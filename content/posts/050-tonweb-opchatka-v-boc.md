+++
title = "№ 50 — Дело о букве, сбившейся с дороги: tonweb считает байты с лишним l"
date = 2026-09-07T17:00:00+03:00
description = "Пятидесятый выпуск «Вечернего Валидатора»: в десериализаторе tonweb флаг fullfilledBytes получил лишнюю букву, а серьезная процедура разбора ячеек пошла дальше как ни в чем не бывало."
tags = ["tonweb"]
+++

# 📰 ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

**№ 50 · Понедельник, 7 сентября 2026 г. · Цена: 0.05 TON (за каждую букву доплата)**

---

## ДЕЛО О ЛИШНЕЙ БУКВЕ В МАШИННОМ ЗАЛЕ

Лондон утонул в тумане, мостовая блестела, а газовый рожок у редакции сипел, словно ему поручили проверять документы у каждой буквы. Сыщик «Вечернего Валидатора» получил папку из конторы [toncenter/tonweb](https://github.com/toncenter/tonweb) — библиотеки на JavaScript, которая разбирает ячейки, собирает сообщения и водит приложения по переулкам TON.

Внутри не нашлось ни взлома, ни пропавшего кошелька. Но в машинном зале стояла улика тоньше волоса и заметнее пятна на белом воротнике: флаг, отвечающий за завершенность байтов, назван `fullfilledBytes`. В английском языке в этом слове одна лишняя `l`. В коде TON это не меняет поведение, зато превращает рабочую криптографическую механику в табличку, которую писарь прибил криво и оставил на фасаде.

Место происшествия: репозиторий [toncenter/tonweb](https://github.com/toncenter/tonweb), коммит [`76dfd07`](https://github.com/toncenter/tonweb/commit/76dfd0701714c0a316aee503c2962840acaf74ef), файл [`src/boc/Cell.js`](https://github.com/toncenter/tonweb/blob/76dfd0701714c0a316aee503c2962840acaf74ef/src/boc/Cell.js), строки [432–452](https://github.com/toncenter/tonweb/blob/76dfd0701714c0a316aee503c2962840acaf74ef/src/boc/Cell.js#L432-L452). Цитата сверена с содержимым файла на этом коммите.

Протокол, файл [`src/boc/Cell.js`](https://github.com/toncenter/tonweb/blob/76dfd0701714c0a316aee503c2962840acaf74ef/src/boc/Cell.js), строки [432–452](https://github.com/toncenter/tonweb/blob/76dfd0701714c0a316aee503c2962840acaf74ef/src/boc/Cell.js#L432-L452):

```javascript
function deserializeCellData(cellData, referenceIndexSize) {
    if (cellData.length < 2)
        throw "Not enough bytes to encode cell descriptors";
    const d1 = cellData[0], d2 = cellData[1];
    cellData = cellData.slice(2);
    const level = Math.floor(d1 / 32);
    const isExotic = d1 & 8;
    const refNum = d1 % 8;
    const dataBytesize = Math.ceil(d2 / 2);
    const fullfilledBytes = !(d2 % 2);
    let cell = new Cell();
    cell.isExotic = isExotic;
    if (cellData.length < dataBytesize + referenceIndexSize * refNum)
        throw "Not enough bytes to encode cell data";
    cell.bits.setTopUppedArray(cellData.slice(0, dataBytesize), fullfilledBytes);
    cellData = cellData.slice(dataBytesize);
    for (let r = 0; r < refNum; r++) {
        cell.refs.push(readNBytesUIntFromArray(referenceIndexSize, cellData));
        cellData = cellData.slice(referenceIndexSize);
    }
    return {cell: cell, residue: cellData};
}
```

## ПЕРВЫЙ СЛЕД: СЕРЬЕЗНЫЙ КАБИНЕТ

На строке [432](https://github.com/toncenter/tonweb/blob/76dfd0701714c0a316aee503c2962840acaf74ef/src/boc/Cell.js#L432) начинается `deserializeCellData`. Это не демонстрационный скрипт с именем вроде `tryStuff`. Функция получает сериализованные данные ячейки и размер индекса ссылок. На строках [433–436](https://github.com/toncenter/tonweb/blob/76dfd0701714c0a316aee503c2962840acaf74ef/src/boc/Cell.js#L433-L436) она проверяет длину, читает два дескриптора и отрезает их от входного массива.

Строки [437–440](https://github.com/toncenter/tonweb/blob/76dfd0701714c0a316aee503c2962840acaf74ef/src/boc/Cell.js#L437-L440) достают уровень, признак exotic-ячейки, число ссылок и размер данных. Тут все выглядит как положено в машинном отделении: короткие поля дескриптора превращаются в параметры дальнейшего разбора.

И вот строка [441](https://github.com/toncenter/tonweb/blob/76dfd0701714c0a316aee503c2962840acaf74ef/src/boc/Cell.js#L441) выкатывает на свет `const fullfilledBytes = !(d2 % 2);`. Смысл ясен: определить, полностью ли заполнен последний байт. Но имя получило лишнюю букву, будто писарь дважды ударил по клавише и решил, что так даже солиднее.

## ВТОРОЙ СЛЕД: БУКВА ДОХОДИТ ДО ДВИГАТЕЛЯ

Можно было бы списать все на безобидную надпись в комментарии. Но строка [446](https://github.com/toncenter/tonweb/blob/76dfd0701714c0a316aee503c2962840acaf74ef/src/boc/Cell.js#L446) передает флаг в настоящий разбор битов: `cell.bits.setTopUppedArray(cellData.slice(0, dataBytesize), fullfilledBytes);`.

В этом месте имя не лежит мертвым грузом. Оно связывает результат вычисления на строке [441](https://github.com/toncenter/tonweb/blob/76dfd0701714c0a316aee503c2962840acaf74ef/src/boc/Cell.js#L441) с настройкой, которая говорит `setTopUppedArray`, как трактовать последний байт. Лишняя `l` не ломает путь, потому что JavaScript не требует правильного английского от локальной переменной. Но она идет от вычисления к вызову, получает пропуск в зал и навсегда остается частью публичного протокола.

Газовый рожок кашлянул. Содержимое ячейки может быть выверено до бита, ссылки могут быть прочитаны по счетчику, остаток может вернуться вызывающему коду. А рядом с этой точностью живет слово, которое выглядит так, словно его проверяли на слух в темной карете.

## ТРЕТИЙ СЛЕД: НИ ОДНОГО ПОПРАВОЧНОГО КАРАНДАША

Коммит [`76dfd07`](https://github.com/toncenter/tonweb/commit/76dfd0701714c0a316aee503c2962840acaf74ef) носит спокойное название `0.0.66 build`. В такой летописи нет драматического `fix typo`, нет признания `spellcheck`, нет сообщения о том, что кто-то наконец заметил лишнюю букву. Сборка прошла, библиотека продолжила путь, а имя осталось в строках [441](https://github.com/toncenter/tonweb/blob/76dfd0701714c0a316aee503c2962840acaf74ef/src/boc/Cell.js#L441) и [446](https://github.com/toncenter/tonweb/blob/76dfd0701714c0a316aee503c2962840acaf74ef/src/boc/Cell.js#L446).

Редакция не станет изображать катастрофу. Из этих строк не следует, что ячейки разбираются неверно или что деньги пропадают из-за одной опечатки. Логика флага остается прежней: на строке [441](https://github.com/toncenter/tonweb/blob/76dfd0701714c0a316aee503c2962840acaf74ef/src/boc/Cell.js#L441) вычисляется булево значение, а на строке [446](https://github.com/toncenter/tonweb/blob/76dfd0701714c0a316aee503c2962840acaf74ef/src/boc/Cell.js#L446) оно передается дальше.

Но кринж от этого не исчезает. В библиотеке, где `d1`, `d2`, ссылки и верхушка байта требуют аккуратности, имя переменной становится частью ремесла. Новичок копирует его. Документация повторяет его. Поиск по проекту находит его. Потом новый человек тратит вечер, проверяя, не значит ли `fullfilled` что-то особенное в терминологии TON. А это всего лишь лишняя буква, получившая пожизненный вид на жительство.

## ВЕРДИКТ СЫЩИКА

[toncenter/tonweb](https://github.com/toncenter/tonweb) — полезная мастерская, а не кабак, где байты считают на глаз. Но коммит [`76dfd07`](https://github.com/toncenter/tonweb/commit/76dfd0701714c0a316aee503c2962840acaf74ef) оставил выразительную улику в файле [`src/boc/Cell.js`](https://github.com/toncenter/tonweb/blob/76dfd0701714c0a316aee503c2962840acaf74ef/src/boc/Cell.js): строка [441](https://github.com/toncenter/tonweb/blob/76dfd0701714c0a316aee503c2962840acaf74ef/src/boc/Cell.js#L441) называет флаг `fullfilledBytes`, а строка [446](https://github.com/toncenter/tonweb/blob/76dfd0701714c0a316aee503c2962840acaf74ef/src/boc/Cell.js#L446) отправляет его в разбор ячейки.

Приговор мягок: поправьте лишнюю `l`, добавьте проверку стиля и не позволяйте машинному залу выдавать случайную опечатку за термин. Ошибка в имени не всегда становится ошибкой исполнения. Но в публичном коде каждая такая буква оставляет след, а сыщик обязан его заметить.

*Сыщик закрыл папку, обвел строку [441](https://github.com/toncenter/tonweb/blob/76dfd0701714c0a316aee503c2962840acaf74ef/src/boc/Cell.js#L441) и погас газовую лампу. В тумане Лондона легко потерять целый байт. Но иногда вся улика — это одна буква, которая решила жить в слове дважды.*

🐀
