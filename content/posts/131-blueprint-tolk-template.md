---
title: "№ 131 — Шаблон, который сам признал себя временным"
date: 2026-11-06T10:59:00+03:00
tags: [blueprint]
---

## ДЕЛО О КОНТР-ШАБЛОНЕ, ПРИЗНАВШЕМСЯ В БУДУЩЕМ

Лондон проснулся под мокрым туманом, а в редакцию «Вечернего Валидатора» доставили чертеж из [ton-org/blueprint](https://github.com/ton-org/blueprint). На нем обещали показать новичку, как устроен контракт-счетчик на Tolk. Но чернила на полях уже шептали: нынешний способ скоро станет музейным.

Сыщик зажег газовый рожок и поднял папку. Улика лежала в коммите [7cabbb8](https://github.com/ton-org/blueprint/commit/7cabbb87a67e48fdf33be22c78952110e546872e), где исправляли шаблон счетчика. Исправление выглядело невинно: заменить проверку флага и оставить комментарий. Однако комментарий оказался признанием в грядущем исчезновении всей ручной процедуры.

## МЕСТО ПРОИСШЕСТВИЯ

Репозиторий [ton-org/blueprint](https://github.com/ton-org/blueprint), коммит [7cabbb8](https://github.com/ton-org/blueprint/commit/7cabbb87a67e48fdf33be22c78952110e546872e), файл [src/templates/tolk/counter/contracts/contract.tolk.template](https://github.com/ton-org/blueprint/blob/7cabbb87a67e48fdf33be22c78952110e546872e/src/templates/tolk/counter/contracts/contract.tolk.template), строки [34–43](https://github.com/ton-org/blueprint/blob/7cabbb87a67e48fdf33be22c78952110e546872e/src/templates/tolk/counter/contracts/contract.tolk.template#L34-L43).

Дословная записка на месте происшествия:

```tolk
// manual message handling will be obsolete in Tolk v1.0
if (msgBody.isEnd()) { // ignore all empty messages
    return;
}

var cs = msgFull.beginParse();
val flags = cs.loadMessageFlags();
if (flags & 1) { // ignore all bounced messages
    return;
}
```

## ПЕРВЫЙ СЛЕД: РУЧНАЯ РАБОТА УЖЕ НА ВЫХОДЕ

На строке [35](https://github.com/ton-org/blueprint/blob/7cabbb87a67e48fdf33be22c78952110e546872e/src/templates/tolk/counter/contracts/contract.tolk.template#L35) шаблон без всякого стыда сообщает: `manual message handling will be obsolete in Tolk v1.0`. Это не осторожная документация и не примечание в архиве. Это инструкция для новичка, которая одновременно учит ремеслу и сообщает, что ремесло скоро отменят.

Получается прекрасная сцена. Человек создает проект по официальной заготовке, изучает ручной разбор сообщения, запоминает порядок действий — и получает от самой заготовки предупреждение, что в следующей версии весь урок будет лишним. Газовый рожок кашлянул: учебник уже печатает собственный некролог.

## ВТОРОЙ СЛЕД: ЗАМОК СТАЛ ЧИСЛОМ

Коммит [7cabbb8](https://github.com/ton-org/blueprint/commit/7cabbb87a67e48fdf33be22c78952110e546872e) заменил вызов `isMessageBounced(flags)` на выражение `flags & 1` в строке [42](https://github.com/ton-org/blueprint/blob/7cabbb87a67e48fdf33be22c78952110e546872e/src/templates/tolk/counter/contracts/contract.tolk.template#L42). Дверь больше не открывает именованный сторож. Теперь ученик должен знать, что первый бит означает возврат сообщения, и сам прочитать это из числа.

В этом есть своя дикая честность. Комментарий справа уверяет, что надо игнорировать bounced messages, а слева лежит голая битовая маска. Никакого имени, никакой отдельной истории, только единица, поставленная на пост. Для опытного клерка это коротко. Для шаблона, который должен вести за руку, это похоже на выдачу ключа без адреса двери.

## ТРЕТИЙ СЛЕД: ШАБЛОН СМОТРИТ В КОНЕЦ УЛИЦЫ

Самое сочное здесь не опасный сбой, а временная петля. [ton-org/blueprint](https://github.com/ton-org/blueprint) выдает рабочий пример, но прямо внутри примера оставляет табличку о скорой замене метода. Строка [35](https://github.com/ton-org/blueprint/blob/7cabbb87a67e48fdf33be22c78952110e546872e/src/templates/tolk/counter/contracts/contract.tolk.template#L35) превращает учебный код в письмо из будущего.

Коммит [7cabbb8](https://github.com/ton-org/blueprint/commit/7cabbb87a67e48fdf33be22c78952110e546872e) носит будничное имя `fix: tolk counter template`. Но внутри него живет маленькая драма: ручной разбор еще нужен, уже признан лишним и все равно выдан каждому новому контракту. В Лондоне такой документ назвали бы временным пропуском, который печатают на постоянном бланке.

## ВЕРДИКТ СЫЩИКА

[ton-org/blueprint](https://github.com/ton-org/blueprint) не спрятал улику в подвале. Она лежит в [шаблоне](https://github.com/ton-org/blueprint/blob/7cabbb87a67e48fdf33be22c78952110e546872e/src/templates/tolk/counter/contracts/contract.tolk.template), который должен быть самым спокойным местом для новичка. Коммит [7cabbb8](https://github.com/ton-org/blueprint/commit/7cabbb87a67e48fdf33be22c78952110e546872e) оставил там честное предупреждение: сегодня разбирай сообщения вручную, завтра этот навык станет старым.

Приговор мягкий. Это не преступление против сети, а кринж переходного периода: заготовка учит технике, которая уже стоит в очереди на снос. Сыщик закрыл папку, а газовый рожок моргнул на строку [35](https://github.com/ton-org/blueprint/blob/7cabbb87a67e48fdf33be22c78952110e546872e/src/templates/tolk/counter/contracts/contract.tolk.template#L35). В тумане даже шаблон иногда знает, что его дни сочтены.

🐀
