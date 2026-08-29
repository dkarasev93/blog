+++
title = "№ 31 — Палата суеверий: заклинание «dont work», тридцать две единицы ручной работы, дверь, запертая словом false, и сторожевой пароль «w0w»"
date = 2026-08-29T14:35:00+03:00
description = "Тридцать первый выпуск «Вечернего Валидатора»: сыщик входит в спящую типографию ton-community/tonweb, где пять лет висит заклинание «todo: dont work» — отменённая строка writeInt(-1, 32), заменённая ручным циклом из тридцати двух единиц и переписанная копировальным клерком в шесть контор подряд. Лабораторная проверка редакции показала: заклинание врёт, запрещённая строка исправно печатает те самые тридцать две единицы. В придачу — ветка, навсегда запертая словом «false &&» из-за парсера, умеющего читать только ссылки, и закомментированный сторожевой пароль Ledger «w0w», записанный в книгу первого апреля."
tags = ["tonweb"]
+++

# 📰 Вечерний Валидатор

**Выпуск № 31.** *Лондон. Туман нынче густ, как чужой JavaScript без типов. Сыщик нашей газеты, перейдя мост через канал, вошёл в старую типографию [ton-community/tonweb](https://github.com/ton-community/tonweb) — ту самую, что печатала адреса, ячейки и подписи для половины города ещё до того, как нынешние модные конторы открыли свои лавочки. Дверь была не заперта, но внутри спало всё: последняя запись в журнале — [`76dfd07`](https://github.com/ton-community/tonweb/commit/76dfd0701714c0a316aee503c2962840acaf74ef), «0.0.66 build», март месяц 2024 года. Типография спит второй год. Но стены, читатель, стены говорят.*

*Место осмотра: репозиторий [ton-community/tonweb](https://github.com/ton-community/tonweb), коммит [`76dfd07`](https://github.com/ton-community/tonweb/commit/76dfd0701714c0a316aee503c2962840acaf74ef) (вершина master). Файлы: [`src/contract/wallet/WalletContractV3.js`](https://github.com/ton-community/tonweb/blob/76dfd0701714c0a316aee503c2962840acaf74ef/src/contract/wallet/WalletContractV3.js), [`src/boc/BitString.js`](https://github.com/ton-community/tonweb/blob/76dfd0701714c0a316aee503c2962840acaf74ef/src/boc/BitString.js), [`src/contract/index.js`](https://github.com/ton-community/tonweb/blob/76dfd0701714c0a316aee503c2962840acaf74ef/src/contract/index.js), [`src/ledger/AppTon.js`](https://github.com/ton-community/tonweb/blob/76dfd0701714c0a316aee503c2962840acaf74ef/src/ledger/AppTon.js). Все цитаты дословные.*

---

## ДЕЛО ПЕРВОЕ: ЗАКЛИНАНИЕ «DONT WORK»

Начнём с сути ремесла. Когда кошелёк TON ещё не развёрнут в сети, его первое подписываемое послание — при `seqno`, равном нулю — обязано нести в поле срока годности не дату, а тридцать две единицы подряд: `0xFFFFFFFF`, «вечность», минус единица беззнакового тридцатидвухбитного. Так велит контракт, так записано в протоколе.

Открываем [`WalletContractV3.js`](https://github.com/ton-community/tonweb/blob/76dfd0701714c0a316aee503c2962840acaf74ef/src/contract/wallet/WalletContractV3.js), строки [20–24](https://github.com/ton-community/tonweb/blob/76dfd0701714c0a316aee503c2962840acaf74ef/src/contract/wallet/WalletContractV3.js#L20-L24):

```js
// message.bits.writeInt(-1, 32);// todo: dont work
for (let i = 0; i < 32; i++) {
    message.bits.writeBit(1);
}
```

Переведём для читателя, не знакомого с типографским жаргоном. Наборщик хотел набрать тридцать две единицы одной строкой — `writeInt(-1, 32)`, минус единица в тридцати двух битах дополнительного кода и есть тридцать две единицы. Но над строкой повешено заклинание: **«todo: dont work»** — «не работает». И вместо одной строки — ручной станок: цикл, тридцать два оборота, по одному биту за оборот, единица за единицей, как чеканщик, стучащий тридцать две монеты молотком, потому что «станок, говорят, сломан».

Заклинание висит здесь с ноября 2021 года — редакция сняла показания с дактилоскопии (`git blame`, коммит `67f6379c`, 1 ноября 2021-го, три часа ночи по местному времени, что само по себе многое поясняет).

Но это ещё не дело. Дело в том, что **заклинание размножилось**. Сыщик обошёл типографию с фонарём и насчитал шесть одинаковых надгробий, переписанных копировальным клерком слово в слово, запятая в запятую:

- [`WalletContractV2.js`](https://github.com/ton-community/tonweb/blob/76dfd0701714c0a316aee503c2962840acaf74ef/src/contract/wallet/WalletContractV2.js), строка [18](https://github.com/ton-community/tonweb/blob/76dfd0701714c0a316aee503c2962840acaf74ef/src/contract/wallet/WalletContractV2.js#L18);
- [`WalletContractV3.js`](https://github.com/ton-community/tonweb/blob/76dfd0701714c0a316aee503c2962840acaf74ef/src/contract/wallet/WalletContractV3.js), строка [20](https://github.com/ton-community/tonweb/blob/76dfd0701714c0a316aee503c2962840acaf74ef/src/contract/wallet/WalletContractV3.js#L20);
- [`WalletContractV4.js`](https://github.com/ton-community/tonweb/blob/76dfd0701714c0a316aee503c2962840acaf74ef/src/contract/wallet/WalletContractV4.js), строка [39](https://github.com/ton-community/tonweb/blob/76dfd0701714c0a316aee503c2962840acaf74ef/src/contract/wallet/WalletContractV4.js#L39);
- [`WalletContractV4R2.js`](https://github.com/ton-community/tonweb/blob/76dfd0701714c0a316aee503c2962840acaf74ef/src/contract/wallet/WalletContractV4R2.js), строка [45](https://github.com/ton-community/tonweb/blob/76dfd0701714c0a316aee503c2962840acaf74ef/src/contract/wallet/WalletContractV4R2.js#L45);
- [`LockupWalletV1.js`](https://github.com/ton-community/tonweb/blob/76dfd0701714c0a316aee503c2962840acaf74ef/src/contract/lockup/LockupWalletV1.js), строка [56](https://github.com/ton-community/tonweb/blob/76dfd0701714c0a316aee503c2962840acaf74ef/src/contract/lockup/LockupWalletV1.js#L56);
- [`VestingWalletV1.js`](https://github.com/ton-community/tonweb/blob/76dfd0701714c0a316aee503c2962840acaf74ef/src/contract/lockup/VestingWalletV1.js), строка [49](https://github.com/ton-community/tonweb/blob/76dfd0701714c0a316aee503c2962840acaf74ef/src/contract/lockup/VestingWalletV1.js#L49).

Шесть контор — кошельки второй, третьей, четвёртой ревизий, оба lockup-заведения — и в каждой одно и то же: закомментированная строка, эпитафия «dont work» без глагольной грамоты, и ручной цикл из тридцати двух единиц. Новый кошелёк входил в город — и первым делом ему выдавали копию суеверия, как униформу.

## ЭКСПЕРТИЗА: РАБОТАЕТ ЛИ ЗАПРЕЩЁННАЯ СТРОКА?

Сыщик не привык верить заклинаниям. Откроем сам «сломанный станок» — [`BitString.js`](https://github.com/ton-community/tonweb/blob/76dfd0701714c0a316aee503c2962840acaf74ef/src/boc/BitString.js), метод `writeInt`, строки [160–164](https://github.com/ton-community/tonweb/blob/76dfd0701714c0a316aee503c2962840acaf74ef/src/boc/BitString.js#L160-L164):

```js
if (number.isNeg()) {
    this.writeBit(true);
    const b = new BN(2);
    const nb = b.pow(new BN(bitLength - 1));
    this.writeUint(nb.add(number), bitLength - 1);
}
```

Арифметика на полях протокола: число отрицательное — пишем старший бит `1`, затем прибавляем число к 2³¹ и записываем оставшийся тридцать один бит. Для минус единицы: 2³¹ − 1 = 2147483647, то есть тридцать одна единица. Итого — старшая единица плюс тридцать одна единица равно **ровно тридцать две единицы**. То самое, что клерк набивает вручную циклом.

Редакция не удовлетворилась арифметикой и провела экспертизу в лаборатории: подняли ту самую библиотеку `bn.js`, что стоит в типографии, и произнесли запрещённое заклинание. Лабораторный журнал гласит: `isNeg true`, `value 2147483647`, тридцать одна единица в хвосте. Станок **исправен**. Строка `writeInt(-1, 32)` печатает в точности то же, что ручной цикл, — и, судя по всему, печатала всегда. Либо поломка когда-то была и давно починена, либо её не было вовсе, а был испуг трёх часов ночи. Но заклинание осталось, разошлось по шести конторам и висит до сих пор — пятый год, оберегая типографию от исправного станка.

## ДЕЛО ВТОРОЕ: ДВЕРЬ, ЗАПЕРТАЯ СЛОВОМ FALSE

В архивной палате [`src/contract/index.js`](https://github.com/ton-community/tonweb/blob/76dfd0701714c0a316aee503c2962840acaf74ef/src/contract/index.js), строки [216–223](https://github.com/ton-community/tonweb/blob/76dfd0701714c0a316aee503c2962840acaf74ef/src/contract/index.js#L216-L223), сыщик обнаружил дверь, которую заперли изнутри — навсегда, но «временно»:

```js
// TODO we also should check for free refs here
// TODO: temporary always push in ref because WalletQueryParser can parse only ref
if (false && (commonMsgInfo.bits.getFreeBits() - 1 >= stateInit.bits.getUsedBits())) {
    commonMsgInfo.bits.writeBit(false);
    commonMsgInfo.writeCell(stateInit);
} else {
    commonMsgInfo.bits.writeBit(true);
    commonMsgInfo.refs.push(stateInit);
}
```

Здесь, читатель, ювелирная работа. По замыслу, `stateInit` следовало класть прямо в биты сообщения, когда места хватает, — экономия ячейки, экономия газа, градостроительная норма. Но условие, проверяющее место, навечно обезврежено приставкой **`false &&`** — ложь в связке «и», после которой правая часть не вычисляется никогда, а ветка с записью в биты мертва при рождении. Причина записана честно: `WalletQueryParser can parse only ref` — домашний разборщик посланий умеет читать только ссылки, и потому вся улица обязана писать только ссылками. «Временно» — датировано дактилоскопией 2 октября 2022 года (коммит `51f06898`). Временно четвёртый год; разборщик так и не научился читать биты, дверь так и заперта, а миллионы посланий, собранных этой типографией, несут stateInit в ссылке, потому что когда-то кому-то было недосуг дописать парсер.

## ДЕЛО ТРЕТЬЕ: СТОРОЖЕВОЙ ПАРОЛЬ «W0W»

И вишенка на ночном столике — сейфовая палата аппаратных ключей, [`src/ledger/AppTon.js`](https://github.com/ton-community/tonweb/blob/76dfd0701714c0a316aee503c2962840acaf74ef/src/ledger/AppTon.js), строки [21–31](https://github.com/ton-community/tonweb/blob/76dfd0701714c0a316aee503c2962840acaf74ef/src/ledger/AppTon.js#L21-L31):

```js
// todo: узнать зачем вызывается decorateAppAPIMethods
// const scrambleKey = "w0w";
// transport.decorateAppAPIMethods(
//     this,
//     [
//         "getAppConfiguration",
//         "getAddress",
//         "sign",
//         "signTransfer",
//     ],
//     scrambleKey
// );
```

Пункт первый: деловая переписка в сейфовой палате ведётся **по-русски** — «узнать зачем вызывается». В типографии, где весь прочий протокол писан по-английски, самый интимный вопрос — зачем вообще вызывается главный метод обвязки сейфа — задан на языке ночных сомнений. Пункт второй: сторожевой пароль к сейфу, `scrambleKey`, — это слово **«w0w»**. «Уау» с нулём вместо буквы. Пароль, достойный дверного коврика, предназначался для обвязки методов подписи — того самого `signTransfer`, которым подписывают переводы. Пункт третий: дактилоскопия датирует запись **1 апреля 2021 года** (коммит `c0a829be`). Первое апреля, читатель. Вся конструкция, к счастью для вкладчиков, закомментирована целиком — сейф работает без сторожевого «уау». Но запись в книге осталась: было желание, и было первое апреля, и пароль был подобран под дату.

## РЕЗЮМЕ СЫЩИКА

Типография [tonweb](https://github.com/ton-community/tonweb) спит с марта 2024-го, и, возможно, оно к лучшему. Но спит она крепко, по-солдатски, и вещи в ней лежат, как их оставили:

1. Заклинание «dont work» над исправным станком — переписано в шесть контор и охраняет типографию от работающего кода пятый год.
2. «Временная» заглушка `false &&` — четвёртый год запирает экономную ветку из-за парсера, умеющего читать только ссылки.
3. Сторожевой пароль «w0w» — закомментирован, записан первого апреля и, кажется, единственный в городе признался в этом честно.

Суеверия, читатель, — единственный код, который не гниёт. Его не рефакторят, не ревьюят и не удаляют: его переписывают — из конторы в контору, из года в год, буква в букву. Проверяйте заклинания в лаборатории. Туман рассеивается, `writeInt` работает, а «временно» в нашем городе — самое долгое слово.

🐀
