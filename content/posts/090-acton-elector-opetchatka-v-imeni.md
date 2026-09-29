+++
title = "№ 90 — Выборы, которые чуть не стали корректными"
date = 2026-09-29T08:00:00+03:00
description = "Девяностый выпуск «Вечернего Валидатора»: в Elector.tolk слово currectElections пришлось вылавливать из контракта по всему дому."
tags = ["acton-contracts"]
+++

# ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

**№ 90.** *Лондон. Утренний туман лип к мостовой, газовый рожок у редакции кашлял в ладонь, а сыщик получил из [ton-blockchain/acton-contracts](https://github.com/ton-blockchain/acton-contracts) записку о выборах. В ней одно слово долго ходило по коридорам в чужом пальто: `currectElections`.*

## МЕСТО ПРОИСШЕСТВИЯ

Улика найдена в репозитории [ton-blockchain/acton-contracts](https://github.com/ton-blockchain/acton-contracts), в коммите [`58e02a6`](https://github.com/ton-blockchain/acton-contracts/commit/58e02a6a6f7a161647f5861c57a27b05fae38fd4), названном `fix(elector): correct typos (#101)`. Главная комната дела — файл [`elector/contracts/Elector.tolk`](https://github.com/ton-blockchain/acton-contracts/blob/58e02a6a6f7a161647f5861c57a27b05fae38fd4/elector/contracts/Elector.tolk). Цитаты сняты с этого коммита.

## ПЕРВЫЙ СЛЕД: ИМЯ, КОТОРОЕ НЕ УМЕЛО ПИСАТЬ САМО СЕБЯ

В старой ведомости поле звалось `currectElections` — не «current», а некое кректное создание из двух ошибок. Коммит [`58e02a6`](https://github.com/ton-blockchain/acton-contracts/commit/58e02a6a6f7a161647f5861c57a27b05fae38fd4) прошел по Elector и заменил это имя на `currentElections`. В новом снимке место хранения уже выглядит так, на строках [37–42](https://github.com/ton-blockchain/acton-contracts/blob/58e02a6a6f7a161647f5861c57a27b05fae38fd4/elector/contracts/types.tolk#L37-L42) файла [`elector/contracts/types.tolk`](https://github.com/ton-blockchain/acton-contracts/blob/58e02a6a6f7a161647f5861c57a27b05fae38fd4/elector/contracts/types.tolk):

```tolk
struct Storage {
    currentElections: Cell<Elections>?
    credits: CreditsMap
    pastElections: map<uint32, PastElection>
    grams: coins
    activeId: uint32
}
```

Слово `currentElections` теперь говорит ровно то, что от него ждут: текущие выборы. Но редакция не может не отметить театральность сцены. Речь идет о поле в хранилище контракта выборов, а преступление выглядит как опечатка школьного переписчика, который запомнил слово «current» по слуху.

## ВТОРОЙ СЛЕД: СТРАЖА ВЫЗЫВАЮТ ПО НОВОМУ ИМЕНИ

Самая сочная часть дела не в том, что в одном месте поселилась лишняя буква. Старое имя было размазано по контракту: его исправляли в функциях приема ставки, проверки выборов, обработки тиков и выдачи списков. В актуальном коммите функция `processNewStake` уже обращается к исправленному полю на строках [145–150](https://github.com/ton-blockchain/acton-contracts/blob/58e02a6a6f7a161647f5861c57a27b05fae38fd4/elector/contracts/Elector.tolk#L145-L150):

```tolk
fun processNewStake(msg: NewStakeMessage): void {
    val srcWcAndHash = InMessage.source().getWorkchainAndHash();
    var storage = Storage.load();
    if ((storage.currentElections == null) || (srcWcAndHash.0 != -1)) {
        return returnStake(InMessage.source(), msg.queryId, StakeReason.Ok);
    }
```

Дальше, на строках [166–170](https://github.com/ton-blockchain/acton-contracts/blob/58e02a6a6f7a161647f5861c57a27b05fae38fd4/elector/contracts/Elector.tolk#L166-L170), тот же служащий достает выборы и сверяет размер ставки:

```tolk
var elect = storage.currentElections.load();
var msgValue = InMessage.value() - ton("1"); // Deduct 1 ton for sending confirmation
if ((msgValue << 12) < elect.totalStake) {
    return returnStake(InMessage.source(), msg.queryId, StakeReason.StakeTooSmall);
}
```

Наконец, на строках [214–218](https://github.com/ton-blockchain/acton-contracts/blob/58e02a6a6f7a161647f5861c57a27b05fae38fd4/elector/contracts/Elector.tolk#L214-L218), результат возвращается в то же хранилище:

```tolk
elect.isFailed = false;
elect.isFinished = false;

storage.currentElections = elect.toCell();
storage.save();
```

Итак, в одной комнате поле носит исправленное имя, в другой его загружают, в третьей сохраняют. Контракт выбора валидаторов пережил переименование без смены сюжета: ставка вошла, проверка прошла, ведомость ушла обратно в сейф. Но до санитарного коммита весь этот коридор был заставлен табличками `currectElections`.

## ТРЕТИЙ СЛЕД: ПРИЗНАНИЕ В НАЗВАНИИ КОММИТА

Сам коммит [`58e02a6`](https://github.com/ton-blockchain/acton-contracts/commit/58e02a6a6f7a161647f5861c57a27b05fae38fd4) не прячется за туманной формулировкой. На двери прямо написано: `fix(elector): correct typos (#101)`. Не «переписать модель хранения», не «пересобрать механизм ставок», а буквально — исправить опечатки.

В той же папке коммит поправляет `gas efficency` на `gas efficiency` в [`README.md`](https://github.com/ton-blockchain/acton-contracts/blob/58e02a6a6f7a161647f5861c57a27b05fae38fd4/README.md), но редакция оставляет эту добычу на полке. Одной опечатки в тексте хватило бы для школьной хроники. Ошибка в имени поля, которое проходит через хранилище Elector, пахнет куда гуще.

Особенно пикантно, что слово `currectElections` не было кличкой одного случайного свидетеля. В коммите [`58e02a6`](https://github.com/ton-blockchain/acton-contracts/commit/58e02a6a6f7a161647f5861c57a27b05fae38fd4) его меняют и в описании типа [`elector/contracts/types.tolk`](https://github.com/ton-blockchain/acton-contracts/blob/58e02a6a6f7a161647f5861c57a27b05fae38fd4/elector/contracts/types.tolk), и в логике [`elector/contracts/Elector.tolk`](https://github.com/ton-blockchain/acton-contracts/blob/58e02a6a6f7a161647f5861c57a27b05fae38fd4/elector/contracts/Elector.tolk). Опечатка успела стать семейным именем.

## ЧЕТВЕРТЫЙ СЛЕД: КАК СЫЩИК ОТЛИЧАЕТ КРИНЖ ОТ КАТАСТРОФЫ

Редакция не станет кричать, что одна лишняя буква немедленно украла бы ставки. У нас есть более надежный факт: коммит [`58e02a6`](https://github.com/ton-blockchain/acton-contracts/commit/58e02a6a6f7a161647f5861c57a27b05fae38fd4) массово исправляет имя поля во всех местах, где оно описывается, читается и сохраняется. Это санитарный ремонт, а не доказательство уже случившегося ограбления.

Кринж здесь в другом. Контракт, где имена полей являются частью общей схемы и проникают в десятки обращений, носил слово, которое само выглядело как черновик к исправлению. Пока все служащие синхронно повторяют ошибку, она притворяется архитектурой. Стоит одному новому хранителю написать `currentElections`, и семейная тайна выходит на свет.

## ХРОНИКА МЕЛКОЙ КАНЦЕЛЯРИИ

*Отдел причин.* В [`elector/contracts/Elector.tolk`](https://github.com/ton-blockchain/acton-contracts/blob/58e02a6a6f7a161647f5861c57a27b05fae38fd4/elector/contracts/Elector.tolk) комментарий на строках [229–234](https://github.com/ton-blockchain/acton-contracts/blob/58e02a6a6f7a161647f5861c57a27b05fae38fd4/elector/contracts/Elector.tolk#L229-L234) по-прежнему говорит: `somebody's joke?` — кто-то пошутил? Газета ценит, когда контракт сам оставляет записку для сыщика.

*Отдел вычетов.* На строке [167](https://github.com/ton-blockchain/acton-contracts/blob/58e02a6a6f7a161647f5861c57a27b05fae38fd4/elector/contracts/Elector.tolk#L167) из ставки вынимают ровно один TON на подтверждение. Финансовый протокол лаконичен: один TON за письмо, даже если письмо несет весть о недостаточной ставке.

*Отдел чистописания.* В [`elector/contracts/types.tolk`](https://github.com/ton-blockchain/acton-contracts/blob/58e02a6a6f7a161647f5861c57a27b05fae38fd4/elector/contracts/types.tolk) на строке [38](https://github.com/ton-blockchain/acton-contracts/blob/58e02a6a6f7a161647f5861c57a27b05fae38fd4/elector/contracts/types.tolk#L38) теперь стоит аккуратное `currentElections`. Сыщик проверил трижды: буквы на месте, выборы тоже.

## ВЕРДИКТ СЫЩИКА

Репозиторий [ton-blockchain/acton-contracts](https://github.com/ton-blockchain/acton-contracts) пойман в коммите [`58e02a6`](https://github.com/ton-blockchain/acton-contracts/commit/58e02a6a6f7a161647f5861c57a27b05fae38fd4) на деле о семейной опечатке. В файле [`elector/contracts/types.tolk`](https://github.com/ton-blockchain/acton-contracts/blob/58e02a6a6f7a161647f5861c57a27b05fae38fd4/elector/contracts/types.tolk) строка [38](https://github.com/ton-blockchain/acton-contracts/blob/58e02a6a6f7a161647f5861c57a27b05fae38fd4/elector/contracts/types.tolk#L38) теперь хранит `currentElections`, а функция в [`elector/contracts/Elector.tolk`](https://github.com/ton-blockchain/acton-contracts/blob/58e02a6a6f7a161647f5861c57a27b05fae38fd4/elector/contracts/Elector.tolk) читает и сохраняет это поле на строках [166–170](https://github.com/ton-blockchain/acton-contracts/blob/58e02a6a6f7a161647f5861c57a27b05fae38fd4/elector/contracts/Elector.tolk#L166-L170) и [214–218](https://github.com/ton-blockchain/acton-contracts/blob/58e02a6a6f7a161647f5861c57a27b05fae38fd4/elector/contracts/Elector.tolk#L214-L218).

Приговор мягкий, но с печатью: если поле хранит текущие выборы, не называй его так, будто корректор уже ушел домой. В Лондоне туман скрывает номера домов, но не должен скрывать лишнюю букву в имени сейфа.

Газовый рожок дернулся. Сыщик закрыл папку, перечеркнул `currectElections`, вывел `currentElections` и растворился в тумане.

🐀
