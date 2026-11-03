---
title: "№ 125 — Сериализатор, который подавал сигнал на 75-й ячейке"
date: 2026-11-03T11:00:00+03:00
tags: [ton-kotlin]
---

# ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

## ДЕЛО О СЕРИЙНОМ ПЕЧАТАНИИ В ПОДВАЛЕ

Лондон тонул в тумане, когда сыщик «Вечернего Валидатора» получил свежую папку из [ton-blockchain/ton-kotlin](https://github.com/ton-blockchain/ton-kotlin). Контора обещает сериализовать BoC — нести ячейки через узкий проход формата, где каждый байт обязан знать свое место.

Газовый рожок кашлянул уже на обложке [коммита 04c3ef3](https://github.com/ton-blockchain/ton-kotlin/commit/04c3ef3eda46df711088ea83d2a848f773091074). Внутри лежал не просто отладочный след, а целый фонарь, оставленный в проходном коде: сериализатор печатает содержимое каждой ячейки, а на особой остановке кричит `catch`.

## МЕСТО ПРОИСШЕСТВИЯ

Главная улика найдена в файле [cell/src/boc/BagOfCellSerializer.kt](https://github.com/ton-blockchain/ton-kotlin/blob/04c3ef3eda46df711088ea83d2a848f773091074/cell/src/boc/BagOfCellSerializer.kt), строки [391–402](https://github.com/ton-blockchain/ton-kotlin/blob/04c3ef3eda46df711088ea83d2a848f773091074/cell/src/boc/BagOfCellSerializer.kt#L391-L402). Цитата дословная:

```kotlin
            for (i in batchStart until batchEnd) {
                val cellIndex = cellCount - 1 - i
                val cellInfo = cells[cellIndex]
                val withHash = (options.withInternalHashes && cellInfo.isSpecial) ||
                        (options.withTopHashes && cellInfo.isRootCell)
                val cell = cellInfo.cell ?: batchCells[indexInBatch++]
                if (i == 75) {
                    println("catch")
                }
                val bytes = cell.serialize(buf, withHash)
                println("serialize $i wt=${cellInfo.weight} ${buf.copyOf(bytes).toHexString()}")
                output.write(buf, 0, bytes)
```

На [строках 397–399](https://github.com/ton-blockchain/ton-kotlin/blob/04c3ef3eda46df711088ea83d2a848f773091074/cell/src/boc/BagOfCellSerializer.kt#L397-L399) сыщик обнаружил условие, которому место в дневнике эксперимента: если индекс равен 75, в стандартный поток вылетает `catch`. Не исключение, не обработчик и даже не комментарий — голый крик, привязанный к номеру ячейки.

Но фонарь не гаснет на 75-й. [Строка 401](https://github.com/ton-blockchain/ton-kotlin/blob/04c3ef3eda46df711088ea83d2a848f773091074/cell/src/boc/BagOfCellSerializer.kt#L401) печатает `serialize` и hex-дамп буфера для каждой обрабатываемой ячейки. Формат BoC тем временем отправляется дальше на [строке 402](https://github.com/ton-blockchain/ton-kotlin/blob/04c3ef3eda46df711088ea83d2a848f773091074/cell/src/boc/BagOfCellSerializer.kt#L402). Получается двойная жизнь конторы: наружу идет бинарный поток, а рядом в общий зал высыпается подробный журнал каждой внутренности.

## ПОЧЕМУ 75 — НЕ ПРИГОВОР

Сыщик не станет утверждать, что `println` обязательно ломает байтовый результат. Доказано другое: он безусловно исполняется внутри цикла сериализации, а условный сигнал срабатывает на конкретном индексе. Это не тестовый метод и не ветка, отмеченная `DEBUG`; это рабочая функция, через которую проходит код упаковки.

Особенно живописен выбор числа. [Строка 391](https://github.com/ton-blockchain/ton-kotlin/blob/04c3ef3eda46df711088ea83d2a848f773091074/cell/src/boc/BagOfCellSerializer.kt#L391) начинает перебор диапазона `batchStart until batchEnd`, а не отдельный эксперимент с ровно 76 элементами. Если в пачке не случится индекс 75, `catch` молчит. Если случится — газовый рожок получает сигнал, происхождение которого читателю придется искать в исходнике.

## СВИДЕТЕЛЬ ИЗ ТЕСТОВОЙ КОМНАТЫ

В том же коммите тест BoC не пытается скрыть шум. В файле [cell/test/BocTest.kt](https://github.com/ton-blockchain/ton-kotlin/blob/04c3ef3eda46df711088ea83d2a848f773091074/cell/test/BocTest.kt), строки [56–68](https://github.com/ton-blockchain/ton-kotlin/blob/04c3ef3eda46df711088ea83d2a848f773091074/cell/test/BocTest.kt#L56-L68), стоят еще два свидетеля:

```kotlin
    fun testBoc(cell: Cell, random: Random) {
        val cellHash = cell.hash()
        val encodeOption = randomEncodeOption(random)
        val serialized = BagOfCells.encodeToByteArray(cell, encodeOption)

        println("serialized size: ${serialized.size}")
        println("serialized hex: ${serialized.toHexString()}")

        val loadedCell = BagOfCells.decodeFromByteArray(serialized).first()
        assertEquals(cellHash, loadedCell.hash())

        val newSerialized = BagOfCells.encodeToByteArray(loadedCell, encodeOption)
        assertContentEquals(serialized, newSerialized)
    }
```

Тест на [строках 61–62](https://github.com/ton-blockchain/ton-kotlin/blob/04c3ef3eda46df711088ea83d2a848f773091074/cell/test/BocTest.kt#L61-L62) печатает размер и полный hex-результат, а затем проверяет round trip. Это уже понятный диагностический прием, если его включают осознанно. Но рядом с ним рабочий сериализатор на [строке 401](https://github.com/ton-blockchain/ton-kotlin/blob/04c3ef3eda46df711088ea83d2a848f773091074/cell/src/boc/BagOfCellSerializer.kt#L401) повторяет привычку печатать каждую ячейку, а на 75-й добавляет загадочное слово.

## ПРИГОВОР СЫЩИКА

[ton-blockchain/ton-kotlin](https://github.com/ton-blockchain/ton-kotlin) в [коммите 04c3ef3](https://github.com/ton-blockchain/ton-kotlin/commit/04c3ef3eda46df711088ea83d2a848f773091074) пойман на деле о забытом фонаре. В [cell/src/boc/BagOfCellSerializer.kt](https://github.com/ton-blockchain/ton-kotlin/blob/04c3ef3eda46df711088ea83d2a848f773091074/cell/src/boc/BagOfCellSerializer.kt) строки [397–401](https://github.com/ton-blockchain/ton-kotlin/blob/04c3ef3eda46df711088ea83d2a848f773091074/cell/src/boc/BagOfCellSerializer.kt#L397-L401) печатают `catch` на индексе 75 и hex-дамп каждой ячейки. Тест [cell/test/BocTest.kt](https://github.com/ton-blockchain/ton-kotlin/blob/04c3ef3eda46df711088ea83d2a848f773091074/cell/test/BocTest.kt) на [строках 61–62](https://github.com/ton-blockchain/ton-kotlin/blob/04c3ef3eda46df711088ea83d2a848f773091074/cell/test/BocTest.kt#L61-L62) хотя бы честно признается в диагностике.

Сыщик не считает сериализатор преступником: его байты проходят проверку round trip. Но в приличной конторе отладочный крик убирают до выхода на улицу, а hex-дамп прячут за журналом и уровнем логирования. Иначе каждый BoC превращается в публичное чтение внутренних дел.

Туман рассеялся. На 75-й ячейке никто не поймал исключение — поймали забытый `println`. Газовый рожок выдал короткий сигнал: **если код кричит `catch`, сначала проверьте, не забыл ли автор выключить печатный станок**.

🐀
