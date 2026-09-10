+++
title = "№ 54 — Дело о ловушке catch: ton-kotlin оставил крик на 75-й ячейке"
date = 2026-09-10T10:58:00+03:00
description = "Вечерний Валидатор раскрывает отладочную ловушку в сериализаторе Bag of Cells: ровно на ячейке 75 код печатает catch, а затем шепчет в рожок весь hex-след."
tags = ["ton-kotlin"]
+++

# 📰 ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

**№ 54 · Четверг, 10 сентября 2026 г. · Цена: 0.05 TON (за отладку доплата)**

---

## ДЕЛО О СЕМЬДЕСЯТ ПЯТОЙ ЯЧЕЙКЕ

Лондон утонул в тумане, когда сыщик «Вечернего Валидатора» получил депешу из [ton-blockchain/ton-kotlin](https://github.com/ton-blockchain/ton-kotlin). В этой конторе собирают Kotlin-инструменты для клеток, ячеек и прочих мелких обитателей TON. Работа тонкая: одна лишняя запятая — и байтовый экипаж уезжает не в тот переулок.

На стол лег коммит [`04c3ef3`](https://github.com/ton-blockchain/ton-kotlin/commit/04c3ef3eda46df711088ea83d2a848f773091074) с невинной вывеской `fix serialization of pruned branch`. Но в файле [`cell/src/boc/BagOfCellSerializer.kt`](https://github.com/ton-blockchain/ton-kotlin/blob/04c3ef3eda46df711088ea83d2a848f773091074/cell/src/boc/BagOfCellSerializer.kt) сыщик заметил нечто, что пахнет не исправлением, а забытым капканом.

### МЕСТО ПРОИСШЕСТВИЯ

Репозиторий [ton-blockchain/ton-kotlin](https://github.com/ton-blockchain/ton-kotlin), коммит [`04c3ef3`](https://github.com/ton-blockchain/ton-kotlin/commit/04c3ef3eda46df711088ea83d2a848f773091074), файл [`cell/src/boc/BagOfCellSerializer.kt`](https://github.com/ton-blockchain/ton-kotlin/blob/04c3ef3eda46df711088ea83d2a848f773091074/cell/src/boc/BagOfCellSerializer.kt), строки [391–402](https://github.com/ton-blockchain/ton-kotlin/blob/04c3ef3eda46df711088ea83d2a848f773091074/cell/src/boc/BagOfCellSerializer.kt#L391-L402). Цитата сверена с содержимым файла на этом коммите.

Протокол, строки [391–402](https://github.com/ton-blockchain/ton-kotlin/blob/04c3ef3eda46df711088ea83d2a848f773091074/cell/src/boc/BagOfCellSerializer.kt#L391-L402):

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

### КАПКАН С НОМЕРОМ

Сцена начинается на строке [391](https://github.com/ton-blockchain/ton-kotlin/blob/04c3ef3eda46df711088ea83d2a848f773091074/cell/src/boc/BagOfCellSerializer.kt#L391): цикл идет по пачке. На строках [392–396](https://github.com/ton-blockchain/ton-kotlin/blob/04c3ef3eda46df711088ea83d2a848f773091074/cell/src/boc/BagOfCellSerializer.kt#L392-L396) вычисляется индекс, достается описание ячейки и решается, нужны ли ей хеши. Все чинно, все пахнет библиотечной пылью.

А затем строка [397](https://github.com/ton-blockchain/ton-kotlin/blob/04c3ef3eda46df711088ea83d2a848f773091074/cell/src/boc/BagOfCellSerializer.kt#L397) ставит поперек дороги условие `i == 75`. Если счетчик добрался до этого номера, строка [398](https://github.com/ton-blockchain/ton-kotlin/blob/04c3ef3eda46df711088ea83d2a848f773091074/cell/src/boc/BagOfCellSerializer.kt#L398) печатает в стандартный вывод одно слово: `catch`.

Почему семьдесят пять? В летописи коммита [`04c3ef3`](https://github.com/ton-blockchain/ton-kotlin/commit/04c3ef3eda46df711088ea83d2a848f773091074) видно, что прежнее число было 67. Его заменили на 75 — без пояснения, без имени ошибки, без теста, который сообщил бы, что именно поймано. Получился не сторож, а табличка на двери: «если дошел до этого места, скажи catch».

Но рожок не замолкает. Строка [400](https://github.com/ton-blockchain/ton-kotlin/blob/04c3ef3eda46df711088ea83d2a848f773091074/cell/src/boc/BagOfCellSerializer.kt#L400) сериализует ячейку, а строка [401](https://github.com/ton-blockchain/ton-kotlin/blob/04c3ef3eda46df711088ea83d2a848f773091074/cell/src/boc/BagOfCellSerializer.kt#L401) печатает номер, вес и полный hex-оттиск буфера. Так что случайный запуск получает не только загадочный крик, но и поток байтов в лог. Отладка, вошедшая в зал без приглашения и занявшая весь стол.

### ПОЧЕМУ ЭТО СОЧНО

Отладочная печать сама по себе не преступление. В мастерской сериализатора лог иногда полезнее свечи. Но здесь рядом стоят три улики: магическое число 75, слово `catch`, не связанное ни с исключением, ни с обработчиком, и безусловная печать полного буфера на каждой итерации.

Особенно хороша контрастная вывеска коммита [`04c3ef3`](https://github.com/ton-blockchain/ton-kotlin/commit/04c3ef3eda46df711088ea83d2a848f773091074): `fix serialization of pruned branch`. Читатель ждет строгий ремонт формата. А на месте ремонта обнаруживает маленький театральный пистолет, который стреляет словом `catch`, когда процесс проходит ровно через 75-й фонарь.

Редакция не станет считать [ton-blockchain/ton-kotlin](https://github.com/ton-blockchain/ton-kotlin) преступной конторой. В том же коммите исправлены важные детали сериализации, включая запись CRC и маску ветви. Но файл [`cell/src/boc/BagOfCellSerializer.kt`](https://github.com/ton-blockchain/ton-kotlin/blob/04c3ef3eda46df711088ea83d2a848f773091074/cell/src/boc/BagOfCellSerializer.kt) на строках [397–401](https://github.com/ton-blockchain/ton-kotlin/blob/04c3ef3eda46df711088ea83d2a848f773091074/cell/src/boc/BagOfCellSerializer.kt#L397-L401) оставил нам редкую ночную сцену: серьезный сериализатор вдруг подмигивает счетчику и кричит в туман.

Сыщик закрыл папку. Газовый рожок кашлянул: `catch`. В Лондоне снова стало тихо — до следующей ячейки.
