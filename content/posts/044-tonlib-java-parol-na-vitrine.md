+++
title = "№ 44 — Дело о пароле на витрине: tonlib-java кормит кошельки одним словом"
date = 2026-09-04T17:00:27+03:00
description = "Сорок четвертый выпуск «Вечернего Валидатора»: в Java-тесте tonlib-java один и тот же пароль local password раздается двум ключам, а рядом лежит mnemonic password и перевод на 6.66 TON."
tags = ["tonlib-java"]
+++

# 📰 ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

**№ 44 · Пятница, 4 сентября 2026 г. · Цена: 0.05 TON (пароль напечатан крупным шрифтом)**

---

## ДЕЛО О ПАРОЛЕ, КОТОРЫЙ ЗНАЛИ ВСЕ

Лондон тонул в вечернем тумане, газовый рожок у редакции сипел, а сыщик «Вечернего Валидатора» получил папку из мастерской [ton-blockchain/tonlib-java](https://github.com/ton-blockchain/tonlib-java). Это Java-обвязка для tonlib, серьезного инструмента общения с TON. Внутри папки лежал тест, где охрана ключей устроена по принципу: один пароль на весь участок, и тот уже выведен на доску для всех.

Место происшествия: репозиторий [ton-blockchain/tonlib-java](https://github.com/ton-blockchain/tonlib-java), коммит [`49b228f`](https://github.com/ton-blockchain/tonlib-java/commit/49b228fe710cd742e82012875ea5c09a177b1904), файл [`src/drinkless/org/ton/TonTestJava.java`](https://github.com/ton-blockchain/tonlib-java/blob/49b228fe710cd742e82012875ea5c09a177b1904/src/drinkless/org/ton/TonTestJava.java), строки [90–99](https://github.com/ton-blockchain/tonlib-java/blob/49b228fe710cd742e82012875ea5c09a177b1904/src/drinkless/org/ton/TonTestJava.java#L90-L99). Цитата сверена с содержимым файла на этом коммите.

Протокол, строки [90–99](https://github.com/ton-blockchain/tonlib-java/blob/49b228fe710cd742e82012875ea5c09a177b1904/src/drinkless/org/ton/TonTestJava.java#L90-L99):

```java
        TonApi.Key key = (TonApi.Key) client.send(new TonApi.CreateNewKey("local password".getBytes(), "mnemonic password".getBytes(), "".getBytes()));
        TonApi.InputKey inputKey = new TonApi.InputKeyRegular(key, "local password".getBytes());
        TonApi.AccountAddress walletAddress = (TonApi.AccountAddress)client.send(new TonApi.GetAccountAddress(new TonApi.WalletV3InitialAccountState(key.publicKey, info.configInfo.defaultWalletId), 1, 0));

        TonApi.Key giverKey = (TonApi.Key)client.send(new TonApi.ImportKey("local password".getBytes(), "".getBytes(), new TonApi.ExportedKey(words))) ;
        TonApi.InputKey giverInputKey = new TonApi.InputKeyRegular(giverKey, "local password".getBytes());
        TonApi.AccountAddress giverAddress = (TonApi.AccountAddress)client.send(new TonApi.GetAccountAddress(new TonApi.WalletV3InitialAccountState(giverKey.publicKey, info.configInfo.defaultWalletId), 1, 0));

        appendLog("sending coins...");
        TonApi.QueryInfo queryInfo = (TonApi.QueryInfo)client.send(new TonApi.CreateQuery(giverInputKey, giverAddress, 60, new TonApi.ActionMsg(new TonApi.MsgMessage[]{new TonApi.MsgMessage(walletAddress, "", 6660000000L, new TonApi.MsgDataText("Hello".getBytes()) )}, true), new TonApi.WalletV3InitialAccountState(giverKey.publicKey, info.configInfo.defaultWalletId)));
```

На строке [90](https://github.com/ton-blockchain/tonlib-java/blob/49b228fe710cd742e82012875ea5c09a177b1904/src/drinkless/org/ton/TonTestJava.java#L90) рождается новый ключ. Для него торжественно названы две строки: `local password` и `mnemonic password`. Первая выглядит как пароль для местного сейфа, вторая — как пароль для мнемоники. Но это не переменные окружения и не секреты тестового раннера, а открытые литералы прямо в исходнике.

Дальше сыщик заметил странную семейную черту. Строка [91](https://github.com/ton-blockchain/tonlib-java/blob/49b228fe710cd742e82012875ea5c09a177b1904/src/drinkless/org/ton/TonTestJava.java#L91) снова передает `local password` в `InputKeyRegular`. Пароль не ищут в защищенном хранилище и не получают от теста. Он стоит рядом с созданием ключа, как дворецкий, который сам выкрикивает код от сейфа.

## ОДИН КОД НА ДВА КОШЕЛЬКА

Если бы на этом все закончилось, мы имели бы обычную учебную заглушку. Но строка [94](https://github.com/ton-blockchain/tonlib-java/blob/49b228fe710cd742e82012875ea5c09a177b1904/src/drinkless/org/ton/TonTestJava.java#L94) импортирует второй ключ, `giverKey`, и вновь получает тот же самый `local password`. Строка [95](https://github.com/ton-blockchain/tonlib-java/blob/49b228fe710cd742e82012875ea5c09a177b1904/src/drinkless/org/ton/TonTestJava.java#L95) закрепляет пароль за этим ключом уже для последующего запроса.

Итак, в комнате два персонажа: обычный `key` и ключ дарителя `giverKey`. У каждого собственный адрес, созданный на строках [92](https://github.com/ton-blockchain/tonlib-java/blob/49b228fe710cd742e82012875ea5c09a177b1904/src/drinkless/org/ton/TonTestJava.java#L92) и [96](https://github.com/ton-blockchain/tonlib-java/blob/49b228fe710cd742e82012875ea5c09a177b1904/src/drinkless/org/ton/TonTestJava.java#L96). А ключ от обоих ящиков один и тот же, причем лежит в протоколе.

Редакция не утверждает, что этот пароль открывает реальные средства. Перед нами тестовый сценарий, а слова могут быть частью локальной демонстрации. Но учебный код имеет дурную привычку переселяться в копипасту. Строка `local password` выглядит не как предупреждение «замените меня», а как готовый рецепт. Читатель может решить, что так и надо строить постоянное хранилище ключей.

Еще живописнее выглядит соседство с пустой строкой на строке [94](https://github.com/ton-blockchain/tonlib-java/blob/49b228fe710cd742e82012875ea5c09a177b1904/src/drinkless/org/ton/TonTestJava.java#L94): импорт получает `"".getBytes()` в качестве пароля мнемоники. Один секрет назван слишком просто, другой отсутствует вовсе. Газовый рожок кашлянул: «Господа, это не защита, а гардероб без номерков!»

## КАРЕТА ДЛЯ ПЕРЕВОДА

Сыщик перелистнул протокол до строки [98](https://github.com/ton-blockchain/tonlib-java/blob/49b228fe710cd742e82012875ea5c09a177b1904/src/drinkless/org/ton/TonTestJava.java#L98). Там тест сообщает, что собирается отправлять монеты. На следующей строке [99](https://github.com/ton-blockchain/tonlib-java/blob/49b228fe710cd742e82012875ea5c09a177b1904/src/drinkless/org/ton/TonTestJava.java#L99) `giverInputKey` создает запрос с переводом `6660000000L` нанотонов — то есть 6.66 TON — на адрес кошелька.

Получается цельная сцена, а не мертвый пример API: сначала создается ключ, потом импортируется ключ дарителя, затем оба получают один открытый пароль, а после этого даритель готовит перевод. Никакой злой умысел из этого не следует, но постановка щедрая до нелепости. Тест показывает не только вызов библиотеки, но и привычку хранить пароль в коде, повторять его для разных ключей и соседствовать с пустой строкой.

Тут важно разделить две вещи. Для автоматического теста допустимы предсказуемые данные: иначе проверка не будет воспроизводимой. Но предсказуемость не требует печатать пароль в каждом вызове. Можно использовать отдельные тестовые значения, собрать их в одном месте и крупно пометить, что они не годятся для настоящего кошелька. В нынешней сцене читателю выдают ключ, адрес и карету, а табличка «только для полигона» потерялась в тумане.

## ВЕРДИКТ СЫЩИКА

[ton-blockchain/tonlib-java](https://github.com/ton-blockchain/tonlib-java) — полезный мост между Java и tonlib, не злодейская контора. Но коммит [`49b228f`](https://github.com/ton-blockchain/tonlib-java/commit/49b228fe710cd742e82012875ea5c09a177b1904) оставил сочную улику в файле [`src/drinkless/org/ton/TonTestJava.java`](https://github.com/ton-blockchain/tonlib-java/blob/49b228fe710cd742e82012875ea5c09a177b1904/src/drinkless/org/ton/TonTestJava.java): на строках [90–95](https://github.com/ton-blockchain/tonlib-java/blob/49b228fe710cd742e82012875ea5c09a177b1904/src/drinkless/org/ton/TonTestJava.java#L90-L95) пароль `local password` обслуживает два ключа, рядом стоит `mnemonic password`, а импорт оставляет пароль мнемоники пустым. На строке [99](https://github.com/ton-blockchain/tonlib-java/blob/49b228fe710cd742e82012875ea5c09a177b1904/src/drinkless/org/ton/TonTestJava.java#L99) тем временем готовится перевод на 6.66 TON.

Приговор редакции прост: тестовые пароли должны быть собраны в явные фикстуры, помечены как ненастоящие и не подаваться как образец хранения ключей. Один пароль на несколько кошельков — плохая привычка даже в викторианском сейфе. А пустой пароль рядом с операцией импорта пусть хотя бы получает табличку, чтобы новичок не унес этот сувенир в продакшен.

*Сыщик закрыл папку, накрыл слова `local password` черной бумагой и погас газовую лампу. В Лондоне сейф может быть учебным. Код от него все же не стоит печатать на первой полосе.*

🐀
