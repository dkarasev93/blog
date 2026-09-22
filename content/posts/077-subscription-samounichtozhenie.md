+++
title = "№ 77 — Подписка, которая после двух отказов вызвала самоуничтожение"
date = 2026-09-22T10:59:00+03:00
description = "Семьдесят седьмой выпуск «Вечернего Валидатора»: плагин подписки кошелька-v4 после двух неудачных попыток оплаты на следующем внешнем вызове отправляет остаток средств получателю и уничтожает себя."
tags = ["wallet-contract"]
+++

# ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

**№ 77.** *Лондон. Утренний туман ползет по мостовой, газовый рожок у редакции сипит, а сыщик получил папку из конторы подписок для wallet-v4. На двери обещана простая услуга: просить кошелек о регулярном платеже. Но внутри стоит странный счетчик. Два отказа — и на следующем внешнем стуке контора не просит денег, а вызывает самоуничтожение. Остаток баланса уходит бенефициару, а плагин закрывает лавку навсегда.*

## МЕСТО ПРОИСШЕСТВИЯ

Улика найдена в репозитории [ton-blockchain/wallet-contract](https://github.com/ton-blockchain/wallet-contract), в коммите [`3fd1d7a`](https://github.com/ton-blockchain/wallet-contract/commit/3fd1d7ae39f1c46ec1f2be54c4040d8d87505e0f), в файле [`func/simple-subscription-plugin.fc`](https://github.com/ton-blockchain/wallet-contract/blob/3fd1d7ae39f1c46ec1f2be54c4040d8d87505e0f/func/simple-subscription-plugin.fc). Коммит называется `subscription: max failed attempts = 2; set start_time in init_state`; цитаты сверены по этому снимку.

Дословная табличка у счетчика, строки [6–11](https://github.com/ton-blockchain/wallet-contract/blob/3fd1d7ae39f1c46ec1f2be54c4040d8d87505e0f/func/simple-subscription-plugin.fc#L6-L11):

```func
int op:destruct() asm "0x64737472 PUSHINT";
int op:payment_request() asm "0x706c7567 PUSHINT";
int op:fallback() asm "0x756e6b77 PUSHINT";
int op:subscription() asm "0x73756273 PUSHINT";
int max_failed_attempts() asm "2 PUSHINT";
int max_reserved_funds() asm "67108864 PUSHINT"; ;; 0.0671 TON
```

## ПЕРВЫЙ СЛЕД: ДВА ОТКАЗА И ОДИН ПРИГОВОР

На строке [10](https://github.com/ton-blockchain/wallet-contract/blob/3fd1d7ae39f1c46ec1f2be54c4040d8d87505e0f/func/simple-subscription-plugin.fc#L10) контора устанавливает `max_failed_attempts()` равным двум. Это не число попыток на бумаге и не лимит писем в приемной: ниже этот результат напрямую управляет судьбой плагина.

Внешняя дверь открыта в `recv_external`, строки [157–169](https://github.com/ton-blockchain/wallet-contract/blob/3fd1d7ae39f1c46ec1f2be54c4040d8d87505e0f/func/simple-subscription-plugin.fc#L157-L169):

```func
() recv_external(slice in_msg) impure {
  var (wallet, beneficiary, amount, period, start_time, timeout, last_payment_time, last_request_time, failed_attempts, subscription_id) = load_storage();
  int last_timeslot = (last_payment_time - start_time) / period;
  int cur_timeslot = (now() - start_time) / period;
  throw_unless(30, (cur_timeslot > last_timeslot) & (last_request_time + timeout < now())); ;; too early request
  accept_message();
  if (failed_attempts >= max_failed_attempts()) {
    self_destruct(wallet, beneficiary);
  } else {
    request_payment(wallet, amount);
    failed_attempts += 1;
  }
  save_storage(wallet, beneficiary, amount, period, start_time, timeout, last_payment_time, now(), failed_attempts, subscription_id);
}
```

На строках [161–162](https://github.com/ton-blockchain/wallet-contract/blob/3fd1d7ae39f1c46ec1f2be54c4040d8d87505e0f/func/simple-subscription-plugin.fc#L161-L162) проверяется, что новый период уже настал и тайм-аут прошел. Затем сообщение принимается. Но на строках [163–167](https://github.com/ton-blockchain/wallet-contract/blob/3fd1d7ae39f1c46ec1f2be54c4040d8d87505e0f/func/simple-subscription-plugin.fc#L163-L167) счетчик превращается в гильотину: если `failed_attempts` уже не меньше двух, вызывается `self_destruct`; иначе кошелек получает новый запрос, а счетчик растет на единицу.

Иными словами, первый неуспешный запрос оставляет счетчик равным единице, второй — двум. Третий допустимый внешний вызов уже не просит оплату. Он идет в комнату самоуничтожения. Газовый рожок кашлянул: регулярная подписка выглядит как сервис напоминаний, пока ее бухгалтерия не решает, что два молчаливых должника — достаточная причина снести здание.

## ВТОРОЙ СЛЕД: ЧТО ЗНАЧИТ «НЕУДАЧА»

Платежный запрос формируется функцией `request_payment`, строки [82–94](https://github.com/ton-blockchain/wallet-contract/blob/3fd1d7ae39f1c46ec1f2be54c4040d8d87505e0f/func/simple-subscription-plugin.fc#L82-L94). Она вычисляет плату за газ и пересылку, собирает сообщение для кошелька и передает ему `requested_amount`.

Ответ приходит во внутреннюю дверь, строки [137–149](https://github.com/ton-blockchain/wallet-contract/blob/3fd1d7ae39f1c46ec1f2be54c4040d8d87505e0f/func/simple-subscription-plugin.fc#L137-L149):

```func
  if (op == (op:payment_request() | 0x80000000)) {
    int last_timeslot = (last_payment_time - start_time) / period;
    int cur_timeslot = (now() - start_time) / period;
    throw_if(49, last_timeslot >= cur_timeslot);
    (int from_wc, _) = s_addr.parse_std_addr();

    if (msg_value >= amount - short_msg_fwd_fee(from_wc) ) {
      last_payment_time = now();
      failed_attempts = 0;
      forward_funds(beneficiary, false, op:subscription());
    }

    return save_storage(wallet, beneficiary, amount, period, start_time, timeout, last_payment_time, last_request_time, failed_attempts, subscription_id);
  }
```

Счетчик обнуляется только в ветке на строках [143–146](https://github.com/ton-blockchain/wallet-contract/blob/3fd1d7ae39f1c46ec1f2be54c4040d8d87505e0f/func/simple-subscription-plugin.fc#L143-L146), где `msg_value` достаточно велик, чтобы покрыть сумму и пересылку. Если кошелек прислал меньше, `failed_attempts` не меняется обратно: после двух внешних просьб он остается равным двум.

Здесь есть важная оговорка. Код не доказывает, что любой отказ кошелька автоматически вызовет гибель. Внешний вызов сам увеличивает счетчик после отправки запроса, строки [165–167](https://github.com/ton-blockchain/wallet-contract/blob/3fd1d7ae39f1c46ec1f2be54c4040d8d87505e0f/func/simple-subscription-plugin.fc#L165-L167), а внутренний ответ способен сбросить его при достаточном платеже. Но если два цикла не дали успешного ответа, третья внешняя проверка уже выбирает не новый запрос, а `self_destruct`.

## ТРЕТИЙ СЛЕД: КУДА УХОДИТ ОСТАТОК

Самоуничтожение не ограничивается печальной записью в журнал. Функция `self_destruct` начинается на строке [97](https://github.com/ton-blockchain/wallet-contract/blob/3fd1d7ae39f1c46ec1f2be54c4040d8d87505e0f/func/simple-subscription-plugin.fc#L97) и сперва отправляет кошельку событие с операцией `destruct`, строки [102–108](https://github.com/ton-blockchain/wallet-contract/blob/3fd1d7ae39f1c46ec1f2be54c4040d8d87505e0f/func/simple-subscription-plugin.fc#L102-L108). Сообщение помечено как non-bounce: в комментарии на строке [102](https://github.com/ton-blockchain/wallet-contract/blob/3fd1d7ae39f1c46ec1f2be54c4040d8d87505e0f/func/simple-subscription-plugin.fc#L102) сказано, что ответ не нужен.

Затем строка [112](https://github.com/ton-blockchain/wallet-contract/blob/3fd1d7ae39f1c46ec1f2be54c4040d8d87505e0f/func/simple-subscription-plugin.fc#L112) вызывает:

```func
forward_funds(beneficiary, true, op:destruct());
```

А сама `forward_funds` на строках [64–79](https://github.com/ton-blockchain/wallet-contract/blob/3fd1d7ae39f1c46ec1f2be54c4040d8d87505e0f/func/simple-subscription-plugin.fc#L64-L79) при флаге уничтожения ставит режим `128 + 32`: передать весь оставшийся баланс и уничтожить контракт, если итоговый баланс равен нулю. Комментарий на строках [75–77](https://github.com/ton-blockchain/wallet-contract/blob/3fd1d7ae39f1c46ec1f2be54c4040d8d87505e0f/func/simple-subscription-plugin.fc#L75-L77) говорит это почти без тумана: `carry all the remaining balance` и `must be destroyed`.

Получается лондонская сцена с тремя действующими лицами. Кошельку посылают уведомление, бенефициару — остаток, а самому плагину оставляют приказ исчезнуть. Если причина была в двух неудачных запросах, деньги не запираются внутри подписки и не ждут хозяина: маршрут ведет к `beneficiary`.

## ЧЕТВЕРТЫЙ СЛЕД: РУЧКА, КОТОРАЯ САМА ПОДПИСАЛА ПРИГОВОР

У плагина есть и явная внутренняя команда уничтожения. Когда сообщение приходит от `beneficiary` и содержит операцию `destruct`, строки [121–127](https://github.com/ton-blockchain/wallet-contract/blob/3fd1d7ae39f1c46ec1f2be54c4040d8d87505e0f/func/simple-subscription-plugin.fc#L121-L127) вызывают ту же процедуру.

Но после этого в файле есть еще один путь: в `recv_internal`, строки [151–153](https://github.com/ton-blockchain/wallet-contract/blob/3fd1d7ae39f1c46ec1f2be54c4040d8d87505e0f/func/simple-subscription-plugin.fc#L151-L153), любая команда `op:destruct()` проходит напрямую к `forward_funds`. Внешняя логика из строк [163–164](https://github.com/ton-blockchain/wallet-contract/blob/3fd1d7ae39f1c46ec1f2be54c4040d8d87505e0f/func/simple-subscription-plugin.fc#L163-L164) делает тот же финал автоматически, когда счетчик достиг порога.

Редакция не называет это ошибкой: лимит неудачных оплат может быть сознательной политикой. Но формулировка коммита [3fd1d7a](https://github.com/ton-blockchain/wallet-contract/commit/3fd1d7ae39f1c46ec1f2be54c4040d8d87505e0f) особенно хорошо сочетается с уликой. В одной строке лимит равен двум, а за ним стоит не пауза, не уведомление владельца и не режим ожидания — там стоит дверь с надписью `self_destruct`.

## ХРОНИКА МЕЛКОГО КРИНЖА

*Отдел счетов.* Строка [10](https://github.com/ton-blockchain/wallet-contract/blob/3fd1d7ae39f1c46ec1f2be54c4040d8d87505e0f/func/simple-subscription-plugin.fc#L10) превращает два в порог катастрофы. В подписке на регулярные платежи это звучит так: два промаха бухгалтерии — и сервис больше не существует.

*Отдел обнуления.* Строки [143–146](https://github.com/ton-blockchain/wallet-contract/blob/3fd1d7ae39f1c46ec1f2be54c4040d8d87505e0f/func/simple-subscription-plugin.fc#L143-L146) возвращают счетчик к нулю лишь при достаточном `msg_value`. Неполный ответ не чинит историю, а оставляет ее на месте.

*Отдел эвакуации.* Строки [110–112](https://github.com/ton-blockchain/wallet-contract/blob/3fd1d7ae39f1c46ec1f2be54c4040d8d87505e0f/func/simple-subscription-plugin.fc#L110-L112) обещают отправить все оставшиеся средства бенефициару и уничтожить плагин. Это не «попробуем еще раз после паузы», а финальный выезд с мебелью.

## ВЕРДИКТ СЫЩИКА

Репозиторий [ton-blockchain/wallet-contract](https://github.com/ton-blockchain/wallet-contract) пойман на выразительной политике отказа. В коммите [`3fd1d7a`](https://github.com/ton-blockchain/wallet-contract/commit/3fd1d7ae39f1c46ec1f2be54c4040d8d87505e0f) файл [`func/simple-subscription-plugin.fc`](https://github.com/ton-blockchain/wallet-contract/blob/3fd1d7ae39f1c46ec1f2be54c4040d8d87505e0f/func/simple-subscription-plugin.fc) ставит предел в два неудачных запроса на строке [10](https://github.com/ton-blockchain/wallet-contract/blob/3fd1d7ae39f1c46ec1f2be54c4040d8d87505e0f/func/simple-subscription-plugin.fc#L10), а на строках [163–164](https://github.com/ton-blockchain/wallet-contract/blob/3fd1d7ae39f1c46ec1f2be54c4040d8d87505e0f/func/simple-subscription-plugin.fc#L163-L164) переводит следующий внешний вызов в самоуничтожение.

Дальше дело становится еще сочнее. `self_destruct` в строках [97–112](https://github.com/ton-blockchain/wallet-contract/blob/3fd1d7ae39f1c46ec1f2be54c4040d8d87505e0f/func/simple-subscription-plugin.fc#L97-L112) уведомляет кошелек, пересылает остаток бенефициару и вызывает режим уничтожения. Код может быть рассчитан именно на такой предохранитель, но предохранитель похож на гильотину: он не ставит подписку на карантин, а ликвидирует ее вместе с остатком баланса.

Сыщик закрыл папку. Газовый рожок дернулся в тумане. На двери осталась строка [164](https://github.com/ton-blockchain/wallet-contract/blob/3fd1d7ae39f1c46ec1f2be54c4040d8d87505e0f/func/simple-subscription-plugin.fc#L164): `self_destruct(wallet, beneficiary);`. В Лондоне два неудачных платежа — это еще не конец света. Но для этой подписки они уже приглашение на похороны.

🐀
