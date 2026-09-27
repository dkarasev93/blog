+++
title = "№ 86 — Миграция, которая не знала слова EXISTS"
date = 2026-09-27T10:58:00+03:00
tags = ["ton-indexer"]
+++

# ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

**№ 86.** *Лондон. Утренний туман залег на мостовой, газовый рожок сипит у редакции, а сыщик получил из [toncenter/ton-indexer](https://github.com/toncenter/ton-indexer) срочную записку: база данных должна была спокойно пережить очередной запуск. Внутри записки нашлась одна лишняя буква. Не в романе, не в комментарии, а прямо в SQL, где она решила стать стеной между сервисом и новой колонкой.*

## МЕСТО ПРОИСШЕСТВИЯ

Улика найдена в [toncenter/ton-indexer](https://github.com/toncenter/ton-indexer), в коммите [`1981e76`](https://github.com/toncenter/ton-indexer/commit/1981e7666c71dad5886528a3eaa19f2e8a4038de), в файле [`ton-index-worker/ton-index-postgres/src/migrate.cpp`](https://github.com/toncenter/ton-indexer/blob/1981e7666c71dad5886528a3eaa19f2e8a4038de/ton-index-worker/ton-index-postgres/src/migrate.cpp), строка [1120](https://github.com/toncenter/ton-indexer/blob/1981e7666c71dad5886528a3eaa19f2e8a4038de/ton-index-worker/ton-index-postgres/src/migrate.cpp#L1120). Это состояние до исправляющего коммита [`9ab50a1`](https://github.com/toncenter/ton-indexer/commit/9ab50a12c7a708d44ba266a1dcdd253d25f1c723), названного без маскировки: `Fix typo in migration`. Цитата снята с родительского снимка и сверена по строке.

Дословная улика, одна строка, один лишний звук:

```cpp
    query += "ALTER TABLE getgems_nft_auctions ADD COLUMN IF NOTN EXISTS jetton_wallet tonaddr;\n";
```

## ПЕРВЫЙ СЛЕД: ПАРОЛЬ ОТ ДВЕРИ ИСПОРЧЕН ОДНОЙ БУКВОЙ

Вежливая дверь SQL зовется `IF NOT EXISTS`. Она обещает: если колонка уже есть, не поднимай тревогу; если ее нет, добавь. Но в строке [1120](https://github.com/toncenter/ton-indexer/blob/1981e7666c71dad5886528a3eaa19f2e8a4038de/ton-index-worker/ton-index-postgres/src/migrate.cpp#L1120) между `NOT` и `EXISTS` поселилась буква `N`: `NOTN EXISTS`.

Это не новый синтаксис PostgreSQL и не загадочный диалект аукционного квартала. Запрос строится как часть миграции `run_1_2_7_migrations`, а затем отправляется в базу вместе с соседними приказами. На строках [1117–1123](https://github.com/toncenter/ton-indexer/blob/1981e7666c71dad5886528a3eaa19f2e8a4038de/ton-index-worker/ton-index-postgres/src/migrate.cpp#L1117-L1123) стоят несколько почти одинаковых распоряжений для таблицы `getgems_nft_auctions`: время шага, последний номер запроса, кошелек jetton, мастер jetton и еще несколько полей. Все соседи знают пароль. Один стражник на строке [1120](https://github.com/toncenter/ton-indexer/blob/1981e7666c71dad5886528a3eaa19f2e8a4038de/ton-index-worker/ton-index-postgres/src/migrate.cpp#L1120) назвал его неверно.

## ВТОРОЙ СЛЕД: АУКЦИОН, КОТОРЫЙ ЗАПЕРЛИ НА ВХОДЕ

Само имя колонки невинно: `jetton_wallet`. Беда не в типе `tonaddr` и не в таблице аукционов. Беда в том, что служебная записка не доходит до смысла: PostgreSQL должен сначала разобрать команду `ALTER TABLE`, а встреча с `NOTN` оставляет миграцию у порога.

Здесь особенно хороша криминальная геометрия. В той же пачке строка [1119](https://github.com/toncenter/ton-indexer/blob/1981e7666c71dad5886528a3eaa19f2e8a4038de/ton-index-worker/ton-index-postgres/src/migrate.cpp#L1119) безупречно добавляет `last_query_id`, а строка [1121](https://github.com/toncenter/ton-indexer/blob/1981e7666c71dad5886528a3eaa19f2e8a4038de/ton-index-worker/ton-index-postgres/src/migrate.cpp#L1121) уже снова пишет правильное `IF NOT EXISTS`. Ошибка не гуляет по всему дому. Она стоит посередине коридора, в одном экземпляре, как швейцар, который знает всех жильцов, но не знает собственного имени.

Редакция не утверждает по одной строке, что вся схема непременно останется без изменений: точный эффект зависит от обработки ошибки и порядка запуска миграций. Но утверждение крепкое и скромное: этот SQL-текст не является корректной заменой соседнему `IF NOT EXISTS`, а исправляющий коммит прямо указывает на опечатку.

## ТРЕТИЙ СЛЕД: ПРИЗНАНИЕ В ИСПРАВЛЯЮЩЕМ КОММИТЕ

Через дверь архива сыщик достал исправление [`9ab50a1`](https://github.com/toncenter/ton-indexer/commit/9ab50a12c7a708d44ba266a1dcdd253d25f1c723). В том же файле [`ton-index-worker/ton-index-postgres/src/migrate.cpp`](https://github.com/toncenter/ton-indexer/blob/9ab50a12c7a708d44ba266a1dcdd253d25f1c723/ton-index-worker/ton-index-postgres/src/migrate.cpp), строка [1120](https://github.com/toncenter/ton-indexer/blob/9ab50a12c7a708d44ba266a1dcdd253d25f1c723/ton-index-worker/ton-index-postgres/src/migrate.cpp#L1120), буква исчезла:

```cpp
    query += "ALTER TABLE getgems_nft_auctions ADD COLUMN IF NOT EXISTS jetton_wallet tonaddr;\n";
```

Коммит называется `Fix typo in migration` и меняет ровно два символа: лишняя `N` покидает участок, пробел возвращается на службу. Никакой реформы, никакого нового плана, только санитарная обработка одной строки. Так в Лондоне чинят замок: не меняют весь особняк, а вынимают щепку из скважины.

## ПОЧЕМУ ЭТО СОЧНО

Сочная часть не в масштабе диффа, а в несоответствии декора и угрозы. Вокруг лежит солидная C++-канцелярия, таблица NFT-аукционов, тип TON-адреса и пачка миграций. А виновник выглядит как опечатка, которую можно принять за кличку местного нотариуса: `NOTN EXISTS`.

Газовый рожок особенно громко хрипит из-за соседства. В одной строке база получает обычную защиту от повторного добавления колонки. В следующей редакция видит почти тот же текст, но с лишней буквой, способной превратить спокойный запуск схемы в допрос у синтаксического инспектора. Большие системы иногда спотыкаются не о пропасть, а о маленькую букву, оставленную на ступеньке.

## ВЕРДИКТ СЫЩИКА

[toncenter/ton-indexer](https://github.com/toncenter/ton-indexer) пойман на деле о миграционном двойнике. В коммите [`1981e76`](https://github.com/toncenter/ton-indexer/commit/1981e7666c71dad5886528a3eaa19f2e8a4038de) файл [`ton-index-worker/ton-index-postgres/src/migrate.cpp`](https://github.com/toncenter/ton-indexer/blob/1981e7666c71dad5886528a3eaa19f2e8a4038de/ton-index-worker/ton-index-postgres/src/migrate.cpp) на строке [1120](https://github.com/toncenter/ton-indexer/blob/1981e7666c71dad5886528a3eaa19f2e8a4038de/ton-index-worker/ton-index-postgres/src/migrate.cpp#L1120) велел базе искать `NOTN EXISTS`. Исправляющий коммит [`9ab50a1`](https://github.com/toncenter/ton-indexer/commit/9ab50a12c7a708d44ba266a1dcdd253d25f1c723) вернул строке правильное `IF NOT EXISTS`.

Приговор мягкий, но с печатью: перед тем как отправлять миграцию в туман, перечитай ее как пароль. В SQL одна лишняя буква не выглядит подозрительно. Она просто не открывает дверь.

*Сыщик погасил газовый рожок, положил карандаш на `IF NOT EXISTS` и растворился в лондонском тумане.*
