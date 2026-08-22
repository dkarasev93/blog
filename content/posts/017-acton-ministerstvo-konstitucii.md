+++
title = "№ 17 — Министерство конституции: новый закон по одной депеше, архив, который не забывает отвергнутых, и инспектор, не трогающий вещи"
date = 2026-08-22T11:03:50+03:00
description = "Семнадцатый выпуск «Вечернего Валидатора»: сыщик дежурит в полночь у окошка /acton_setConfig, где всю конституцию локального города меняют одной депешей без подписи, блок для нового закона майнят без единого жителя, отвергнутые редакции конституции навсегда оседают в архиве, а инспектор обходит все комнаты словаря, не притрагиваясь ни к одной вещи."
tags = ["acton"]
+++

# 📰 Вечерний Валидатор

**№ 17.** *Лондон. Туман нынче дошёл до Вестминстера и встал у самых дверей парламента — ждёт, видимо, когда оттуда вынесут что-нибудь ещё. Наш корреспондент вновь в конторе [ton-blockchain/acton](https://github.com/ton-blockchain/acton) — газета уже писала про телеграфную контору (выпуск № 15) и раздаточную будку (выпуск № 16). А сегодня в ноль часов три минуты ночи — редакция сверила штемпель: [`8562a1f`](https://github.com/ton-blockchain/acton/commit/8562a1f69c46fa7c160d6b7c0e23ada1e4d20421), Sat Aug 22 00:03:59 — в конторе открыли новое учреждение. Вывеска лаконична: «feat(localnet): add /acton_setConfig endpoint to change localnet config». По-нашему — министерство по пересмотру конституции. Сыщик явился к открытию и просидел в приёмной до рассвета.*

*Место происшествия: репозиторий [ton-blockchain/acton](https://github.com/ton-blockchain/acton), коммит [`5535f11`](https://github.com/ton-blockchain/acton/commit/5535f119f6b0f45babc1f7e840483284aba00d6c) (master от 22.08.2026). Файлы: [`crates/ton-localnet/src/node.rs`](https://github.com/ton-blockchain/acton/blob/5535f119f6b0f45babc1f7e840483284aba00d6c/crates/ton-localnet/src/node.rs), [`crates/ton-localnet/src/server/handlers/admin.rs`](https://github.com/ton-blockchain/acton/blob/5535f119f6b0f45babc1f7e840483284aba00d6c/crates/ton-localnet/src/server/handlers/admin.rs), [`crates/ton-localnet/src/server/router.rs`](https://github.com/ton-blockchain/acton/blob/5535f119f6b0f45babc1f7e840483284aba00d6c/crates/ton-localnet/src/server/router.rs), [`crates/ton-localnet/src/storage.rs`](https://github.com/ton-blockchain/acton/blob/5535f119f6b0f45babc1f7e840483284aba00d6c/crates/ton-localnet/src/storage.rs). Все цитаты дословные.*

---

## СЕНСАЦИЯ ПЕРВАЯ: КОНСТИТУЦИЯ — ПО ОДНОЙ ДЕПЕШЕ

В настоящем городе TON смена конституции — дело долгое и скучное: конфиг-предложение, голосование валидаторов, раунды, кворум, подписи. Три четверти города должны сказать «да», прежде чем поменяется хоть один параметр. В локальном отделении acton процедуру упростили. Реестр маршрутов, строка [203](https://github.com/ton-blockchain/acton/blob/5535f119f6b0f45babc1f7e840483284aba00d6c/crates/ton-localnet/src/server/router.rs#L203):

```rust
        .route("/acton_setConfig", post(set_config))
```

Одна строка. Одно окошко. Никаких выборов: приходишь с base64-грамотой — уходишь с новой конституцией. Протокол приёма прошений, строки [303–311](https://github.com/ton-blockchain/acton/blob/5535f119f6b0f45babc1f7e840483284aba00d6c/crates/ton-localnet/src/server/handlers/admin.rs#L303-L311):

```rust
pub async fn set_config(
    State(node): State<Arc<Localnet>>,
    Json(payload): Json<SetConfigRequest>,
) -> Response {
    handle_result(node.set_config(payload.config), |res| {
        serde_json::to_value(res).unwrap_or(Value::Null)
    })
    .await
}
```

Проситель — кто угодно, грамота — что угодно, лишь бы словарь разбирался. Документация честно предупреждает, строка [306](https://github.com/ton-blockchain/acton/blob/5535f119f6b0f45babc1f7e840483284aba00d6c/docs/content/docs/localnet/control-api.mdx#L306): «These routes are intended for local development tooling and should not be exposed publicly» — мол, не выставляйте окошко на улицу. Предупреждение записано в самой брошюре учреждения, прямо под перечнем услуг — как надпись «не влезай» на трансформаторной будке, которую будка же и отпирает.

Но вот что заставило сыщика поднести газовый рожок ближе. Новая конституция вступает в силу в специальном блоке — и вот его протокол, строка [545](https://github.com/ton-blockchain/acton/blob/5535f119f6b0f45babc1f7e840483284aba00d6c/crates/ton-localnet/src/node.rs#L545):

```rust
        match self.commit_transaction_block(seqno, prev_lt, gen_utime, Vec::new(), Vec::new()) {
```

`Vec::new(), Vec::new()` — два пустых списка. Блок-майнится, блок-печатается, блок-кладётся в цепь, а жителей в нём нет: ни одной транзакции, ни одного сообщения. Парламент собрался в полном составе из нуля депутатов и единогласно принял новую конституцию. Сыщик перечитал протокол дважды: да, пустой зал, пустая трибуна, и на этом пустом месте — свежий параграф городского устройства. В настоящем городе такой блок сочли бы сфабрикованным. Здесь он называется «config-only block» и считается услугой.

## СЕНСАЦИЯ ВТОРАЯ: АРХИВ, КОТОРЫЙ НЕ ОТДАЁТ ОТВЕРГНУТЫХ

Теперь о канцелярской аккуратности. Процедура смены конституции, строки [536–552](https://github.com/ton-blockchain/acton/blob/5535f119f6b0f45babc1f7e840483284aba00d6c/crates/ton-localnet/src/node.rs#L536-L552), устроена с откатом — похвально. Смотрите, в каком порядке идут дела:

```rust
        let config_hash = Hash256::from(config_cell.repr_hash());
        self.cas.put(config_boc, config_hash);

        let previous_config_hash = self.globals.config_boc_hash;
        let previous_config_cell = std::mem::replace(&mut self.config_cell, config_cell);
        self.globals.config_boc_hash = config_hash;
        match self.commit_transaction_block(seqno, prev_lt, gen_utime, Vec::new(), Vec::new()) {
            Ok(block) => Ok((config_hash, block)),
            Err(error) => {
                self.globals.config_boc_hash = previous_config_hash;
                self.config_cell = previous_config_cell;
                Err(error.context("Failed to commit blockchain config"))
            }
        }
```

Грамоту сперва сдают в архив — строка [540](https://github.com/ton-blockchain/acton/blob/5535f119f6b0f45babc1f7e840483284aba00d6c/crates/ton-localnet/src/node.rs#L540), `self.cas.put(config_boc, config_hash)`, — и лишь потом пытаются провести блок. Если блок не прошёл, министерство честно откатывает конторские книги: старый хэш на место, старая ячейка на место, строки [548–549](https://github.com/ton-blockchain/acton/blob/5535f119f6b0f45babc1f7e840483284aba00d6c/crates/ton-localnet/src/node.rs#L548-L549). Всё вернули. Кроме одного: **из архива отвергнутую редакцию не вынули**. Сыщик заглянул в устав архива, [`storage.rs`](https://github.com/ton-blockchain/acton/blob/5535f119f6b0f45babc1f7e840483284aba00d6c/crates/ton-localnet/src/storage.rs), и обнаружил, что у `CellStore` есть `put`, есть `get` — а метода «вынуть» нет вовсе. Процедура сдачи, строки [48–61](https://github.com/ton-blockchain/acton/blob/5535f119f6b0f45babc1f7e840483284aba00d6c/crates/ton-localnet/src/storage.rs#L48-L61):

```rust
    pub fn put(&mut self, boc: BocBytes, hash: Hash256) -> Hash256 {
        self.cell_by_hash.remove(&hash);

        if let Some(conn) = &self.conn {
            let conn = conn.lock().expect("Failed to lock DB connection");
            let _ = conn.execute(
                "INSERT OR IGNORE INTO cas (hash, boc) VALUES (?1, ?2)",
                params![hash.to_bytes(), boc],
            );
        } else {
            self.boc_by_hash.insert(hash, boc);
        }
        hash
    }
```

Читатель, тут три жемчужины в одном ящике стола. Первая: `INSERT OR IGNORE`, строка [54](https://github.com/ton-blockchain/acton/blob/5535f119f6b0f45babc1f7e840483284aba00d6c/crates/ton-localnet/src/storage.rs#L54) — «вставь или сделай вид, что вставил». Вторая: `let _ =`, строка [53](https://github.com/ton-blockchain/acton/blob/5535f119f6b0f45babc1f7e840483284aba00d6c/crates/ton-localnet/src/storage.rs#L53) — если архив не принял грамоту, об этом молчат даже внутренне. Третья: `INSERT` есть, `IGNORE` есть, а `DELETE` не существует как понятия. Итог: всякая конституция, которую город рассмотрел и **отверг**, навсегда оседает в SQLite-архиве — бессрочный сор брошенных проектов реформ. Министерство откатывает все книги, кроме той, что пылится глубже всех. В нуаре это называется: компромат остаётся в деле.

## СЕНСАЦИЯ ТРЕТЬЯ: ИНСПЕКТОР, КОТОРЫЙ НИК ЧЕМУ НЕ ПРИКАСАЕТСЯ

Как министерство проверяет присланную конституцию? Очень тщательно и очень странно. Строки [525–534](https://github.com/ton-blockchain/acton/blob/5535f119f6b0f45babc1f7e840483284aba00d6c/crates/ton-localnet/src/node.rs#L525-L534):

```rust
        let config =
            Dict::<u32, Cell>::load_from_root_ext(&mut config_slice, Cell::empty_context())
                .context("Failed to parse blockchain config dictionary")?;
        anyhow::ensure!(
            config.root().is_some(),
            "Blockchain config dictionary is empty"
        );
        for entry in config.iter() {
            entry.context("Failed to parse blockchain config dictionary entry")?;
        }
```

Строки [532–534](https://github.com/ton-blockchain/acton/blob/5535f119f6b0f45babc1f7e840483284aba00d6c/crates/ton-localnet/src/node.rs#L532-L534) — вот они, венец инспекторского искусства: обойти **каждую** комнату словаря, постучать в каждую дверь, проверить, не рассыпается ли она при разборе — и не взять с собой ничего. Значения не читаются, параметры не сверяются, газ-цены не нюхаются: инспекция чисто тактильная, «лишь бы парсилось». Конституцию, где под номером 8 записана версия сети «abracadabra», а под номером 20 — цена газа из стихов, примут, если ячейки складываются ровно. Сыщик отметил в блокноте: единственное, что город требует от нового закона, — чтобы он был грамотно нарезан на клетки. Содержание — дело десятое, закон есть закон.

И венчает приёмную маленькая вежливость, которую газета не может обойти молчанием. В обработчике, строка [308](https://github.com/ton-blockchain/acton/blob/5535f119f6b0f45babc1f7e840483284aba00d6c/crates/ton-localnet/src/server/handlers/admin.rs#L308), результат упаковывается так: `serde_json::to_value(res).unwrap_or(Value::Null)`. Если сериализация ответа вдруг провалится — министерство не рапортует о провале. Оно выдаёт просителю чистый лист: `result: null`. А сверху, в канцелярии ответов, строки [91–95](https://github.com/ton-blockchain/acton/blob/5535f119f6b0f45babc1f7e840483284aba00d6c/crates/ton-localnet/src/server/handlers/utils.rs#L91-L95), уже отпечатано `ok: true`. То есть в самом странном из миров конституция сменилась, печать поставлена, а квитанция пуста — и всё равно «ok». Справка о реформе без реквизитов, зато с доброжелательным штампом.

---

## ХРОНИКА МЕЛКИХ ПРОИСШЕСТВИЙ

*Отдел магических печатей.* Испытательный лист министерства, [`tests/integration/localnet_tests.rs`](https://github.com/ton-blockchain/acton/blob/5535f119f6b0f45babc1f7e840483284aba00d6c/tests/integration/localnet_tests.rs), вписывает в пробную конституцию параметр номер девятьсот девяносто девять, строка [3551](https://github.com/ton-blockchain/acton/blob/5535f119f6b0f45babc1f7e840483284aba00d6c/tests/integration/localnet_tests.rs#L3551): `const TEST_PARAM: u32 = 999;` — на единичку не дотянули до тысячи, чтобы не мешаться с взрослыми номерами. А содержимое параметра, строка [3578](https://github.com/ton-blockchain/acton/blob/5535f119f6b0f45babc1f7e840483284aba00d6c/tests/integration/localnet_tests.rs#L3578): `0xfeed_cafe`. «Накорми кафе» — классическая шутка печатного двора, шестнадцатеричная частушка. Газета не против шуток в испытательных листах; газета против того, чтобы шутка была самой осмысленной строкой во всей процедуре приёма конституции.

*Отдел точных наук.* Там же, строка [3629](https://github.com/ton-blockchain/acton/blob/5535f119f6b0f45babc1f7e840483284aba00d6c/tests/integration/localnet_tests.rs#L3629): `"invalid_requests_did_not_mine": head_after_invalid_requests == 2`. Неверные прошения не смайнили блоков — это проверяется сравнением с константой **два**. Почему два? Потому что один блок смайнили раньше в этом же листе, а второй — сама смена конституции. Арифметика верна, но записана как у лавочника: «было два, стало два — значит, ничего не украли». Сыщик предпочитает арифметику с именами, но признаёт: в полночь, когда писался коммит, числа без имён считаются быстрее.

*Отдел ночных штампов.* Штемпель на коммите: Sat Aug 22 **00:03:59**. Конституционное учреждение распахнуло двери в третьей минуте первого часа ночи. Газета не делает из этого выводов о здоровье муниципальных служащих, газета лишь фиксирует: лучшие учреждения этого города открываются, когда туман гуще всего и свидетелей нет. Кстати, вхождение в министерство простое до изящества, строки [1075–1079](https://github.com/ton-blockchain/acton/blob/5535f119f6b0f45babc1f7e840483284aba00d6c/crates/ton-localnet/src/localnet.rs#L1075-L1079): грамоту декодируют из base64 — «Invalid config base64» — и отправляют дальше по трубе. Ни капли воска на конверте, ни одной подписи. Депеша как депеша.

---

## ВЕРДИКТ РЕДАКЦИИ

Скажем по совести: учреждение нужное. Локальный город без права сменить конфиг — это театр, где декорации прибиты гвоздями; разработчику, желающему погонять контракт на чужих газ-ценах, окошко `/acton_setConfig` — подарок. Проверка словаря пусть тактильная, но не нулевая: пустой словарь отвергнут, битую ячейку вернут с 422, а невалидные прошения блоков не майнят — испытательный лист это сторожит. Вреда нет никому за пределами локальной сети, и предупреждение «не выставляйте наружу» напечатано в брошюре. Но газета ведёт хронику почерка. И почерк этой полночи таков: конституцию меняют одной депешей без подписи, закон вступает в силу в блоке без единого жителя, архив бережно хранит все отвергнутые редакции — с пометкой `INSERT OR IGNORE`, чтобы никто не смутился, — а инспектор обходит комнаты нового закона, не прикасаясь к вещам. Туман над Темзой всё принял: он сам меняет устройство города каждую ночь — без депеш, без блоков и, разумеется, без записи в архиве.
