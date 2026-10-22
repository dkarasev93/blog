---
title: "Выпуск № 103: Канал, который закрыли до начала разговора"
date: 2026-10-22T16:59:00+03:00
tags: [bridge]
---

# ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

## ДЕЛО О КАНАЛЕ, ЗАКРЫТОМ ДО ПЕРВОГО ПРИВЕТСТВИЯ

Лондон утонул в тумане, когда сыщик «Вечернего Валидатора» получил свежую папку из [ton-connect/bridge](https://github.com/ton-connect/bridge). Внутри лежала сцена, достойная плохого водевиля: клиент ушел во время рукопожатия, сторож закрыл канал, а следующий сторож все равно решил подписать его на новости.

На обложке папки стоял коммит [918dff2](https://github.com/ton-connect/bridge/commit/918dff2736d64ecdeb0162a1f423a8811eda1a18) с названием `fix: stop the bridge from crashing with send on closed channel`. Сыщик прочел заголовок и позвонил в газовый рожок. Когда ремонт сам обещает остановить падение из-за отправки в закрытый канал, в подвале уже есть что осматривать.

## МЕСТО ПРОИСШЕСТВИЯ

Главная улика найдена в коммите [918dff2](https://github.com/ton-connect/bridge/commit/918dff2736d64ecdeb0162a1f423a8811eda1a18), в файле [internal/v3/handler/session.go](https://github.com/ton-connect/bridge/blob/918dff2736d64ecdeb0162a1f423a8811eda1a18/internal/v3/handler/session.go), строки [39–59](https://github.com/ton-connect/bridge/blob/918dff2736d64ecdeb0162a1f423a8811eda1a18/internal/v3/handler/session.go#L39-L59). Цитата дословная:

```go
// Close stops the session and cleans up resources. Safe to call multiple times.
func (s *Session) Close() {
	s.mux.Lock()
	defer s.mux.Unlock()
	s.closeLocked()
}

// Start begins the session by subscribing to storage. A session closed before it starts, as
// one whose client left during the handshake is, stays unsubscribed.
func (s *Session) Start() {
	s.mux.Lock()
	defer s.mux.Unlock()

	if s.closed {
		return
	}
	err := s.storage.Sub(context.Background(), s.ClientIds, s.lastEventId, s.messageCh)
	if err != nil {
		s.closeLocked()
	}
}
```

Сцена проста: закрытие и начало подписки оба входят в одну комнату с замком. Но раньше дверь запиралась одним механизмом, а сам факт закрытия нигде не записывался. Поэтому начало могло явиться после финала и повторно сунуть канал в подписку.

## ПЕРВЫЙ СЛЕД: УШЕДШИЙ КЛИЕНТ ОСТАВИЛ СТОРОЖА

В [строках 46–57](https://github.com/ton-connect/bridge/blob/918dff2736d64ecdeb0162a1f423a8811eda1a18/internal/v3/handler/session.go#L46-L57) сам коммит оставил аккуратное признание: клиент способен уйти во время рукопожатия, до того как обработчик доберется до `Start`. Горутинка, следящая за `ctx.Done`, успевает вызвать `Close`, закрывает `messageCh`, а затем поздний `Start` приходит на пустой бал.

Новая строка [52](https://github.com/ton-connect/bridge/blob/918dff2736d64ecdeb0162a1f423a8811eda1a18/internal/v3/handler/session.go#L52) теперь спрашивает `s.closed`. Если ответ положительный, [строка 53](https://github.com/ton-connect/bridge/blob/918dff2736d64ecdeb0162a1f423a8811eda1a18/internal/v3/handler/session.go#L53) делает `return`. Подписка не создается, канал не отправляется обратно в контору, и ушедший клиент не возвращается через черный ход.

До ремонта эта мелодрама имела второй акт. `Start` мог подписать уже закрытый канал, а хранилище при воспроизведении накопленных сообщений пыталось писать в него. В Go это не грустная записка, а паника. Если канал оставался среди подписчиков, следующая публикация могла снова ударить в тот же капкан.

## ВТОРОЙ СЛЕД: ЗАМЕНА ОДНОГО ЗАМКА НА ДВА ПРИЗНАКА

В [строках 12–20](https://github.com/ton-connect/bridge/blob/918dff2736d64ecdeb0162a1f423a8811eda1a18/internal/v3/handler/session.go#L12-L20) старый `sync.Once` уступил место полю `closed` типа `bool`:

```go
type Session struct {
	mux         sync.RWMutex
	ClientIds   []string
	storage     storagev3.Storage
	messageCh   chan models.SseMessage
	Closer      chan interface{}
	lastEventId int64
	closed      bool
}
```

Это не косметика. `sync.Once` хорошо не дает выполнить один блок дважды, но сам по себе не сообщает `Start`, что блок уже отыграл. Поле `closed` стало общей вывеской для обоих маршрутов: [строка 52](https://github.com/ton-connect/bridge/blob/918dff2736d64ecdeb0162a1f423a8811eda1a18/internal/v3/handler/session.go#L52) читает ее перед подпиской, а [строка 66](https://github.com/ton-connect/bridge/blob/918dff2736d64ecdeb0162a1f423a8811eda1a18/internal/v3/handler/session.go#L66) ставит ее при закрытии.

Газовый рожок особенно уважает соседство. Старый механизм говорил: «закрывай один раз». Новый механизм говорит: «закрыто уже сейчас, не начинай». Для гонки жизненного цикла это две разные реплики, и раньше второй в пьесе не было.

## ТРЕТИЙ СЛЕД: СНАЧАЛА ОТПИСКА, ПОТОМ ПАДЕНИЕ ЗАНАВЕСА

Самая важная дверь находится в `closeLocked`, строки [61–75](https://github.com/ton-connect/bridge/blob/918dff2736d64ecdeb0162a1f423a8811eda1a18/internal/v3/handler/session.go#L61-L75):

```go
// closeLocked unsubscribes before closing messageCh, so no storage still holds the channel.
func (s *Session) closeLocked() {
	if s.closed {
		return
	}
	s.closed = true

	logger := slog.With("prefix", "Session.Close")
	err := s.storage.Unsub(context.Background(), s.ClientIds, s.messageCh)
	if err != nil {
		logger.Error("failed to unsubscribe from storage", "err", err)
	}

	close(s.Closer)
	close(s.messageCh)
}
```

На строке [69](https://github.com/ton-connect/bridge/blob/918dff2736d64ecdeb0162a1f423a8811eda1a18/internal/v3/handler/session.go#L69) хранилище сначала получает приказ забыть канал. Только потом строки [74–75](https://github.com/ton-connect/bridge/blob/918dff2736d64ecdeb0162a1f423a8811eda1a18/internal/v3/handler/session.go#L74-L75) закрывают `Closer` и `messageCh`. Это порядок, которого не хватало старой сцене: нельзя выдавать канал из списка подписчиков после того, как его уже захлопнули.

Есть и маленькая, но сочная деталь. Если подписка внутри `Start` сама провалилась, [строка 57](https://github.com/ton-connect/bridge/blob/918dff2736d64ecdeb0162a1f423a8811eda1a18/internal/v3/handler/session.go#L57) ведет ее в тот же `closeLocked`. Значит, аварийный выход не заводит отдельный ритуал с собственным шансом закрыть канал второй раз.

## ЧЕТВЕРТЫЙ СЛЕД: ПРИЗНАНИЕ ИЗ ПРИЕМНОЙ

В описании коммита [918dff2](https://github.com/ton-connect/bridge/commit/918dff2736d64ecdeb0162a1f423a8811eda1a18) сказано, что после поздней подписки воспроизведение очереди могло вызвать панику, а следующий pub/sub для того же клиента уже ронял процесс в `handlePubSub`. Это не домысел редакции, а описание ремонта, пришитое к коммиту.

Впрочем, сыщик не станет превращать каждую гонку в доказанное падение всей сети. Доказано другое: жизненный цикл сессии имел путь «закрыть до старта», после которого старый `Start` мог продолжить работу. Коммит добавил проверку состояния, отписку до закрытия и тестовую папку [internal/v3/handler/session_test.go](https://github.com/ton-connect/bridge/blob/918dff2736d64ecdeb0162a1f423a8811eda1a18/internal/v3/handler/session_test.go), где этот маршрут теперь разыгрывается отдельно.

## ВЕРДИКТ СЫЩИКА

Репозиторий [ton-connect/bridge](https://github.com/ton-connect/bridge) пойман в коммите [918dff2](https://github.com/ton-connect/bridge/commit/918dff2736d64ecdeb0162a1f423a8811eda1a18) на старой лондонской шутке: клиент уже ушел, канал уже закрыт, но подписка еще не знала, что спектакль окончен. В файле [internal/v3/handler/session.go](https://github.com/ton-connect/bridge/blob/918dff2736d64ecdeb0162a1f423a8811eda1a18/internal/v3/handler/session.go) строки [52–57](https://github.com/ton-connect/bridge/blob/918dff2736d64ecdeb0162a1f423a8811eda1a18/internal/v3/handler/session.go#L52-L57) не дают стартовать мертвой сессии, а строки [61–75](https://github.com/ton-connect/bridge/blob/918dff2736d64ecdeb0162a1f423a8811eda1a18/internal/v3/handler/session.go#L61-L75) сначала отписывают канал и лишь потом закрывают его.

Приговор без скидки: асинхронность не обязана быть мистикой, но ее двери надо подписывать. Если клиент может исчезнуть между `AddConnection` и `Start`, поздний сторож обязан увидеть след закрытия. Иначе газовый рожок звонит не на рассвете, а в момент следующего сообщения.

🐀
