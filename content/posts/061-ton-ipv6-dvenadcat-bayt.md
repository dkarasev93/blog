---
title: "Выпуск № 61: IPv6, которому отрезали четыре байта"
date: 2026-09-14T11:00:00+03:00
tags: [ton]
---

# 📰 ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

## ДЕЛО О ДЛИНЕ, КОТОРАЯ НЕ ДОШЛА ДО АДРЕСА

Лондон проснулся под серым туманом. Мостовая блестела, газовый рожок у редакции сипел, а сыщик «Вечернего Валидатора» получил папку из главной конторы [ton-blockchain/ton](https://github.com/ton-blockchain/ton). Внутри лежала не пропавшая монета и не взломанный сейф, а куда более изящная улика: адрес IPv6, которому выдали размер меньше, чем он заслуживает.

На обложке стоял коммит [`866cb88`](https://github.com/ton-blockchain/ton/commit/866cb883e8da44a367dc211f264c8ffa13cb7ca1) с вывеской `Improve adnl packet size calculation`. Исправление честно признавалось: в сетевой картотеке IPv6-адрес считали 12 байтами, хотя сам адресный реквизит занимает 128 бит, то есть 16 байт. Четыре байта исчезали в тумане еще до отправки письма.

## МЕСТО ПРОИСШЕСТВИЯ

Репозиторий [ton-blockchain/ton](https://github.com/ton-blockchain/ton), исправляющий коммит [`866cb88`](https://github.com/ton-blockchain/ton/commit/866cb883e8da44a367dc211f264c8ffa13cb7ca1), файл [`adnl/adnl-address-list.hpp`](https://github.com/ton-blockchain/ton/blob/866cb883e8da44a367dc211f264c8ffa13cb7ca1/adnl/adnl-address-list.hpp), строка [81–83](https://github.com/ton-blockchain/ton/blob/866cb883e8da44a367dc211f264c8ffa13cb7ca1/adnl/adnl-address-list.hpp#L81-L83). Цитата сверена с содержимым файла на этом коммите.

Протокол после ремонта:

```cpp
td::uint32 serialized_size() const override {
  return 24;
}
```

А теперь старый протокол — родительский коммит [`f7a473a`](https://github.com/ton-blockchain/ton/commit/f7a473a0e2990b5460794564fb604f1dba5344ab), тот же файл [`adnl/adnl-address-list.hpp`](https://github.com/ton-blockchain/ton/blob/f7a473a0e2990b5460794564fb604f1dba5344ab/adnl/adnl-address-list.hpp), строки [81–83](https://github.com/ton-blockchain/ton/blob/f7a473a0e2990b5460794564fb604f1dba5344ab/adnl/adnl-address-list.hpp#L81-L83):

```cpp
td::uint32 serialized_size() const override {
  return 12;
}
```

## ПЕРВЫЙ СЛЕД: АДРЕС С ТРЕМЯ ЧЕТВЕРТЯМИ ПАСПОРТА

Класс `AdnlAddressUdp6` хранит поле `ip_` типа `td::Bits128`. Это видно в [файле](https://github.com/ton-blockchain/ton/blob/866cb883e8da44a367dc211f264c8ffa13cb7ca1/adnl/adnl-address-list.hpp), на строке [65](https://github.com/ton-blockchain/ton/blob/866cb883e8da44a367dc211f264c8ffa13cb7ca1/adnl/adnl-address-list.hpp#L65). Рядом лежит порт `td::uint16`, строка [66](https://github.com/ton-blockchain/ton/blob/866cb883e8da44a367dc211f264c8ffa13cb7ca1/adnl/adnl-address-list.hpp#L66). Но метод оценки размера в родительском коммите [`f7a473a`](https://github.com/ton-blockchain/ton/commit/f7a473a0e2990b5460794564fb604f1dba5344ab) оставлял для всей записи только `12`.

Сыщик не станет делать вид, что каждая цифра обязана равняться простой сумме полей: у сетевого формата есть свои заголовки и выравнивание. Но разница между [12](https://github.com/ton-blockchain/ton/blob/f7a473a0e2990b5460794564fb604f1dba5344ab/adnl/adnl-address-list.hpp#L81-L83) и [24](https://github.com/ton-blockchain/ton/blob/866cb883e8da44a367dc211f264c8ffa13cb7ca1/adnl/adnl-address-list.hpp#L81-L83) не похожа на косметику. Сам коммит [`866cb88`](https://github.com/ton-blockchain/ton/commit/866cb883e8da44a367dc211f264c8ffa13cb7ca1) пришел именно с починкой расчета размера пакета.

## ВТОРОЙ СЛЕД: СЧЕТЧИК, КОТОРЫЙ СТОЯЛ У ВХОДА

Размер адреса нужен не для красоты в музейном каталоге. Он участвует в предварительном расчете сетевого сообщения: если контора думает, что адрес короче, она может неверно оценить, сколько места осталось для остальных данных.

В том же коммите [`866cb88`](https://github.com/ton-blockchain/ton/commit/866cb883e8da44a367dc211f264c8ffa13cb7ca1) в файле [`adnl/adnl-message.h`](https://github.com/ton-blockchain/ton/blob/866cb883e8da44a367dc211f264c8ffa13cb7ca1/adnl/adnl-message.h) появился счетчик строк [30–42](https://github.com/ton-blockchain/ton/blob/866cb883e8da44a367dc211f264c8ffa13cb7ca1/adnl/adnl-message.h#L30-L42), который учитывает служебные байты и выравнивание:

```cpp
inline td::uint32 bytes_serialized_size(size_t size) {
  // TlStorerCalcLength::store_string
  if (size < 254) {
    size += 1;
  } else if (size < (1 << 24)) {
    size += 4;
  } else {
    size += 8;
  }
  return static_cast<td::uint32>((size + 3) & -4);
}
```

Контраст выразителен. Для сообщения контора завела отдельную арифметику с порогами и выравниванием, а IPv6 до ремонта проходил через короткую записку `return 12`. Газовый рожок кашлянул: у одной двери лежит линейка, у соседней — оценка на глаз.

## ТРЕТИЙ СЛЕД: ПОСЫЛЬНЫЙ, КОТОРЫЙ НЕ СТАЛ ВРАТЬ ДАЛЬШЕ

Коммит [`866cb88`](https://github.com/ton-blockchain/ton/commit/866cb883e8da44a367dc211f264c8ffa13cb7ca1) поправил не только строку в заголовке. В файле [`adnl/adnl-address-list.cpp`](https://github.com/ton-blockchain/ton/blob/866cb883e8da44a367dc211f264c8ffa13cb7ca1/adnl/adnl-address-list.cpp), строки [118–125](https://github.com/ton-blockchain/ton/blob/866cb883e8da44a367dc211f264c8ffa13cb7ca1/adnl/adnl-address-list.cpp#L118-L125), сетевой посыльный еще и перестал передавать слишком крупное сообщение вслепую:

```cpp
if (message.size() > AdnlNetworkManager::get_mtu()) {
  VLOG(adnl, ERROR) << "Dropping too big outbound packet dst=" << addr_ << " size=" << message.size();
  return;
}
```

Это уже не улика против новой версии, а печать на конверте ремонта. Сначала контора пересчитала адресные размеры, затем добавила явный отказ для сообщения, которое не влезает в MTU. Старый расчет IPv6 был частью той же ночной истории: счетчик мог оставить лишнее доверие к свободному месту.

Редакция не считает [ton-blockchain/ton](https://github.com/ton-blockchain/ton) преступным заведением. Наоборот, починка пришла открыто и с ясным названием. Но старый фрагмент прекрасен своей простотой: серьезный сетевой класс носил `Bits128`, а паспорт размера выдавал ему `12`.

## ВЕРДИКТ СЫЩИКА

[ton-blockchain/ton](https://github.com/ton-blockchain/ton) пойман на старом деле о коротком IPv6-паспорте. В родительском коммите [`f7a473a`](https://github.com/ton-blockchain/ton/commit/f7a473a0e2990b5460794564fb604f1dba5344ab) файл [`adnl/adnl-address-list.hpp`](https://github.com/ton-blockchain/ton/blob/f7a473a0e2990b5460794564fb604f1dba5344ab/adnl/adnl-address-list.hpp) обещал размер [12](https://github.com/ton-blockchain/ton/blob/f7a473a0e2990b5460794564fb604f1dba5344ab/adnl/adnl-address-list.hpp#L81-L83). В коммите [`866cb88`](https://github.com/ton-blockchain/ton/commit/866cb883e8da44a367dc211f264c8ffa13cb7ca1) тот же файл стал обещать [24](https://github.com/ton-blockchain/ton/blob/866cb883e8da44a367dc211f264c8ffa13cb7ca1/adnl/adnl-address-list.hpp#L81-L83).

Приговор мягок: если поле зовется `Bits128`, не стоит отправлять его в расчет с половиной паспорта. Сыщик закрыл папку, сверил строки [81–83](https://github.com/ton-blockchain/ton/blob/866cb883e8da44a367dc211f264c8ffa13cb7ca1/adnl/adnl-address-list.hpp#L81-L83), погас газовый рожок и растворился в тумане. В Лондоне адрес можно потерять на мостовой. Но в сетевой арифметике хуже, когда его теряют еще до конверта.

🐀
