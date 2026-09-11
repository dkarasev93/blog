---
title: "Выпуск № 56: Посылка, ушедшая в отложенный ящик"
date: 2026-09-11T16:58:00+03:00
tags: [bridge]
---
# 📰 ВЕЧЕРНИЙ ВАЛИДАТОР

## ДЕЛО О ПОСЫЛКЕ, КОТОРУЮ ПРИНЯЛИ ДО ТОГО, КАК ЕЕ ПОЛОЖИЛИ В ЯЩИК

Лондон тонул в тумане, мостовая блестела, а газовый рожок редакции сипел, будто его назначили дежурным у почтового ящика. В такую ночь сыщик «Вечернего Валидатора» получил депешу из конторы [ton-connect/bridge](https://github.com/ton-connect/bridge), где мосты переправляют сообщения между кошельками и приложениями.

Улика нашлась в [коммите 3ec5ae0](https://github.com/ton-connect/bridge/commit/3ec5ae04a159b4456c34d0fd7457fdae1310058b), в файле [internal/v1/handler/handler.go](https://github.com/ton-connect/bridge/blob/3ec5ae04a159b4456c34d0fd7457fdae1310058b/internal/v1/handler/handler.go), строки [443–450](https://github.com/ton-connect/bridge/blob/3ec5ae04a159b4456c34d0fd7457fdae1310058b/internal/v1/handler/handler.go#L443-L450). Цитата сверена с содержимым файла на этом коммите.

## МЕСТО ПРОИСШЕСТВИЯ

Протокол, файл [internal/v1/handler/handler.go](https://github.com/ton-connect/bridge/blob/3ec5ae04a159b4456c34d0fd7457fdae1310058b/internal/v1/handler/handler.go), строки [443–450](https://github.com/ton-connect/bridge/blob/3ec5ae04a159b4456c34d0fd7457fdae1310058b/internal/v1/handler/handler.go#L443-L450):

```go
		go func() {
			log := log.WithField("prefix", "SendMessageHandler.storge.Add")
			err = h.storage.Add(context.Background(), sseMessage, ttl)
			if err != nil {
				// TODO ooops
				log.Errorf("db error: %v", err)
			}
		}()
```

Так выглядела почтовая служба до вмешательства редакции. Посыльный запускал отдельного курьера, тут же продолжал разговор с отправителем и позволял обработчику вернуть успешный ответ. Само занесение письма в хранилище происходило где-то потом, в тумане, без обещания, что оно завершится раньше следующего визита адресата.

## ПЕРВЫЙ СЛЕД: «200» РАНЬШЕ, ЧЕМ ЧЕРНИЛА ВЫСОХЛИ

В [строках 436–442](https://github.com/ton-connect/bridge/blob/3ec5ae04a159b4456c34d0fd7457fdae1310058b/internal/v1/handler/handler.go#L436-L442) живые сессии получают сообщение прямо в очередь. Это парадный зал: посылка видна тем, кто уже стоит у окна. А на [строке 443](https://github.com/ton-connect/bridge/blob/3ec5ae04a159b4456c34d0fd7457fdae1310058b/internal/v1/handler/handler.go#L443) начинается другая жизнь — `go func()`, отдельная горничная для записи в запасной ящик.

Проблема обнаруживается не в философии, а в порядке событий. Обработчик мог сообщить клиенту об успехе, пока строка [445](https://github.com/ton-connect/bridge/blob/3ec5ae04a159b4456c34d0fd7457fdae1310058b/internal/v1/handler/handler.go#L445) еще не записала письмо. Если клиент после ответа немедленно разрывал связь и возвращался, он просил уже не живую доставку, а архив. Архив мог быть еще пуст.

Получается отменный викторианский фокус: письмо уже принято по квитанции, но в почтовом ящике его нет. Живой свидетель видел посылку. Следующий свидетель, пришедший на минуту позже, видел только ночь и мокрый камень.

## ВТОРОЙ СЛЕД: ОШИБКА, КОТОРОЙ ВЕЛЕНО СКАЗАТЬ «ОЙ»

На [строках 446–449](https://github.com/ton-connect/bridge/blob/3ec5ae04a159b4456c34d0fd7457fdae1310058b/internal/v1/handler/handler.go#L446-L449) хранительница ящика узнает о провале записи. Но вместо ответа отправителю, отмены успеха или надежного журнала контора оставляет нам бессмертную реплику:

```go
// TODO ooops
```

Цитата дословна: именно `ooops`, с тремя буквами `o`. Ошибка базы данных превращена в легкое пожатие плечами внутри фоновой горничной. Отправитель уже получил свою квитанцию, а запись о провале остается в руках логгера. Если лог не дошел до нужного глаза, письмо исчезло без траурной ленты.

Еще выразительнее выглядит соседняя вывеска на [строке 444](https://github.com/ton-connect/bridge/blob/3ec5ae04a159b4456c34d0fd7457fdae1310058b/internal/v1/handler/handler.go#L444): `SendMessageHandler.storge.Add`. Слово `storge` потеряло одну букву, словно архивариус спешил закрыть ящик. Это не причина самой гонки, но прекрасная печать на конверте: даже имя места хранения записано не вполне уверенно.

## ТРЕТИЙ СЛЕД: ПОЧЕМУ ГАЗОВЫЙ РОЖОК ЗВОНИТ

Коммит [97c750f](https://github.com/ton-connect/bridge/commit/97c750fa2cde50e389d623986e700fb0677e2484) с заголовком `fix: store message synchronously before acking send (#233)` пришел как признание, написанное уже после ночного дознания. В исправленном [файле internal/v1/handler/handler.go](https://github.com/ton-connect/bridge/blob/97c750fa2cde50e389d623986e700fb0677e2484/internal/v1/handler/handler.go) строки [443–449](https://github.com/ton-connect/bridge/blob/97c750fa2cde50e389d623986e700fb0677e2484/internal/v1/handler/handler.go#L443-L449) больше не запускают отдельную горничную: хранилище вызывается до ответа, а провал превращается в HTTP 500.

Сам заголовок коммита [97c750f](https://github.com/ton-connect/bridge/commit/97c750fa2cde50e389d623986e700fb0677e2484) говорит все без маски: сообщение надо сохранить синхронно, прежде чем подтвердить отправку. В описании той же улики контора признает, что тесты дрожали: после `200` клиент мог читать запасной ящик раньше, чем туда попадала запись.

Редакция не станет считать [ton-connect/bridge](https://github.com/ton-connect/bridge) преступной шайкой. Наоборот, [коммит 97c750f](https://github.com/ton-connect/bridge/commit/97c750fa2cde50e389d623986e700fb0677e2484) показывает аккуратное лечение: запись сделана обязательной частью пути, а ошибка получает настоящий ответ. Но старый [коммит 3ec5ae0](https://github.com/ton-connect/bridge/commit/3ec5ae04a159b4456c34d0fd7457fdae1310058b) оставил сочную сцену: отдельный курьер, квитанция до доставки, `TODO ooops` и опечатка в имени кладовой.

## ВЕРДИКТ СЫЩИКА

[ton-connect/bridge](https://github.com/ton-connect/bridge) пойман не на краже посылок, а на их преждевременной регистрации. В [internal/v1/handler/handler.go](https://github.com/ton-connect/bridge/blob/3ec5ae04a159b4456c34d0fd7457fdae1310058b/internal/v1/handler/handler.go) строки [443–450](https://github.com/ton-connect/bridge/blob/3ec5ae04a159b4456c34d0fd7457fdae1310058b/internal/v1/handler/handler.go#L443-L450) отправляли письмо в отложенный ящик и почти сразу делали вид, что дело закрыто. Потом пришел [коммит 97c750f](https://github.com/ton-connect/bridge/commit/97c750fa2cde50e389d623986e700fb0677e2484) и запер дверь до завершения записи.

Сыщик обвел [строку 447](https://github.com/ton-connect/bridge/blob/3ec5ae04a159b4456c34d0fd7457fdae1310058b/internal/v1/handler/handler.go#L447), где живет `// TODO ooops`, и погас лампу. В тумане Лондона и без того трудно отличить доставленное письмо от обещания его доставить. А если обещание еще и написано с тремя `o`, газовый рожок имеет полное право звонить до рассвета.
