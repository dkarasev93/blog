---
title: "№ 138 — Рефунд, который сам себе не верит"
date: 2026-11-09T16:59:00+03:00
tags: [verifier-registry]
---

# 📰 ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

**№ 138.** *Лондонский вечер тонул в тумане, когда сыщик получил папку из нового реестра. Газовый рожок мигнул: внутри обещали доставку с оплатой в USDT, а на полях уже лежало признание, что одна ошибка способна оставить деньги в кармане контракта навсегда.*

## ДЕЛО О РЕЕСТРЕ, КОТОРЫЙ КЛЯНЕТСЯ НЕ ПАДАТЬ

Улика пришла из репозитория [tolk-labs/verifier-registry](https://github.com/tolk-labs/verifier-registry), коммита [f7c630e](https://github.com/tolk-labs/verifier-registry/commit/f7c630e6048eb523dcc303f34c927f708e85173a). Это свежая запись бота-верификатора: он закрепил исходники контракта Marketplace за хешем кода.

Главный фолиант — файл [contracts/Marketplace.tolk](https://github.com/tolk-labs/verifier-registry/blob/f7c630e6048eb523dcc303f34c927f708e85173a/sources/ff/bf812264e70fc65fb336a38d0db024e3d11a753c1ec930b7a47b0893e57212/files/contracts/Marketplace.tolk). В строках [189–193](https://github.com/tolk-labs/verifier-registry/blob/f7c630e6048eb523dcc303f34c927f708e85173a/sources/ff/bf812264e70fc65fb336a38d0db024e3d11a753c1ec930b7a47b0893e57212/files/contracts/Marketplace.tolk#L189-L193) контора дает показания дословно:

```text
// Real USDT is now in our wallet. From here on NOTHING may throw:
// this contract has no admin, so USDT stuck in its wallet would be
// stuck forever. Every problem becomes a refund instead. The only
// step that can fail is reading the payload, so that — and nothing
// with side effects, TVM does not roll those back — goes in `try`.
```

Сыщик снял шляпу. Контракт без администратора торжественно обещает, что после получения настоящего USDT не позволит себе падать. Даже формулировка не прячется за дипломатией: `NOTHING may throw`, а застрявшие монеты будут застрявшими навсегда.

## УЛИКА ВТОРАЯ: ВСЕ ДЕНЬГИ КЛИЕНТА — НО НЕ ТОЛЬКО ЕГО

Дальше начинается арифметика, от которой газовый рожок кашляет. В строках [259–264](https://github.com/tolk-labs/verifier-registry/blob/f7c630e6048eb523dcc303f34c927f708e85173a/sources/ff/bf812264e70fc65fb336a38d0db024e3d11a753c1ec930b7a47b0893e57212/files/contracts/Marketplace.tolk#L259-L264) контора резервирует собственный запас так:

```text
reserveToncoinsOnBalance(
    contract.getOriginalBalance() - in.valueCoins,
    RESERVE_MODE_AT_MOST,
);
```

Затем, в строках [265–282](https://github.com/tolk-labs/verifier-registry/blob/f7c630e6048eb523dcc303f34c927f708e85173a/sources/ff/bf812264e70fc65fb336a38d0db024e3d11a753c1ec930b7a47b0893e57212/files/contracts/Marketplace.tolk#L265-L282), создается перевод USDT и отправляется с режимом `SEND_MODE_CARRY_ALL_BALANCE`:

```text
// SAFETY: the reserve right above keeps the Marketplace's own
// balance; what this sends is what is left of the client's coin, and
// all of it ends up on their order.
forward.send(SEND_MODE_CARRY_ALL_BALANCE);
```

Замысел понятен: отложить остаток конторы, а затем пустить в путь все остальное. Но в деле есть неприятная цифра. Резерв вычисляется как баланс до входящего сообщения минус сумма этого сообщения. Если Marketplace был пуст, а клиент принес монеты впервые, формула просит резервировать отрицательную величину. Режим `AT_MOST` смягчает нехватку, но не превращает отрицательную арифметику в ясный счет. После этого `CARRY_ALL_BALANCE` получает право забрать все, что осталось на балансе.

Именно тут показания расходятся с заголовком «только клиентская монета». Код не устанавливает положительный нижний предел для резервируемой величины и не проверяет, что после резерва на отправку осталась нужная сумма. В контракте, который называет себя бессмертным перед ошибками, денежная безопасность держится на комментарии `SAFETY`.

## ТРЕТЬЯ УЛИКА: КОНТОРА ЗНАЕТ, ЧТО ЕЙ НЕЛЬЗЯ ПАДАТЬ

На строках [37–45](https://github.com/tolk-labs/verifier-registry/blob/f7c630e6048eb523dcc303f34c927f708e85173a/sources/ff/bf812264e70fc65fb336a38d0db024e3d11a753c1ec930b7a47b0893e57212/files/contracts/Marketplace.tolk#L37-L45) стоят три тарифа: минимальный вход `0.05 TON`, уведомление заказа `0.01 TON` и возврат `0.05 TON`. А на строках [131–155](https://github.com/tolk-labs/verifier-registry/blob/f7c630e6048eb523dcc303f34c927f708e85173a/sources/ff/bf812264e70fc65fb336a38d0db024e3d11a753c1ec930b7a47b0893e57212/files/contracts/Marketplace.tolk#L131-L155) возврат при нехватке входящего газа должен взять эти `0.05 TON` из собственного баланса.

То есть одна и та же контора одновременно говорит: «ничего не должно бросить исключение», «если денег клиента мало, заплати сам», а ниже оставляет перевод с `SEND_MODE_PAY_FEES_SEPARATELY`. Если у пустого Marketplace нет собственного газа, этот спасательный круг сделан из воздуха. Сыщик не утверждает, что каждая транзакция сломается: он фиксирует более точное преступление — комментарий обещает безусловный возврат, а код не показывает безусловного источника средств для него.

## ПРИГОВОР СЫЩИКА

Репозиторий [tolk-labs/verifier-registry](https://github.com/tolk-labs/verifier-registry) в коммите [f7c630e](https://github.com/tolk-labs/verifier-registry/commit/f7c630e6048eb523dcc303f34c927f708e85173a) показал миру Marketplace, который сам написал себе клятву: после получения USDT ему нельзя падать. В файле [contracts/Marketplace.tolk](https://github.com/tolk-labs/verifier-registry/blob/f7c630e6048eb523dcc303f34c927f708e85173a/sources/ff/bf812264e70fc65fb336a38d0db024e3d11a753c1ec930b7a47b0893e57212/files/contracts/Marketplace.tolk), строки [189–193](https://github.com/tolk-labs/verifier-registry/blob/f7c630e6048eb523dcc303f34c927f708e85173a/sources/ff/bf812264e70fc65fb336a38d0db024e3d11a753c1ec930b7a47b0893e57212/files/contracts/Marketplace.tolk#L189-L193), эта клятва стоит рядом с резервом по формуле [259–264](https://github.com/tolk-labs/verifier-registry/blob/f7c630e6048eb523dcc303f34c927f708e85173a/sources/ff/bf812264e70fc65fb336a38d0db024e3d11a753c1ec930b7a47b0893e57212/files/contracts/Marketplace.tolk#L259-L264) и опустошением баланса на [строке 282](https://github.com/tolk-labs/verifier-registry/blob/f7c630e6048eb523dcc303f34c927f708e85173a/sources/ff/bf812264e70fc65fb336a38d0db024e3d11a753c1ec930b7a47b0893e57212/files/contracts/Marketplace.tolk#L282).

Газовый рожок погас. Сыщик записал: самый опасный персонаж в лондонском контракте — не исключение, а комментарий, который заранее назвал исключения невозможными.

---

*Выпуск № 138. Цитаты кода дословные; номера строк сверены с коммитом.*