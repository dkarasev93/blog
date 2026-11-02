---
title: "№ 123 — Вектор, который сам признал себя очень глупым"
date: 2026-11-02T11:00:00+03:00
tags: [ton]
---

# ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

## ДЕЛО О ГЛУПОЙ ОЧЕРЕДИ

Лондон тонул в тумане, когда сыщик «Вечернего Валидатора» получил папку из [ton-blockchain/ton](https://github.com/ton-blockchain/ton). Внутри лежал не новый протокол, а честная записка из машинного подвала: в ядре сети кто-то назвал собственную многопоточную конструкцию очень глупой, а затем пустил ее работать с ячейками.

Газовый рожок кашлянул. В таких делах нельзя судить по названию: иногда самоуничижение — это скромность, а иногда оно просто точная техническая документация.

## МЕСТО ПРОИСШЕСТВИЯ

След ведет в [коммит 1c131db](https://github.com/ton-blockchain/ton/commit/1c131db450011491771dea600f1fde545571b317), в файл [crypto/vm/db/DynamicBagOfCellsDbV2.cpp](https://github.com/ton-blockchain/ton/blob/1c131db450011491771dea600f1fde545571b317/crypto/vm/db/DynamicBagOfCellsDbV2.cpp), строки [17–18](https://github.com/ton-blockchain/ton/blob/1c131db450011491771dea600f1fde545571b317/crypto/vm/db/DynamicBagOfCellsDbV2.cpp#L17-L18). Цитата дословная:

```cpp
// Very stupid Vector/MpmcQueue
template <class T>
```

На [строке 17](https://github.com/ton-blockchain/ton/blob/1c131db450011491771dea600f1fde545571b317/crypto/vm/db/DynamicBagOfCellsDbV2.cpp#L17) автор не оставил читателю ни тумана, ни юридической защиты. Это не «экспериментальный контейнер», не «легковесная очередь» и не «новая архитектура хранения». Это *Very stupid Vector/MpmcQueue* — вежливое признание, сделанное до того, как его успел сделать ревьюер.

## КАК РАБОТАЕТ ПОДОЗРЕВАЕМЫЙ

В [строках 19–29](https://github.com/ton-blockchain/ton/blob/1c131db450011491771dea600f1fde545571b317/crypto/vm/db/DynamicBagOfCellsDbV2.cpp#L19-L29) конструкция заводит первый блок, кладет туда вектор и помечает его готовым. Далее [строки 35–58](https://github.com/ton-blockchain/ton/blob/1c131db450011491771dea600f1fde545571b317/crypto/vm/db/DynamicBagOfCellsDbV2.cpp#L35-L58) вычисляют блок по старшим битам индекса, проверяют флаг готовности и при необходимости создают очередную порцию данных под мьютексом.

С виду это похоже на аккуратную лестницу: первый пролет, затем блоки крупнее, потом атомарный флаг. Но вывеска уже испортила впечатление. Когда класс называет себя глупым, любой следующий сдвиг бита звучит как признание в соучастии.

## СВИДЕТЕЛЬ, КОТОРЫЙ НЕ УВЕРЕН

Самая сочная улика обнаружена в [строках 64–70](https://github.com/ton-blockchain/ton/blob/1c131db450011491771dea600f1fde545571b317/crypto/vm/db/DynamicBagOfCellsDbV2.cpp#L64-L70):

```cpp
T pop_front() {
  auto pos = begin_.fetch_add(1, std::memory_order_relaxed);
  while (pos >= end_.load(std::memory_order_acquire)) {
    // This may (or may not) use too much CPU
    td::this_thread::yield();
  }
  return std::move(at(pos));
}
```

На [строке 67](https://github.com/ton-blockchain/ton/blob/1c131db450011491771dea600f1fde545571b317/crypto/vm/db/DynamicBagOfCellsDbV2.cpp#L67) программа признается: ожидание «может (а может и нет)» потребить слишком много процессора. Это не совсем комментарий и не совсем прогноз. Это маленький метеорологический бюллетень для CPU: туман будет, но синоптик не ручается.

Сыщик не станет выдавать эту фразу за доказанный баг. Она честно описывает сомнение автора, а код ниже действительно уступает управление потоком через `yield`. Но для газеты важен сам контраст: в одном файле живет самоуверенная атомарная арифметика и осторожное «а может, процессор перегреется».

## ВТОРАЯ ОЧЕРЕДЬ В ПОДВАЛЕ

Для сравнения сыщик заглянул в соседний бенчмарк того же [коммита 1c131db](https://github.com/ton-blockchain/ton/commit/1c131db450011491771dea600f1fde545571b317), файл [tdactor/benchmark/benchmark.cpp](https://github.com/ton-blockchain/ton/blob/1c131db450011491771dea600f1fde545571b317/tdactor/benchmark/benchmark.cpp). Там в [строке 1208](https://github.com/ton-blockchain/ton/blob/1c131db450011491771dea600f1fde545571b317/tdactor/benchmark/benchmark.cpp#L1208) встречается класс `StupidQueue`, а его описание на [строках 1212–1214](https://github.com/ton-blockchain/ton/blob/1c131db450011491771dea600f1fde545571b317/tdactor/benchmark/benchmark.cpp#L1212-L1214) возвращает обычное `Mutex queue`.

Но эту улику редакция не смешивает с главным делом: бенчмарк — другая сцена, другой класс и другой сюжет. В сегодняшнем выпуске важна именно внутренняя очередь базы ячеек, которая назвала себя глупой, а затем призналась, что не знает, сколько CPU поглотит ее ожидание.

## ПРИГОВОР СЫЩИКА

[ton-blockchain/ton](https://github.com/ton-blockchain/ton) в [коммите 1c131db](https://github.com/ton-blockchain/ton/commit/1c131db450011491771dea600f1fde545571b317) пойман не на падении сети, а на редкой форме технической откровенности. В [crypto/vm/db/DynamicBagOfCellsDbV2.cpp](https://github.com/ton-blockchain/ton/blob/1c131db450011491771dea600f1fde545571b317/crypto/vm/db/DynamicBagOfCellsDbV2.cpp) строки [17–18](https://github.com/ton-blockchain/ton/blob/1c131db450011491771dea600f1fde545571b317/crypto/vm/db/DynamicBagOfCellsDbV2.cpp#L17-L18) дают контейнеру кличку «очень глупый», а строка [67](https://github.com/ton-blockchain/ton/blob/1c131db450011491771dea600f1fde545571b317/crypto/vm/db/DynamicBagOfCellsDbV2.cpp#L67) оставляет сомнение насчет аппетита CPU.

Вердикт прост: если код способен сам написать на двери «я очень глупый», не надо заставлять его притворяться джентльменом. Пусть ревьюер сначала проверит индексы, мьютексы и атомарность, а потом уже снимет шляпу перед такой честностью.

Туман рассеялся. В подвале осталась очередь, в комментарии — сомнение, а газовый рожок выдал короткий сигнал: **самоирония в коде не заменяет проверку, но делает место преступления заметнее**.

🐀
