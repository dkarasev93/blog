+++
title = "№ 133 — Кошелек, который вынес кошелек из дома"
date = 2026-11-07T10:58:00+03:00
description = "В репозитории ton-wallet приложение уехало в другой дом, а на месте остались релизные ворота и архив старой витрины."
tags = ["ton-wallet"]
+++

# ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

**№ 133.** *Лондон. Туман полз по мостовой, когда сыщик получил адрес с вызывающей вывеской [ton-blockchain/ton-wallet](https://github.com/ton-blockchain/ton-wallet). Он ожидал найти кошелек. Но дверь открылась в контору, где кошелек уже уехал, приложение живет у соседей, а старые страницы оставлены как запасной выход. Газовый рожок кашлянул: «Поищите средства в другом доме».*

## МЕСТО ПРОИСШЕСТВИЯ

Улика лежит в коммите [`ab40e45`](https://github.com/ton-blockchain/ton-wallet/commit/ab40e454406bace589d9bb086364f97ca9143637), в файле [`AGENTS.md`](https://github.com/ton-blockchain/ton-wallet/blob/ab40e454406bace589d9bb086364f97ca9143637/AGENTS.md). В строках [3–4](https://github.com/ton-blockchain/ton-wallet/blob/ab40e454406bace589d9bb086364f97ca9143637/AGENTS.md#L3-L4) контора поясняет свое семейное положение дословно:

```text
This repository owns Gram Wallet release workflows, artifact policy and publishing. Application changes belong in `mytonwallet-org/mytonwallet-dev` and arrive through its public release mirror.
```

Дело уже пахнет не просто переездом. Дом с названием `ton-wallet` владеет выпуском кошелька, но не самим кошельком. Сыщик проверил соседнюю табличку в том же файле: строки [15–16](https://github.com/ton-blockchain/ton-wallet/blob/ab40e454406bace589d9bb086364f97ca9143637/AGENTS.md#L15-L16) велят хранить `docs/` как старый снимок для отката и запускать тесты из другой папки.

## ПАСПОРТ ПРОПАВШЕГО

В файле [`README.md`](https://github.com/ton-blockchain/ton-wallet/blob/ab40e454406bace589d9bb086364f97ca9143637/README.md) строки [1–3](https://github.com/ton-blockchain/ton-wallet/blob/ab40e454406bace589d9bb086364f97ca9143637/README.md#L1-L3) оформляют алиби:

```text
# Gram Wallet releases

This repository builds and publishes [Gram Wallet Web](https://wallet.ton.org) and the [Gram Wallet Chrome extension](https://chromewebstore.google.com/detail/nphplpgoakhhjchkkhmiggakijnkhfnd). Application source lives in [mytonwallet-org/mytonwallet](https://github.com/mytonwallet-org/mytonwallet). Every build uses an exact public source commit and records it separately from the pipeline commit.
```

Название обещает кошелек, первая строка уточняет: это уже не кошелек, а *Gram Wallet releases*. Исходники живут в другом доме, а этот хранит конвейер, разрешения и печати. В викторианском Лондоне это называлось бы не жилищем, а конторой по пересылке чужих ключей.

Особенно хороша строка [17](https://github.com/ton-blockchain/ton-wallet/blob/ab40e454406bace589d9bb086364f97ca9143637/README.md#L17). Там сказано, что старый код сохранен в коммите [`ae5161e`](https://github.com/ton-blockchain/ton-wallet/commit/ae5161e75f232eb9364231bce9bdd87a2f3f2c4e), а папка `docs/` — лишь снимок старого сайта для отката и «not a second application source tree». Словно дворецкий заранее оставил записку: не ищите тут жильцов, это только декорация.

## ОТДЕЛ РЕЛИЗНЫХ ВОРОТ

В том же коммите [`ab40e45`](https://github.com/ton-blockchain/ton-wallet/commit/ab40e454406bace589d9bb086364f97ca9143637) файл [`.github/workflows/gram-chrome-publish.yml`](https://github.com/ton-blockchain/ton-wallet/blob/ab40e454406bace589d9bb086364f97ca9143637/.github/workflows/gram-chrome-publish.yml) на строках [38–42](https://github.com/ton-blockchain/ton-wallet/blob/ab40e454406bace589d9bb086364f97ca9143637/.github/workflows/gram-chrome-publish.yml#L38-L42) называет свою единственную миссию без маскировки:

```yaml
jobs:
  publish:
    name: Publish the original reviewed Chrome package
    runs-on: ubuntu-latest
```

Тут не осталось даже попытки притвориться приложением. Есть публикация проверенного Chrome-пакета, есть строгий конвейер, есть секреты издателя. Сам кошелек в материалах дела проходит как «application source» с адресом на стороне. Это не баг, который роняет баланс, а комедия имени: вывеска обещает сейф, внутри сидит нотариус релизов.

## ВЕРДИКТ СЫЩИКА

[ton-blockchain/ton-wallet](https://github.com/ton-blockchain/ton-wallet) пойман не на краже средств, а на исчезновении собственной личности. В коммите [`ab40e45`](https://github.com/ton-blockchain/ton-wallet/commit/ab40e454406bace589d9bb086364f97ca9143637) файл [`AGENTS.md`](https://github.com/ton-blockchain/ton-wallet/blob/ab40e454406bace589d9bb086364f97ca9143637/AGENTS.md), строки [3–4](https://github.com/ton-blockchain/ton-wallet/blob/ab40e454406bace589d9bb086364f97ca9143637/AGENTS.md#L3-L4), честно признает: приложение живет в другом репозитории. Файл [`README.md`](https://github.com/ton-blockchain/ton-wallet/blob/ab40e454406bace589d9bb086364f97ca9143637/README.md), строки [1–3](https://github.com/ton-blockchain/ton-wallet/blob/ab40e454406bace589d9bb086364f97ca9143637/README.md#L1-L3), оставляет на месте лишь сборку и публикацию Gram Wallet.

Сыщик записал адрес в блокнот: это кошелек без кошелька, дом без жильца, архив с газовым светом и Chrome-пакетом на пороге. Потом туман сомкнулся, а газовый рожок прошептал: «Средства не найдены. Ищите в соседнем репозитории».

🐀
