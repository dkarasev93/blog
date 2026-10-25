---
title: "Выпуск № 108: Пакетный туман без досмотра"
date: 2026-10-25T11:00:00+03:00
tags: [adnl-tunnel]
---

# Пакетный туман без досмотра

*«Вечерний Валидатор», Лондон. Спецкор сыщик ведет дело при газовом рожке.*

Туман в Лондоне нынче густ. Он лег на новый [adnl-tunnel](https://github.com/ton-blockchain/adnl-tunnel) — шлюз, что обещает вести ADNL-пакеты через чужие серверы, брать плату за трафик и прятать узел от прямых атак. На вывеске все чинно: пакет вошел, монета звякнула, сеть пошла дальше.

Но сыщик поднял [коммит c00a157](https://github.com/ton-blockchain/adnl-tunnel/commit/c00a15763f7a10f3b7426beea6b728834f2a86c9) и постучал в [tunnel/instructions.go](https://github.com/ton-blockchain/adnl-tunnel/blob/c00a15763f7a10f3b7426beea6b728834f2a86c9/tunnel/instructions.go). Там, в платном проходе, висит записка на [строке 510](https://github.com/ton-blockchain/adnl-tunnel/blob/c00a15763f7a10f3b7426beea6b728834f2a86c9/tunnel/instructions.go#L510):

```go
// TODO: recover payments after restart
```

Платежи после рестарта еще надо вернуть. А значит, у ворот может проснуться процесс без памяти о прежних расчетах. Для платного туннеля это не романтическая амнезия, а бухгалтер с пустым сейфом.

## Свидетель первый: кто прислал пакет

Дальше сыщик прошел к [строкам 1219–1221](https://github.com/ton-blockchain/adnl-tunnel/blob/c00a15763f7a10f3b7426beea6b728834f2a86c9/tunnel/instructions.go#L1219-L1221) того же [файла](https://github.com/ton-blockchain/adnl-tunnel/blob/c00a15763f7a10f3b7426beea6b728834f2a86c9/tunnel/instructions.go):

```go
// TODO: filter "from", add bans

//TODO: verify packets as much as possible
```

Это не план капитального ремонта в подвале. Это почти протокол досмотра, оставленный на потом: не фильтровать отправителя, не ставить запреты, проверять пакет «насколько возможно». В городе, где пакет платит за проезд, такая формула звучит как: «Паспорт предъявлять по желанию, печать ставить по настроению».

Сверху README [adnl-tunnel](https://github.com/ton-blockchain/adnl-tunnel) уверяет, что инструмент создан для защиты от прямых атак в децентрализованной сети TON. Снизу [строка 1221](https://github.com/ton-blockchain/adnl-tunnel/blob/c00a15763f7a10f3b7426beea6b728834f2a86c9/tunnel/instructions.go#L1221) честно просит еще подумать, как проверять то, что уже приехало. Газовый рожок кашлянул. Защита от атаки стоит у двери и ищет инструкцию по досмотру.

## Свидетель второй: посылка с невидимой начинкой

На [строке 1258](https://github.com/ton-blockchain/adnl-tunnel/blob/c00a15763f7a10f3b7426beea6b728834f2a86c9/tunnel/instructions.go#L1258) [того же исходника](https://github.com/ton-blockchain/adnl-tunnel/blob/c00a15763f7a10f3b7426beea6b728834f2a86c9/tunnel/instructions.go) сыщик находит еще одну записку:

```go
//TODO: add random padding to payload
```

Случайная набивка для полезной нагрузки тоже еще не пришита. Пока пакет идет в пальто без подкладки, а наблюдатель может прикинуть его размер. В этом деле даже конверт оставили без тумана.

## Резюме сыщика

Улика не в одном TODO, а в связке. [Коммит c00a157](https://github.com/ton-blockchain/adnl-tunnel/commit/c00a15763f7a10f3b7426beea6b728834f2a86c9) показывает платный шлюз, где:

1. память о платежах после рестарта еще не возвращена — [строка 510](https://github.com/ton-blockchain/adnl-tunnel/blob/c00a15763f7a10f3b7426beea6b728834f2a86c9/tunnel/instructions.go#L510);
2. отправителя еще не фильтруют, а запреты еще не добавляют — [строка 1219](https://github.com/ton-blockchain/adnl-tunnel/blob/c00a15763f7a10f3b7426beea6b728834f2a86c9/tunnel/instructions.go#L1219);
3. проверку пакетов оставили с туманным «as much as possible» — [строка 1221](https://github.com/ton-blockchain/adnl-tunnel/blob/c00a15763f7a10f3b7426beea6b728834f2a86c9/tunnel/instructions.go#L1221);
4. случайную набивку пока не положили — [строка 1258](https://github.com/ton-blockchain/adnl-tunnel/blob/c00a15763f7a10f3b7426beea6b728834f2a86c9/tunnel/instructions.go#L1258).

Вердикт «Вечернего Валидатора»: шлюз уже умеет брать пакет в ладони, но еще не решил, кто его прислал, что в нем лежит и что будет с оплатой после ночного перезапуска. В Лондоне такие ворота называют не крепостью, а дверью с красивой табличкой и запиской: «Замок добавим позже».

🐀
