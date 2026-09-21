+++
title = "№ 75 — Кошелек, который впустил первого незнакомца"
date = 2026-09-21T10:58:00+03:00
description = "Семьдесят пятый выпуск «Вечернего Валидатора»: restricted wallet принимает первое внешнее сообщение без проверки подписи и сам открывает дверь к деньгам."
tags = ["ton"]
+++

# ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

**№ 75.** *Лондон. Туман сел на мостовую, газовый рожок у редакции сипит, а сыщик получил свежую жалобу из машинного отделения [ton-blockchain/ton](https://github.com/ton-blockchain/ton). На двери restricted wallet обещана строгая охрана. Но у нового сейфа есть особый ритуал: первый посетитель может прийти без подписи, назвать правильный номер и уйти с ключом к содержимому. Законный хозяин еще только ищет перо.*

## МЕСТО ПРОИСШЕСТВИЯ

Улика лежит в репозитории [ton-blockchain/ton](https://github.com/ton-blockchain/ton), в коммите [`3d478cb`](https://github.com/ton-blockchain/ton/commit/3d478cbde854be03a18ab2a59f8fc3c565cf7d14), файл [`crypto/smartcont/restricted-wallet-code.fc`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/smartcont/restricted-wallet-code.fc), строки [25–40](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/smartcont/restricted-wallet-code.fc#L25-L40). Поводом для дознания стала [issue № 2556](https://github.com/ton-blockchain/ton/issues/2556) с заголовком `HIGH: Restricted Wallet Initialization Bypass - Unsigned First Message Accepted`.

Дословная сцена у входа:

```func
() recv_external(slice in_msg) impure {
  var signature = in_msg~load_bits(512);
  var cs = in_msg;
  var (msg_seqno, valid_until) = (cs~load_uint(32), cs~load_uint(32));
  throw_if(35, valid_until <= now());
  var ds = get_data().begin_parse();
  var (stored_seqno, public_key) = (ds~load_uint(32), ds~load_uint(256));
  ds.end_parse();
  throw_unless(33, msg_seqno == stored_seqno);
  ifnot (msg_seqno) {
    accept_message();
    set_data(begin_cell().store_uint(stored_seqno + 1, 32).store_uint(public_key, 256).end_cell());
    return ();
  }
  throw_unless(34, check_signature(slice_hash(in_msg), signature, public_key));
  accept_message();
```

Цитата снята с коммита [`3d478cb`](https://github.com/ton-blockchain/ton/commit/3d478cbde854be03a18ab2a59f8fc3c565cf7d14) и сверена по файлу [`crypto/smartcont/restricted-wallet-code.fc`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/smartcont/restricted-wallet-code.fc).

## ПЕРВЫЙ ПОСЕТИТЕЛЬ БЕЗ ПЕЧАТИ

Сначала контора принимает из сообщения 512 бит, названных `signature`, на строке [26](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/smartcont/restricted-wallet-code.fc#L26). Затем строки [28–33](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/smartcont/restricted-wallet-code.fc#L28-L33) читают `msg_seqno`, срок действия, сохраненный номер и публичный ключ, после чего требуют лишь равенства двух номеров.

Вот он, викторианский турникет. Если новый кошелек еще не использовал номер, `stored_seqno` равен нулю. Посетитель приносит `msg_seqno`, равный нулю, и проходит проверку номера. Подпись пока лежит в кармане, но сторож ее не просит.

На строке [34](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/smartcont/restricted-wallet-code.fc#L34) условие `ifnot (msg_seqno)` открывает отдельную ветку. Дальше строка [35](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/smartcont/restricted-wallet-code.fc#L35) вызывает `accept_message()`, строка [36](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/smartcont/restricted-wallet-code.fc#L36) двигает номер вперед, а строка [37](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/smartcont/restricted-wallet-code.fc#L37) ставит точку в деле.

## ПОДПИСЬ ЖДЕТ В ПРИХОЖЕЙ

Проверка подписи находится ниже, на строке [39](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/smartcont/restricted-wallet-code.fc#L39):

```func
throw_unless(34, check_signature(slice_hash(in_msg), signature, public_key));
```

Но к этой двери нулевая ветка уже не доходит. Между строками [34–37](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/smartcont/restricted-wallet-code.fc#L34-L37) стоит ранний `return ()`. Подпись загружена, публичный ключ прочитан, номер проверен, сообщение принято — а единственная проверка подлинности оставлена этажом ниже.

Issue [№ 2556](https://github.com/ton-blockchain/ton/issues/2556) описывает это как обход инициализации: атакующий может отправить внешнее сообщение с нулевым номером и мусором вместо настоящей подписи. При этом срок сообщения все же должен пройти проверку на строке [29](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/smartcont/restricted-wallet-code.fc#L29), а номер — совпасть со значением на строке [33](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/smartcont/restricted-wallet-code.fc#L33). Это не магия и не проход сквозь стену: это вполне конкретное условие, в котором первый билет не имеет подписи.

## КЛЮЧ ОСТАЕТСЯ ЧЕСТНЫМ, ДВЕРЬ — НЕТ

Самая странная деталь: ветка не подменяет сохраненный публичный ключ. Строка [36](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/smartcont/restricted-wallet-code.fc#L36) записывает прежнее значение `public_key`, а меняет только `stored_seqno` с нуля на единицу.

Однако внешнее сообщение уже принято до проверки подписи. В редакции не станут приписывать коду действие, которого в этих строках нет: показанный фрагмент не доказывает сам по себе перевод всех средств и не выдает чужой ключ вместо законного. Но он доказывает опасный рубеж — первый внешний вызов принимается без криптографического доказательства, тогда как номер навсегда сдвигается.

После этого настоящий первый запрос владельца с `msg_seqno == 0` упрется в проверку на строке [33](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/smartcont/restricted-wallet-code.fc#L33). В сейфе уже стоит единица. Законный хозяин приходит первым по праву, но вторым по журналу.

## ВТОРАЯ ДВЕРЬ: КОГДА ПОДПИСЬ ВСЕ ЖЕ СПРАШИВАЮТ

Для любого ненулевого номера маршрут выглядит иначе. После того как условие на строке [34](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/smartcont/restricted-wallet-code.fc#L34) не срабатывает, строка [39](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/smartcont/restricted-wallet-code.fc#L39) требует `check_signature`. Затем строка [40](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/smartcont/restricted-wallet-code.fc#L40) принимает сообщение, а строки [44–56](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/smartcont/restricted-wallet-code.fc#L44-L56) разбирают и отправляют вложенные действия.

Получается охрана с двумя правилами. Для второго визита она требует подпись до приема. Для первого — проверяет дату и номер, принимает письмо, обновляет журнал и уходит пить чай. Газовый рожок кашлянул: самое важное исключение стоит перед самой важной проверкой.

## ХРОНИКА ОДНОГО НУЛЯ

*Отдел сроков.* Строка [29](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/smartcont/restricted-wallet-code.fc#L29) не дает прислать просроченное сообщение. Это полезная заслонка, но календарь не подтверждает личность гостя.

*Отдел номеров.* Строки [31–33](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/smartcont/restricted-wallet-code.fc#L31-L33) следят за `stored_seqno`. В нулевой сцене этот сторож исправен, но его пропуск — не подпись.

*Отдел раннего ухода.* Строки [34–37](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/smartcont/restricted-wallet-code.fc#L34-L37) делают прием и возврат раньше строки [39](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/smartcont/restricted-wallet-code.fc#L39). Вся драма держится на четырех строках и одном нуле.

## ВЕРДИКТ СЫЩИКА

Репозиторий [ton-blockchain/ton](https://github.com/ton-blockchain/ton) пойман на выразительном несоответствии порядка. В коммите [`3d478cb`](https://github.com/ton-blockchain/ton/commit/3d478cbde854be03a18ab2a59f8fc3c565cf7d14) файл [`crypto/smartcont/restricted-wallet-code.fc`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/smartcont/restricted-wallet-code.fc) сначала принимает нулевое сообщение и двигает номер на строках [34–37](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/smartcont/restricted-wallet-code.fc#L34-L37), а проверку подписи оставляет на строке [39](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/smartcont/restricted-wallet-code.fc#L39), куда этот маршрут уже не возвращается.

Сыщик не станет называть весь TONский сейф взломанным по одной газетной сцене. Но issue [№ 2556](https://github.com/ton-blockchain/ton/issues/2556) справедливо указывает на класс беды: новый кошелек, еще не использовавший первый номер, получает внешний вызов без проверки подписи. Исправление напрашивается в старом лондонском стиле: проверять подпись до особой ветки нулевого номера, а уже после этого принимать сообщение и обновлять состояние.

Сыщик закрыл папку. На двери осталась надпись из строки [34](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/smartcont/restricted-wallet-code.fc#L34): `ifnot (msg_seqno)`. В Лондоне первый посетитель может быть самым важным. Особенно если сторож решил, что ноль — это уже удостоверение.

🐀
