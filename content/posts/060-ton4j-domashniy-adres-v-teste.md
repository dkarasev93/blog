---
title: "Выпуск № 60: Тест, который заблудился в чужом доме"
date: 2026-09-13T16:59:00+03:00
tags: [ton4j]
---

# ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

## ДЕЛО О ФАЙЛЕ, КОТОРЫЙ ЖДЕТ ЧУЖУЮ КВАРТИРУ

Лондон тонул в вечернем тумане, когда сыщик «Вечернего Валидатора» получил папку из [ton-blockchain/ton4j](https://github.com/ton-blockchain/ton4j). Контора выпускает Java-инструменты для разговора с TON, а значит, ее тестам полагается знать дорогу к библиотекам не хуже, чем констеблю — дорогу к Скотланд-Ярду.

Но на стол лег коммит [`338c6b3`](https://github.com/ton-blockchain/ton4j/commit/338c6b3851f841aef5a0128f6624354de88c3bae) от 13 июня 2026 года с вывеской `rebrand to back to gram`. Среди переименований граммов и старых тонкоинов сыщик нашел улику, которая пахнет не блокчейном, а личным домашним каталогом.

## МЕСТО ПРОИСШЕСТВИЯ

Дело лежит в коммите [`338c6b3`](https://github.com/ton-blockchain/ton4j/commit/338c6b3851f841aef5a0128f6624354de88c3bae), файл [`tonlib/src/test/java/org/ton/ton4j/tonlib/TestTonlibJson.java`](https://github.com/ton-blockchain/ton4j/blob/338c6b3851f841aef5a0128f6624354de88c3bae/tonlib/src/test/java/org/ton/ton4j/tonlib/TestTonlibJson.java), строки [63–68](https://github.com/ton-blockchain/ton4j/blob/338c6b3851f841aef5a0128f6624354de88c3bae/tonlib/src/test/java/org/ton/ton4j/tonlib/TestTonlibJson.java#L63-L68):

```java
  @Test
  public void testIssue13() {
    Tonlib tonlib =
        Tonlib.builder()
            .pathToTonlibSharedLib(tonlibPath)
            .pathToGlobalConfig("/home/neodix/gitProjects/global-config-archive.json")
            .build();
```

Цитата дословная. Тест не просит конфигурацию из ресурсов проекта, не берет ее из переменной среды и не принимает путь через настройку. Он уверенно идет по адресу `/home/neodix/gitProjects/global-config-archive.json`, словно каждый клон [ton-blockchain/ton4j](https://github.com/ton-blockchain/ton4j) обязан жить в прихожей господина Neodix.

## ПЕРВЫЙ СЛЕД: ДОМАШНИЙ АДРЕС В КАЗЕННОМ ДЕЛЕ

Имя теста — `testIssue13`, строка [63](https://github.com/ton-blockchain/ton4j/blob/338c6b3851f841aef5a0128f6624354de88c3bae/tonlib/src/test/java/org/ton/ton4j/tonlib/TestTonlibJson.java#L63). Дальше строка [67](https://github.com/ton-blockchain/ton4j/blob/338c6b3851f841aef5a0128f6624354de88c3bae/tonlib/src/test/java/org/ton/ton4j/tonlib/TestTonlibJson.java#L67) выдает тайну: глобальный конфиг должен находиться в чужой домашней папке, внутри каталога `gitProjects`.

Это не абстрактный пример вроде `/path/to/config`. Это полноценный личный маршрут с именем пользователя. На машине автора он мог быть совершенно разумен. На машине коллеги, в CI или у любого читателя тест превращается в вежливый стук в стену, за которой нет дома.

## ВТОРОЙ СЛЕД: СТРОИТЕЛЬ ДОШЕЛ, А ДОМ — НЕТ

Строка [66](https://github.com/ton-blockchain/ton4j/blob/338c6b3851f841aef5a0128f6624354de88c3bae/tonlib/src/test/java/org/ton/ton4j/tonlib/TestTonlibJson.java#L66) еще использует переменную `tonlibPath`, то есть путь к общей библиотеке уже вынесен в настройку теста. Но строка [67](https://github.com/ton-blockchain/ton4j/blob/338c6b3851f841aef5a0128f6624354de88c3bae/tonlib/src/test/java/org/ton/ton4j/tonlib/TestTonlibJson.java#L67) для глобального конфига выбирает жесткий адрес.

В итоге один и тот же строитель `Tonlib.builder()` получает два разных мира: первый можно подать снаружи, второй велит пройти в конкретную комнату конкретного жильца. Коммит [`338c6b3`](https://github.com/ton-blockchain/ton4j/commit/338c6b3851f841aef5a0128f6624354de88c3bae) меняет множество имен вокруг `toncoins` и `grams`, но этот домашний адрес остается стоять на посту, как дворецкий, который признает только один паспорт.

## ТРЕТИЙ СЛЕД: ПОЧЕМУ ЭТО СОЧНО

Сломанная ссылка в документации — неприятность. Личный абсолютный путь внутри теста — уже маленькая пьеса о том, как локальная машина выдает себя за весь мир. Код выглядит готовым к публикации: аннотация `@Test`, строитель, вызов `getLast()`, журналирование блока. Но на строке [67](https://github.com/ton-blockchain/ton4j/blob/338c6b3851f841aef5a0128f6624354de88c3bae/tonlib/src/test/java/org/ton/ton4j/tonlib/TestTonlibJson.java#L67) тест оставляет записку: «ищите файл там, где живу я».

Редакция не станет считать [ton-blockchain/ton4j](https://github.com/ton-blockchain/ton4j) негодной конторой. Перенос пути в ресурс тестов, параметр сборки или переменную среды решает дело без драматического вмешательства. Но публичный тест обязан быть гостеприимнее частной квартиры.

## ВЕРДИКТ СЫЩИКА

[ton-blockchain/ton4j](https://github.com/ton-blockchain/ton4j) пойман на простой, но сочной улике: в коммите [`338c6b3`](https://github.com/ton-blockchain/ton4j/commit/338c6b3851f841aef5a0128f6624354de88c3bae) файл [`tonlib/src/test/java/org/ton/ton4j/tonlib/TestTonlibJson.java`](https://github.com/ton-blockchain/ton4j/blob/338c6b3851f841aef5a0128f6624354de88c3bae/tonlib/src/test/java/org/ton/ton4j/tonlib/TestTonlibJson.java) отправляет `pathToGlobalConfig` прямо в `/home/neodix/gitProjects/global-config-archive.json`.

Приговор мягок: тесту пора выдать карту города, а не ключ от чужой двери. Сыщик сверил строки [63–68](https://github.com/ton-blockchain/ton4j/blob/338c6b3851f841aef5a0128f6624354de88c3bae/tonlib/src/test/java/org/ton/ton4j/tonlib/TestTonlibJson.java#L63-L68), погас газовый рожок и растворился в тумане. В Лондоне можно потерять адрес. Но хуже, когда адрес потерять невозможно — он навечно зашит в тест.
