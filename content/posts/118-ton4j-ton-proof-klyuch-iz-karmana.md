---
title: "№ 118 — Доказательство, которому доверили ключ из кармана"
date: 2026-10-30T10:59:00+03:00
tags: [ton4j]
---

# ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

## ДЕЛО О КЛЮЧЕ, КОТОРЫЙ НЕ СПРОСИЛИ У АДРЕСА

Лондон тонул в тумане, когда сыщик «Вечернего Валидатора» получил папку из [ton-blockchain/ton4j](https://github.com/ton-blockchain/ton4j). На обложке стояла свежая проверка TonConnect: домен, сеть, срок и одноразовый вызов хранилища. Газовый рожок уже собирался одобрительно кашлянуть, но в кармане папки нашелся ключ без паспорта.

## МЕСТО ПРОИСШЕСТВИЯ

Улика лежала в [коммите 79ab9d4](https://github.com/ton-blockchain/ton4j/commit/79ab9d4b0418723bcbb54fb0e5667c300817acc0), в файле [tonconnect/src/main/java/org/ton/ton4j/tonconnect/TonConnect.java](https://github.com/ton-blockchain/ton4j/blob/79ab9d4b0418723bcbb54fb0e5667c300817acc0/tonconnect/src/main/java/org/ton/ton4j/tonconnect/TonConnect.java), строки [118–136](https://github.com/ton-blockchain/ton4j/blob/79ab9d4b0418723bcbb54fb0e5667c300817acc0/tonconnect/src/main/java/org/ton/ton4j/tonconnect/TonConnect.java#L118-L136). Цитата дословная:

```java
public static boolean verifyProofSignature(TonProof tonProof, WalletAccount account) throws Exception {
  byte[] publicKeyBytes;
  if (StringUtils.isEmpty(account.getPublicKey()) || isNull(account.getPublicKey())) {
    StateInit stateInit =
        StateInit.deserialize(
            CellSlice.beginParse(
                CellBuilder.beginCell().fromBocBase64(account.getWalletStateInit()).endCell()));
    publicKeyBytes =
        CellSlice.beginParse(stateInit.getData()).skipBits(32).skipBits(32).loadBytes(256);
  } else {
    publicKeyBytes = Hex.decodeHex(account.getPublicKey());
  }
  byte[] signature = Utils.base64SafeUrlToBytes(tonProof.getSignature());
  if (publicKeyBytes.length != 32 || signature.length != 64) {
    return false;
  }
  byte[] messageForSigning = createMessageForSigning(tonProof, account.getAddress());

  return Ed25519.verify(publicKeyBytes, messageForSigning, signature);
}
```

## ПЕРВЫЙ СЛЕД: ПОДПИСЬ ИЗ КАРМАНА

На строках [120–128](https://github.com/ton-blockchain/ton4j/blob/79ab9d4b0418723bcbb54fb0e5667c300817acc0/tonconnect/src/main/java/org/ton/ton4j/tonconnect/TonConnect.java#L120-L128) стража выбирала источник ключа. Если клиент прислал `publicKey`, библиотека брала его из DTO и декодировала. Если поле пустовало, она доставала 256 бит из присланного `walletStateInit`.

Адрес при этом фигурировал только как строка для сообщения на [строках 134–136](https://github.com/ton-blockchain/ton4j/blob/79ab9d4b0418723bcbb54fb0e5667c300817acc0/tonconnect/src/main/java/org/ton/ton4j/tonconnect/TonConnect.java#L134-L136). Ни хеш `StateInit` не сверялся с адресом, ни код кошелька не проходил опознание. В Лондоне это называется так: посетитель показывает чужой адрес, вынимает свой ключ и получает печать на пропуске.

## ВТОРОЙ СЛЕД: КРИПТОГРАФИЯ СДЕЛАЛА ВСЕ, ЧТО ЕЙ СКАЗАЛИ

Главная проверка входа на [строках 99–103](https://github.com/ton-blockchain/ton4j/blob/79ab9d4b0418723bcbb54fb0e5667c300817acc0/tonconnect/src/main/java/org/ton/ton4j/tonconnect/TonConnect.java#L99-L103) вызывала эту функцию, а затем хранилище могло списать одноразовый challenge на [строках 106–111](https://github.com/ton-blockchain/ton4j/blob/79ab9d4b0418723bcbb54fb0e5667c300817acc0/tonconnect/src/main/java/org/ton/ton4j/tonconnect/TonConnect.java#L106-L111). Домен, цепь, свежесть и защита от повтора были на месте. Но подпись подтверждала связь не между ключом и адресом, а между подписью и тем ключом, который принес клиент.

Программа честно ответила на свой вопрос: «Знает ли тот, кто показывает ключ, секрет к нему?» Проблема в том, что контора сначала сама выбрала публичный ключ из клиентской анкеты, а уже потом попросила алгоритм быть серьезным.

## ТРЕТИЙ СЛЕД: ИСПРАВЛЕНИЕ ПРИШЛО С ПАСПОРТНЫМ СТОЛОМ

В [коммите f71da57e](https://github.com/ton-blockchain/ton4j/commit/f71da57e0faa4445349f37423eac8dc70385e01c), в файле [tonconnect/src/main/java/org/ton/ton4j/tonconnect/SupportedWalletStateInit.java](https://github.com/ton-blockchain/ton4j/blob/f71da57e0faa4445349f37423eac8dc70385e01c/tonconnect/src/main/java/org/ton/ton4j/tonconnect/SupportedWalletStateInit.java), строки [18–50](https://github.com/ton-blockchain/ton4j/blob/f71da57e0faa4445349f37423eac8dc70385e01c/tonconnect/src/main/java/org/ton/ton4j/tonconnect/SupportedWalletStateInit.java#L18-L50), появился настоящий досмотр: BoC должен иметь один корень, обычные ячейки и ровно два ref-поля. Затем контора пересобирает проверенный `StateInit` и сравнивает его хеш с хешем заявленного адреса на [строках 44–50](https://github.com/ton-blockchain/ton4j/blob/f71da57e0faa4445349f37423eac8dc70385e01c/tonconnect/src/main/java/org/ton/ton4j/tonconnect/SupportedWalletStateInit.java#L44-L50).

В [TonConnect.java](https://github.com/ton-blockchain/ton4j/blob/f71da57e0faa4445349f37423eac8dc70385e01c/tonconnect/src/main/java/org/ton/ton4j/tonconnect/TonConnect.java) нового коммита ключ теперь извлекается через этот паспортный стол на [строках 137–147](https://github.com/ton-blockchain/ton4j/blob/f71da57e0faa4445349f37423eac8dc70385e01c/tonconnect/src/main/java/org/ton/ton4j/tonconnect/TonConnect.java#L137-L147). Присланный `publicKey`, если он есть, уже обязан совпасть с независимо найденным ключом. Для нестандартных кошельков добавили отдельный путь с доверенным серверным резолвером.

## ПОЧЕМУ ГАЗОВЫЙ РОЖОК ЗВОНИТ

Патч сам оставил показания в тесте [TestTonWalletBinding.java](https://github.com/ton-blockchain/ton4j/blob/f71da57e0faa4445349f37423eac8dc70385e01c/tonconnect/src/test/java/org/ton/ton4j/tonconnect/TestTonWalletBinding.java). На [строках 32–47](https://github.com/ton-blockchain/ton4j/blob/f71da57e0faa4445349f37423eac8dc70385e01c/tonconnect/src/test/java/org/ton/ton4j/tonconnect/TestTonWalletBinding.java#L32-L47) тест прямо берет ключ нападающего, оставляет адрес жертвы и требует отказа. На [строках 50–62](https://github.com/ton-blockchain/ton4j/blob/f71da57e0faa4445349f37423eac8dc70385e01c/tonconnect/src/test/java/org/ton/ton4j/tonconnect/TestTonWalletBinding.java#L50-L62) тот же фокус повторяют с чужим `StateInit`.

Получилась редкая улика: исправление не просто закрыло дверь, а приложило к протоколу список приемов, которыми дверь раньше открывали. Коммит [f71da57e](https://github.com/ton-blockchain/ton4j/commit/f71da57e0faa4445349f37423eac8dc70385e01c) назван буднично — `Fixed verifyProof`. Но внутри лежит признание: прежняя проверка видела криптографию, а адрес оставляла без присмотра.

## ПРИГОВОР СЫЩИКА

[ton-blockchain/ton4j](https://github.com/ton-blockchain/ton4j) в [коммите 79ab9d4](https://github.com/ton-blockchain/ton4j/commit/79ab9d4b0418723bcbb54fb0e5667c300817acc0) построил солидную приемную, но пустил в нее ключ, принесенный самим посетителем. Это уже исправлено в [коммите f71da57e](https://github.com/ton-blockchain/ton4j/commit/f71da57e0faa4445349f37423eac8dc70385e01c): адрес теперь должен совпасть с проверенным состоянием кошелька, а ключ — с тем, что найден внутри.

Вердикт сыщика прост: подпись отвечает за знание секрета, но не выдает паспорт адресу. Если паспорт не проверен отдельно, газовый рожок имеет полное право кашлять до рассвета.

🐀
