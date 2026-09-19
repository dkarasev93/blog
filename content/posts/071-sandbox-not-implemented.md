+++
title = "№ 71 — Песочница с табличкой Not implemented"
date = 2026-09-19T10:58:00+03:00
description = "Семьдесят первый выпуск «Вечернего Валидатора»: генератор схем в TON Sandbox выдает наружу десятки функций Not implemented, хотя каждая из них стоит ровно не у той двери."
tags = ["sandbox"]
+++

# 📰 ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

**№ 71.** *Лондон. Утренний туман прижался к мостовой, газовый рожок у редакции сипит, а сыщик получил сверток из новой песочницы TON. Внутри не было ни грабителя, ни сломанного замка. Там лежал аккуратный машинный чертеж, в котором слово `Not implemented` встречалось снова и снова. Самое подозрительное: каждая заглушка стояла в помещении, где ее вроде бы не должны были вызвать.*

## МЕСТО ПРОИСШЕСТВИЯ

Репозиторий [ton-blockchain/sandbox](https://github.com/ton-blockchain/sandbox), коммит [`4fb4a5c`](https://github.com/ton-blockchain/sandbox/commit/4fb4a5ce9d4e6368cbfe0ca2e3197797e4a08750), файл [`src/config/config.tlb-gen.ts`](https://github.com/ton-blockchain/sandbox/blob/4fb4a5ce9d4e6368cbfe0ca2e3197797e4a08750/src/config/config.tlb-gen.ts), строки [4606–4631](https://github.com/ton-blockchain/sandbox/blob/4fb4a5ce9d4e6368cbfe0ca2e3197797e4a08750/src/config/config.tlb-gen.ts#L4606-L4631) и [5133–5168](https://github.com/ton-blockchain/sandbox/blob/4fb4a5ce9d4e6368cbfe0ca2e3197797e4a08750/src/config/config.tlb-gen.ts#L5133-L5168). Цитаты сверены по содержимому файла на этом коммите.

На столе лежали два протокола.

```typescript
// _ {n:#} _:(Hashmap n True) = BitstringSet n;

export function loadBitstringSet(slice: Slice, n: number): BitstringSet {
    let _: Dictionary<bigint, True> = Dictionary.loadDirect(Dictionary.Keys.BigUint(n), {
        serialize: () => { throw new Error('Not implemented') },
        parse: loadTrue,
    }, slice);
    return {
        kind: 'BitstringSet',
        n: n,
        _: _,
    }

}

export function storeBitstringSet(bitstringSet: BitstringSet): (builder: Builder) => void {
    return ((builder: Builder) => {
        builder.storeDictDirect(bitstringSet._, Dictionary.Keys.BigUint(bitstringSet.n), {
            serialize: ((arg: True, builder: Builder) => {
            storeTrue(arg)(builder);
        }),
            parse: () => { throw new Error('Not implemented') },
        });
    })

}
```

И второй, из зала валют:

```typescript
export function loadExtraCurrencyCollection(slice: Slice): ExtraCurrencyCollection {
    let dict: Dictionary<number, bigint> = Dictionary.load(Dictionary.Keys.Uint(32), {
        serialize: () => { throw new Error('Not implemented') },
        parse: ((slice: Slice) => {
        return slice.loadVarUintBig(bitLen((32 - 1)))

    }),
    }, slice);
    return {
        kind: 'ExtraCurrencyCollection',
        dict: dict,
    }

}

export function storeExtraCurrencyCollection(extraCurrencyCollection: ExtraCurrencyCollection): (builder: Builder) => void {
    return ((builder: Builder) => {
        builder.storeDict(extraCurrencyCollection.dict, Dictionary.Keys.Uint(32), {
            serialize: ((arg: bigint, builder: Builder) => {
            ((arg: bigint) => {
                return ((builder: Builder) => {
                    builder.storeVarUint(arg, bitLen((32 - 1)));
                })

            })(arg)(builder);
        }),
            parse: () => { throw new Error('Not implemented') },
        });
    })

}
```

## ПЕРВЫЙ СЛЕД: ЗАГЛУШКА В УНИФОРМЕ СЕРВИСА

В строке [4608](https://github.com/ton-blockchain/sandbox/blob/4fb4a5ce9d4e6368cbfe0ca2e3197797e4a08750/src/config/config.tlb-gen.ts#L4608) начинается загрузчик `loadBitstringSet`. Он достает словарь из среза, а в строке [4610](https://github.com/ton-blockchain/sandbox/blob/4fb4a5ce9d4e6368cbfe0ca2e3197797e4a08750/src/config/config.tlb-gen.ts#L4610) передает словарю функцию `serialize`, которая при вызове бросит `Not implemented`. Рядом, в строке [4611](https://github.com/ton-blockchain/sandbox/blob/4fb4a5ce9d4e6368cbfe0ca2e3197797e4a08750/src/config/config.tlb-gen.ts#L4611), стоит рабочий `parse: loadTrue`.

Сыщик остановил взгляд на этой паре. Загрузчик читает данные, значит ему нужен разбор. Но рядом лежит заряженная кнопка для обратной операции. Она пока не нажата, однако публичный модуль уже носит ее в кармане.

На другой стороне двери картина зеркальная. Функция `storeBitstringSet` в строке [4621](https://github.com/ton-blockchain/sandbox/blob/4fb4a5ce9d4e6368cbfe0ca2e3197797e4a08750/src/config/config.tlb-gen.ts#L4621) готовит запись, а в строках [4623–4626](https://github.com/ton-blockchain/sandbox/blob/4fb4a5ce9d4e6368cbfe0ca2e3197797e4a08750/src/config/config.tlb-gen.ts#L4623-L4626) получает рабочий `serialize`. Зато строка [4627](https://github.com/ton-blockchain/sandbox/blob/4fb4a5ce9d4e6368cbfe0ca2e3197797e4a08750/src/config/config.tlb-gen.ts#L4627) оставляет `parse: () => { throw new Error('Not implemented') }`.

Получается чинная витрина: у загрузчика есть неиспользуемая дубинка записи, у сохранения есть неиспользуемая дубинка чтения. Кодогенератор не забыл про половину схемы. Он просто разложил половины по соседним кабинетам, чтобы каждая могла честно сообщить о своей неготовности.

## ВТОРОЙ СЛЕД: ВАЛЮТНАЯ КАНЦЕЛЯРИЯ С ТЕМ ЖЕ ПОЧЕРКОМ

Сыщик прошел дальше и нашел `ExtraCurrencyCollection`. В строке [5138](https://github.com/ton-blockchain/sandbox/blob/4fb4a5ce9d4e6368cbfe0ca2e3197797e4a08750/src/config/config.tlb-gen.ts#L5138) загрузчик собирает словарь дополнительных валют. В строке [5140](https://github.com/ton-blockchain/sandbox/blob/4fb4a5ce9d4e6368cbfe0ca2e3197797e4a08750/src/config/config.tlb-gen.ts#L5140) снова висит `serialize`, который при вызове поднимет красный флаг. При этом настоящий разбор суммы находится в строках [5141–5144](https://github.com/ton-blockchain/sandbox/blob/4fb4a5ce9d4e6368cbfe0ca2e3197797e4a08750/src/config/config.tlb-gen.ts#L5141-L5144): там вызывается `slice.loadVarUintBig`.

В строке [5153](https://github.com/ton-blockchain/sandbox/blob/4fb4a5ce9d4e6368cbfe0ca2e3197797e4a08750/src/config/config.tlb-gen.ts#L5153) начинается обратная процедура. Она умеет сериализовать `bigint` в строках [5155–5162](https://github.com/ton-blockchain/sandbox/blob/4fb4a5ce9d4e6368cbfe0ca2e3197797e4a08750/src/config/config.tlb-gen.ts#L5155-L5162), но в строке [5164](https://github.com/ton-blockchain/sandbox/blob/4fb4a5ce9d4e6368cbfe0ca2e3197797e4a08750/src/config/config.tlb-gen.ts#L5164) оставляет второй заряд `Not implemented`.

Это уже не одинокая записка на полях. Один и тот же почерк прошел через словарь битов и через дополнительные валюты. Где функция читает, там кодогенератор оставляет не нужную ей запись. Где функция пишет, там оставляет не нужное ей чтение. Если кто-то передаст эти параметры не той внутренней двери, песочница сама вынесет ему приговор.

## ТРЕТИЙ СЛЕД: ЗАГОЛОВОК ОБЕЩАЕТ СЕРИАЛИЗАТОРЫ

В коммите [`4fb4a5c`](https://github.com/ton-blockchain/sandbox/commit/4fb4a5ce9d4e6368cbfe0ca2e3197797e4a08750) выпуск помечен как `chore: release 0.45.0`. В его журнале строка [15](https://github.com/ton-blockchain/sandbox/blob/4fb4a5ce9d4e6368cbfe0ca2e3197797e4a08750/CHANGELOG.md#L15) обещает обновленные схемы и сгенерированные сериализаторы. Формально это правда: функции есть, типы есть, рабочие половины есть.

Но в файле [`src/config/config.tlb-gen.ts`](https://github.com/ton-blockchain/sandbox/blob/4fb4a5ce9d4e6368cbfe0ca2e3197797e4a08750/src/config/config.tlb-gen.ts) слово `Not implemented` появляется как служебный герб. Для `BitstringSet` оно стоит в строках [4610](https://github.com/ton-blockchain/sandbox/blob/4fb4a5ce9d4e6368cbfe0ca2e3197797e4a08750/src/config/config.tlb-gen.ts#L4610) и [4627](https://github.com/ton-blockchain/sandbox/blob/4fb4a5ce9d4e6368cbfe0ca2e3197797e4a08750/src/config/config.tlb-gen.ts#L4627), а для дополнительных валют — в строках [5140](https://github.com/ton-blockchain/sandbox/blob/4fb4a5ce9d4e6368cbfe0ca2e3197797e4a08750/src/config/config.tlb-gen.ts#L5140) и [5164](https://github.com/ton-blockchain/sandbox/blob/4fb4a5ce9d4e6368cbfe0ca2e3197797e4a08750/src/config/config.tlb-gen.ts#L5164).

Редакция не станет заявлять, что обычная загрузка конфига падает на каждой улице. В показанных путях загрузчик использует `parse`, а сохранение — `serialize`. Для этих штатных маршрутов заглушки находятся в обратных параметрах и могут никогда не сработать. Но газетная улика от этого не худеет: публичный сгенерированный код выглядит как полный набор двусторонних операций, хотя четыре из них являются ловушками с одинаковой надписью.

## ХРОНИКА МЕЛКОГО КРИНЖА

*Отдел вывесок.* В строках [4606–4608](https://github.com/ton-blockchain/sandbox/blob/4fb4a5ce9d4e6368cbfe0ca2e3197797e4a08750/src/config/config.tlb-gen.ts#L4606-L4608) перед функцией стоит схема `Hashmap n True`, а ниже читателю выдают параметр `serialize`, который не умеет сериализовать ничего. Песочница сначала рисует карту, а потом ставит на ней табличку «вход только по рабочим дням».

*Отдел зеркал.* В строках [4623–4628](https://github.com/ton-blockchain/sandbox/blob/4fb4a5ce9d4e6368cbfe0ca2e3197797e4a08750/src/config/config.tlb-gen.ts#L4623-L4628) рабочая запись и нерабочее чтение стоят рядом, как два портрета одного клерка: один принимает бумаги, другой сразу сообщает, что прием не ведется.

*Отдел валют.* В строках [5155–5165](https://github.com/ton-blockchain/sandbox/blob/4fb4a5ce9d4e6368cbfe0ca2e3197797e4a08750/src/config/config.tlb-gen.ts#L5155-L5165) `bigint` чинно превращается в данные, а обратный путь закрыт. Казначей умеет выдать монету, но не умеет принять ее обратно — даже в коде, который был создан именно для описания обеих дорог.

## ВЕРДИКТ СЫЩИКА

[ton-blockchain/sandbox](https://github.com/ton-blockchain/sandbox) поймано на сочной машинной бюрократии. Коммит [`4fb4a5c`](https://github.com/ton-blockchain/sandbox/commit/4fb4a5ce9d4e6368cbfe0ca2e3197797e4a08750) принес релиз, где схема и генератор выглядят солидно, но файл [`src/config/config.tlb-gen.ts`](https://github.com/ton-blockchain/sandbox/blob/4fb4a5ce9d4e6368cbfe0ca2e3197797e4a08750/src/config/config.tlb-gen.ts) разложил четыре одинаковые ловушки `Not implemented` по двум направлениям.

Приговор осторожный. Это не доказательство, что штатная песочница немедленно рушится: рабочая сторона каждой пары стоит на своем месте. Но это отличный пример того, как генератор может напечатать видимость полноты. Пользователь видит `Dictionary`, `serialize`, `parse` и спокойный типовой интерфейс. Только при ближайшем повороте ключа из кармана выскакивает исключение.

Сыщик закрыл папку. Газовый рожок дернулся от ветра. На двери песочницы осталась записка из строки [4610](https://github.com/ton-blockchain/sandbox/blob/4fb4a5ce9d4e6368cbfe0ca2e3197797e4a08750/src/config/config.tlb-gen.ts#L4610): `throw new Error('Not implemented')`. В Лондоне даже неиспользуемая дверь обязана быть заперта честно.

🐀
