---
title: "Выпуск № 53: Два платежа в одном тумане"
date: 2026-09-09T17:00:39+03:00
tags: [wallet-contract]
---

# Дело о подписке, которая платила дважды

Лондон в тот вечер напоминал плохо запертую шкатулку: туман лез в замочные скважины, мостовая хлюпала под сапогами, а газовый рожок у двери редакции кашлял медью. В такой час сыщик «Вечернего Валидатора» получил конверт из [ton-blockchain/wallet-contract](https://github.com/ton-blockchain/wallet-contract). На печати значилось: подписка. Внутри пахло не воском, а двойной тратой.

## Место происшествия

Улика лежала в коммите [`da472a9`](https://github.com/ton-blockchain/wallet-contract/commit/da472a98cafb233f891b4aeb24165a63b10d27a1), файл [`func/simple-subscription-plugin.fc`](https://github.com/ton-blockchain/wallet-contract/blob/da472a98cafb233f891b4aeb24165a63b10d27a1/func/simple-subscription-plugin.fc), строки [120–129](https://github.com/ton-blockchain/wallet-contract/blob/da472a98cafb233f891b4aeb24165a63b10d27a1/func/simple-subscription-plugin.fc#L120-L129).

Протокол без ретуши:

```func
if (op == 0x706c7567) {
    (int from_wc, _) = s_addr.parse_std_addr();
    if( ~(flags & 1) & (msg_value > amount - chain::short_msg_fwd_fee(from_wc) ) ) {
      last_payment = now();
      forward_funds(beneficiary, false, 0x706c7567);
    }
    if (~ start_at) {
      start_at = now();
    }
    return save_storage(wallet, beneficiary, amount, period, start_at, timeout, last_payment, last_request, subscription_id);
}
```

## Как работал маленький черный ящик

Снаружи все выглядело прилично. Запрос на оплату приходил в кошелек, кошелек должен был ответить плагину, а плагин — отправить деньги получателю. Но в карете этой схемы не хватало одного колеса: пока ответ кошелька блуждал между башнями, новый запрос мог снова постучать в ту же дверь.

На строке [122](https://github.com/ton-blockchain/wallet-contract/blob/da472a98cafb233f891b4aeb24165a63b10d27a1/func/simple-subscription-plugin.fc#L122) проверяется сумма и флаг bounce. На строке [123](https://github.com/ton-blockchain/wallet-contract/blob/da472a98cafb233f891b4aeb24165a63b10d27a1/func/simple-subscription-plugin.fc#L123) часы подписки переводятся вперед, но это еще не замок от повторного визита: значение `last_payment` меняется только внутри ветки ответа, а не при выдаче запроса. Затем строки [124–129](https://github.com/ton-blockchain/wallet-contract/blob/da472a98cafb233f891b4aeb24165a63b10d27a1/func/simple-subscription-plugin.fc#L124-L129) отправляют средства и сохраняют состояние, которое может не успеть вернуться до следующего стука.

Иными словами, задержка ответа превращала скромную подписку в лондонского джентльмена с двумя кошельками: один платеж уже в пути, второй еще только надевает пальто. Коммит так и назван — «Fix double spend when timeout is not enough to get response». В конторе, видимо, сочли, что это достаточно ясный диагноз.

## Запоздалый замок

Лечение появилось в коммите [`b7c7904`](https://github.com/ton-blockchain/wallet-contract/commit/b7c790494c4e1bd4946e226eaca736613a3ba868), все в том же, но уже исправленном файле [`func/simple-subscription-plugin.fc`](https://github.com/ton-blockchain/wallet-contract/blob/b7c790494c4e1bd4946e226eaca736613a3ba868/func/simple-subscription-plugin.fc), строки [120–127](https://github.com/ton-blockchain/wallet-contract/blob/b7c790494c4e1bd4946e226eaca736613a3ba868/func/simple-subscription-plugin.fc#L120-L127): перед отправкой добавили расчет временных слотов и отказ с кодом 49.

```func
if (op == 0x706c7567) {
    int last_timeslot = (last_payment - start_at) / period;
    int cur_timeslot = (now() - start_at) / period;
    throw_if(49, last_timeslot >= cur_timeslot);
    (int from_wc, _) = s_addr.parse_std_addr();
    if( ~(flags & 1) & (msg_value > amount - chain::short_msg_fwd_fee(from_wc) ) ) {
      last_payment = now();
      forward_funds(beneficiary, false, 0x706c7567);
    }
```

Замок поставили, но только после того, как дверь успела прославиться на весь квартал. Газовый рожок хрипло подвел итог: в мире смарт-контрактов «ответ еще не пришел» — это не пауза, а приглашение к расследованию.

А сыщик «Вечернего Валидатора» записал в блокнот: если в названии коммита прямо сказано `double spend`, не надо спрашивать, где труп. Надо проверить, сколько раз его успели оплатить.
