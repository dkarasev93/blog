+++
title = "№ 33 — Признание в полночь: «Have no idea hot to parallelize this» в подвале celldb и костыли, гордые собой"
date = 2026-08-30T11:10:00+03:00
description = "Тридцать третий выпуск «Вечернего Валидатора»: в ton-blockchain/ton найдена записка инженера «Have no idea hot to parallelize this in case of rocksdb» с опечаткой от волнения; tdutils чеканит девиз «we have our own hacks»; ошибка компиляции вежливо просит закомментировать саму себя; управление памятью в многопоточном коде описано словами «stupid gc»."
tags = ["ton"]
+++

# 📰 Вечерний Валидатор

**№ 33.** *Лондон. Туман над мемпулом стоял густой, как очередь неподтверждённых транзакций. Сыщик спустился по скрипучей лестнице в недра [ton-blockchain/ton](https://github.com/ton-blockchain/ton) — в подвал celldb, где валидаторы хранят самое сокровенное: состояние всей сети. И там, при свете газового реторта, обнаружил галерею полуночных исповедей, оставленных инженерами прямо на стенах продакшена.*

*Место осмотра: репозиторий [ton-blockchain/ton](https://github.com/ton-blockchain/ton), коммит [`3d478cb`](https://github.com/ton-blockchain/ton/commit/3d478cbde854be03a18ab2a59f8fc3c565cf7d14) («TON 2026.08», master). Файлы: [`crypto/vm/db/DynamicBagOfCellsDbV2.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/vm/db/DynamicBagOfCellsDbV2.cpp), [`tdutils/td/utils/port/Stat.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tdutils/td/utils/port/Stat.cpp), [`tdutils/td/utils/HazardPointers.h`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tdutils/td/utils/HazardPointers.h). Все цитаты дословные, опечатки оригинала сохранены как улики.*

---

## СЕНСАЦИЯ ПЕРВАЯ: ЗАПИСКА НА ПОЛЯХ ВЕЧНОСТИ

Файл [`DynamicBagOfCellsDbV2.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/vm/db/DynamicBagOfCellsDbV2.cpp) — полторы тысячи строк честного, мозолистого C++. На строке [1539](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/vm/db/DynamicBagOfCellsDbV2.cpp#L1539) — записка, оставленная неизвестным инженером:

```cpp
// Have no idea hot to parallelize this in case of rocksdb
```

Остановитесь, читатель. Перечитайте. Смакуйте. Человек, чьими руками написан код, хранящий **каждую ячейку состояния сети на сотни миллиардов**, стоит перед циклом по чанкам диффов — и честно, по-детски, с опущенной головой признаётся: *не знаю, как это распараллелить*. Не в личном дневнике. Не на исповеди. В продакшене. В мастер-ветке. Навсегда.

А опечатка — **«hot» вместо «how»** — вишенка на этом пудинге из смирения. Палец дрогнул, сэр. От волнения. От честности. Редакция намеренно не правит её: это часть истории, как прожжённая дыра на сюртуке самоубийцы.

## СЕНСАЦИЯ ВТОРАЯ: ГЕРОЙ ЗНАЛ, КУДА ИДЁТ

Тот же файл, строка [17](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/vm/db/DynamicBagOfCellsDbV2.cpp#L17). Прежде чем признаться в бессилии, автор честно предупреждает публику об инструменте, которым орудовал:

```cpp
// Very stupid Vector/MpmcQueue
```

«Очень глупый вектор». Он знал. Он *знал*, леди и джентльмены. И всё равно пошёл до конца — как истинный герой нуара идёт в бар, где его ждут.

## СЕНСАЦИЯ ТРЕТЬЯ: ДЕВИЗ НА ФАМИЛЬНОМ ГРЕБЕ

Перебираемся в контору tdutils, [`Stat.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tdutils/td/utils/port/Stat.cpp). Строка [88](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tdutils/td/utils/port/Stat.cpp#L88) — здесь смирение уступает место гордому цинизму ремесленника:

```cpp
// remove libc compatibility hacks if any: we have our own hacks
```

«Уберите костыли libc, если найдёте: у нас есть собственные костыли». Это не комментарий. Это **девиз на фамильном гребе**. Так говорили старейшины лондонских доков: чужой грязи у нас не надо, своей хватает.

А пятью строками выше, на строке [83](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tdutils/td/utils/port/Stat.cpp#L83), — шедевр жанра, достойный отдельной рамки:

```cpp
T().warning("Platform lacks support of precise access/modification file times, comment this line to continue");
```

Ошибка компиляции, которая вежливо просит вас **закомментировать саму себя, чтобы продолжить**. Палач, протягивающий осуждённому ножницы. Бюрократия, доведённая до совершенства.

## СЕНСАЦИЯ ЧЕТВЁРТАЯ: ЭПИТАФИЯ В ДВА СЛОВА

[`HazardPointers.h`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tdutils/td/utils/HazardPointers.h), строка [109](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tdutils/td/utils/HazardPointers.h#L109). Лаконично, как надгробие:

```cpp
// stupid gc
```

Два слова. Управление памятью в многопоточном коде, где одна ошибка стоит слота валидатора, описано словами, которыми кучер ругает упрямую клячу.

## РЕЗЮМЕ СЫЩИКА

Мы не осуждаем. «Вечерний Валидатор» видел вещи похуже: трёхлетние TODO, закомментированные блоки размером с роман Толстого, конфиги, захардкоженные ещё при царе-тестнете. Напротив — есть что-то трогательно викторианское в этой голой честности. В эпоху, когда каждый стартап рисует диаграммы своей «идеальной масштабируемой архитектуры», инженер TON просто пишет в коде: *«понятия не имею»* — и жмёт merge.

Файл жив и правится по сей день. Признание осталось. Цикл остался. Блокчейн работает. Может, в этом и есть весь секрет мейннета, сэр.

🐀
