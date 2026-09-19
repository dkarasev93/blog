+++
title = "№ 72 — Кошелек, который сначала коммитит, а потом читает"
date = 2026-09-19T16:58:00+03:00
description = "Семьдесят второй выпуск «Вечернего Валидатора»: новый Telegram-кошелек TON принимает подписанное сообщение, навсегда двигает seqno, а затем может обнаружить, что массив был криво упакован."
tags = ["tg-wallet-contract"]
+++

# 📰 ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

**№ 72.** *Лондон. Вечерний туман обнял мостовую, газовый рожок у редакции кашлянул, а сыщик получил папку из новой кошельковой конторы. На обложке обещали защиту от повтора. Внутри нашлась другая защита: подписанное сообщение сначала получает новый номер, потом отправляется на разбор. Если разбор споткнется, номер уже ушел в архив, а деньги остались на столе.*

## МЕСТО ПРОИСШЕСТВИЯ

Репозиторий [ton-blockchain/tg-wallet-contract](https://github.com/ton-blockchain/tg-wallet-contract), коммит [`10a0d2b`](https://github.com/ton-blockchain/tg-wallet-contract/commit/10a0d2b9e87bd0a5e9135c2431e3eb15c6a0485c), файл [`contracts/WalletTg/WalletTg.tolk`](https://github.com/ton-blockchain/tg-wallet-contract/blob/10a0d2b9e87bd0a5e9135c2431e3eb15c6a0485c/contracts/WalletTg/WalletTg.tolk), строки [129–140](https://github.com/ton-blockchain/tg-wallet-contract/blob/10a0d2b9e87bd0a5e9135c2431e3eb15c6a0485c/contracts/WalletTg/WalletTg.tolk#L129-L140). Цитата сверена по содержимому файла на этом коммите.

Дословный протокол из комнаты внешних сообщений:

```tolk
        SendBulkMessagesRequestE => {
            msg.header.validateRequest(storage);
            acceptExternalMessage();

            storage.seqno += 1;
            storage.save();
            commitContractDataAndActions();

            // if parsing fails, the contract throws, and its balance is charged for gas;
            // this is expected: if we are here, the user signed this incorrectly packed message;
            // anyway, it cannot be replayed, because seqno is already bumped
            msg.msgArr.parseArrayAndSend(true);
        }
```

## ПЕРВЫЙ СЛЕД: ПЕЧАТЬ ПЕРЕД ДОСМОТРОМ

На строках [129–130](https://github.com/ton-blockchain/tg-wallet-contract/blob/10a0d2b9e87bd0a5e9135c2431e3eb15c6a0485c/contracts/WalletTg/WalletTg.tolk#L129-L130) кошелек проверяет заголовок запроса. В нем сверяются номер `seqno`, идентификатор кошелька и срок действия: та самая сторожевая будка против повтора.

Затем на строке [131](https://github.com/ton-blockchain/tg-wallet-contract/blob/10a0d2b9e87bd0a5e9135c2431e3eb15c6a0485c/contracts/WalletTg/WalletTg.tolk#L131) вызывается `acceptExternalMessage()`. Сыщик не спорит: для внешнего сообщения это нормальная церемония, после которой контракт принимает его к исполнению. Но дальше порядок событий становится куда интереснее.

Строка [133](https://github.com/ton-blockchain/tg-wallet-contract/blob/10a0d2b9e87bd0a5e9135c2431e3eb15c6a0485c/contracts/WalletTg/WalletTg.tolk#L133) увеличивает `storage.seqno`, а строки [134–135](https://github.com/ton-blockchain/tg-wallet-contract/blob/10a0d2b9e87bd0a5e9135c2431e3eb15c6a0485c/contracts/WalletTg/WalletTg.tolk#L134-L135) сохраняют его и фиксируют данные вместе с действиями. Только после этого строка [140](https://github.com/ton-blockchain/tg-wallet-contract/blob/10a0d2b9e87bd0a5e9135c2431e3eb15c6a0485c/contracts/WalletTg/WalletTg.tolk#L140) зовет `msg.msgArr.parseArrayAndSend(true)`.

Получается строгая викторианская очередь: сначала клерк ставит печать «номер использован», затем открывает посылку и выясняет, что в ней лежит. Если массив сообщений не соответствует формату, номер уже нельзя вернуть на полку.

## ВТОРОЙ СЛЕД: САМА КАНЦЕЛЯРИЯ ВСЕ ПОДПИСАЛА

Самая сочная деталь находится в комментариях на строках [137–139](https://github.com/ton-blockchain/tg-wallet-contract/blob/10a0d2b9e87bd0a5e9135c2431e3eb15c6a0485c/contracts/WalletTg/WalletTg.tolk#L137-L139). Код прямо признает: если разбор провалится, контракт бросит исключение, баланс заплатит за газ, а сообщение нельзя будет повторить, потому что `seqno` уже увеличен.

Дословная формула обвинения такова:

```tolk
// if parsing fails, the contract throws, and its balance is charged for gas;
// this is expected: if we are here, the user signed this incorrectly packed message;
// anyway, it cannot be replayed, because seqno is already bumped
```

Это не тайная улика, которую сыщик вытащил из машинного подвала. Это официальное описание поведения прямо над вызовом. Кошелек знает, что подписанная, но неверно упакованная пачка может сгореть на разборе. Он знает и второе: повторная попытка будет отвергнута новым номером.

Справедливости ради, сообщение подписано текущим владельцем. Контракт не пропускает случайного прохожего: внешняя подпись проверяется раньше, на строках [108–114](https://github.com/ton-blockchain/tg-wallet-contract/blob/10a0d2b9e87bd0a5e9135c2431e3eb15c6a0485c/contracts/WalletTg/WalletTg.tolk#L108-L114). Но подпись доказывает право отправителя, а не безупречность внутреннего массива. Ошибка клиента превращается в платный визит без второй попытки.

## ТРЕТИЙ СЛЕД: ОДНА ДВЕРЬ, ДВА МАРШРУТА

Сыщик сравнил это с одиночной отправкой. В соседней ветке `SendOneMessageRequestE` строки [118–126](https://github.com/ton-blockchain/tg-wallet-contract/blob/10a0d2b9e87bd0a5e9135c2431e3eb15c6a0485c/contracts/WalletTg/WalletTg.tolk#L118-L126) порядок тот же: проверка, `acceptExternalMessage`, увеличение `seqno`, сохранение, фиксация и затем отправка одного сообщения.

Для одного сообщения это выглядит почти безобидно: после проверки заголовка операция отправляется в `validateAndSend`. В пачке же последняя дверь ведет в `parseArrayAndSend`, где еще предстоит разобрать длину и элементы массива. Чем больше посылка, тем больше места остается для ошибки упаковки после уже необратимой печати.

Сыщик не станет называть это кражей средств. В показанном пути явно описан расход газа, а не потеря всей суммы перевода. Но для пользователя картина неприятна: он может получить отказ на поздней стадии, потратить газ и потерять текущий `seqno`, хотя ни один элемент пачки еще не был отправлен.

## ХРОНИКА МЕЛКИХ ПРОИСШЕСТВИЙ

*Отдел заголовков.* Комментарий на строке [28](https://github.com/ton-blockchain/tg-wallet-contract/blob/10a0d2b9e87bd0a5e9135c2431e3eb15c6a0485c/contracts/WalletTg/WalletTg.tolk#L28) обещает проверку против повторов. Она действительно работает для валидного запроса, но в пачечной ветке номер становится новым еще до полного разбора тела.

*Отдел честных признаний.* Фраза `its balance is charged for gas` на строке [137](https://github.com/ton-blockchain/tg-wallet-contract/blob/10a0d2b9e87bd0a5e9135c2431e3eb15c6a0485c/contracts/WalletTg/WalletTg.tolk#L137) звучит как расписка кассира: ошибка упаковки не скрывается и не возвращается вежливым результатом.

*Отдел повторов.* Строка [139](https://github.com/ton-blockchain/tg-wallet-contract/blob/10a0d2b9e87bd0a5e9135c2431e3eb15c6a0485c/contracts/WalletTg/WalletTg.tolk#L139) превращает защиту от повтора в односторонний турникет. После сохранения нового номера старый запрос уже не пройдет, даже если его поздняя часть не пережила досмотр.

## ВЕРДИКТ СЫЩИКА

[ton-blockchain/tg-wallet-contract](https://github.com/ton-blockchain/tg-wallet-contract) пойман на маленьком, но выразительном порядке действий. В коммите [`10a0d2b`](https://github.com/ton-blockchain/tg-wallet-contract/commit/10a0d2b9e87bd0a5e9135c2431e3eb15c6a0485c) файл [`contracts/WalletTg/WalletTg.tolk`](https://github.com/ton-blockchain/tg-wallet-contract/blob/10a0d2b9e87bd0a5e9135c2431e3eb15c6a0485c/contracts/WalletTg/WalletTg.tolk) сначала принимает внешнее сообщение и навсегда двигает `seqno` на строках [129–135](https://github.com/ton-blockchain/tg-wallet-contract/blob/10a0d2b9e87bd0a5e9135c2431e3eb15c6a0485c/contracts/WalletTg/WalletTg.tolk#L129-L135), а потом разбирает массив на строке [140](https://github.com/ton-blockchain/tg-wallet-contract/blob/10a0d2b9e87bd0a5e9135c2431e3eb15c6a0485c/contracts/WalletTg/WalletTg.tolk#L140).

Это может быть осознанной политикой: подписанный запрос с плохой упаковкой считается использованным, чтобы его нельзя было гонять по кругу. Но цена политики записана в том же протоколе: газ списан, номер сдвинут, а пачка не прошла. В идеальном кошельке клиенту стоило бы дать отдельную предварительную проверку или надежный форматировщик, чтобы ошибка упаковки находилась до необратимой отметки.

Сыщик закрыл папку. Газовый рожок дернулся в тумане. На двери осталась записка из строки [139](https://github.com/ton-blockchain/tg-wallet-contract/blob/10a0d2b9e87bd0a5e9135c2431e3eb15c6a0485c/contracts/WalletTg/WalletTg.tolk#L139): `seqno is already bumped`. В Лондоне даже честный турникет может взять плату за вход раньше, чем проверит билет.

🐀
