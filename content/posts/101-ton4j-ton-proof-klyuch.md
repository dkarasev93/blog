---
title: "Выпуск № 101: Ключ, который клиент принес сам"
date: 2026-10-21T16:58:00+03:00
tags: [ton4j]
---

# ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

## ДЕЛО О КЛЮЧЕ, ПРИНЕСЕННОМ НА ДОВЕРИИ

Лондон затянуло туманом, когда сыщик «Вечернего Валидатора» получил свежую папку из [ton-blockchain/ton4j](https://github.com/ton-blockchain/ton4j). Внутри лежала не кража монет, а более тонкая сцена: вход в систему проверял подпись, но долго позволял самому посетителю принести ключ и назвать его своим.

На обложке нового ремонта стоит [коммит f71da57](https://github.com/ton-blockchain/ton4j/commit/f71da57e0faa4445349f37423eac8dc70385e01c) — `Fixed verifyProof: it validates supported wallet StateInit code/data, matches its hash to the claimed address, and uses the extracted key`. Сыщик прочел заголовок дважды. Когда коммит так подробно объясняет, кого именно перестали пускать без проверки, газовый рожок уже кашляет.

## МЕСТО ПРОИСШЕСТВИЯ

Старый след лежит в родительском коммите [79ab9d4](https://github.com/ton-blockchain/ton4j/commit/79ab9d4b0418723bcbb54fb0e5667c300817acc0), в файле [tonconnect/src/main/java/org/ton/ton4j/tonconnect/TonConnect.java](https://github.com/ton-blockchain/ton4j/blob/79ab9d4b0418723bcbb54fb0e5667c300817acc0/tonconnect/src/main/java/org/ton/ton4j/tonconnect/TonConnect.java), строки [118–129](https://github.com/ton-blockchain/ton4j/blob/79ab9d4b0418723bcbb54fb0e5667c300817acc0/tonconnect/src/main/java/org/ton/ton4j/tonconnect/TonConnect.java#L118-L129). Цитата дословная:

```java
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
```

Картина прелестная. Если гость не принес `publicKey`, контора пыталась достать ключ из `walletStateInit`. Но если гость ключ принес, охрана брала `account.getPublicKey()` и шла с ним на проверку подписи. Связь между этим ключом и адресом кошелька не доказывалась. Подпись могла быть математически верной, а ключ — не принадлежать заявленному адресу.

Это не означает, что любой вход автоматически открывал дверь: оставались домен, цепь, срок, challenge и сама подпись. Но главный вопрос владения адресом отправлялся в туман. Сыщик записал в протокол: посетитель предъявлял удостоверение личности, а затем сам выбирал, каким ключом его подписать.

## ПЕРВЫЙ СЛЕД: НОВАЯ ОХРАНА НЕ ВЕРИТ БУМАЖНОМУ КЛЮЧУ

В исправляющем коммите [f71da57](https://github.com/ton-blockchain/ton4j/commit/f71da57e0faa4445349f37423eac8dc70385e01c) появился файл [tonconnect/src/main/java/org/ton/ton4j/tonconnect/SupportedWalletStateInit.java](https://github.com/ton-blockchain/ton4j/blob/f71da57e0faa4445349f37423eac8dc70385e01c/tonconnect/src/main/java/org/ton/ton4j/tonconnect/SupportedWalletStateInit.java). В строках [20–42](https://github.com/ton-blockchain/ton4j/blob/f71da57e0faa4445349f37423eac8dc70385e01c/tonconnect/src/main/java/org/ton/ton4j/tonconnect/SupportedWalletStateInit.java#L20-L42) входной BoC разбирается, проверяется число корней, форма StateInit, код и данные кошелька:

```java
      if (walletStateInit == null || walletStateInit.isEmpty()) {
        throw invalidState();
      }
      // TON Connect accepts both standard and URL-safe base64 BoCs.
      byte[] boc =
          Base64.getDecoder().decode(walletStateInit.replace('-', '+').replace('_', '/'));
      List<Cell> roots = Cell.fromBocMultiRoots(boc);
      if (roots.size() != 1) {
        throw invalidState();
      }
      Cell root = roots.get(0);
      requireOrdinaryCells(root);
      if (root.getBitLength() != 5 || root.getRefs().size() != 2) {
        throw invalidState();
      }
      CellSlice state = CellSlice.beginParse(root);
      // No split depth, special flags or libraries; code and data must both be present.
      if (state.loadUint(5).intValue() != 0b00110) {
        throw invalidState();
      }
      Cell code = state.loadRef();
      Cell data = state.loadRef();
      byte[] key = extractPublicKey(code, data);
```

Теперь клиентский `publicKey` не является источником истины. Сначала контора извлекает ключ из проверенного состояния стандартного кошелька. Для нестандартного кошелька новый путь требует серверный resolver, а не еще одну бумагу из кармана клиента.

## ВТОРОЙ СЛЕД: АДРЕС СВЕРЯЮТ С СОБСТВЕННЫМ СОСТОЯНИЕМ

Самая важная сцена стоит в том же файле [SupportedWalletStateInit.java](https://github.com/ton-blockchain/ton4j/blob/f71da57e0faa4445349f37423eac8dc70385e01c), строки [44–50](https://github.com/ton-blockchain/ton4j/blob/f71da57e0faa4445349f37423eac8dc70385e01c/tonconnect/src/main/java/org/ton/ton4j/tonconnect/SupportedWalletStateInit.java#L44-L50):

```java
      // Rebuild the validated StateInit so the address uses the actual code and data hashes.
      Cell validatedState =
          CellBuilder.beginCell().storeUint(0b00110, 5).storeRef(code).storeRef(data).endCell();
      String claimedHash = canonicalAddress.substring(canonicalAddress.indexOf(':') + 1);
      if (!claimedHash.equals(Hex.encodeHexString(root.getHash()))
          || !claimedHash.equals(Hex.encodeHexString(validatedState.getHash()))) {
        throw new IllegalArgumentException("Wallet StateInit does not match the account address");
      }
```

Вот и вся улика, от которой в участке стало тихо. Адрес больше не принимают за чистую монету: его хеш должен совпасть с фактическим StateInit. Код кошелька, его данные и заявленный адрес обязаны выйти из одной кареты.

## ТРЕТИЙ СЛЕД: ПОДПИСЬ ПОЛУЧАЕТ НЕ ТОТ КЛЮЧ, КОТОРЫЙ ЕЙ ПОДСУНУЛИ

В файле [tonconnect/src/main/java/org/ton/ton4j/tonconnect/TonConnect.java](https://github.com/ton-blockchain/ton4j/blob/f71da57e0faa4445349f37423eac8dc70385e01c/tonconnect/src/main/java/org/ton/ton4j/tonconnect/TonConnect.java), строки [137–152](https://github.com/ton-blockchain/ton4j/blob/f71da57e0faa4445349f37423eac8dc70385e01c/tonconnect/src/main/java/org/ton/ton4j/tonconnect/TonConnect.java#L137-L152), новый порядок записан почти бухгалтерски:

```java
      byte[] publicKey =
          keyResolver == null
              ? SupportedWalletStateInit.resolvePublicKey(
                  wallet.getAddress(), wallet.getWalletStateInit())
              : keyResolver.resolvePublicKey(wallet.getAddress(), context.getExpectedChain());
      if (publicKey == null || publicKey.length != 32) {
        return false;
      }
      publicKey = publicKey.clone();
      if (!StringUtils.isEmpty(wallet.getPublicKey())
          && !MessageDigest.isEqual(publicKey, Hex.decodeHex(wallet.getPublicKey()))) {
        return false;
      }
      if (!verifySignature(proof, wallet.getAddress(), publicKey)) {
        return false;
      }
```

Сначала ключ привязывают к адресу через StateInit либо через серверную проверку цепи. Потом, если клиент все же прислал свой `publicKey`, его сверяют с найденным. И только после этого проверяют подпись. В старом порядке клиентский ключ мог стать судьей; в новом он всего лишь свидетель, которого проверяют.

## ПРИГОВОР СЫЩИКА

Репозиторий [ton-blockchain/ton4j](https://github.com/ton-blockchain/ton4j) пойман не на готовой краже, а на опасной старой привычке: криптографическая подпись и владение адресом стояли рядом, но не были связаны. В родительском коммите [79ab9d4](https://github.com/ton-blockchain/ton4j/commit/79ab9d4b0418723bcbb54fb0e5667c300817acc0) файл [TonConnect.java](https://github.com/ton-blockchain/ton4j/blob/79ab9d4b0418723bcbb54fb0e5667c300817acc0/tonconnect/src/main/java/org/ton/ton4j/tonconnect/TonConnect.java), строки [118–129](https://github.com/ton-blockchain/ton4j/blob/79ab9d4b0418723bcbb54fb0e5667c300817acc0/tonconnect/src/main/java/org/ton/ton4j/tonconnect/TonConnect.java#L118-L129), принимал ключ из `account.getPublicKey()`. Исправление [f71da57](https://github.com/ton-blockchain/ton4j/commit/f71da57e0faa4445349f37423eac8dc70385e01c) добавило проверку StateInit и адреса в [SupportedWalletStateInit.java](https://github.com/ton-blockchain/ton4j/blob/f71da57e0faa4445349f37423eac8dc70385e01c/tonconnect/src/main/java/org/ton/ton4j/tonconnect/SupportedWalletStateInit.java).

Приговор таков: подпись доказывает, что кто-то владеет ключом. Она не доказывает, что ключ принадлежит адресу, пока контора не проверит связь. Газовый рожок пробил сто первый удар, охрана сверила паспорт с домом, а сыщик растворился в лондонском тумане.

🐀
