+++
title = "№ 18 — Часовая палата: машина времени со сроком годности, туман по вызову и шахта, где заказано три, а выкопано ноль"
date = 2026-08-22T17:07:00+03:00
description = "Восемнадцатый выпуск «Вечернего Валидатора»: сыщик дежурит в часовой палате локального города acton, где время ускоряют одной депешей, но не далее февраля 2106 года, где «установить время» означает «установить отставание от Гринвича», где туман на город нагоняют вручную и без верхнего предела, а на шахте принимают заказ на четыре миллиарда блоков и честно рапортуют: выкопано ноль, пропущено три."
tags = ["acton"]
+++

# 📰 Вечерний Валидатор

**№ 18.** *Лондон. Туман нынче пришёл по расписанию — что для тумана подозрительно, а для нашего города удобно: значит, кто-то его заказал. Наш корреспондент вновь в конторе [ton-blockchain/acton](https://github.com/ton-blockchain/acton) — газета уже писала про телеграфную контору (выпуск № 15), раздаточную будку (выпуск № 16) и министерство конституции (выпуск № 17). Нынче сыщик поднялся этажом выше, туда, где под стеклом тикает городской хронометр: часовая палата локальной сети, учреждение с окошками `/acton_increaseTime`, `/acton_setTime`, `/acton_setNextBlockTimestamp`, а по соседству — распорядитель тумана `/acton_setNetworkConditions` и шахтёрская касса `/acton_mine`. Сыщик занял стул у самого маятника и просидел до тех пор, пока маятник не попросил слово.*

*Место происшествия: репозиторий [ton-blockchain/acton](https://github.com/ton-blockchain/acton), коммит [`5535f11`](https://github.com/ton-blockchain/acton/commit/5535f119f6b0f45babc1f7e840483284aba00d6c) (master от 22.08.2026). Файлы: [`crates/ton-localnet/src/virtual_clock.rs`](https://github.com/ton-blockchain/acton/blob/5535f119f6b0f45babc1f7e840483284aba00d6c/crates/ton-localnet/src/virtual_clock.rs), [`crates/ton-localnet/src/server/router.rs`](https://github.com/ton-blockchain/acton/blob/5535f119f6b0f45babc1f7e840483284aba00d6c/crates/ton-localnet/src/server/router.rs), [`crates/ton-localnet/src/server/handlers/admin.rs`](https://github.com/ton-blockchain/acton/blob/5535f119f6b0f45babc1f7e840483284aba00d6c/crates/ton-localnet/src/server/handlers/admin.rs), [`crates/ton-localnet/src/server/mod.rs`](https://github.com/ton-blockchain/acton/blob/5535f119f6b0f45babc1f7e840483284aba00d6c/crates/ton-localnet/src/server/mod.rs), [`crates/ton-localnet/src/localnet.rs`](https://github.com/ton-blockchain/acton/blob/5535f119f6b0f45babc1f7e840483284aba00d6c/crates/ton-localnet/src/localnet.rs). Все цитаты дословные.*

---

## СЕНСАЦИЯ ПЕРВАЯ: МАШИНА ВРЕМЕНИ СО СРОКОМ ГОДНОСТИ

Часовая палата принимает депеши без подписи и очереди — реестр маршрутов, строки [211–212](https://github.com/ton-blockchain/acton/blob/5535f119f6b0f45babc1f7e840483284aba00d6c/crates/ton-localnet/src/server/router.rs#L211-L212). Хочешь в будущее — стучись в окошко `/acton_increaseTime`, ведомость приёма, строки [233–241](https://github.com/ton-blockchain/acton/blob/5535f119f6b0f45babc1f7e840483284aba00d6c/crates/ton-localnet/src/server/handlers/admin.rs#L233-L241). А дальше депеша попадает к самому механизму. Вот он, сердце хронометра, строки [53–66](https://github.com/ton-blockchain/acton/blob/5535f119f6b0f45babc1f7e840483284aba00d6c/crates/ton-localnet/src/virtual_clock.rs#L53-L66):

```rust
    pub(crate) fn increase_time(&mut self, seconds: u64) -> anyhow::Result<NodeClockInfo> {
        anyhow::ensure!(seconds > 0, "seconds must be greater than 0");
        let current = u64::from(self.now_unix()?);
        let next = current
            .checked_add(seconds)
            .context("localnet time overflow")?;
        anyhow::ensure!(
            next <= u64::from(u32::MAX),
            "localnet time cannot exceed {}",
            u32::MAX
        );
```

Читатель, разберём по винтикам. Первое: строка [54](https://github.com/ton-blockchain/acton/blob/5535f119f6b0f45babc1f7e840483284aba00d6c/crates/ton-localnet/src/virtual_clock.rs#L54) — «seconds must be greater than 0». Путешествие на ноль секунд запрещено законом: остаться в настоящем — не услуга. Часовая палата как извозчик, который согласен везти куда угодно, кроме «стой, где стоишь». Второе — и вот тут сыщик снял очки и протёр их дважды: строки [59–62](https://github.com/ton-blockchain/acton/blob/5535f119f6b0f45babc1f7e840483284aba00d6c/crates/ton-localnet/src/virtual_clock.rs#L59-L62). Будущее города кончается на числе `u32::MAX` — 4 294 967 295 секунд от начала эпохи, то есть **седьмого февраля 2106 года, в шесть часов двадцать восемь минут утра по Гринвичу**. Машина времени со сроком годности: до 2106-го — пожалуйста, а дальше табличка «localnet time cannot exceed 4294967295». Сыщик ценит честность: конец света здесь не скрывают, его печатают в квитанции об отказе — впрочем, как есть, числом, без даты: канцелярия пересчитывать вечность в календарь не обучена.

Теперь главное открытие ночи. Окошко `/acton_setTime` обещает в графе услуг — официальная брошюра, строка [756](https://github.com/ton-blockchain/acton/blob/5535f119f6b0f45babc1f7e840483284aba00d6c/docs/public/openapi/acton-localnet-control.openapi.json#L756): «Sets the localnet virtual clock to a Unix timestamp» — установить часы на метку времени. А механизм внутри делает иное, строки [72–79](https://github.com/ton-blockchain/acton/blob/5535f119f6b0f45babc1f7e840483284aba00d6c/crates/ton-localnet/src/virtual_clock.rs#L72-L79):

```rust
    pub(crate) fn set_time(
        &mut self,
        timestamp: u32,
        latest_block_timestamp: u32,
    ) -> anyhow::Result<NodeClockInfo> {
        ensure_timestamp_not_before_latest_block(timestamp, latest_block_timestamp)?;
        self.offset_seconds = i64::from(timestamp) - system_unix_now_i64()?;
        self.clock_info()
    }
```

Строка [77](https://github.com/ton-blockchain/acton/blob/5535f119f6b0f45babc1f7e840483284aba00d6c/crates/ton-localnet/src/virtual_clock.rs#L77) — вот весь фокус: палата **не останавливает часы на указанной дате**. Она записывает разницу с настенным хронометром Гринвича и идёт дальше. Сказал «полночь 1912-го»... стоп, прошлое запрещено стражем у дверей, строки [115–121](https://github.com/ton-blockchain/acton/blob/5535f119f6b0f45babc1f7e840483284aba00d6c/crates/ton-localnet/src/virtual_clock.rs#L115-L121), — сказал «полночь 2030-го»: часы покажут полночь 2030-го, но через час будет час ночи 2030-го, а через год — 2031-й. «Установить время» в этой палате означает «установить отставание». Будущее, однажды заказанное, продолжает бежать — его нельзя ни пригвоздить к месту, ни повернуть вспять. Идеальная машина времени для нуарного города: назад хода нет, а вперёд — с перезаправкой не далее 2106-го.

И венчает механизм сторож, нанятый сторожить невозможное: строка [63](https://github.com/ton-blockchain/acton/blob/5535f119f6b0f45babc1f7e840483284aba00d6c/crates/ton-localnet/src/virtual_clock.rs#L63), `i64::try_from(seconds).context("localnet time delta is too large")`. Дельта «слишком велика»? Но любая дельта, прошедшая проверку двумя строками выше, не превосходит четырёх с лишним миллиардов, а `i64` вмещает девять квинтиллионов. Страж бодрствует у двери, в которую физически не пролезает ни один посетитель. Сыщик записал в блокнот: «присяга верна, случай наступит никогда» — вторая такая присяга в архиве конторы после телеграфной из выпуска № 15.

## СЕНСАЦИЯ ВТОРАЯ: РАСПОРЯДИТЕЛЬ ТУМАНА

По соседству с часовой палатой — кабинет, о котором в Лондоне мечтают века: распорядитель погоды. Окошко `/acton_setNetworkConditions`, строка [208](https://github.com/ton-blockchain/acton/blob/5535f119f6b0f45babc1f7e840483284aba00d6c/crates/ton-localnet/src/server/router.rs#L208), бланк прошения предельно лаконичен, строки [66–68](https://github.com/ton-blockchain/acton/blob/5535f119f6b0f45babc1f7e840483284aba00d6c/crates/ton-localnet/src/server/models.rs#L66-L68):

```rust
pub struct SetNetworkConditionsRequest {
    pub response_delay_ms: u64,
}
```

Одна графа: «задержка ответа, миллисекунд». Без верхней границы, без справки о вменяемости — `u64`, читатель, это девятнадцать цифр. Приёмщик записывает, не моргнув, строки [124–134](https://github.com/ton-blockchain/acton/blob/5535f119f6b0f45babc1f7e840483284aba00d6c/crates/ton-localnet/src/server/handlers/admin.rs#L124-L134): `network_conditions.set_response_delay_ms(payload.response_delay_ms)` — и сразу печатает квитанцию. А в кладовой значение просто кладут на полку, строки [83–86](https://github.com/ton-blockchain/acton/blob/5535f119f6b0f45babc1f7e840483284aba00d6c/crates/ton-localnet/src/server/mod.rs#L83-L86):

```rust
    pub fn set_response_delay_ms(&self, response_delay_ms: u64) {
        self.response_delay_ms
            .store(response_delay_ms, Ordering::Relaxed);
    }
```

Сыщик прикинул на полях газеты: закажи проситель `u64::MAX` миллисекунд — и город обязуется выдержать паузу в **пятьсот восемьдесят четыре с половиной миллиона лет**. Сопоставьте, читатель: часовая палата, стоящая через стенку, будущее дальше 2106 года не выдаёт — «cannot exceed», точка. А распорядитель тумана согласен морозить ответ до тепловой смерти вселенной. В одном здании две конторы: одна ставит времени предел в восемьдесят лет, другая не ставит предела вообще. Горизонт событий у города — 2106-й, а терпение — вечное.

Но самое изящное — механика нагона тумана, строки [259–266](https://github.com/ton-blockchain/acton/blob/5535f119f6b0f45babc1f7e840483284aba00d6c/crates/ton-localnet/src/server/router.rs#L259-L266):

```rust
async fn delay_response(request: Request, next: Next, conditions: NetworkConditions) -> Response {
    let response = next.run(request).await;
    let delay_ms = conditions.response_delay_ms();
    if delay_ms > 0 {
        sleep(Duration::from_millis(delay_ms)).await;
    }
    response
}
```

Вчитайтесь в порядок строк: сперва строка [260](https://github.com/ton-blockchain/acton/blob/5535f119f6b0f45babc1f7e840483284aba00d6c/crates/ton-localnet/src/server/router.rs#L260) — `next.run(request).await`, — ответ **готов**, блюдо подано на серебряном блюде, квитанция отпечатана, и лишь потом, строки [262–264](https://github.com/ton-blockchain/acton/blob/5535f119f6b0f45babc1f7e840483284aba00d6c/crates/ton-localnet/src/server/router.rs#L262-L264), служка садится на неё и ждёт по часам. Это не медленная кухня — это медленный официант, гордость заведения: всё сварено вовремя, но подано с драматической паузой. Сыщик отдал должное реализму: туман в Лондоне точно так же не мешает делам твориться — он мешает о делах узнавать.

И последний штрих портрета: туман нагоняют избирательно. Механизм приторочен к народным окошкам toncenter-совместимого проезда — v2, v3, emulate, строки [164–176](https://github.com/ton-blockchain/acton/blob/5535f119f6b0f45babc1f7e840483284aba00d6c/crates/ton-localnet/src/server/router.rs#L164-L176), — а служебные двери `/acton_*`, включая саму часовую палату, стоят без задержки. Чиновник курьеру: «городу туман, нам — просвет». Сыщик прошёлся вдоль фасада: действительно, у муниципальных окон стёкла чистые.

## СЕНСАЦИЯ ТРЕТЬЯ: ШАХТА «ЗАКАЗАНО ТРИ — ВЫКОПАНО НОЛЬ»

В подвальном этаже — шахтёрская касса `/acton_mine`. Порядок приёма заказов, строки [136–150](https://github.com/ton-blockchain/acton/blob/5535f119f6b0f45babc1f7e840483284aba00d6c/crates/ton-localnet/src/server/handlers/admin.rs#L136-L150):

```rust
pub async fn mine_blocks(State(node): State<Arc<Localnet>>, body: Bytes) -> Response {
    handle_result(
        async move {
            let payload = if body.is_empty() {
                MineBlocksRequest::default()
            } else {
                serde_json::from_slice::<MineBlocksRequest>(&body)
                    .map_err(|e| anyhow::anyhow!("Invalid mine request JSON: {e}"))?
            };
            node.mine_blocks(payload.blocks.unwrap_or(1)).await
        },
```

Строки [139–140](https://github.com/ton-blockchain/acton/blob/5535f119f6b0f45babc1f7e840483284aba00d6c/crates/ton-localnet/src/server/handlers/admin.rs#L139-L140) и [145](https://github.com/ton-blockchain/acton/blob/5535f119f6b0f45babc1f7e840483284aba00d6c/crates/ton-localnet/src/server/handlers/admin.rs#L145): пришёл без бланка — заказано по умолчанию; бланк пуст — копаем один. Брошюра подтверждает, строка [704](https://github.com/ton-blockchain/acton/blob/5535f119f6b0f45babc1f7e840483284aba00d6c/docs/public/openapi/acton-localnet-control.openapi.json#L704): «When the request body is omitted or blocks is not provided, one block is mined». Шахта не спрашивает, нужен ли тебе блок: стоишь у кассы — значит, заказал. Теперь спускаемся в штольню, строки [2106–2135](https://github.com/ton-blockchain/acton/blob/5535f119f6b0f45babc1f7e840483284aba00d6c/crates/ton-localnet/src/localnet.rs#L2106-L2135):

```rust
fn handle_mine_blocks(
    node: &mut Node,
    count: u32,
    mining_mode: LocalnetMiningMode,
) -> anyhow::Result<LocalnetMineResult> {
    anyhow::ensure!(count > 0, "blocks must be greater than 0");

    let mut blocks = Vec::with_capacity(count as usize);
    let mut skipped_empty_blocks = 0;
    for _ in 0..count {
        match mine_block_with_mode(node, mining_mode)? {
            Some(block) => blocks.push(block.block_id()),
            None => skipped_empty_blocks += 1,
        }
    }
```

Единственный страж у клети, строка [2111](https://github.com/ton-blockchain/acton/blob/5535f119f6b0f45babc1f7e840483284aba00d6c/crates/ton-localnet/src/localnet.rs#L2111): «count must be greater than 0». Нуля нельзя — это понятно, ноль блоков уже выкопала часовая палата. А миллиард? А четыре миллиарда двести девяносто четыре миллиона девятьсот шестьдесят семь тысяч двести девяносто пять — полный `u32`? Можно. И вот тут строка [2113](https://github.com/ton-blockchain/acton/blob/5535f119f6b0f45babc1f7e840483284aba00d6c/crates/ton-localnet/src/localnet.rs#L2113) делает шаг, который сыщик расценил как главную улику ночи: `Vec::with_capacity(count as usize)` — шахта **заранее** бронирует вагонетки под весь заказ. Не по мере добычи, не по мере проветривания штольни — сразу, в полном составе, четыре миллиарда мест. Вагонетки, разумеется, не влезают в депо: оперативная память города кончается раньше, чем кончается `u32`, и касса падает под собственный заказ, так и не начав копать. Хроника напоминает: за стенкой контора отказывалась выдать время дальше 2106 года — осторожность, достойная лучшего дома. А здесь та же контора готова выделить место под четыре миллиарда блоков, не спросив, чем платить будут.

А внизу, у выхода, ещё один сторож-фантазёр, строки [2125–2128](https://github.com/ton-blockchain/acton/blob/5535f119f6b0f45babc1f7e840483284aba00d6c/crates/ton-localnet/src/localnet.rs#L2125-L2128): `blocks.len().try_into().context("mined blocks count exceeds u32")` — «число выкопанных блоков превысило u32». Сыщик перепроверил арифметику трижды: заказ — `u32`, цикл — от нуля до заказа, вагонеток — не больше итераций. Превысить `u32` можно, лишь выкопав больше, чем заказали. Третья несбывшаяся присяга за один вечер: у этой конторы явный природный дар нанимать стражу против явлений, исключённых законами арифметики.

Финал шахтёрской истории — в испытательном листе, и газета перепечатывает его без сокращений, строки [3769–3781](https://github.com/ton-blockchain/acton/blob/5535f119f6b0f45babc1f7e840483284aba00d6c/crates/ton-localnet/src/localnet.rs#L3769-L3781):

```rust
    #[test]
    fn handle_mine_blocks_skips_empty_blocks_by_default() {
        let mut node = make_test_node();

        let result = handle_mine_blocks(&mut node, 3, LocalnetMiningMode::default())
            .expect("manual mining must succeed");

        assert_eq!(result.blocks_mined, 0);
        assert_eq!(result.skipped_empty_blocks, 3);
```

Заказано три блока. Выкопано **ноль**. Пропущено три — пустые, видите ли. И рапорт: «manual mining must succeed» — успех, строка [3774](https://github.com/ton-blockchain/acton/blob/5535f119f6b0f45babc1f7e840483284aba00d6c/crates/ton-localnet/src/localnet.rs#L3774). Клиент приходит к кассе с зелёной квитанцией, в которой написано: добыча состоялась, добыча равна нулю. Режим по умолчанию `skip_empty_blocks` честно назван в графе режима — но касса всё равно принимает заказ на три, зная заранее, что выдаст ничего. Сыщик любил шахту за прямоту: здесь не обещают золота. Здесь обещают процесс.

---

## ХРОНИКА МЕЛКИХ ПРОИСШЕСТВИЙ

*Отдел скромных путешественников.* Испытательный лист часовой палаты, [`tests/integration/localnet_tests.rs`](https://github.com/ton-blockchain/acton/blob/5535f119f6b0f45babc1f7e840483284aba00d6c/tests/integration/localnet_tests.rs), отправляет город в будущее на... строка [910](https://github.com/ton-blockchain/acton/blob/5535f119f6b0f45babc1f7e840483284aba00d6c/tests/integration/localnet_tests.rs#L910): `let set_time_target = tx_block_gen_utime + 1800;` — на полчаса. Машина времени с горизонтом в восемьдесят лет, а инспектор гоняет её на дачу и обратно. Газета одобряет: с таким сроком годности бережное обращение — не трусость, а консерваторство.

*Отдел одноразовых билетов.* Окошко `/acton_setNextBlockTimestamp` продаёт билет в будущее ровно на один блок: брошюра, строка [782](https://github.com/ton-blockchain/acton/blob/5535f119f6b0f45babc1f7e840483284aba00d6c/docs/public/openapi/acton-localnet-control.openapi.json#L782), — «Sets a one-shot Unix timestamp for the next mined block. The timestamp is consumed...». Механизм подтверждает, строки [92–101](https://github.com/ton-blockchain/acton/blob/5535f119f6b0f45babc1f7e840483284aba00d6c/crates/ton-localnet/src/virtual_clock.rs#L92-L101): билет поглощается ближайшим блоком, `self.next_block_timestamp = None`, и часы снова идут по-гринвичски со сдвигом. Компостер времени: побывал в 2030-м на один квартал — и вышел.

*Отдел утренней побудки.* Что происходит, когда город просыпается после ночи? Процедура пробуждения, строки [126–132](https://github.com/ton-blockchain/acton/blob/5535f119f6b0f45babc1f7e840483284aba00d6c/crates/ton-localnet/src/virtual_clock.rs#L126-L132): если последний блок отстал от настенных часов, отставание обнуляется — `Ok(required.max(0))`, строка [131](https://github.com/ton-blockchain/acton/blob/5535f119f6b0f45babc1f7e840483284aba00d6c/crates/ton-localnet/src/virtual_clock.rs#L131). Город не может проспать: утром его время молча дёргают к настоящему. А вот если город перед сном убежал в будущее — будущее аккуратно сохранят. Палата бережёт чужие завтра и безжалостна к вчера. Профессиональная деформация учреждения, торгующего временем.

*Отдел древней истории.* Нижняя граница мира бдится в строках [133–139](https://github.com/ton-blockchain/acton/blob/5535f119f6b0f45babc1f7e840483284aba00d6c/crates/ton-localnet/src/virtual_clock.rs#L133-L139): `anyhow::ensure!(now >= 0, "localnet time cannot be before unix epoch")`. До 1970 года этот город не ездит даже в мыслях. Сыщик, человек викторианский, запротестовал было — 1895-й так удобен для хронометража, — но признал: контора последовательна. Вчера запрещено, позавчера — тем более.

---

## ВЕРДИКТ РЕДАКЦИИ

Скажем по совести: палата нужная и местами образцовая. Прошлое запрещено — блоки не поедут задом наперёд, переполнения перехватываются на каждом перекрёстке, одноразовый билет гасится честно, а в брошюре по-прежнему красуется предупреждение не выставлять служебные окошки на улицу. Разработчику, гоняющему вестинг-контракт через пять лет эпохи за одну секунду лаборатории, эта машина — дар небес, и вреда от неё нет никому дальше локальной сети. Но газета ведёт хронику почерка, и почерк сей ночи таков: будущее отпускается до февраля 2106 года и ни секундой дальше, тогда как пауза в ответе может длиться пятьсот восемьдесят четыре миллиона лет; «установить время» означает «установить отставание»; туман на город нагоняют готовеньким, после того как ответ уже сварен; а шахта, отказывающаяся копать ноль блоков, принимает заказ на четыре миллиарда — и рапортует об успешной добыче нуля. Туман над Темзой всё принял: он сам работает по расписанию `delay_response` — делает давно всё сделано, а до города доходит с паузой, которую никто не заказывал, но все оплачивают.
