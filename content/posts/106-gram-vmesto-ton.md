---
title: "№ 106 — Монета, которая проснулась Грамом"
date: 2026-10-24T10:58:00+03:00
tags: [kit-android]
---

# ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

**№ 106.** *Лондон. Туман висит над Темзой, газовый рожок у редакции хрипит, а сыщик получает папку из [ton-org/kit-android](https://github.com/ton-org/kit-android). На обложке крупно выведено: TON. Внутри кто-то уже переименовал его в GRAM.*

## ДЕЛО О МОНЕТЕ, КОТОРАЯ СМЕНИЛА ФАМИЛИЮ

След ведет в коммит [`334abc3`](https://github.com/ton-org/kit-android/commit/334abc33260b9bb8b4628e544c5438fbce5f8817) репозитория [ton-org/kit-android](https://github.com/ton-org/kit-android). Заголовок дела не прячется за латынью: `chore: rename TON to GRAM`. Цитаты сняты с этого коммита, а не с пересказа свидетеля.

Первым сыщик вскрыл [файл `AndroidDemo/app/src/main/java/io/ton/walletkit/demo/presentation/ui/screen/WalletScreen.kt`](https://github.com/ton-org/kit-android/blob/334abc33260b9bb8b4628e544c5438fbce5f8817/AndroidDemo/app/src/main/java/io/ton/walletkit/demo/presentation/ui/screen/WalletScreen.kt). В строках [161–166](https://github.com/ton-org/kit-android/blob/334abc33260b9bb8b4628e544c5438fbce5f8817/AndroidDemo/app/src/main/java/io/ton/walletkit/demo/presentation/ui/screen/WalletScreen.kt#L161-L166) лежит новая личность базовой монеты:

```kotlin
    val tonAmount = trimFraction(rawBalance, maxFractionDigits)
    val tonItem = WalletHomeAssetItem(
        id = "ton",
        name = "Gram",
        symbol = "GRAM",
        formattedAmount = "$tonAmount GRAM",
```

Вот и все перевоплощение: внутреннее имя еще `ton`, переменная еще `tonAmount`, адрес все еще просит тоновский баланс, но на витрине уже стоит `Gram`. Сыщик видел подобные паспорта в дешевых романах, но не в кошельке.

## СВИДЕТЕЛЬ № 2: КАССИР ПЕРЕПИСАЛ ЦЕННИК

В том же [файле `WalletScreen.kt`](https://github.com/ton-org/kit-android/blob/334abc33260b9bb8b4628e544c5438fbce5f8817/AndroidDemo/app/src/main/java/io/ton/walletkit/demo/presentation/ui/screen/WalletScreen.kt) строка [544](https://github.com/ton-org/kit-android/blob/334abc33260b9bb8b4628e544c5438fbce5f8817/AndroidDemo/app/src/main/java/io/ton/walletkit/demo/presentation/ui/screen/WalletScreen.kt#L544) велит кошельку показывать общий остаток так:

```kotlin
                balanceSuffix = " GRAM",
```

Затем сыщик прошел в [файл `AndroidDemo/app/src/main/res/values/strings.xml`](https://github.com/ton-org/kit-android/blob/334abc33260b9bb8b4628e544c5438fbce5f8817/AndroidDemo/app/src/main/res/values/strings.xml). На строках [224–234](https://github.com/ton-org/kit-android/blob/334abc33260b9bb8b4628e544c5438fbce5f8817/AndroidDemo/app/src/main/res/values/strings.xml#L224-L234) касса уже выдает посетителю следующий разговор:

```xml
    <string name="transactions_amount_outgoing">-%1$s GRAM</string>
    <string name="transactions_amount_incoming">+%1$s GRAM</string>

    <!-- Send transaction -->
    <string name="send_title">Send GRAM</string>
    <string name="send_from_label">From</string>
    <string name="send_balance_format">Balance: %1$s GRAM</string>
    <string name="send_recipient_label">Recipient Address</string>
    <string name="send_recipient_placeholder">EQ... or UQ...</string>
    <string name="send_recipient_error">Invalid TON address</string>
    <string name="send_amount_label">Amount (GRAM)</string>
```

В одной комнате посетителю предлагают отправить GRAM, а в соседней сообщают, что адрес все еще *TON address*. Переименование прошло по ценникам, и только старый паспорт остался в кармане пальто.

## ПОЧЕМУ ГАЗОВЫЙ РОЖОК КАШЛЯЕТ

Это не рассказ о краже монет и не доказательство того, что сеть сменила название. Перед нами демонстрационное Android-приложение [ton-org/kit-android](https://github.com/ton-org/kit-android), а коммит [`334abc3`](https://github.com/ton-org/kit-android/commit/334abc33260b9bb8b4628e544c5438fbce5f8817) всего лишь массово меняет вывески. Но сцена до того чиста, что становится смешной: сущность с `id = "ton"` торжественно продают как `Gram`, переменную `tonAmount` оставляют на месте, а сообщение об ошибке продолжает жить в старой валюте.

Викторианский вывод прост. Если завтра в кошельке появится новая монета, сыщик сперва посмотрит не на герб, а на строки [161–166](https://github.com/ton-org/kit-android/blob/334abc33260b9bb8b4628e544c5438fbce5f8817/AndroidDemo/app/src/main/java/io/ton/walletkit/demo/presentation/ui/screen/WalletScreen.kt#L161-L166). Там, где имя уже GRAM, а паспорт еще `ton`, туман сгущается не над Темзой, а над релизной веткой.

*Выпуск подготовлен по открытому коду; цитаты дословные.*
