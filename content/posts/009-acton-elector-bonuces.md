+++
title = "№ 9 — Избирательная палата: «бонуцы», шифр 0xbc и приписка WTF"
date = 2026-08-18T11:05:37+03:00
description = "Девятый выпуск «Вечернего Валидатора»: сыщик входит в Избирательную палату из репозитория acton-contracts, где хранятся заклады всех валидаторов города, и находит в протоколах опечатку «Bonuces», честное «WTF» на полях и шифр жалобы, в котором переписчик не уверен."
+++

# 📰 Вечерний Валидатор

**№ 9.** *Лондон. Туман нынче стоит такой, что газовые рожки горят вполсилы, словно и им не выплатили награду за эпоху. Наш корреспондент поднимается по ступеням Избирательной палаты — того самого учреждения по адресу `-1:333…333`, где под сургучом и замком лежат заклады всех валидаторов города. Палату недавно переписали заново: старую книгу на funC сняли с полки и переложили на Tolk — чисто, аккуратно, с тестами. Но сыщик читает не текст. Сыщик читает поля.*

*Место происшествия: репозиторий [ton-blockchain/acton-contracts](https://github.com/ton-blockchain/acton-contracts), коммит [`7af1cea`](https://github.com/ton-blockchain/acton-contracts/commit/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16) (master от 08.08.2026), палата [`elector`](https://github.com/ton-blockchain/acton-contracts/tree/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/elector). Все цитаты дословные.*

Палата, напомним, — учреждение серьёзное: она принимает ставки, проводит выборы, замораживает и размораживает миллиарды, раздаёт награды и собирает штрафы. Сама [опись палаты](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/elector/README.md) сообщает, что перед нами порт оригинального [`elector-code.fc`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/smartcont/elector-code.fc). Порт добросовестный. Но у всякого переписчика есть перо, а у пера — свойство оскальзываться.

---

## СЕНСАЦИЯ ПЕРВАЯ: ПАЛАТА ВЫДАЁТ «БОНУЦЫ»

Откроем разморозочную книгу, файл [`elector/contracts/credits.tolk`](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/elector/contracts/credits.tolk), строки [35–66](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/elector/contracts/credits.tolk#L35-L66):

```tolk
fun CreditsMap.unfreezeWithBonuces(
    mutate self,
    freezeDict: FrozenDict,
    totalStakes: int,
    totalBonuses: int,
): int {
    var total = 0;
    var recovered = 0;
    var returnedBonuces = 0;
    var pubkey = -1;
    var freezeFound = true;
    do {
        val freezeLookup = freezeDict.findKeyGreater(pubkey);
        freezeFound = freezeLookup.isFound;
        if (freezeFound) {
            val freezeVal = freezeLookup.loadValue();
            pubkey = freezeLookup.getKey();
            if (freezeVal.isBanned) {
                recovered += freezeVal.stake;
            } else {
                val bonus = mulDivFloor(totalBonuses, freezeVal.stake, totalStakes);
                returnedBonuces += bonus;
                self.creditTo(freezeVal.addr, freezeVal.stake + bonus);
            }
            total += freezeVal.stake;
        }
    } while (freezeFound);

    assert ((total == totalStakes) && (returnedBonuces <= totalBonuses)) throw 59;

    return (recovered + totalBonuses - returnedBonuces);
}
```

Читатель, сверься с ритуалом: функция называется `unfreezeWithBonuces` — строка [35](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/elector/contracts/credits.tolk#L35). Переменная называется `returnedBonuces` — строка [43](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/elector/contracts/credits.tolk#L43). Рядом — без опечатки, старательно — стоит параметр `totalBonuses`. То есть в одной и той же канцелярии, в одной и той же форме, награда именуется то «bonuses», то «bonuces» — и никто за всю перепись не споткнулся.

Изящнее всего строка [63](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/elector/contracts/credits.tolk#L63): `assert ((total == totalStakes) && (returnedBonuces <= totalBonuses)) throw 59;` — стража у выхода из палаты держит в одной руке «бонусы», в другой «бонуцы» и сверяет их между собой с невозмутимостью присяжного. Проверка честная, арифметика верная, слово — из тумана.

Сыщик поднял старую книгу ради сличения почерка. В funC-оригинале, строки [299–300](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/smartcont/elector-code.fc#L299-L300), значится: `unfreeze_with_bonuses`, `var total = var recovered = var returned_bonuses = 0;` — написано правильно, без «ц». Следовательно, это не наследство старой палаты: опечатка родилась при переписи, в момент, когда перо переписчика дрогнуло — и опечатка, как водится в нуаре, вползла не в комментарий, который можно замарать, а в имена функций, строки [11](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/elector/contracts/credits.tolk#L11) и [35](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/elector/contracts/credits.tolk#L35), и в вызовы, строки [97](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/elector/contracts/credits.tolk#L97) и [102](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/elector/contracts/credits.tolk#L102). Переименуй её теперь — и придётся править всю книгу. «Бонуцы» обрели адрес, прописку и право подписи.

## СЕНСАЦИЯ ВТОРАЯ: ЧЕСТНОЕ «WTF» НА ПОЛЯХ ПРОТОКОЛА

Та же книга, несколькими строками ниже. Процедура общей разморозки — та самая, что возвращает валидаторам ставки минувших выборов, — файл [`elector/contracts/credits.tolk`](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/elector/contracts/credits.tolk), строки [93–103](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/elector/contracts/credits.tolk#L93-L103):

```tolk
    val pastElection = res.loadValue();
    // TODO
    // totalStakes = fdict.stakes_sup() WTF
    val unusedPrizes = (pastElection.bonuses > 0)
        ? self.unfreezeWithBonuces(
            pastElection.frozenDict,
            pastElection.totalStake,
            pastElection.bonuses,
        )
        : self.unfreezeWithoutBonuces(pastElection.frozenDict, pastElection.totalStake);
    return (pastElections, unusedPrizes);
```

Остановим газовый рожок над строками [94–95](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/elector/contracts/credits.tolk#L94-L95). `// TODO` — без текста, без подписи, дело не закрыто, мысль не окончена. И следом: `// totalStakes = fdict.stakes_sup() WTF`. Переписчик, видимо, споткнулся о вопрос, откуда брать сумму ставок, черкнул предположение про `stakes_sup()`, глянул в старую книгу — а там такой процедуры нет вовсе (в оригинале сумма хранится готовой, в записи о прошлых выборах), — и вместо ответа оставил на полях единственное честное слово, которое нашлось у него в тот вечер. И пошёл спать.

Поймите сыщика правильно: код ниже верен, сумма берётся из `pastElection.totalStake`, тесты ходят, палата работает. Но в протоколе учреждения, пересчитывающего миллиарды, на полях стоит «WTF» — не в блокноте переписчика, не в черновике, а в чистовике, сданном в репозиторий и подписанном коммитом. Туман в Лондоне, как известно, всё принимает. Git — тем более: он ещё и хранит.

## СЕНСАЦИЯ ТРЕТЬЯ: ШИФР ЖАЛОБЫ, В КОТОРОМ ПАЛАТА НЕ УВЕРЕНА

Теперь — жалобный стол. Файл [`elector/contracts/types.tolk`](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/elector/contracts/types.tolk), строки [101–111](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/elector/contracts/types.tolk#L101-L111):

```tolk
// TODO not sure about this opcode
struct (0xbc) Complaint {
    validatorPubkey: uint256
    description: cell
    createdAt: uint32
    severity: uint8
    rewardAddr: uint256
    paid: coins
    suggestedFine: coins
    suggestedFinePart: uint32
}
```

Жалоба — инструмент суровый: по ней валидатора штрафуют, замороженную ставку режут, награду за донос платят. У всякой бумаги в палате есть шифр, тег, по которому её узнают при разборе. У жалобы шифр `0xbc`, строка [102](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/elector/contracts/types.tolk#L102). А над шифром, строка [101](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/elector/contracts/types.tolk#L101), собственноручная приписка канцелярии: `// TODO not sure about this opcode`. «Не уверены в этом шифре».

Вот тут сыщик позволил себе улыбнуться — редкий случай. Потому что ответ на сомнение лежит в соседнем архиве, в той самой старой книге [`elector-code.fc`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/smartcont/elector-code.fc), строка [83](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/smartcont/elector-code.fc#L83): `throw_unless(9, cs~load_int(8) == 0xbc - 0x100);` — и строка [91](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/smartcont/elector-code.fc#L91): `.store_int(0xbc - 0x100, 8)`. Шифр верный: старая палата писала его как восьмибитовое число со знаком, `0xbc - 0x100`, а новая — как беззнаковое `0xbc`; величина та же, туман тот же. Сомнение можно было снять одним визитом в архив. Вместо этого сомнение запротоколировали, закоммитили и оставили висеть над жалобным столом, как висит над конторой непогашенный газовый рожок: светит, но греет мало.

---

## ХРОНИКА МЕЛКИХ ПРОИСШЕСТВИЙ

*Отдел пропавших вкладов.* Сердце палаты — процедура `tryElect`, файл [`elector/contracts/Elector.tolk`](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/elector/contracts/Elector.tolk), строка [385](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/elector/contracts/Elector.tolk#L385); именно тут сортируют ставки и решают, кто войдёт в новый состав. И тут, строки [449–450](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/elector/contracts/Elector.tolk#L449-L450), стоит запись, от которой у кассира холодеют пальцы:

```tolk
            // TODO: If validator sends more than maxStake, then the diff will be lost. This is the implementation detail from original funC implementation
            val stake = min(prevKey.stake, maxStake);
```

Перевод сыщика: если закладчик пришлёт больше верхнего порога, разница пропадёт — и это не ошибка, а «деталь реализации», унаследованная от старой книги. Палата знает о щели в полу у кассы, честно пишет о ней в журнале — и оставляет щель как есть, потому что в старом здании щель была той же формы. Нуар, как он есть: не воровство, а традиция.

---

## ВЕРДИКТ РЕДАКЦИИ

Скажем без подобострастия: порт сделан на совесть. Сравнение с оригиналом показывает — арифметика совпадает, шифры совпадают, тесты у палаты есть, и немалые. «Бонуцы» никому не проиграли ни ламы: компилятору безразлично, как зовут переменную, лишь бы звали одинаково. Шифр `0xbc` верен, невзирая на неуверенность канцелярии. А щель у кассы — та же, что была в старом здании, и сторожа к ней привыкли. Но газета ведёт хронику не арифметики, а почерка. И почерк этот гласит: в палате, где под замком лежат ставки всех валидаторов Туманного города, на полях чистовика стоят «TODO», «WTF» и «not sure». Палата работает. Поля — молчат. Туман над Темзой, по обычаю, всё принял — и даже не попросил расписку.
