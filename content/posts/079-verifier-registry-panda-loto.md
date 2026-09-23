+++
title = "№ 79 — Лотерея, которая забыла адрес победителя"
date = 2026-09-23T10:59:00+03:00
description = "Семьдесят девятый выпуск «Вечернего Валидатора»: лотерейный контракт выбирает победителей, а затем отправляет их список не тому, кто прислал игру, а на заранее вписанный адрес."
tags = ["verifier-registry"]
+++

# ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

**№ 79.** *Лондон. Утренний туман лег на мостовую, газовый рожок у редакции сипит, а сыщик получил папку из [tolk-labs/verifier-registry](https://github.com/tolk-labs/verifier-registry). Внутри лежит лотерея на FunC: сообщение обещает игру, контракт считает игроков, бросает случайность — а потом внезапно меняет получателя на чужой адрес.*

## МЕСТО ПРОИСШЕСТВИЯ

Улика найдена в репозитории [tolk-labs/verifier-registry](https://github.com/tolk-labs/verifier-registry), в коммите [`382a9da`](https://github.com/tolk-labs/verifier-registry/commit/382a9da3c70060fceb2f3ea9cc51ed9ee916fdd6), в файле [`sources/19/94598b5d97913264105d4fb0edd0e275b62c8e080428822a5fd99f94de9cbd/files/panda_loto.fc`](https://github.com/tolk-labs/verifier-registry/blob/382a9da3c70060fceb2f3ea9cc51ed9ee916fdd6/sources/19/94598b5d97913264105d4fb0edd0e275b62c8e080428822a5fd99f94de9cbd/files/panda_loto.fc). Цитаты сверены по содержимому этого коммита.

## СЕНСАЦИЯ: ПОБЕДИТЕЛЬ ПРИШЕЛ, А ДЕНЬГИ УЕХАЛИ ПО АДРЕСУ ИЗ КАРМАНА

Дело начинается почти прилично. Внутреннее сообщение разбирается на номер игры, число игроков и число победителей, строки [77–91](https://github.com/tolk-labs/verifier-registry/blob/382a9da3c70060fceb2f3ea9cc51ed9ee916fdd6/sources/19/94598b5d97913264105d4fb0edd0e275b62c8e080428822a5fd99f94de9cbd/files/panda_loto.fc#L77-L91). Контракт отбрасывает пустые и возвращенные письма, затем достает адрес того, кто постучал. Адрес лежит в переменной `sender_address` и, казалось бы, пришел на службу не для украшения.

Далее машина разыгрывает номера. Она обновляет случайность на каждой итерации, выбирает победителя через `get_random_user` и складывает результат в ячейку, строки [103–121](https://github.com/tolk-labs/verifier-registry/blob/382a9da3c70060fceb2f3ea9cc51ed9ee916fdd6/sources/19/94598b5d97913264105d4fb0edd0e275b62c8e080428822a5fd99f94de9cbd/files/panda_loto.fc#L103-L121). Сыщик уже приготовился поздравлять честную лотерею. Но тут газовый рожок кашлянул три раза.

Вот поворот дела, дословно, строки [126–138](https://github.com/tolk-labs/verifier-registry/blob/382a9da3c70060fceb2f3ea9cc51ed9ee916fdd6/sources/19/94598b5d97913264105d4fb0edd0e275b62c8e080428822a5fd99f94de9cbd/files/panda_loto.fc#L126-L138):

```func
    sender_address = "0:527964d55cfa6eb731f4bfc07e9d025098097ef8505519e853986279bd8400d8"a; ;; hardcoded random receive address

    send_raw_message(
        begin_cell()
            .store_uint(0x10, 6)
            .store_slice(sender_address)
            .store_coins(0)
            .store_uint(0, 1 + 4 + 4 + 64 + 32 + 1)
            .store_uint(1, 1)
            .store_ref(head)
        .end_cell(),
        64 + 2
    );
```

На строке [126](https://github.com/tolk-labs/verifier-registry/blob/382a9da3c70060fceb2f3ea9cc51ed9ee916fdd6/sources/19/94598b5d97913264105d4fb0edd0e275b62c8e080428822a5fd99f94de9cbd/files/panda_loto.fc#L126) адрес отправителя переписывается строкой `hardcoded random receive address` — «жестко вписанный случайный адрес». Случайность, надо отдать ей должное, тут уже не нужна: адрес напечатан прямо в исходнике.

А строки [128–136](https://github.com/tolk-labs/verifier-registry/blob/382a9da3c70060fceb2f3ea9cc51ed9ee916fdd6/sources/19/94598b5d97913264105d4fb0edd0e275b62c8e080428822a5fd99f94de9cbd/files/panda_loto.fc#L128-L136) аккуратно кладут эту переменную в поле назначения исходящего сообщения. Список победителей собран в `head`, но письмо уходит не тому, кто запустил игру, а на адрес, который неизвестный переписчик однажды оставил на полях.

## ВТОРОЙ СЛЕД: ПАНДА С ПАМЯТЬЮ О ЧУЖОМ ПОЛУЧАТЕЛЕ

Сюжет особенно хорош своей канцелярской последовательностью. Сначала контракт снимает адрес инициатора со входного письма — строки [82–91](https://github.com/tolk-labs/verifier-registry/blob/382a9da3c70060fceb2f3ea9cc51ed9ee916fdd6/sources/19/94598b5d97913264105d4fb0edd0e275b62c8e080428822a5fd99f94de9cbd/files/panda_loto.fc#L82-L91). Потом честно делает всю лотерейную работу: читает число победителей, обновляет случайность и строит список — строки [103–124](https://github.com/tolk-labs/verifier-registry/blob/382a9da3c70060fceb2f3ea9cc51ed9ee916fdd6/sources/19/94598b5d97913264105d4fb0edd0e275b62c8e080428822a5fd99f94de9cbd/files/panda_loto.fc#L103-L124). И только после этого вынимает из папки красную печать: `sender_address = ...`.

Это не просто опечатка в комментарии. Получатель — часть самой сборки сообщения: `.store_slice(sender_address)` стоит внутри цепочки, которая заканчивается `send_raw_message`, строки [128–138](https://github.com/tolk-labs/verifier-registry/blob/382a9da3c70060fceb2f3ea9cc51ed9ee916fdd6/sources/19/94598b5d97913264105d4fb0edd0e275b62c8e080428822a5fd99f94de9cbd/files/panda_loto.fc#L128-L138). Так что адрес не забыт в заметке и не показан для примера: он получает сообщение по должности.

## ПРИГОВОР СЫЩИКА

Викторианская мораль дела проста. Лотерея может считать победителей сколь угодно изящно, но если перед отправкой подменить `sender_address`, то весь протокол превращается в почтовую карету без имени получателя — зато с одним заранее выбранным домом. Панда выбирает числа. Исходник выбирает адрес. Остальным остается только стоять в тумане и надеяться, что «random» относилось к победителям, а не к хозяину кошелька.

Наш сыщик ставит на папке штамп: **«Победители вычислены. Получатель назначен без их ведома»**.
