---
title: "Выпуск № 59: Канал, который запер сам себя"
date: 2026-09-13T11:01:00+03:00
tags: [tongo]
---

# 📰 ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

## ДЕЛО О ПОСЫЛЬНОМ, КОТОРЫЙ ЗАЖАЛ ДВЕРЬ

Лондон тонул в утреннем тумане. Мостовая блестела, газовый рожок у редакции сипел, а сыщик «Вечернего Валидатора» получил папку из конторы [tonkeeper/tongo](https://github.com/tonkeeper/tongo), библиотеки для разговора с лайтсерверами TON.

На обложке стояла короткая надпись: `fix: deadlock`. Внутри лежал не просто ремонт, а прекрасная сцена: пул соединений держал блокировку и ждал, пока канал под этой же блокировкой освободится. Вежливый посыльный запер дверь изнутри и стал ждать дворецкого.

## МЕСТО ПРОИСШЕСТВИЯ

Репозиторий [tonkeeper/tongo](https://github.com/tonkeeper/tongo), родитель коммита [`cec7f34`](https://github.com/tonkeeper/tongo/commit/cec7f346b41b3285430d6887514a1bbd7cba12ae), файл [`liteapi/pool/conn_pool.go`](https://github.com/tonkeeper/tongo/blob/cec7f346b41b3285430d6887514a1bbd7cba12ae/liteapi/pool/conn_pool.go), строки [353–365](https://github.com/tonkeeper/tongo/blob/cec7f346b41b3285430d6887514a1bbd7cba12ae/liteapi/pool/conn_pool.go#L353-L365). Цитата сверена с содержимым файла на этом коммите.

Протокол, строки [353–365](https://github.com/tonkeeper/tongo/blob/cec7f346b41b3285430d6887514a1bbd7cba12ae/liteapi/pool/conn_pool.go#L353-L365):

```go
func (p *ConnPool) notifySubscribers(update masterHeadUpdated) {
	p.mu.RLock()
	defer p.mu.RUnlock()

	if p.bestConn == nil {
		return
	}
	if update.Conn.ID() != p.bestConn.ID() {
		return
	}
	for _, ch := range p.waitList {
		ch <- update.Head
```

Цитата обрывается на самом выразительном месте: после `RLock` поток идет к каналу обычной блокирующей отправкой.

## ПЕРВЫЙ СЛЕД: ОХРАНА ВЗЯЛА ПОСЫЛЬНОГО ЗА ВОРОТНИК

Метод [`notifySubscribers`](https://github.com/tonkeeper/tongo/blob/cec7f346b41b3285430d6887514a1bbd7cba12ae/liteapi/pool/conn_pool.go#L353) сначала берет общую блокировку на строке [354](https://github.com/tonkeeper/tongo/blob/cec7f346b41b3285430d6887514a1bbd7cba12ae/liteapi/pool/conn_pool.go#L354). На строке [355](https://github.com/tonkeeper/tongo/blob/cec7f346b41b3285430d6887514a1bbd7cba12ae/liteapi/pool/conn_pool.go#L355) обещает отпустить ее лишь при выходе. А на строке [364](https://github.com/tonkeeper/tongo/blob/cec7f346b41b3285430d6887514a1bbd7cba12ae/liteapi/pool/conn_pool.go#L364) без всякой проверки пишет в канал подписчика.

Канал создан с емкостью один: [`conn_pool.go`](https://github.com/tonkeeper/tongo/blob/cec7f346b41b3285430d6887514a1bbd7cba12ae/liteapi/pool/conn_pool.go#L374-L375), строки [374–375](https://github.com/tonkeeper/tongo/blob/cec7f346b41b3285430d6887514a1bbd7cba12ae/liteapi/pool/conn_pool.go#L374-L375). Первая голова мастерчейна помещается внутрь. Следующая уже должна ждать читателя.

Но читатель, возможно, ждет не просто так. Получатель мог застрять на другой операции пула, которой нужна та же блокировка. Тогда картина становится безупречной по-нуарному: [Run](https://github.com/tonkeeper/tongo/blob/cec7f346b41b3285430d6887514a1bbd7cba12ae/liteapi/pool/conn_pool.go#L187-L201) вызывает уведомление, уведомление держит `RLock`, канал заполнен, а получатель не может добраться до места, где он этот канал прочитает.

## ВТОРОЙ СЛЕД: ФОКУС С ПУСТЫМ ВЫХОДОМ

Коммит [`0f15564`](https://github.com/tonkeeper/tongo/commit/0f15564b505de2b070c58f09eb18d63f977beb94) пришел с заголовком `fix: deadlock`. В измененном файле [`liteapi/pool/conn_pool.go`](https://github.com/tonkeeper/tongo/blob/0f15564b505de2b070c58f09eb18d63f977beb94/liteapi/pool/conn_pool.go), строки [363–370](https://github.com/tonkeeper/tongo/blob/0f15564b505de2b070c58f09eb18d63f977beb94/liteapi/pool/conn_pool.go#L363-L370), контора поставила перед отправкой маленький турникет:

```go
for _, ch := range p.waitList {
	select {
	default:
		// subscriber hasn't drained the previous head yet; don't block
		// the pool's Run() goroutine while holding the lock — it will
		// pick up the next update on the following iteration.
	case ch <- update.Head:
	}
}
```

Теперь если канал занят, срабатывает `default`: новая голова пропускается мимо, а поток не остается у запертой двери. Комментарий на строках [366–368](https://github.com/tonkeeper/tongo/blob/0f15564b505de2b070c58f09eb18d63f977beb94/liteapi/pool/conn_pool.go#L366-L368) прямо называет виновника: нельзя блокировать `Run` во время удержания блокировки.

## ТРЕТИЙ СЛЕД: ПОЧЕМУ ЭТО СОЧНО

Смешон не сам канал и не сама подписка. Смешон порядок церемонии. Сначала контора надевает на себя замок, потом подходит к окну, которое может потребовать живого посетителя. Если посетитель не успел принять прошлую голову, служба обновления могла повиснуть прямо внутри общего сейфа.

В [файле `liteapi/pool/conn_pool.go`](https://github.com/tonkeeper/tongo/blob/cec7f346b41b3285430d6887514a1bbd7cba12ae/liteapi/pool/conn_pool.go) на строке [363](https://github.com/tonkeeper/tongo/blob/cec7f346b41b3285430d6887514a1bbd7cba12ae/liteapi/pool/conn_pool.go#L363) перебираются все каналы, а строка [364](https://github.com/tonkeeper/tongo/blob/cec7f346b41b3285430d6887514a1bbd7cba12ae/liteapi/pool/conn_pool.go#L364) не оставляет запасного выхода. Это не тайм-аут и не мягкий отказ: обычная отправка ждет бесконечно, пока кто-нибудь не заберет значение.

А затем [коммит `0f15564`](https://github.com/tonkeeper/tongo/commit/0f15564b505de2b070c58f09eb18d63f977beb94) приносит лекарство в четыре строки: `select`, `default` и честное признание про удержание замка. Газовый рожок кашлянул. В Лондоне это называется не «тонкая оптимизация», а «вспомнить, что дверь иногда открывают с обеих сторон».

Редакция не станет называть [tonkeeper/tongo](https://github.com/tonkeeper/tongo) негодной конторой. Наоборот, исправление точное: оно убирает блокирующую передачу, пока пул держит `RLock`. Но старый след в родительском коммите [`cec7f34`](https://github.com/tonkeeper/tongo/commit/cec7f346b41b3285430d6887514a1bbd7cba12ae) великолепен своей простотой: канал на одну ячейку, вечное ожидание и общий замок в одном почтовом зале.

## ВЕРДИКТ СЫЩИКА

[tonkeeper/tongo](https://github.com/tonkeeper/tongo) пойман на деле о канале, который мог превратить обновление головы мастерчейна в неподвижную сцену. В родительском коммите [`cec7f34`](https://github.com/tonkeeper/tongo/commit/cec7f346b41b3285430d6887514a1bbd7cba12ae) файл [`liteapi/pool/conn_pool.go`](https://github.com/tonkeeper/tongo/blob/cec7f346b41b3285430d6887514a1bbd7cba12ae/liteapi/pool/conn_pool.go) держал `RLock` на строках [353–355](https://github.com/tonkeeper/tongo/blob/cec7f346b41b3285430d6887514a1bbd7cba12ae/liteapi/pool/conn_pool.go#L353-L355) и затем мог застрять на отправке строки [364](https://github.com/tonkeeper/tongo/blob/cec7f346b41b3285430d6887514a1bbd7cba12ae/liteapi/pool/conn_pool.go#L364).

Потом коммит [`0f15564`](https://github.com/tonkeeper/tongo/commit/0f15564b505de2b070c58f09eb18d63f977beb94) на строках [363–370](https://github.com/tonkeeper/tongo/blob/0f15564b505de2b070c58f09eb18d63f977beb94/liteapi/pool/conn_pool.go#L363-L370) сделал передачу неблокирующей. Приговор прост: если держишь замок, не жди гостя у окна. Иначе весь пул превратится в клуб джентльменов, где каждый вежливо пропускает другого первым — до самого конца сети.

*Сыщик сверил строки [353–365](https://github.com/tonkeeper/tongo/blob/cec7f346b41b3285430d6887514a1bbd7cba12ae/liteapi/pool/conn_pool.go#L353-L365), погас лампу и растворился в тумане. Газовый рожок еще раз кашлянул: `fix: deadlock` — короткая надпись, за которой иногда стоит целая ночь у запертой двери.*

🐀
