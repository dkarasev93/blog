+++
title = "№ 34 — Блокчейн в банке: MyLocalTon хранит мнемонику фаусета на миллион граммов прямо в исходниках, а JSON ковыряет стеком"
date = 2026-09-01T12:00:00+03:00
description = "Тридцать четвертый выпуск «Вечернего Валидатора»: в neodiX42/MyLocalTon найдены зашитые в код мнемоники фаусет-кошельков с балансом 1 000 001 грамм; два «разных» приватных ключа отличаются одним байтом; глобальный конфиг редактируется ручным парсером скобок; на вопрос «как обрезать последний символ» отвечают заклинанием substring(0, -1)."
tags = ["mylocalton"]
+++

# 📰 Вечерний Валидатор

**№ 34.** *Город. В подвале под конторой «ТОН-разработка» сыщик обнаружил консервную банку с этикеткой «собственный блокчейн, вскрыть перед запуском тестов». Банка называется [MyLocalTon](https://github.com/neodiX42/MyLocalTon) — 184 Java-файла, 27 тысяч строк, локальная сеть из валидаторов, DHT-серверов и фаусетов, упакованная в один jar. Сыщик вскрыл банку. Внутри обнаружено следующее.*

*Место осмотра: репозиторий [neodiX42/MyLocalTon](https://github.com/neodiX42/MyLocalTon), коммит [`bf54138`](https://github.com/neodiX42/MyLocalTon/commit/bf54138638633167f9e03a4bece8207bedbbfc7f) (master, 13 июня 2026). Файлы: [`DhtServer.java`](https://github.com/neodiX42/MyLocalTon/blob/bf54138638633167f9e03a4bece8207bedbbfc7f/src/main/java/org/ton/mylocalton/executors/dhtserver/DhtServer.java), [`MyLocalTonSettings.java`](https://github.com/neodiX42/MyLocalTon/blob/bf54138638633167f9e03a4bece8207bedbbfc7f/src/main/java/org/ton/mylocalton/settings/MyLocalTonSettings.java), [`MyLocalTonUtils.java`](https://github.com/neodiX42/MyLocalTon/blob/bf54138638633167f9e03a4bece8207bedbbfc7f/src/main/java/org/ton/mylocalton/utils/MyLocalTonUtils.java). Все цитаты дословные.*

---

## СЕНСАЦИЯ ПЕРВАЯ: МИЛЛИОН И ОДИН ГРАММ, КЛЮЧИ ПРИЛАГАЮТСЯ

На строке [186](https://github.com/neodiX42/MyLocalTon/blob/bf54138638633167f9e03a4bece8207bedbbfc7f/src/main/java/org/ton/mylocalton/settings/MyLocalTonSettings.java#L186) файла настроек сыщик нашел сейф, дверца которого распахнута настежь:

```java
BigInteger initialBalance = Utils.toNano(1_000_001);
String privateKey = "a51e8fb6f0fae3834bf430f5012589d319e7b3b3303ceb82c816b762fccf2d05";
String walletRawAddress = "-1:22f53b7d9aba2cef44755f7078b01614cd4dde2388a1729c2c386cf8f9898afe";
String mnemonic =
    "viable model canvas decade neck soap turtle asthma bench crouch bicycle grief history envelope valid intact invest like offer urban adjust popular draft coral";
```

Фаусет с балансом **1 000 001 грамм** — не миллион ровно, а миллион и один, с бухгалтерской аккуратностью человека, который знает, что последний грамм уйдет на комиссии. Приватный ключ и полная мнемоника из 24 слов лежат рядом, в том же файле, в публичном репозитории. Сыщик проверил: это не пример и не заглушка — это рабочие дефолты, с которыми поднимается каждая локальная сеть.

Справедливости ради: сеть локальная, граммы нарисованные, ключи ни к чему не привязаны. Но осадочек остался: привычка хранить мнемоники в `Settings.java` — она заразна, как привычка не мыть руки.

## СЕНСАЦИЯ ВТОРАЯ: БЛИЗНЕЦЫ, РАЗЛУЧЕННЫЕ ОДНИМ БАЙТОМ

Ниже по файлу — второй фаусет, highload. Сыщик сверил отпечатки:

```java
// FaucetWalletSettings:
String privateKey = "a51e8fb6f0fae3834bf430f5012589d319e7b3b3303ceb82c816b762fccf2d05";
// FaucetHighloadWalletSettings:
String privateKey = "e1480435871753a968ef08edabb24f5532dab4cd904dbdc683b8576fb45fa697";
// FaucetDataWalletSettings:
String privateKey = "f2480435871753a968ef08edabb24f5532dab4cd904dbdc683b8576fb45fa697";
```

Второй и третий ключи идентичны, кроме **первого байта**: `e148…` и `f248…`. Двадцать девять байтов из тридцати двух — общие. Это не генерация ключей, это фотокопия с подправленной цифрой в углу. Криптографически, конечно, другой ключ. По духу — тот же самый, только в шляпе.

## СЕНСАЦИЯ ТРЕТЬЯ: JSON, РАЗБИРАЕМЫЙ ГАЕЧНЫМ КЛЮЧОМ

Главная улика вечера — функция, ради которой сыщика и позвали: [`addDhtNodesToGlobalConfig`](https://github.com/neodiX42/MyLocalTon/blob/bf54138638633167f9e03a4bece8207bedbbfc7f/src/main/java/org/ton/mylocalton/executors/dhtserver/DhtServer.java#L152) (DhtServer.java:152). Ее задача — вписать свежие DHT-ноды в глобальный конфиг сети. Конфиг — JSON. В проекте есть полноценные JSON-библиотеки. Но нет:

```java
if (globalConfigContent.contains("\"nodes\": []")) { // very first creation
      ...
      String replaced =
          StringUtils.replace(
              globalConfigContent,
              "\"nodes\": []",
              "\"nodes\": [" + String.join(",", dhtNodes) + "]");
```

JSON редактируется **строковой заменой по вхождению** `"nodes": []`. А если ноды уже есть — в ход идет ручной парсер скобок из `MyLocalTonUtils.sbb()` (строка [574](https://github.com/neodiX42/MyLocalTon/blob/bf54138638633167f9e03a4bece8207bedbbfc7f/src/main/java/org/ton/mylocalton/utils/MyLocalTonUtils.java#L574)), который находит кусок конфига от `"nodes": [` до закрывающей скобки, балансируя их **стеком**:

```java
ArrayDeque<Integer> st = new ArrayDeque<>();
for (i = index; i < str.length(); i++) {
  if (str.charAt(i) == '[') {
    st.push((int) str.charAt(i));
  } else if (str.charAt(i) == ']') {
    st.pop();
    if (st.isEmpty()) {
      return i;
```

Парсинг JSON стеком на `ArrayDeque<Integer>` в 2026 году — это как вскрывать сейф отмычкой, когда ключ лежит у тебя в кармане. Апогей же — строка [179](https://github.com/neodiX42/MyLocalTon/blob/bf54138638633167f9e03a4bece8207bedbbfc7f/src/main/java/org/ton/mylocalton/executors/dhtserver/DhtServer.java#L179), где нужно отрезать последний символ (ту самую `]`), чтобы дописать ноды:

```java
StringUtils.substring(existingNodes, 0, -1) + "," + String.join(",", dhtNodes) + "]"
```

`substring(0, -1)`. Минус один. Сыщик поначалу записал это как «выход за границы строки» и уже потирал руки — но нет: в Apache Commons Lang отрицательный индекс означает «от конца строки». Код **рабочий**. Он просто написан так, что любой ревьюер обязан споткнуться, полезть в документацию и вернуться с чувством, что его разыграли. Сыщик оценил: легальная ловушка для коллег, замаскированная под баг.

---

## ХРОНИКА МЕЛКИХ ПРОИСШЕСТВИЙ

*Отдел перезагрузок.* Сброс блокчейна из UI ([ResetBlockchainPaneController.java:34](https://github.com/neodiX42/MyLocalTon/blob/bf54138638633167f9e03a4bece8207bedbbfc7f/src/main/java/org/ton/mylocalton/ui/custom/layout/ResetBlockchainPaneController.java#L34)) на Windows выглядит так: `Runtime.getRuntime().exec("cmd /c start java -jar " + MyLocalTonUtils.getMyPath() + " restart")` — приложение заказывает само себя в новом окне и честно умирает через `System.exit(0)`. Цепочка из четырех посредников: Java → cmd → start → java.

*Отдел установщиков.* ton-http-api ставится прямо из рантайма: `pip3 install --user ton-http-api`, затем `p.waitFor(30, TimeUnit.SECONDS)` и сразу `p.exitValue()` ([MyLocalTonUtils.java:1283](https://github.com/neodiX42/MyLocalTon/blob/bf54138638633167f9e03a4bece8207bedbbfc7f/src/main/java/org/ton/mylocalton/utils/MyLocalTonUtils.java#L1283)). Если pip копается дольше 30 секунд — а pip копается дольше 30 секунд — `exitValue()` бросит `IllegalThreadStateException`, ведь процесс еще жив. Таймаут, который наказывает за свое срабатывание.

*Отдел лингвистики.* DataDB.java:178 сообщает: `log.error("cant add request - wallet {} found", walletAddr)`. Кошелек *найден* — и это ошибка. Пропавшее «not» сыщик так и не разыскал.

*Отдел сценарного искусства.* В пакете `data/scenarios/` обнаружены `Scenario.java`, `Scenario1.java` … `Scenario16.java` — семнадцать сценариев, пронумерованных как сезоны сериала, прямо в основном коде приложения. Нумерация вместо имен — табличка «осторожно, злая собака» без указания, что за собака.

*Отдел архитектуры.* Самый крупный файл проекта — `MainController.java`, 4439 строк. Это не контроллер, это целый район: кнопки, валидаторы, кошельки и блоки живут в нем коммуналкой. Второй по величине — `MyLocalTon.java`, 2342 строки. Всего в проекте 207 закомментированных строк кода, 44 вызова `Thread.sleep` и 17 `printStackTrace()` — диагностическая система уровня «постучать по прибору».

*Отдел складского учета.* В корне репозитория, рядом с `pom.xml`, лежат бинарники: `objectdb-2.8.6.jar` и `ant-contrib-1.0b3.jar`. Maven смотрит на это с неодобрением, но молчит — в Maven же все равно никто не читает вслух.

---

## ВЕРДИКТ РЕДАКЦИИ

MyLocalTon — честный рабочий инструмент: поднимает локальную сеть, генерит блоки, дает фаусет на миллион и один грамм и не притворяется продакшеном. Зашитые ключи тут — не дыра, а дизайн: сеть нарисованная, деньги нарисованные. Но манера исполнения говорит сама за себя: JSON ковыряется стеком, ключи клонируются правкой одного байта, перезапуск устроен через четыре уровня шелла, а ошибка про найденный кошелек сообщает, что кошелек найден. Консервная банка вскрыта, содержимое годится в пищу, этикетка врет местами.

*Сыщик записал показания, аккуратно закрыл банку и поставил обратно на полку — пусть тесты бегают дальше.*
