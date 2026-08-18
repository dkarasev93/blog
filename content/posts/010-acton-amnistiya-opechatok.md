+++
title = "№ 10 — Амнистия опечаток: «readed», «lazer» и список неприкосновенных слов"
date = 2026-08-18T17:06:11+03:00
description = "Десятый выпуск «Вечернего Валидатора»: сыщик спускается в подвал конторы acton, где служит штатный цензор опечаток, и обнаруживает, что цензору вручили список слов, на которые он обязан закрывать глаза."
+++

# 📰 Вечерний Валидатор

**№ 10.** *Лондон. Туман нынче вползает не только в окна, но и в словари. Наш корреспондент впервые переступает порог конторы [ton-blockchain/acton](https://github.com/ton-blockchain/acton) — нового сыскного агентства для разработчиков, того самого CLI, что собирает контракты, гоняет тесты и будит локальные сети. Газета уже бывала в их картотеке контрактов (выпуск № 9, Избирательная палата с «бонуцами»), но в самой конторе до сего вечера не бывал никто. Зря откладывали.*

*Место происшествия: репозиторий [ton-blockchain/acton](https://github.com/ton-blockchain/acton), коммит [`857c24b`](https://github.com/ton-blockchain/acton/commit/857c24b40f40d7a70782537603ee64017fd0f806) (master от 18.08.2026, свежее утреннего тумана). Все цитаты дословные.*

В конторе, надо отдать должное, чистота: семь с лишним сотен ржавых folio, CI, проверки, линтеры. И штатный цензор опечаток — программа `typos`, нанятая на постоянной основе: в [описи ночных обходов](https://github.com/ton-blockchain/acton/blob/857c24b40f40d7a70782537603ee64017fd0f806/justfile#L126-L127), строки [126–127](https://github.com/ton-blockchain/acton/blob/857c24b40f40d7a70782537603ee64017fd0f806/justfile#L126-L127), значится коротко: `typos .` — «обойди весь дом». В казённой [ведомости сборки](https://github.com/ton-blockchain/acton/blob/857c24b40f40d7a70782537603ee64017fd0f806/.github/workflows/build.yml#L257), строка [257](https://github.com/ton-blockchain/acton/blob/857c24b40f40d7a70782537603ee64017fd0f806/.github/workflows/build.yml#L257), цензор числится по имени и званию: `typos@1.47.2`. Служит не покладая глаз. И вот тут сыщик попросил у дежурного второй документ — тот, что цензор носит в нагрудном кармане.

---

## СЕНСАЦИЯ ПЕРВАЯ: СПИСОК СЛОВ, КОТОРЫЕ ЦЕНЗОР ОБЯЗАН НЕ ВИДЕТЬ

Файл [`_typos.toml`](https://github.com/ton-blockchain/acton/blob/857c24b40f40d7a70782537603ee64017fd0f806/_typos.toml), строки [29–45](https://github.com/ton-blockchain/acton/blob/857c24b40f40d7a70782537603ee64017fd0f806/_typos.toml#L29-L45):

```toml
[default]
extend-ignore-re = [
  "\\b[a-zA-Z0-9_+/\\-]{48,}\\b", # user-friendly addresses
  "\\bADDD\\b",
  "\\bTrUe\\b",
  "\\balse\\b",
  "\\b(?:test-)?ba-stdlib-[A-Za-z0-9-]+\\b",
  "[dD]arcula",
  "BA",
  "Ba",
  "Inferrable",
  "readed",
  "authentificate",
  "authentification",
  "ba",
  "lazer",
  "\\b(?:vart|fo|FO|Fo|tou|overfl|bui|tru|TNam)\\b", # completion prefixes
]
```

Осмотрите этот документ при газовом рожке. Это не список разрешённых адресов и не таблица шифров — это, читатель, **амнистия**. Контора наняла ловца опечаток, а потом собственноручно выдала ему перечень слов, при виде которых он должен отвернуться и засвистеть в туман. Среди амнистированных — «alse» (строка [34](https://github.com/ton-blockchain/acton/blob/857c24b40f40d7a70782537603ee64017fd0f806/_typos.toml#L34)), «readed» (строка [40](https://github.com/ton-blockchain/acton/blob/857c24b40f40d7a70782537603ee64017fd0f806/_typos.toml#L40)), «authentificate» и «authentification» (строки [41–42](https://github.com/ton-blockchain/acton/blob/857c24b40f40d7a70782537603ee64017fd0f806/_typos.toml#L41-L42)) и — любимец редакции — «lazer» (строка [44](https://github.com/ton-blockchain/acton/blob/857c24b40f40d7a70782537603ee64017fd0f806/_typos.toml#L44)). Лазер. Через «з». Навсегда легализован, подписано, скреплено TOML-ом.

Сыщик, разумеется, не поверил бумаге и пошёл проверять, кто именно укрывается за этой амнистией. Результаты — ниже, и они прекрасны каждый по-своему.

## СЕНСАЦИЯ ВТОРАЯ: «READED» ОХРАНЯЕТ ШИФРОВАННУЮ ПЕРЕПИСКУ

За словом «readed» стоит не комментарий и не черновик, а самое настоящее поле структуры. Файл [`crates/ton-liteapi/src/adnl/primitives/codec.rs`](https://github.com/ton-blockchain/acton/blob/857c24b40f40d7a70782537603ee64017fd0f806/crates/ton-liteapi/src/adnl/primitives/codec.rs), строки [12–17](https://github.com/ton-blockchain/acton/blob/857c24b40f40d7a70782537603ee64017fd0f806/crates/ton-liteapi/src/adnl/primitives/codec.rs#L12-L17):

```rust
/// Implementation of ADNL protocol. Connection must be first initialized with [`AdnlHandshake`] to exchange keys.
pub struct AdnlCodec {
    aes_rx: AdnlAes,
    aes_tx: AdnlAes,
    last_readed_length: Option<usize>,
}
```

Поймите масштаб происшествия. ADNL — это шифрованная линия, по которой контора перешёптывается с лайтсерверами; `AdnlCodec` — будочник у телеграфной линии, который отмеряет длину каждого шифрованного пакета. И длина эта хранится в поле под именем `last_readed_length` — строка [16](https://github.com/ton-blockchain/acton/blob/857c24b40f40d7a70782537603ee64017fd0f806/crates/ton-liteapi/src/adnl/primitives/codec.rs#L16). «Последняя прочтённая длина», как говорят в тех кварталах, где грамматика давно не платит за электричество. Поле не прячется: оно инициализируется в строках [24](https://github.com/ton-blockchain/acton/blob/857c24b40f40d7a70782537603ee64017fd0f806/crates/ton-liteapi/src/adnl/primitives/codec.rs#L24) и [32](https://github.com/ton-blockchain/acton/blob/857c24b40f40d7a70782537603ee64017fd0f806/crates/ton-liteapi/src/adnl/primitives/codec.rs#L32), читается в строке [43](https://github.com/ton-blockchain/acton/blob/857c24b40f40d7a70782537603ee64017fd0f806/crates/ton-liteapi/src/adnl/primitives/codec.rs#L43), записывается в строке [60](https://github.com/ton-blockchain/acton/blob/857c24b40f40d7a70782537603ee64017fd0f806/crates/ton-liteapi/src/adnl/primitives/codec.rs#L60) и стирается в строке [72](https://github.com/ton-blockchain/acton/blob/857c24b40f40d7a70782537603ee64017fd0f806/crates/ton-liteapi/src/adnl/primitives/codec.rs#L72). Шесть упоминаний, шесть улик — и каждая под защитой амнистии.

Заметьте изящество конструкции: вместо того чтобы один раз переименовать поле — дело шести правок в одном файле, — в конторе предпочли изменить закон. Опечатка не исправлена. Опечатка **легализована**. Цензор теперь проходит мимо «readed» с видом человека, которому показали ордер.

## СЕНСАЦИЯ ТРЕТЬЯ: «LAZER» ЧИСЛИТСЯ В ОФИЦИАЛЬНОЙ КАРТОТЕКЕ

Третья фигура под амнистией — «lazer». И тут сыщику пришлось снять шляпу, потому что след ведёт в картотеку get-методов, которую контора показывает клиентам. Файл [`crates/ton-indexer-contracts/src/known_get_methods.rs`](https://github.com/ton-blockchain/acton/blob/857c24b40f40d7a70782537603ee64017fd0f806/crates/ton-indexer-contracts/src/known_get_methods.rs), строка [237](https://github.com/ton-blockchain/acton/blob/857c24b40f40d7a70782537603ee64017fd0f806/crates/ton-indexer-contracts/src/known_get_methods.rs#L237):

```rust
    (102_248, "get_lazer_data"),
```

Картотека, заметим, генерируется из [ABI-каталога известных контрактов](https://github.com/ton-blockchain/acton/blob/857c24b40f40d7a70782537603ee64017fd0f806/crates/ton-indexer-contracts/src/known_get_methods.rs#L3) и снабжена бинарным поиском — то есть это не клочок бумаги, а именной указатель, по которому acton называет методы при работе с чужими контрактами. Сыщик поднял первоисточник: `get_lazer_data` прибыл из контрактов Storm Trade, а «Lazer» там — не опечатка вовсе, а чужая визитная карточка: так зовут оракул Pyth Lazer, и его имя собственное, причудливое и неисправимое. Слово вынужденно неприкосновенно — как фамилия на двери чужой конторы. Амнистия тут честна, и газета это фиксирует без улыбки.

По той же причине неприкосновенны «authentificate» и «authentification»: они унаследованы из официальной TL-схемы самого протокола, файл [`crates/ton-liteapi/src/tl/schemas/ton_api.tl`](https://github.com/ton-blockchain/acton/blob/857c24b40f40d7a70782537603ee64017fd0f806/crates/ton-liteapi/src/tl/schemas/ton_api.tl), строки [25–27](https://github.com/ton-blockchain/acton/blob/857c24b40f40d7a70782537603ee64017fd0f806/crates/ton-liteapi/src/tl/schemas/ton_api.tl#L25-L27):

```
tcp.authentificate nonce:bytes = tcp.Message;
tcp.authentificationNonce nonce:bytes = tcp.Message;
tcp.authentificationComplete key:PublicKey signature:bytes = tcp.Message;
```

Тут контора не виновата вовсе: схема — святость, менять её нельзя, и слово «authentificate» придётся терпеть, пока туман стоит над Темзой. А «alse», между прочим, укрывается в тестовых пометках вида `f<caret>alse` — строка [405](https://github.com/ton-blockchain/acton/blob/857c24b40f40d7a70782537603ee64017fd0f806/crates/ton-language-server-core/tests/languages/toml/completion.rs#L405), где каретка автодополнения разрезает «false» пополам. То есть из пяти амнистированных слов четыре — чужие или технические, и лишь одно, «readed», — своё, домашнее, выращенное в собственной оранжерее. И именно его контора исправить не захотела.

---

## ХРОНИКА МЕЛКИХ ПРОИСШЕСТВИЙ

*Отдел поддельных дел.* Пока сыщик был в доме, он заглянул в архив эмуляции — и обнаружил там мастерскую фальшивых транзакций. Файл [`src/ffi/emulation.rs`](https://github.com/ton-blockchain/acton/blob/857c24b40f40d7a70782537603ee64017fd0f806/src/ffi/emulation.rs), строки [1047–1056](https://github.com/ton-blockchain/acton/blob/857c24b40f40d7a70782537603ee64017fd0f806/src/ffi/emulation.rs#L1047-L1056):

```rust
fn build_pseudo_broadcast_tx(now: u32, in_msg: Cell, norm_hash: HashBytes) -> TupleItem {
    let tx = Transaction {
        account: Default::default(),
        lt: 0,
        // HACK: abused slot — carries the TEP-467 normalized hash for the Rust-side lookup.
        prev_trans_hash: norm_hash,
        prev_trans_lt: 0,
        now,
        out_msg_count: Default::default(),
        orig_status: AccountStatus::Uninit,
```

Контора изготавливает транзакцию, которой не было: логическое время — ноль (строка [1050](https://github.com/ton-blockchain/acton/blob/857c24b40f40d7a70782537603ee64017fd0f806/src/ffi/emulation.rs#L1050)), статус счёта — «не существовал» (строка [1056](https://github.com/ton-blockchain/acton/blob/857c24b40f40d7a70782537603ee64017fd0f806/src/ffi/emulation.rs#L1056)), а в графу «хэш предыдущего дела» — `prev_trans_hash`, строка [1052](https://github.com/ton-blockchain/acton/blob/857c24b40f40d7a70782537603ee64017fd0f806/src/ffi/emulation.rs#L1052) — вклеен совсем другой хэш, нормализованный по TEP-467, чтобы по нему потом искать настоящую транзакцию в сети. Сама канцелярия честно признаётся в строке [1051](https://github.com/ton-blockchain/acton/blob/857c24b40f40d7a70782537603ee64017fd0f806/src/ffi/emulation.rs#L1051): «HACK: abused slot» — «слот используется не по назначению». Подлог, доведённый до совершенства: подделка заверена комментарием о подделке и служит благому делу — находить вашу настоящую транзакцию среди тумана. Нуар бывает и милосерден.

---

## ВЕРДИКТ РЕДАКЦИИ

Скажем справедливости ради: большая часть списка неприкосновенных — вынужденная мера. Чужие схемы и чужие имена собственные не исправишь, не сломав протокол; каретка в тестах и адреса кошельков цензору тоже не по зубам. За «lazer» и «authentificate» конторе — оправдание. Но «readed» — другое дело. Это родное поле, в родном файле, с шестью упоминаниями, и оно охраняет длину пакетов на шифрованной линии с лайтсерверами. Исправить его стоило меньше усилий, чем внести в амнистию. Контора выбрала амнистию. Так и живём: цензор обходит дом каждую ночь, фонарь его горит, шаги размеренны — и у двери с табличкой «last_readed_length» он, по распоряжению свыше, прибавляет шаг. Туман над Темзой, по обычаю, всё принял — на этот раз с правильным написанием.
