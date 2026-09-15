---
title: "Выпуск № 63: Замок, который держал звонок"
date: 2026-09-15T10:58:00+03:00
tags: [tonutils-go]
---

# 📰 ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

## ДЕЛО О КОЛОКОЛЬЧИКЕ, КОТОРЫЙ ЗВОНИЛ ВНУТРИ ЗАМКА

Лондон утонул в утреннем тумане. У редакции сипел газовый рожок, а сыщик «Вечернего Валидатора» получил депешу из конторы [xssnick/tonutils-go](https://github.com/xssnick/tonutils-go). В ней лежал коммит [`dee7b0b`](https://github.com/xssnick/tonutils-go/commit/dee7b0b9dc154fccf814c010dc45cc1a92808d27) с деловым названием `Fixed channel init & dht race`.

Деловая вывеска скрывала сцену куда сочнее: при создании защищенного ADNL-канала контора звала пользовательский callback до того, как помечала канал готовым, и делала это внутри уже взятого замка. Колокольчик звонил, пока дворецкий держал дверь запертой.

## МЕСТО ПРОИСШЕСТВИЯ

Репозиторий [xssnick/tonutils-go](https://github.com/xssnick/tonutils-go), родитель коммита [`b3cfd99`](https://github.com/xssnick/tonutils-go/commit/b3cfd99ca00a56a6824b69997949263a1f2879a1), файл [`adnl/channel.go`](https://github.com/xssnick/tonutils-go/blob/b3cfd99ca00a56a6824b69997949263a1f2879a1/adnl/channel.go), строки [99–104](https://github.com/xssnick/tonutils-go/blob/b3cfd99ca00a56a6824b69997949263a1f2879a1/adnl/channel.go#L99-L104). Цитата сверена по родительскому коммиту.

Вот дословный протокол старой сцены:

```go
	h := c.adnl.onChannel
	if h != nil {
		h(c)
	}

	c.ready.Store(true)
```

## ПЕРВЫЙ СЛЕД: ГОСТЬ ДОПУЩЕН ДО РЕГИСТРАЦИИ

На строке [99](https://github.com/xssnick/tonutils-go/blob/b3cfd99ca00a56a6824b69997949263a1f2879a1/adnl/channel.go#L99) берется обработчик `onChannel`. На строках [100–102](https://github.com/xssnick/tonutils-go/blob/b3cfd99ca00a56a6824b69997949263a1f2879a1/adnl/channel.go#L100-L102) он немедленно вызывается. И только на строке [104](https://github.com/xssnick/tonutils-go/blob/b3cfd99ca00a56a6824b69997949263a1f2879a1/adnl/channel.go#L104) появляется `c.ready.Store(true)`.

Снаружи это выглядело как торжественное открытие канала. Но callback мог увидеть канал еще не готовым. Более того, его вызывали из старого пути инициализации, где вокруг создания канала уже стоял замок. Пользовательский код получал приглашение войти в комнату в тот миг, когда ключ все еще торчал в замке с внутренней стороны.

## ВТОРОЙ СЛЕД: КОЛОКОЛЬЧИК ПОД ОХРАНОЙ

Коммит [`dee7b0b`](https://github.com/xssnick/tonutils-go/commit/dee7b0b9dc154fccf814c010dc45cc1a92808d27) принес не только новые имена для жизненного цикла. В исправленном файле [`adnl/channel.go`](https://github.com/xssnick/tonutils-go/blob/dee7b0b9dc154fccf814c010dc45cc1a92808d27/adnl/channel.go), строки [89–100](https://github.com/xssnick/tonutils-go/blob/dee7b0b9dc154fccf814c010dc45cc1a92808d27/adnl/channel.go#L89-L100), callback из `setup` исчез, а готовность стала последней операцией внутри этого метода:

```go
	c.idEnc, err = tl.Hash(keys.PublicKeyAES{Key: c.encKey})
	if err != nil {
		return err
	}

	c.ready.Store(true)
	return nil
```

Теперь канал сначала завершает внутреннюю настройку. Затем новый путь публикации вызывает обработчики уже отдельно, через `channelPublishedLocked`; сам callback запускается после снятия внутренних замков и только если канал все еще текущий. Это видно в том же коммите [`dee7b0b`](https://github.com/xssnick/tonutils-go/commit/dee7b0b9dc154fccf814c010dc45cc1a92808d27), файл [`adnl/adnl.go`](https://github.com/xssnick/tonutils-go/blob/dee7b0b9dc154fccf814c010dc45cc1a92808d27/adnl/adnl.go), строки [507–520](https://github.com/xssnick/tonutils-go/blob/dee7b0b9dc154fccf814c010dc45cc1a92808d27/adnl/adnl.go#L507-L520).

## ТРЕТИЙ СЛЕД: ТЕСТ, КОТОРЫЙ СЛЫШАЛ СКРИП КЛЮЧА

Самое красноречивое признание лежит в новом тесте [`adnl/channel_lifecycle_test.go`](https://github.com/xssnick/tonutils-go/blob/dee7b0b9dc154fccf814c010dc45cc1a92808d27/adnl/channel_lifecycle_test.go). На строках [359–376](https://github.com/xssnick/tonutils-go/blob/dee7b0b9dc154fccf814c010dc45cc1a92808d27/adnl/channel_lifecycle_test.go#L359-L376) тест нарочно задерживает callback и проверяет, что локальная переинициализация не застряла за ним. Его контрольный вопрос звучит так:

```go
	waitChannelLifecycleSignal(t, reinitDone, "local reinit blocked behind the ready callback")
```

Фраза `local reinit blocked behind the ready callback` — почти готовый заголовок для вечернего листка. Пока обработчик держал сцену, переинициализация должна была пройти свободно. Если бы она ждала тот же замок, дверь ADNL могла превратиться в маленький лондонский тупик: callback ждет дальнейшего действия, дальнейшее действие ждет замок, а замок ждет, пока callback закончит.

Тест [TestLocalReinitInvalidatesBlockedInboundChannelCreate](https://github.com/xssnick/tonutils-go/blob/dee7b0b9dc154fccf814c010dc45cc1a92808d27/adnl/channel_lifecycle_test.go#L288-L329) из того же файла добавляет еще один штрих: устаревшее создание канала после локальной переинициализации не должно заново публиковать канал. Вызов уже не просто звонит — он обязан знать, к какой двери относится.

## ПОЧЕМУ ЭТО СОЧНО

Кринж здесь не в одной пропущенной проверке. Сцена собрана из трех приличных деталей, которые вместе сыграли фарс. Есть пользовательский callback, есть внутренний замок, есть флаг готовности, который выставляется после callback. Каждая часть по отдельности выглядит невинно. Встречаются они в [adnl/channel.go](https://github.com/xssnick/tonutils-go/blob/b3cfd99ca00a56a6824b69997949263a1f2879a1/adnl/channel.go) — и канал приглашает свидетеля внутрь до того, как называет себя открытым.

Особенно выразителен контраст с заголовком [`dee7b0b`](https://github.com/xssnick/tonutils-go/commit/dee7b0b9dc154fccf814c010dc45cc1a92808d27): `Fixed channel init & dht race`. Внутри не просто переставили пару строк. Пришлось вынести жизненный цикл канала в отдельную систему, ввести версии состояния, обработку устаревших пакетов и набор тестов, которые проверяют порядок публикации, закрытия и переинициализации.

Редакция не станет называть [xssnick/tonutils-go](https://github.com/xssnick/tonutils-go) темной лавкой. Наоборот, коммит [`dee7b0b`](https://github.com/xssnick/tonutils-go/commit/dee7b0b9dc154fccf814c010dc45cc1a92808d27) показывает серьезный ремонт. Но старая улика в [adnl/channel.go](https://github.com/xssnick/tonutils-go/blob/b3cfd99ca00a56a6824b69997949263a1f2879a1/adnl/channel.go) прекрасна своей викторианской логикой: сначала впустить гостя, потом повесить табличку «готово», а дверь все это время держать запертой.

## ВЕРДИКТ СЫЩИКА

[xssnick/tonutils-go](https://github.com/xssnick/tonutils-go) пойман на деле о callback, который звонил внутри замка. В родительском коммите [`b3cfd99`](https://github.com/xssnick/tonutils-go/commit/b3cfd99ca00a56a6824b69997949263a1f2879a1), файл [`adnl/channel.go`](https://github.com/xssnick/tonutils-go/blob/b3cfd99ca00a56a6824b69997949263a1f2879a1/adnl/channel.go) на строках [99–104](https://github.com/xssnick/tonutils-go/blob/b3cfd99ca00a56a6824b69997949263a1f2879a1/adnl/channel.go#L99-L104) вызывал `onChannel` до `c.ready.Store(true)`.

Исправляющий коммит [`dee7b0b`](https://github.com/xssnick/tonutils-go/commit/dee7b0b9dc154fccf814c010dc45cc1a92808d27) убрал вызов из [`adnl/channel.go`](https://github.com/xssnick/tonutils-go/blob/dee7b0b9dc154fccf814c010dc45cc1a92808d27/adnl/channel.go), а тест в [`adnl/channel_lifecycle_test.go`](https://github.com/xssnick/tonutils-go/blob/dee7b0b9dc154fccf814c010dc45cc1a92808d27/adnl/channel_lifecycle_test.go) закрепил правило: callback не должен запирать переинициализацию за собой.

Приговор прост: сначала открыть дверь, потом звать гостей, а замок отпустить до начала разговора. Сыщик сверил строки [99–104](https://github.com/xssnick/tonutils-go/blob/b3cfd99ca00a56a6824b69997949263a1f2879a1/adnl/channel.go#L99-L104) и [359–376](https://github.com/xssnick/tonutils-go/blob/dee7b0b9dc154fccf814c010dc45cc1a92808d27/adnl/channel_lifecycle_test.go#L359-L376), погас газовый рожок и растворился в тумане. В Лондоне звонок может быть срочным. Но если он раздается из запертой комнаты, весь город узнает об этом слишком поздно.

🐀
