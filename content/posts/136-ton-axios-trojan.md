---
title: "№ 136 — Конверт с трояном в почтовой карете"
date: 2026-11-08T16:59:00+03:00
tags: [ton]
---

# 📰 ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

**№ 136.** *Лондон. Туман цеплялся за фонари, когда сыщик получил депешу из [ton-org/ton](https://github.com/ton-org/ton). Газовый рожок кашлянул: в почтовой карете ехал не новый кошелек и не свежий опкод, а зависимость с именем `axios` — и у нее обнаружили весьма дурную компанию.*

## ДЕЛО О ПАКЕТЕ, КОТОРЫЙ НЕЛЬЗЯ БЫЛО ПОДПУСКАТЬ К ПОЧТЕ

След ведет в коммит [72811dcacf1cf17b9423c9bbd09cfd8a0eab2e2e](https://github.com/ton-org/ton/commit/72811dcacf1cf17b9423c9bbd09cfd8a0eab2e2e) конторы [ton-org/ton](https://github.com/ton-org/ton). Сыщик открыл папку [CHANGELOG.md](https://github.com/ton-org/ton/blob/72811dcacf1cf17b9423c9bbd09cfd8a0eab2e2e/CHANGELOG.md): в строке [11](https://github.com/ton-org/ton/blob/72811dcacf1cf17b9423c9bbd09cfd8a0eab2e2e/CHANGELOG.md#L11) сама контора объясняет, почему решила запереть одну версию библиотеки:

> `- Pin axios to version 1.14.0 to prevent installation of compromised version 1.14.1 containing remote access trojan (RAT). See https://github.com/axios/axios/issues/10604`
>
> — [строка 11](https://github.com/ton-org/ton/blob/72811dcacf1cf17b9423c9bbd09cfd8a0eab2e2e/CHANGELOG.md#L11), коммит [72811dc](https://github.com/ton-org/ton/commit/72811dcacf1cf17b9423c9bbd09cfd8a0eab2e2e)

Это не слух из туманного переулка. В той же папке лежит приговор упаковщику: [package.json](https://github.com/ton-org/ton/blob/72811dcacf1cf17b9423c9bbd09cfd8a0eab2e2e/package.json), строка [42](https://github.com/ton-org/ton/blob/72811dcacf1cf17b9423c9bbd09cfd8a0eab2e2e/package.json#L42):

```json
        "axios": "1.14.0",
```

Раньше там стоял диапазон `^1.6.7`: карета могла сама выбрать более свежий пакет. После обыска замок заменили на точный номер. Вежливый жест, если не считать того, что поводом был пакет, который, согласно записи конторы, содержал удаленный троян.

## УЛИКА ВТОРАЯ: КАРЕТА ПЕРЕВОЗИТ НЕ ТОЛЬКО ПОЧТУ

Сыщик не стал довольствоваться одной запиской. В [yarn.lock](https://github.com/ton-org/ton/blob/72811dcacf1cf17b9423c9bbd09cfd8a0eab2e2e/yarn.lock) на строках [1991–2000](https://github.com/ton-org/ton/blob/72811dcacf1cf17b9423c9bbd09cfd8a0eab2e2e/yarn.lock#L1991-L2000) зафиксирован новый адрес посылки:

```yaml
"axios@npm:1.14.0":
  version: 1.14.0
  resolution: "axios@npm:1.14.0"
  dependencies:
    follow-redirects: ^1.15.11
    form-data: ^4.0.5
    proxy-from-env: ^2.1.0
```

Так выглядит приличная защита после тревоги: не «берите что угодно из этой серии», а «берите ровно этот сверток». Но улика сочна именно своей бытовой нелепостью. Пакетный менеджер не злодей и не сыщик: он честно следует знаку `^`, пока в карету не сядет пассажир с фальшивым паспортом.

## ПРИГОВОР СЫЩИКА

Дело не в том, что [ton-org/ton](https://github.com/ton-org/ton) сам положил троян в почтовый ящик. Наоборот: коммит [72811dc](https://github.com/ton-org/ton/commit/72811dcacf1cf17b9423c9bbd09cfd8a0eab2e2e) показывает, как контора заметила опасную версию и прибила зависимость к `1.14.0`. Кринж сидит в другом месте: один невинный символ диапазона мог оставить дверь открытой, а закрыли ее уже после того, как на улице нашли чужие следы.

Газовый рожок погас. Сыщик записал в блокнот: в Лондоне проверяй не только код, но и курьера, который его приносит.

---

*Выпуск № 136. Все цитаты дословные; номера строк сверены с содержимым коммита.*
