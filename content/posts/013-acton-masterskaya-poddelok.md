+++
title = "№ 13 — Мастерская подделок: чужой паспорт, нулевая подпись и нулевая случайность"
date = 2026-08-20T11:06:03+03:00
description = "Тринадцатый выпуск «Вечернего Валидатора»: сыщик спускается в подвальный квартал acton, где печатают собственный город — локальную сеть, — и обнаруживает, что у города паспорт соседа, подписи из одних нулей, случайность из одних нулей и транзакции-манекены с честной припиской «HACK»."
+++

# 📰 Вечерний Валидатор

**№ 13.** *Лондон. Туман нынче вползает под двери ровными слоями, как листы из-под печатного станка. Наш корреспондент вновь в конторе [ton-blockchain/acton](https://github.com/ton-blockchain/acton) — газета уже описывала здешний список амнистированных опечаток (выпуск № 10), — но на сей раз сыщик миновал канцелярию и спустился по узкой лестнице в подвальный квартал: там, где контора печатает собственный город. Локальную сеть. Своих размеров, своей погоды, своих блоков. Мастерская работает в три смены. Сыщик поднял с верстака четыре изделия — и у каждого нашлось клеймо.*

*Место происшествия: репозиторий [ton-blockchain/acton](https://github.com/ton-blockchain/acton), коммит [`aeaa44b`](https://github.com/ton-blockchain/acton/commit/aeaa44b0aa470e70bfa23b4a82735fa27f72919c) (master от 19.08.2026). Все цитаты дословные.*

---

## СЕНСАЦИЯ ПЕРВАЯ: ГОРОД ПОЛЬЗУЕТСЯ ПАСПОРТОМ СОСЕДА

Всякий город имеет номер в большом реестре — `global_id`, по которому его узнают грамоты, кошельки и телеграф. Главный город значится под номером −239, тестовый — под номером −3; это записано в [описи протокола TonConnect](https://github.com/ton-blockchain/ton-connect/blob/5656a962eee30819a31a9e918e3de0b9614713b6/spec/connect.md#L146-L152), строки [146–152](https://github.com/ton-blockchain/ton-connect/blob/5656a962eee30819a31a9e918e3de0b9614713b6/spec/connect.md#L146-L152): `MAINNET = '-239'`, `TESTNET = '-3'`. Теперь откроем паспортный стол мастерской, файл [`crates/ton-localnet/src/block/types.rs`](https://github.com/ton-blockchain/acton/blob/aeaa44b0aa470e70bfa23b4a82735fa27f72919c/crates/ton-localnet/src/block/types.rs), строки [9–13](https://github.com/ton-blockchain/acton/blob/aeaa44b0aa470e70bfa23b4a82735fa27f72919c/crates/ton-localnet/src/block/types.rs#L9-L13):

```rust
/// Development-network global id written into localnet block/state cells.
///
/// The value is intentionally stable and local to Acton. It is not meant to
/// identify mainnet/testnet consensus data; it only keeps the generated TL-B
/// structures internally consistent for local tooling.
pub(crate) const LOCALNET_GLOBAL_ID: i32 = -3;
```

Осмотритесь при газовом рожке. Мастерская печатает каждому своему блоку — а блоки эти расходятся по всему [`ton-localnet`](https://github.com/ton-blockchain/acton/blob/aeaa44b0aa470e70bfa23b4a82735fa27f72919c/crates/ton-localnet/src/block/builder.rs#L61), строка [61](https://github.com/ton-blockchain/acton/blob/aeaa44b0aa470e70bfa23b4a82735fa27f72919c/crates/ton-localnet/src/block/builder.rs#L61), — паспорт с номером −3: ровно тем самым, под которым в большом реестре ходит тестовая сеть. Над номером, строки [9–12](https://github.com/ton-blockchain/acton/blob/aeaa44b0aa470e70bfa23b4a82735fa27f72919c/crates/ton-localnet/src/block/types.rs#L9-L12), намалёвана честная приписка: значение, мол, «местное», «не предназначено обозначать данные консенсуса mainnet/testnet». Сыщик переведёт с канцелярского: «паспорт выдан нами, номер чужой, но мы им пользуемся аккуратно». Город-подделка ходит по Лондону с паспортом соседа — и с запиской в кармане, что он не сосед. Туман, разумеется, принял и то и другое.

## СЕНСАЦИЯ ВТОРАЯ: ПОДПИСЬ ИЗ ШЕСТЬДЕСЯТИ ЧЕТЫРЁХ НУЛЕЙ

Второй верстак — отдел эмуляции кошельков, файл [`crates/ton-localnet/src/api/toncenter_emulate.rs`](https://github.com/ton-blockchain/acton/blob/aeaa44b0aa470e70bfa23b4a82735fa27f72919c/crates/ton-localnet/src/api/toncenter_emulate.rs). Здесь мастерская принимает TonConnect-грамоты и подшивает к ним подпись. Вот казённая печать, строка [20](https://github.com/ton-blockchain/acton/blob/aeaa44b0aa470e70bfa23b4a82735fa27f72919c/crates/ton-localnet/src/api/toncenter_emulate.rs#L20):

```rust
const DUMMY_SIGNATURE: [u8; 64] = [0; 64];
```

Шестьдесят четыре байта. Шестьдесят четыре нуля. Вся криптография эпохи, все десятилетия эллиптических кривых — и на выходе ровная вереница нулей, как туман над набережной. Процедура подписания — `add_dummy_signature`, строки [236–247](https://github.com/ton-blockchain/acton/blob/aeaa44b0aa470e70bfa23b4a82735fa27f72919c/crates/ton-localnet/src/api/toncenter_emulate.rs#L236-L247) — работает с достоинством старого нотариуса: для кошелька V5R1 нули ставятся после тела грамоты, строка [240](https://github.com/ton-blockchain/acton/blob/aeaa44b0aa470e70bfa23b4a82735fa27f72919c/crates/ton-localnet/src/api/toncenter_emulate.rs#L240), для всех прочих — перед телом, строка [242](https://github.com/ton-blockchain/acton/blob/aeaa44b0aa470e70bfa23b4a82735fa27f72919c/crates/ton-localnet/src/api/toncenter_emulate.rs#L242). Порядок соблюдён до миллиметра; лишь сама подпись не подписывает ничего. А в испытательном листе, строка [315](https://github.com/ton-blockchain/acton/blob/aeaa44b0aa470e70bfa23b4a82735fa27f72919c/crates/ton-localnet/src/api/toncenter_emulate.rs#L315), мастерская добросовестно проверяет, что подпись равна самой себе: `assert_eq!(signature, DUMMY_SIGNATURE);` — нули сверены с нулями, совпадение полное, расхождений не выявлено. Сыщик не удержался и спросил у дежурного, проверял ли кто-нибудь хоть одну настоящую подпись за всю историю отдела. Дежурный показал на табличку «emulate» и вернулся к нулям.

## СЕНСАЦИЯ ТРЕТЬЯ: СЛУЧАЙНОСТЬ, НАЗНАЧЕННАЯ ПРИКАЗОМ

Третий верстак — будочник гет-методов. Каждый, кто стучится в локальную сеть с вопросом к контракту, получает исполнение при свечах: в дело вводится `rand_seed`, «случайное зерно», от которого зависит вся дальнейшая судьба случайности в виртуальной машине. Файл [`crates/ton-localnet/src/localnet.rs`](https://github.com/ton-blockchain/acton/blob/aeaa44b0aa470e70bfa23b4a82735fa27f72919c/crates/ton-localnet/src/localnet.rs), строки [3175–3182](https://github.com/ton-blockchain/acton/blob/aeaa44b0aa470e70bfa23b4a82735fa27f72919c/crates/ton-localnet/src/localnet.rs#L3175-L3182):

```rust
    let args = RunGetMethodArgs {
        code: code_boc,
        data: data_boc,
        method_id,
        address: address.to_string(),
        unixtime: i64::from(node.now_unix()?),
        balance: meta.balance.to_string(),
        rand_seed: "0000000000000000000000000000000000000000000000000000000000000000".to_owned(),
        gas_limit: "10000000".to_owned(),
```

Случайное зерно, читатель, состоит из шестидесяти четырёх шестнадцатеричных нулей, строка [3180](https://github.com/ton-blockchain/acton/blob/aeaa44b0aa470e70bfa23b4a82735fa27f72919c/crates/ton-localnet/src/localnet.rs#L3180). Каждый вопрос, заданный любому контракту в любую минуту, исполняется при одной и той же «случайности». Контракт, призвавший `random`, получает не каприз судьбы, а распорядок учреждения: судьба в мастерской выдаётся по табелю и не меняется годами. Сыщик записал в блокнот старую истину нуара: в городе, где случайность назначается приказом, детективу не нужны улики — достаточно штатного расписания. А соседняя строка [3181](https://github.com/ton-blockchain/acton/blob/aeaa44b0aa470e70bfa23b4a82735fa27f72919c/crates/ton-localnet/src/localnet.rs#L3181) венчает картину: газ лимитирован ровно десятью миллионами — круглая сумма, выписанная от руки, как жалованье привидению.

## СЕНСАЦИЯ ЧЕТВЁРТАЯ: ТРАНЗАКЦИЯ-МАНЕКЕН С ЧЕСТНОЙ ЗАПИСКОЙ «HACK»

Последний верстак — самый изящный. Когда клиент просит мастерскую «отправить грамоту в настоящую сеть и подождать», конторе требуется вернуть расписку в фирменном виде — кортеж из ячеек и чисел. Но настоящей транзакции ещё нет: она лишь ушла в туман. Что делает мастерская? Печатает манекен. Файл [`src/ffi/emulation.rs`](https://github.com/ton-blockchain/acton/blob/aeaa44b0aa470e70bfa23b4a82735fa27f72919c/src/ffi/emulation.rs), строки [1041–1053](https://github.com/ton-blockchain/acton/blob/aeaa44b0aa470e70bfa23b4a82735fa27f72919c/src/ffi/emulation.rs#L1041-L1053):

```rust
/// Build a placeholder `SendResult` tuple for transactions broadcast to a real network.
///
/// The pseudo `tx` carries the external-in message in `in_msg` (for its `dst`) and the
/// TEP-467 normalized hash in `prev_trans_hash` (used as the lookup key on the wait side
/// — avoids re-normalizing locally). Other `Transaction` slots are minimal defaults so the
/// cell round-trips through TL-B serialization.
fn build_pseudo_broadcast_tx(now: u32, in_msg: Cell, norm_hash: HashBytes) -> TupleItem {
    let tx = Transaction {
        account: Default::default(),
        lt: 0,
        // HACK: abused slot — carries the TEP-467 normalized hash for the Rust-side lookup.
        prev_trans_hash: norm_hash,
        prev_trans_lt: 0,
```

Перечтите ещё раз, медленно. Мастерская отливает фальшивую `Transaction`: счёт по умолчанию, логическое время ноль, строка [1050](https://github.com/ton-blockchain/acton/blob/aeaa44b0aa470e70bfa23b4a82735fa27f72919c/src/ffi/emulation.rs#L1050), статус на входе и на выходе — `Uninit`, строки [1056–1057](https://github.com/ton-blockchain/acton/blob/aeaa44b0aa470e70bfa23b4a82735fa27f72919c/src/ffi/emulation.rs#L1056-L1057), то есть транзакция, которая не происходила с аккаунтом, которого не было. А в графу «хеш предыдущей транзакции», `prev_trans_hash`, строка [1052](https://github.com/ton-blockchain/acton/blob/aeaa44b0aa470e70bfa23b4a82735fa27f72919c/src/ffi/emulation.rs#L1052), контрабандой провезён нормализованный хеш исходящей грамоты — чтобы по нему потом разыскать настоящую. Над этой контрабандой собственноручная пометка мастера, строка [1051](https://github.com/ton-blockchain/acton/blob/aeaa44b0aa470e70bfa23b4a82735fa27f72919c/src/ffi/emulation.rs#L1051): `// HACK: abused slot — carries the TEP-467 normalized hash for the Rust-side lookup.` «Графа используется не по назначению». Манекен далее облачается в полный наряд — пропущенную фазу вычислений с причиной `NoState`, пустые списки исходящих — и проходит через казённый TL-B-пропускной пункт как ни в чём не бывало, строка [1082](https://github.com/ton-blockchain/acton/blob/aeaa44b0aa470e70bfa23b4a82735fa27f72919c/src/ffi/emulation.rs#L1082). Сыщик видел подделки грубее. Но подделки, у которых на подкладке честно вышито «abused slot», он встречает впервые — и склоняет шляпу.

---

## ХРОНИКА МЕЛКИХ ПРОИСШЕСТВИЙ

*Отдел исповедей.* В отладочном крыле, файл [`crates/acton-debug/src/core/replayer.rs`](https://github.com/ton-blockchain/acton/blob/aeaa44b0aa470e70bfa23b4a82735fa27f72919c/crates/acton-debug/src/core/replayer.rs), над процедурой, перетасовывающей отладочные метки, вывешена исповедь в пятнадцать строк — строки [687–703](https://github.com/ton-blockchain/acton/blob/aeaa44b0aa470e70bfa23b4a82735fa27f72919c/crates/acton-debug/src/core/replayer.rs#L687-L703), и начинается она капслоком, строка [687](https://github.com/ton-blockchain/acton/blob/aeaa44b0aa470e70bfa23b4a82735fa27f72919c/crates/acton-debug/src/core/replayer.rs#L687): `/// THIS HACK SHOULD BE DELETED LATER`. «Этот хак следует удалить позже». Далее — полное признание: откуда взялся хак, почему он нужен, какие два пути его отменят, строка [702](https://github.com/ton-blockchain/acton/blob/aeaa44b0aa470e70bfa23b4a82735fa27f72919c/crates/acton-debug/src/core/replayer.rs#L702): `/// Both solutions will eliminate the need of this hack.` Преступление описано подробнее, чем иные законы. Сыщик давно заметил: слово «later» в таких исповедях — самый долгоживущий житель Лондона; оно переживает туманы, рефакторинги и самих исповедников.

*Отдел отложенных дел.* В архиве повторных обходов, файл [`crates/ton-retrace/src/tests.rs`](https://github.com/ton-blockchain/acton/blob/aeaa44b0aa470e70bfa23b4a82735fa27f72919c/crates/ton-retrace/src/tests.rs), два следственных эксперимента — `test_retrace_lib_load`, строка [104](https://github.com/ton-blockchain/acton/blob/aeaa44b0aa470e70bfa23b4a82735fa27f72919c/crates/ton-retrace/src/tests.rs#L104), и `test_retrace_v12`, строка [117](https://github.com/ton-blockchain/acton/blob/aeaa44b0aa470e70bfa23b4a82735fa27f72919c/crates/ton-retrace/src/tests.rs#L117), — отстранены от службы с одинаковой формулировкой на погонах, строки [102](https://github.com/ton-blockchain/acton/blob/aeaa44b0aa470e70bfa23b4a82735fa27f72919c/crates/ton-retrace/src/tests.rs#L102) и [115](https://github.com/ton-blockchain/acton/blob/aeaa44b0aa470e70bfa23b4a82735fa27f72919c/crates/ton-retrace/src/tests.rs#L115): `#[ignore] // flaky as hell for some reason`. «Шатки, как сама преисподняя, по причинам неведомым». Оба — слово в слово. Один приговор, две копии, ноль расследований. Туман, как водится, причин не называет.

---

## ВЕРДИКТ РЕДАКЦИИ

Скажем по совести: мастерская подделок безвредна, как безвреден театральный реквизит. Локальная сеть обязана быть фальшивкой — в том её служба: нулевая подпись не подписывает настоящих денег, нулевая случайность не губит настоящих лотерей, транзакция-манекен никогда не покидает витрины, а чужой паспорт носится внутри собственного двора. Больше того: всякий раз мастер честно подписывает изделие — «DUMMY», «HACK», «abused slot», «SHOULD BE DELETED LATER». И всё же газета ведёт хронику почерка, а почерк сей вечер примечателен: город, напечатанный в подвале, ходит с номером −3 соседнего города, подписывает грамоты шестьюдесятью четырьмя нулями, бросает жребий, в котором заранее все нули, и выдаёт клиентам расписки за транзакции, которые ещё бродят в тумане. Если завтра сыщик встретит в настоящем городе нулевую подпись — он будет знать, чья это школа. Туман над Темзой всё принял — но, кажется, впервые попросил показать паспорт. Ему показали чужой. Туман подумал и принял снова.
