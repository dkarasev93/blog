---
title: "№ 139 — Контора, которая держит тестнет в красной строке"
date: 2026-11-10T11:08:00+03:00
tags: [tonutils-rs]
---

# ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

**№ 139.** *Лондон. Туман лежал на мостовой, как мокрый плащ. Сыщик получил папку из молодой конторы [lapismyt/tonutils-rs](https://github.com/lapismyt/tonutils-rs), где обещали строгие проверки TON. Газовый рожок мигнул: один из главных караулов проекта уже заранее признан красным и будет ждать спасения сверху.*

## ДЕЛО О ТЕСТЕ, КОТОРЫЙ САМ СЕБЯ ПРИГОВОРИЛ

След ведет в коммит [`10689ef`](https://github.com/lapismyt/tonutils-rs/commit/10689efb4e3643ecbc7c7195d4c796ad9d840f3b) конторы [lapismyt/tonutils-rs](https://github.com/lapismyt/tonutils-rs). В ее внутреннем справочнике [`docs/reference/README.md`](https://github.com/lapismyt/tonutils-rs/blob/10689efb4e3643ecbc7c7195d4c796ad9d840f3b/docs/reference/README.md) сыщик обнаружил показание, достойное отдельного фонаря. Строки [192–199](https://github.com/lapismyt/tonutils-rs/blob/10689efb4e3643ecbc7c7195d4c796ad9d840f3b/docs/reference/README.md#L192-L199) говорят дословно:

```text
Testnet DHT runs strictly from the scheduled/manual
`.github/workflows/live-testnet-dht.yml` workflow because the canonical testnet
seeds are currently unreachable; that workflow stays red until upstream
recovers.
```

Перевод с канцелярского на лондонский: тестнетная проверка DHT запускается только по расписанию или вручную, потому что эталонные семена сейчас недоступны. А сам workflow будет оставаться красным, пока upstream не выздоровеет. Не «может упасть», не «иногда требует внимания» — красная строка уже вписана в план службы как постоянный элемент интерьера.

## УЛИКА ВТОРАЯ: КРАСНЫЙ СИГНАЛ СТАЛ ЧАСТЬЮ ПРОЦЕДУРЫ

В том же [`docs/reference/README.md`](https://github.com/lapismyt/tonutils-rs/blob/10689efb4e3643ecbc7c7195d4c796ad9d840f3b/docs/reference/README.md) строки [194–199](https://github.com/lapismyt/tonutils-rs/blob/10689efb4e3643ecbc7c7195d4c796ad9d840f3b/docs/reference/README.md#L194-L199) сообщают еще одну прелесть: CI запускает каждый живой шаг с `if: ${{ !cancelled() }}` и сохраняет журналы, чтобы один провал не прятал улики от остальных. Контора не прячет красную лампу. Она аккуратно протирает вокруг нее стекло и вывешивает расписание следующего осмотра.

Сыщик сверил это с самим тестом в файле [`crates/tonutils/src/liteclient/tests/live_workflows.rs`](https://github.com/lapismyt/tonutils-rs/blob/10689efb4e3643ecbc7c7195d4c796ad9d840f3b/crates/tonutils/src/liteclient/tests/live_workflows.rs). На строках [54–59](https://github.com/lapismyt/tonutils-rs/blob/10689efb4e3643ecbc7c7195d4c796ad9d840f3b/crates/tonutils/src/liteclient/tests/live_workflows.rs#L54-L59) лежит второй протокол, дословно:

```rust
/// Testnet replacement for the PR-blocking DHT gate.
///
/// The canonical testnet DHT seeds are currently unreachable, but the testnet
/// liteservers answer normally. Every configured index is tried because the
/// first entries in the published configuration can be closed while later
/// ones serve the same network.
```

Вот она, викторианская архитектура: ворота, блокирующие pull request, пришлось заменить запасными воротами, потому что старые семена не отвечают. Лайтсерверы, по показаниям, еще держатся, но DHT-караул уже переведен из парадного входа в режим ручного обхода.

## ТРЕТЬЯ УЛИКА: ВЫЗОВ ПОМОЩИ С ПОРОГА

Сам тест не оставляет места для романтики. Файл [`crates/tonutils/src/liteclient/tests/live_workflows.rs`](https://github.com/lapismyt/tonutils-rs/blob/10689efb4e3643ecbc7c7195d4c796ad9d840f3b/crates/tonutils/src/liteclient/tests/live_workflows.rs), строки [60–83](https://github.com/lapismyt/tonutils-rs/blob/10689efb4e3643ecbc7c7195d4c796ad9d840f3b/crates/tonutils/src/liteclient/tests/live_workflows.rs#L60-L83), обходит все адреса, собирает отказы, а затем, если никто не ответил, произносит:

```rust
    panic!(
        "no testnet liteserver answered getMasterchainInfo: {}",
        failures.join(" | ")
    );
```

Никакого молчаливого пропуска, никакого «ну сеть сегодня такая». Если все лайтсерверы отвернулись, тест падает с полным списком показаний. Это хорошая строгость для проверки сети — и одновременно комичная афиша: проект заранее печатает в документации, что один из его живых караулов будет красным до тех пор, пока кто-то снаружи не починит семена.

## ПРИГОВОР СЫЩИКА

Дело не в том, что [lapismyt/tonutils-rs](https://github.com/lapismyt/tonutils-rs) обязан заставить чужие тестнетные семена отвечать. У него есть разумная страховка: перебор лайтсерверов и подробный список отказов. Кринж сидит в другом месте. В конторе одновременно требуют строгой проверки без обхода недоступной сети и признают, что главный DHT-караул будет красным до восстановления upstream.

Так живет современный CI: красная строка не авария, если она заранее занесена в инструкцию. Сыщик тушит газовый рожок и записывает: в тумане TON иногда ломается не код, а сама дверь, через которую код клянется быть проверенным.

---

*Выпуск № 139. Все цитаты дословные; номера строк сверены с содержимым коммита.*
