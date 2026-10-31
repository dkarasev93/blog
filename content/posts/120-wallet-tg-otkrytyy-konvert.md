---
title: "№ 120 — Кошелек, который вынес старый ключ на улицу"
date: 2026-10-31T16:59:00+03:00
tags: [tg-wallet-contract]
---

# ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

## ДЕЛО О КЛЮЧЕ, КОТОРЫЙ ВЫШЕЛ ПОСЛЕ СМЕНЫ ЗАМКА

Лондон тонул в тумане, когда сыщик «Вечернего Валидатора» получил папку из [ton-blockchain/tg-wallet-contract](https://github.com/ton-blockchain/tg-wallet-contract). В ней описывалась смена ключа кошелька: старый ключ отходит в историю, новый принимает караул. Все чинно, пока сыщик не заметил, что вместе с историей на улицу выходит еще один конверт.

## МЕСТО ПРОИСШЕСТВИЯ

Улика лежит в [коммите 1a29ae5](https://github.com/ton-blockchain/tg-wallet-contract/commit/1a29ae5d90e03c3d585d3e42e6ecef691d727a3c), в файле [contracts/WalletTg/WalletTg.tolk](https://github.com/ton-blockchain/tg-wallet-contract/blob/1a29ae5d90e03c3d585d3e42e6ecef691d727a3c/contracts/WalletTg/WalletTg.tolk), строки [53–60](https://github.com/ton-blockchain/tg-wallet-contract/blob/1a29ae5d90e03c3d585d3e42e6ecef691d727a3c/contracts/WalletTg/WalletTg.tolk#L53-L60). Цитата дословная:

```tolk
// Send an ext-out event "public key has been changed".
// Used by indexers to create actions, which are used by clients to decrypt messages encrypted by old private key.
fun sendOnKeyChangedEvent(encryptedOldPrivateKey: bits256) {
    val eventMsg = createExternalLogMessage({
        dest: createAddressNone(),
        body: OnKeyHasBeenChangedEvent { encryptedOldPrivateKey }
    });
    eventMsg.send(SEND_MODE_REGULAR);   // not +2
}
```

Газовый рожок поперхнулся на словах `old private key`. Контора не отправляет сам ключ открытым текстом: в запросе лежит XOR от хеша нового секрета с текущим приватным ключом. Но этот результат все равно кладут в ext-out, то есть в публичное событие цепи, предназначенное для индексаторов и клиентов. Конверт запечатан не в сейфе, а приколот к афише на площади.

## ПЕРВЫЙ СЛЕД: СХЕМА ПОДПИСАЛА ПРИГОВОР

В файле [contracts/WalletTg/messages.tolk](https://github.com/ton-blockchain/tg-wallet-contract/blob/1a29ae5d90e03c3d585d3e42e6ecef691d727a3c/contracts/WalletTg/messages.tolk) строки [92–103](https://github.com/ton-blockchain/tg-wallet-contract/blob/1a29ae5d90e03c3d585d3e42e6ecef691d727a3c/contracts/WalletTg/messages.tolk#L92-L103) добавляют к запросу смены ключа поле `encryptedOldPrivateKey`. Комментарий не прячется в подвале:

> `/// Used to calculate old private key after changing it: to render old encrypted messages.`
> `/// Must contain XOR (hash(newPrivateKey+salt), currentPrivateKey).`

Это не случайный лог и не отладочная записка. Само сообщение несет материал, из которого клиент должен восстановить старый приватный ключ для чтения старых зашифрованных сообщений. Так что вопрос не в том, есть ли шифрование, а в том, насколько долго и кем защищен его второй компонент.

## ВТОРОЙ СЛЕД: ИНДЕКСАТОРУ ОСТАВИЛИ КОПИЮ

На [строках 123–125](https://github.com/ton-blockchain/tg-wallet-contract/blob/1a29ae5d90e03c3d585d3e42e6ecef691d727a3c/contracts/WalletTg/messages.tolk#L123-L125) описан внешний выход:

```tolk
/// External-out emitted when a key has been changed. Used by indexers.
struct (0xEBA19948) OnKeyHasBeenChangedEvent {
    encryptedOldPrivateKey: bits256
}
```

А в [contracts/WalletTg/WalletTg.tolk](https://github.com/ton-blockchain/tg-wallet-contract/blob/1a29ae5d90e03c3d585d3e42e6ecef691d727a3c/contracts/WalletTg/WalletTg.tolk) строки [97–109](https://github.com/ton-blockchain/tg-wallet-contract/blob/1a29ae5d90e03c3d585d3e42e6ecef691d727a3c/contracts/WalletTg/WalletTg.tolk#L97-L109) показывают момент передачи: после проверки подписей и сохранения нового публичного ключа контракт вызывает `sendOnKeyChangedEvent`.

Иными словами, смена замка сопровождается публикацией слепка старого ключа. Его видят все, кто читает события. Если соль, новый приватный ключ или логика клиента когда-нибудь окажутся скомпрометированы, старый архив уже не надо искать в сейфе: цепь сама ведет к полке.

## ТРЕТИЙ СЛЕД: ПЕЧАТЬ «NOT +2»

Особенно бодро выглядит [строка 60](https://github.com/ton-blockchain/tg-wallet-contract/blob/1a29ae5d90e03c3d585d3e42e6ecef691d727a3c/contracts/WalletTg/WalletTg.tolk#L60): `eventMsg.send(SEND_MODE_REGULAR);   // not +2`. Смысл комментария ясен: событие должно уйти обычным исходящим сообщением, и его стоимость не списывается по мягкому режиму `+2`.

Получается двойная бухгалтерия. Пользователь меняет ключ и просит сохранить доступ к старым шифрам. Контракт публикует материал для восстановления, а за саму публикацию берет обычную плату из баланса. Газовый рожок сообщил: старый ключ не только вынесли на улицу, но и выписали счет за его доставку.

## ВЕРДИКТ СЫЩИКА

Это не доказательство, что любой наблюдатель прямо сейчас получит приватный ключ. `encryptedOldPrivateKey` назван шифротекстом, а восстановление задумано для владельца нового ключа. Но архитектурный след сочен и прост: секретный переход между двумя ключами превращен в публичный ext-out для индексаторов, причем контракт сам поясняет, что этот кусок нужен для расшифровки старых сообщений.

В Лондоне подобный конверт не назвали бы тайным. Его назвали бы запечатанным письмом, которое сначала зачитали на площади.

**Дело закрыто. Конверт оставлен под наблюдением.**
