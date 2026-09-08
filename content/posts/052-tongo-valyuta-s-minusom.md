+++
title = "№ 52 — Дело о валюте с минусом: tongo превратил 4294967279 в -17"
date = 2026-09-08T16:58:00+03:00
description = "Пятьдесят второй выпуск «Вечернего Валидатора»: tongo сменил беззнаковый ключ словаря extra currencies на знаковый, и токен с номером 4294967279 вышел из камеры уже как -17."
tags = ["tongo"]
+++

# 📰 ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

**№ 52 · Вторник, 8 сентября 2026 г. · Цена: 0.05 TON (сдача может уйти в минус)**

---

## ДЕЛО О МОНЕТЕ, УШЕДШЕЙ НИЖЕ НУЛЯ

Лондон тонул в тумане, мостовая блестела, а газовый рожок у редакции сипел, словно его заставили считать деньги в темной кладовой. Сыщик «Вечернего Валидатора» получил папку из конторы [tonkeeper/tongo](https://github.com/tonkeeper/tongo) — библиотеки на Go, которая раскладывает по полкам ячейки, блоки и прочие улики TON.

Внутри лежала странная арифметика. В коммите [`4514890`](https://github.com/tonkeeper/tongo/commit/4514890fed50db8d03a629004903d81b8d26bda1) ключ словаря дополнительных валют перестал быть беззнаковым числом и стал знаковым. Один и тот же 32-битный узник сменил паспорт: `4294967279` превратился в `-17`.

Место происшествия: репозиторий [tonkeeper/tongo](https://github.com/tonkeeper/tongo), коммит [`4514890`](https://github.com/tonkeeper/tongo/commit/4514890fed50db8d03a629004903d81b8d26bda1), файл [`tlb/models.go`](https://github.com/tonkeeper/tongo/blob/4514890fed50db8d03a629004903d81b8d26bda1/tlb/models.go), строки [150–155](https://github.com/tonkeeper/tongo/blob/4514890fed50db8d03a629004903d81b8d26bda1/tlb/models.go#L150-L155). Цитата сверена с содержимым файла на этом коммите.

Протокол, файл [`tlb/models.go`](https://github.com/tonkeeper/tongo/blob/4514890fed50db8d03a629004903d81b8d26bda1/tlb/models.go), строки [150–155](https://github.com/tonkeeper/tongo/blob/4514890fed50db8d03a629004903d81b8d26bda1/tlb/models.go#L150-L155):

```go
// ExtraCurrencyCollection
// extra_currencies$_ dict:(HashmapE 32 (VarUInteger 32))
// = ExtraCurrencyCollection;
type ExtraCurrencyCollection struct {
    Dict HashmapE[Int32, VarUInteger32]
}
```

## ПЕРВЫЙ СЛЕД: ТРИДЦАТЬ ДВА БИТА, ДВА ХАРАКТЕРА

На строке [151](https://github.com/tonkeeper/tongo/blob/4514890fed50db8d03a629004903d81b8d26bda1/tlb/models.go#L151) схема говорит о словаре с ключом длиной 32 бита: `HashmapE 32`. Это широкий коридор, в котором помещаются и обычные положительные номера, и значения, выглядящие как числа выше двух миллиардов.

Но строка [154](https://github.com/tonkeeper/tongo/blob/4514890fed50db8d03a629004903d81b8d26bda1/tlb/models.go#L154) теперь указывает `HashmapE[Int32, VarUInteger32]`. Раньше там стоял `Uint32`. В дворе машинного типа это не смена шрифта. `Uint32` не знает минуса, а `Int32` умеет прочитать те же 32 бита как знаковое значение.

Поэтому старый номер `4294967279`, то есть хвост диапазона беззнакового типа, получил новое лицо. Те же биты теперь показывают `-17`. Газовый рожок кашлянул: монета не уменьшилась, изменился лишь ярлык на ящике. Но если программа использует этот ярлык как настоящий идентификатор, в дело входит уже не типографская опечатка, а новая личность валюты.

## ВТОРОЙ СЛЕД: ТЕСТОВАЯ КОМНАТА ПЕРЕПИСАЛА ИМЕНА

Коммит [`4514890`](https://github.com/tonkeeper/tongo/commit/4514890fed50db8d03a629004903d81b8d26bda1) не ограничился строкой [154](https://github.com/tonkeeper/tongo/blob/4514890fed50db8d03a629004903d81b8d26bda1/tlb/models.go#L154). В тестовом досье он заменил номер валюты. Файл [`tlb/testdata/block-1/block.expected.json`](https://github.com/tonkeeper/tongo/blob/4514890fed50db8d03a629004903d81b8d26bda1/tlb/testdata/block-1/block.expected.json) на строке [4297](https://github.com/tonkeeper/tongo/blob/4514890fed50db8d03a629004903d81b8d26bda1/tlb/testdata/block-1/block.expected.json#L4297) теперь хранит ключ `-17`.

Протокол перемены, файл [`tlb/testdata/block-1/block.expected.json`](https://github.com/tonkeeper/tongo/blob/4514890fed50db8d03a629004903d81b8d26bda1/tlb/testdata/block-1/block.expected.json), строки [4294–4298](https://github.com/tonkeeper/tongo/blob/4514890fed50db8d03a629004903d81b8d26bda1/tlb/testdata/block-1/block.expected.json#L4294-L4298):

```json
"Other": {
  "-17": "1555555554",
  "239": "2333333332"
}
```

На строке [4297](https://github.com/tonkeeper/tongo/blob/4514890fed50db8d03a629004903d81b8d26bda1/tlb/testdata/block-1/block.expected.json#L4297) номер стал отрицательным, а на строке [4298](https://github.com/tonkeeper/tongo/blob/4514890fed50db8d03a629004903d81b8d26bda1/tlb/testdata/block-1/block.expected.json#L4298) соседняя валюта `239` сохранила невинный вид. В одном JSON стоят рядом два свидетеля: один пережил смену знака, другой даже не снял шляпу.

Само название коммита [`4514890`](https://github.com/tonkeeper/tongo/commit/4514890fed50db8d03a629004903d81b8d26bda1) звучит деловито: `treat extra currency collection key as signed`. Но для читателя это почти признание в протоколе: система решила, что ключ надо считать знаковым, а старые снимки пришлось переписать, чтобы новая арифметика выглядела естественно.

## ТРЕТИЙ СЛЕД: КОГДА КЛЮЧ СТАЛ ПОКАЗАНИЕМ

В строках [142–148](https://github.com/tonkeeper/tongo/blob/4514890fed50db8d03a629004903d81b8d26bda1/tlb/models.go#L142-L148) `ExtraCurrencyCollection` живет внутри `CurrencyCollection` рядом с основным количеством граммов. Это не случайная карта заметок на полях. Она входит в представление денежной коллекции.

Отсюда следует осторожный, но неприятный вывод. Если внешний код получает JSON через методы вокруг строк [157–160](https://github.com/tonkeeper/tongo/blob/4514890fed50db8d03a629004903d81b8d26bda1/tlb/models.go#L157-L160), ключ валюты выходит наружу уже в новом виде. Пользователь, база данных или индексатор, привыкший к `4294967279`, может встретить `-17`. Это не доказывает потерю средств и не означает, что сам блок стал иным. Но договор о форме данных изменился прямо на глазах.

Редакция не станет кричать о краже. Коммит [`4514890`](https://github.com/tonkeeper/tongo/commit/4514890fed50db8d03a629004903d81b8d26bda1) может исправлять неверное толкование схемы TON, а отрицательная запись способна быть честным отображением знакового ключа. Сочная улика в другом: название типа в Go теперь спорит с привычным видом тестовых данных, а смысл номера нельзя понять без знания того, как прочитаны 32 бита.

## ВЕРДИКТ СЫЩИКА

[tonkeeper/tongo](https://github.com/tonkeeper/tongo) не пойман на фальшивой монете. Но коммит [`4514890`](https://github.com/tonkeeper/tongo/commit/4514890fed50db8d03a629004903d81b8d26bda1) оставил чудесную сцену: файл [`tlb/models.go`](https://github.com/tonkeeper/tongo/blob/4514890fed50db8d03a629004903d81b8d26bda1/tlb/models.go) на строке [154](https://github.com/tonkeeper/tongo/blob/4514890fed50db8d03a629004903d81b8d26bda1/tlb/models.go#L154) выдает ключ словаря за `Int32`, а тестовый файл на строке [4297](https://github.com/tonkeeper/tongo/blob/4514890fed50db8d03a629004903d81b8d26bda1/tlb/testdata/block-1/block.expected.json#L4297) показывает результат: `-17`.

Приговор прост: при смене знака надо громко предупредить всех, кто хранит ключи как идентификаторы, а не как абстрактные биты. Иначе в тумане один и тот же валютный номер выйдет из камеры с двумя паспортами. Газовый рожок требует подписи под протоколом: прежде чем обвинять минус, проверьте, кто именно поменял ему имя.

*Сыщик закрыл папку, еще раз сверил строки [151–154](https://github.com/tonkeeper/tongo/blob/4514890fed50db8d03a629004903d81b8d26bda1/tlb/models.go#L151-L154) и погас лампу. В Лондоне давно знают: иногда самая подозрительная перемена — это не новая монета, а новый способ ее назвать.*

🐀
