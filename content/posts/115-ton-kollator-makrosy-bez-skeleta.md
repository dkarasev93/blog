---
title: "Выпуск № 115: Макросы, которые вышли без скелета"
date: 2026-10-28T17:00:00+03:00
tags: [ton]
---

# Вечерний Валидатор

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

## ДЕЛО О КОЛЛАТОРЕ, КОТОРЫЙ ПРИНЯЛ МАКРОС ЗА СТЕНУ

Лондон тонул в тумане, когда сыщик «Вечернего Валидатора» получил папку из [ton-blockchain/ton](https://github.com/ton-blockchain/ton). На обложке стоял коммит [ccc85d6](https://github.com/ton-blockchain/ton/commit/ccc85d6a09a5dc100e88128f09c6c2d4e89b863f) с сухим названием `Various changes in node`. Внутри, среди сетевых перемен и валидаторской алхимии, обнаружилась новая охрана коллатора: три макроса, каждый без скобяного каркаса, зато с правом вернуть из функции.

Газовый рожок кашлянул. Сыщик поднял воротник: в приличном C++ макрос, который изображает оператор, обычно прячется в `do { ... } while (false)`. Здесь сторож поставлен прямо в коридоре, без ограды.

## МЕСТО ПРОИСШЕСТВИЯ

Улика лежит в коммите [ccc85d6](https://github.com/ton-blockchain/ton/commit/ccc85d6a09a5dc100e88128f09c6c2d4e89b863f), в файле [validator/impl/collator.cpp](https://github.com/ton-blockchain/ton/blob/ccc85d6a09a5dc100e88128f09c6c2d4e89b863f/validator/impl/collator.cpp), строки [44–66](https://github.com/ton-blockchain/ton/blob/ccc85d6a09a5dc100e88128f09c6c2d4e89b863f/validator/impl/collator.cpp#L44-L66). Цитата дословная:

```cpp
#define FAIL_UNLESS_MSG(condition, msg) \\
  if (!(condition)) {                   \\
    fatal_error(msg);                   \\
    return {};                          \\
  }
#define FAIL_UNLESS(condition) \\
  FAIL_UNLESS_MSG(condition, PSTRING() << "Check failed at line " << __LINE__ << ": " << #condition)

#define FAIL_UNLESS_ERRCODE_MSG(condition, err, msg) \\
  if (!(condition)) {                                \\
    fatal_error(msg);                                \\
    return (err);                                    \\
  }
#define FAIL_UNLESS_ERRCODE(condition, err) \\
  FAIL_UNLESS_ERRCODE_MSG(condition, err, PSTRING() << "Check failed at line " << __LINE__ << ": " << #condition)

#define FAIL_UNLESS_MSG_VOID(condition, msg) \\
  if (!(condition)) {                        \\
    fatal_error(msg);                        \\
    return;                                  \\
  }
#define FAIL_UNLESS_VOID(condition) \\
  FAIL_UNLESS_MSG_VOID(condition, PSTRING() << "Check failed at line " << __LINE__ << ": " << #condition)
```

Три вывески, один прием: раскрыться в голый `if`, вызвать `fatal_error`, а потом выйти из текущей функции. Макросы обещают проверку, но фактически подменяют часть управления потоком. Сыщик записал в блокнот: дверь выглядит как выражение, а за дверью уже стоит досрочный выход.

## ПЕРВЫЙ СЛЕД: ПОЧЕМУ СКОБЫ НЕ РОСКОШЬ

В том же файле макрос применяется как самостоятельная команда. Например, проверка единственного предыдущего блока стоит на строках [202–204](https://github.com/ton-blockchain/ton/blob/ccc85d6a09a5dc100e88128f09c6c2d4e89b863f/validator/impl/collator.cpp#L202-L204), а проверка занятости — на строках [275–278](https://github.com/ton-blockchain/ton/blob/ccc85d6a09a5dc100e88128f09c6c2d4e89b863f/validator/impl/collator.cpp#L275-L278). Цитата со второй сцены дословная:

```cpp
alarm_timestamp() = timeout_;
FAIL_UNLESS_VOID(pending);
```

В этих местах спектакль не падает: макрос стоит отдельной строкой. Но он не является безопасным составным оператором. Если такой шаблон однажды поставить после `if` без фигурных скобок, внешний `else` сможет связаться с внутренним `if` из макроса. Это старый лондонский фокус: невидимая часть текста меняет смысл видимой конструкции.

Проблема не в том, что C++ не умеет читать макросы. Проблема в том, что читатель читает вызов `FAIL_UNLESS_VOID(...)` как единое действие, а препроцессор подсовывает ему еще один условный оператор. Сыщик проверил карманы: `do { ... } while (false)` в этом месте не найден.

## ВТОРОЙ СЛЕД: ПРИГОВОР С ПУСТЫМ ВОЗВРАТОМ

Особенно театрально выглядит [FAIL_UNLESS](https://github.com/ton-blockchain/ton/blob/ccc85d6a09a5dc100e88128f09c6c2d4e89b863f/validator/impl/collator.cpp#L49-L50): он всегда передает в макрос `return {}`. Такой выход подходит только функциям, где пустая инициализация действительно означает отказ. Рядом живет версия [FAIL_UNLESS_ERRCODE](https://github.com/ton-blockchain/ton/blob/ccc85d6a09a5dc100e88128f09c6c2d4e89b863f/validator/impl/collator.cpp#L52-L58), а для `void` есть отдельная дверь на строках [60–66](https://github.com/ton-blockchain/ton/blob/ccc85d6a09a5dc100e88128f09c6c2d4e89b863f/validator/impl/collator.cpp#L60-L66).

Получилась маленькая контора с тремя сортами молчаливого ухода: пустая вещь, код ошибки и ничего. Снаружи все называется `FAIL_UNLESS`, будто это единый приговор. Внутри каждый вариант меняет судьбу функции по своему тарифу.

Редакция не утверждает, что именно этот фрагмент уже сломал выпуск блока. Такой вывод потребовал бы отдельного запуска и анализа всех мест раскрытия макросов. Но конструкция уже оставила опасную улику: управление потоком спрятано в макросе, а форма макроса не защищена от соседнего `else`.

## ПРИГОВОР СЫЩИКА

[ton-blockchain/ton](https://github.com/ton-blockchain/ton) пойман в коммите [ccc85d6](https://github.com/ton-blockchain/ton/commit/ccc85d6a09a5dc100e88128f09c6c2d4e89b863f) на старом фокусе: назвать кусок управления потоком проверкой и выпустить его на улицу без скобок. Файл [validator/impl/collator.cpp](https://github.com/ton-blockchain/ton/blob/ccc85d6a09a5dc100e88128f09c6c2d4e89b863f/validator/impl/collator.cpp), строки [44–66](https://github.com/ton-blockchain/ton/blob/ccc85d6a09a5dc100e88128f09c6c2d4e89b863f/validator/impl/collator.cpp#L44-L66): три макроса стоят в тумане и делают вид, что они обычные команды.

Рецепт сыщика скучен, зато надежен: завернуть тело каждого макроса в `do { ... } while (false)` либо заменить их на настоящие функции и явные проверки. Газовый рожок пробил сто пятнадцатый удар. Коллатор продолжил считать блоки, а макросы отправились на допрос к препроцессору.

🐀
