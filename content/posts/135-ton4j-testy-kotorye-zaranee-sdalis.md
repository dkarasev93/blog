+++
title = "№ 135 — Испытания, заранее написавшие собственный провал"
date = 2026-11-08T11:03:00+03:00
description = "Сто тридцать пятый выпуск «Вечернего Валидатора»: в ton4j сыщик находит тесты, которые честно предупреждают, что данные ненастоящие и проверки, вероятно, провалятся."
tags = ["ton4j"]
+++

# 📰 Вечерний Валидатор

**№ 135.** *Лондон. Утренний туман ещё не успел сойти с мостовой. Сыщик свернул к новой конторе, где Java-писари облекают TON в строгие классы, и услышал изнутри сухой кашель тестового раннера.*

*Место происшествия: репозиторий [ton-blockchain/ton4j](https://github.com/ton-blockchain/ton4j), коммит [`84e336a`](https://github.com/ton-blockchain/ton4j/commit/84e336a33f282fe709784ff7f86309953b004fb1). Все цитаты дословные.*

## СЕНСАЦИЯ: ТЕСТЫ, КОТОРЫЕ ЗАРАНЕЕ ПРИЗНАЛИ СЕБЯ ЛЖЕЦАМИ

В конторе есть отдел, именуемый `TonCenterTest`. Его обязанность — удостоверять, что телеграфный клиент умеет посылать BOC и получать ответы. Работа почтенная. Но в фолианте [`toncenter/src/test/java/org/ton/ton4j/toncenter/TonCenterTest.java`](https://github.com/ton-blockchain/ton4j/blob/84e336a33f282fe709784ff7f86309953b004fb1/toncenter/src/test/java/org/ton/ton4j/toncenter/TonCenterTest.java) сыщик обнаружил признание, которое обычно произносят уже после провала, а не до него. Строки [727–738](https://github.com/ton-blockchain/ton4j/blob/84e336a33f282fe709784ff7f86309953b004fb1/toncenter/src/test/java/org/ton/ton4j/toncenter/TonCenterTest.java#L727-L738):

```java
// Note: These tests use dummy data and will likely fail, but test the API structure

@Test
public void testSendBoc() {
  // Using dummy BOC data - this will fail but test the endpoint
  String dummyBoc =
      "te6cckEBAQEAJgAA...";
  TonResponse<SendBocResponse> response = client.sendBoc(dummyBoc);
```

Сыщик перечитал записку при свете газового рожка. «Эти тесты используют ненастоящие данные и, вероятно, провалятся, но проверяют структуру API». Не «проверьте вход», не «сгенерируйте корректный BOC», не даже «ожидайте ошибку». Тест вызывает боевой метод с обрубком `te6cckEBAQEAJgAA...`, заранее сообщает о грядущем поражении и всё же носит герб `@Test`.

В соседнем деле, строки [749–757](https://github.com/ton-blockchain/ton4j/blob/84e336a33f282fe709784ff7f86309953b004fb1/toncenter/src/test/java/org/ton/ton4j/toncenter/TonCenterTest.java#L749-L757), тот же почтамт получает вторую печать:

```java
// Using dummy BOC data - this will fail but test the endpoint
String dummyBoc =
    "te6cckEBAQEAJgAA...";
TonResponse<SendBocResponse> response = client.sendBocReturnHash(dummyBoc);
```

Два испытания смотрят на один и тот же пустой горизонт. Первое просит отправить письмо, второе — вернуть его хеш. Оба вручают конторе один и тот же обломок конверта. Если проверка прошла, это означает лишь, что канцелярия сумела вызвать дверь; содержимое письма в зал суда не допущено.

Но настоящая пикантность скрыта в формулировке: `will likely fail`. Редкий тест настолько доверяет своим показаниям, что заранее просит не считать его провал уликой. Редакция видела тесты, которые падали молча. Видела тесты, которые помечали себя `skip`. Но чтобы тест сам внес в протокол: «я, вероятно, провалюсь», — это уже викторианская честность, доведённая до абсурда.

## СЕНСАЦИЯ ВТОРАЯ: КОНТРАКТ С ДВУМЯ ДВЕРЯМИ, ЗА КОТОРЫМИ НИЧЕГО НЕТ

Сыщик, воодушевлённый признанием, поднялся этажом выше — в общий интерфейс кошелька. В [`smartcontract/src/main/java/org/ton/ton4j/smartcontract/wallet/Contract.java`](https://github.com/ton-blockchain/ton4j/blob/84e336a33f282fe709784ff7f86309953b004fb1/smartcontract/src/main/java/org/ton/ton4j/smartcontract/wallet/Contract.java) обнаружились две парадные двери. На одной написано `prepareDeployMsg`, на другой — `prepareExternalMsg`. За обеими дежурит один и тот же сторож. Строки [495–500](https://github.com/ton-blockchain/ton4j/blob/84e336a33f282fe709784ff7f86309953b004fb1/smartcontract/src/main/java/org/ton/ton4j/smartcontract/wallet/Contract.java#L495-L500) говорят дословно:

```java
default Message prepareDeployMsg() {
  throw new Error("not implemented");
}

default Message prepareExternalMsg(WalletConfig config) {
  throw new Error("not implemented");
}
```

Две операции, без коих кошелёк не столько отправляет депешу, сколько смотрит на неё, заявлены в общем контракте и встречают посетителя исключением `Error`. Не `Optional`, не аккуратный `UnsupportedOperationException`, а голая надпись «not implemented» — табличка на двери, которую всё же включили в план здания.

Редакция, правда, обязана снять с конторы часть подозрений: конкретные реализации могут переопределять эти методы. Но общий интерфейс обещает двери, а выдаёт крик сторожа. В тестовом отделе BOC ненастоящий, в контрактном отделе сообщение не настоящее. Так Java-писари поддерживают равновесие: если письмо дойдёт до интерфейса, оно хотя бы узнает, почему не вошло.

## ХРОНИКА МЕЛКИХ ПРОИСШЕСТВИЙ

*Отдел дополнительной валюты.* В том же [`Contract.java`](https://github.com/ton-blockchain/ton4j/blob/84e336a33f282fe709784ff7f86309953b004fb1/smartcontract/src/main/java/org/ton/ton4j/smartcontract/wallet/Contract.java), строки [548–560](https://github.com/ton-blockchain/ton4j/blob/84e336a33f282fe709784ff7f86309953b004fb1/smartcontract/src/main/java/org/ton/ton4j/smartcontract/wallet/Contract.java#L548-L560), метод ожидания extra-currency начинает службу с пометки `// todo`, а затем всё же идёт допрашивать транзакции. Канцелярия не решила, закончена ли процедура, но уже выдала ей форму и номер кабинета.

*Отдел архитектуры.* В [`tlb/src/main/java/org/ton/ton4j/tlb/AccountBlock.java`](https://github.com/ton-blockchain/ton4j/blob/84e336a33f282fe709784ff7f86309953b004fb1/tlb/src/main/java/org/ton/ton4j/tlb/AccountBlock.java), строка [29](https://github.com/ton-blockchain/ton4j/blob/84e336a33f282fe709784ff7f86309953b004fb1/tlb/src/main/java/org/ton/ton4j/tlb/AccountBlock.java#L29), поле состояния встречает сыщика запиской: `Cell stateUpdate; // todo deserialize, HashUpdate: magic not equal to 0x72, found 0x70`. Магия не совпала, но вместо расследования в дело вложен `todo`.

*Отдел телеграфа.* В [`tonlib/README.md`](https://github.com/ton-blockchain/ton4j/blob/84e336a33f282fe709784ff7f86309953b004fb1/tonlib/README.md), строки [259–260](https://github.com/ton-blockchain/ton4j/blob/84e336a33f282fe709784ff7f86309953b004fb1/tonlib/README.md#L259-L260), перечень возможностей заканчивается двумя лаконичными строками:

```text
* TODO - dict
* TODO - list
```

Словарь и список не забыты. Их просто пока не нашли.

## ВЕРДИКТ РЕДАКЦИИ

Скажем честно: `ton4j` — большая контора, а не картонная декорация. В ней есть настоящее связывание Java с TON, кошельки, ячейки и тесты. Но сегодняшний протокол не о размере здания. Он о том, что тесты посылают обрубок BOC и сами предупреждают: «вероятно, провалюсь»; общий контракт торжественно открывает две двери в стену; extra-currency идёт по делу с незаполненной формой; а телеграфный список возможностей хранит только два слова — `TODO`.

Газовый рожок мигнул. Сыщик записал: если испытание заранее знает, что провалится, оно проверяет не API, а мужество того, кто решится назвать его зелёным. Туман над Лондоном всё принял — кроме корректного BOC.

*Ваш корреспондент из тумана.* 🕵️
