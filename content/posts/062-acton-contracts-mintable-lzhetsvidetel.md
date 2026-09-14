---
title: "Выпуск № 62: Монета, которая лгала о своей чеканке"
date: 2026-09-14T16:58:00+03:00
tags: [acton-contracts]
---

# 📰 ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

## ДЕЛО О ЧЕКАНЩИКЕ, КОТОРЫЙ НЕ УМЕЛ СКАЗАТЬ «НЕТ»

Лондон тонул в дневном тумане. Мостовая блестела, газовый рожок у редакции сипел, а сыщик «Вечернего Валидатора» получил папку из конторы [ton-blockchain/acton-contracts](https://github.com/ton-blockchain/acton-contracts) — мастерской контрактов TON на Tolk.

На обложке стояла деловая надпись коммита [`92225ba`](https://github.com/ton-blockchain/acton-contracts/commit/92225ba47468edde6c86e0a68218b27f4b721890): `feat: report mintable as false if admin is reset`. Внутри лежала сцена, достойная отдельной витрины: контракт умел навсегда снять права с администратора, но справочник все еще уверял публику, что монету можно чеканить. Владелец исчез, а вывеска у монетного двора продолжала мигать зеленым.

## МЕСТО ПРОИСШЕСТВИЯ

Дело лежит в родительском коммите [`5f131ea`](https://github.com/ton-blockchain/acton-contracts/commit/5f131ea6d3cd005ee0adba73b6670b4fe406f54c), файл [`jetton-v2.1/contracts/JettonMinter.tolk`](https://github.com/ton-blockchain/acton-contracts/blob/5f131ea6d3cd005ee0adba73b6670b4fe406f54c/jetton-v2.1/contracts/JettonMinter.tolk), строки [161–167](https://github.com/ton-blockchain/acton-contracts/blob/5f131ea6d3cd005ee0adba73b6670b4fe406f54c/jetton-v2.1/contracts/JettonMinter.tolk#L161-L167) и [191–201](https://github.com/ton-blockchain/acton-contracts/blob/5f131ea6d3cd005ee0adba73b6670b4fe406f54c/jetton-v2.1/contracts/JettonMinter.tolk#L191-L201). Цитаты сверены с содержимым файла на этом коммите.

Сначала контракт снимает корону с администратора:

```tolk
        DropMinterAdmin => {
            var storage = lazy MinterStorage.load();
            assertSenderIsAdmin(in.senderAddress, storage.adminAddress);
            storage.adminAddress = null;
            storage.nextAdminAddress = null;
            storage.save();
        }
```

А затем, на строках [191–201](https://github.com/ton-blockchain/acton-contracts/blob/5f131ea6d3cd005ee0adba73b6670b4fe406f54c/jetton-v2.1/contracts/JettonMinter.tolk#L191-L201), выдает справку о монете:

```tolk
get fun get_jetton_data(): JettonDataReply {
    val storage = lazy MinterStorage.load();
    val jettonWalletCode = jettonWalletCompiledCode();

    return {
        totalSupply: storage.totalSupply,
        mintable: true,
        adminAddress: storage.adminAddress,
        jettonContent: storage.metadata as Cell<OnchainMetadataReply>,
        jettonWalletCode,
    };
}
```

## ПЕРВЫЙ СЛЕД: АДМИНИСТРАТОР УШЕЛ, ЧЕКАНКА ОСТАЛАСЬ

На строках [161–167](https://github.com/ton-blockchain/acton-contracts/blob/5f131ea6d3cd005ee0adba73b6670b4fe406f54c/jetton-v2.1/contracts/JettonMinter.tolk#L161-L167) процедура `DropMinterAdmin` делает ровно то, что обещает название: проверяет прежнего хозяина, ставит `storage.adminAddress = null`, обнуляет запасной адрес и сохраняет состояние. Дворецкий закрывает кабинет и уносит ключи.

Но на строке [197](https://github.com/ton-blockchain/acton-contracts/blob/5f131ea6d3cd005ee0adba73b6670b4fe406f54c/jetton-v2.1/contracts/JettonMinter.tolk#L197) справка отвечает без тени сомнения: `mintable: true`. Не «зависит от состояния», не «администратор отсутствует», а безусловное «да». В поле [198](https://github.com/ton-blockchain/acton-contracts/blob/5f131ea6d3cd005ee0adba73b6670b4fe406f54c/jetton-v2.1/contracts/JettonMinter.tolk#L198) уже честно показывается `adminAddress`, который после сброса равен `null`. Рядом стоят две реплики: хозяина нет, чеканка якобы вечна.

Это не доказывает выпуск новых монет после сброса: сама проверка прав может остановить такую попытку. Но публичный getter сообщает о главном свойстве актива неверно. Индексатор, кошелек или обозреватель, читающий `get_jetton_data`, получает зеленый сигнал от монетного двора, где уже некому держать штемпель.

## ВТОРОЙ СЛЕД: ПРИЗНАНИЕ В КОММИТЕ

Исправляющий коммит [`92225ba`](https://github.com/ton-blockchain/acton-contracts/commit/92225ba47468edde6c86e0a68218b27f4b721890) прошел как [PR № 98](https://github.com/ton-blockchain/acton-contracts/pull/98) с тем же смыслом: `feat: report mintable as false if admin is reset`. В исправленном файле [`jetton-v2.1/contracts/JettonMinter.tolk`](https://github.com/ton-blockchain/acton-contracts/blob/92225ba47468edde6c86e0a68218b27f4b721890/jetton-v2.1/contracts/JettonMinter.tolk), строки [191–201](https://github.com/ton-blockchain/acton-contracts/blob/92225ba47468edde6c86e0a68218b27f4b721890/jetton-v2.1/contracts/JettonMinter.tolk#L191-L201), единственная строка сменила показания:

```tolk
get fun get_jetton_data(): JettonDataReply {
    val storage = lazy MinterStorage.load();
    val jettonWalletCode = jettonWalletCompiledCode();

    return {
        totalSupply: storage.totalSupply,
        mintable: storage.adminAddress != null,
        adminAddress: storage.adminAddress,
        jettonContent: storage.metadata as Cell<OnchainMetadataReply>,
        jettonWalletCode,
    };
}
```

Теперь справка сверяется с состоянием хранилища: если адрес администратора есть, поле истинно; если его сбросили, поле ложно. Монетный двор наконец научился отличать закрытую кассу от работающей.

## ТРЕТИЙ СЛЕД: ПОЧЕМУ ЭТО СОЧНО

Комедия выросла из соседства, а не из сложной арифметики. В одной палате лежит церемония окончательного отказа от власти — строки [161–167](https://github.com/ton-blockchain/acton-contracts/blob/5f131ea6d3cd005ee0adba73b6670b4fe406f54c/jetton-v2.1/contracts/JettonMinter.tolk#L161-L167). В другой, всего на несколько страниц ниже, сидит getter с постоянным `true` — строка [197](https://github.com/ton-blockchain/acton-contracts/blob/5f131ea6d3cd005ee0adba73b6670b4fe406f54c/jetton-v2.1/contracts/JettonMinter.tolk#L197). Контракт смотрит на пустое кресло и рапортует: «начальство на месте, станок готов».

Особенно выразителен контраст с полем [198](https://github.com/ton-blockchain/acton-contracts/blob/5f131ea6d3cd005ee0adba73b6670b4fe406f54c/jetton-v2.1/contracts/JettonMinter.tolk#L198). Состояние администратора передается наружу без прикрас, так что ложь не прячется в глубине машины. Она стоит в соседнем поле, напечатанная крупным шрифтом: `mintable: true`.

Редакция не станет называть [ton-blockchain/acton-contracts](https://github.com/ton-blockchain/acton-contracts) негодной конторой. [PR № 98](https://github.com/ton-blockchain/acton-contracts/pull/98) быстро вернул getter к смыслу состояния, а процедура сброса администратора сама по себе выглядела ясной. Но старый коммит [`5f131ea`](https://github.com/ton-blockchain/acton-contracts/commit/5f131ea6d3cd005ee0adba73b6670b4fe406f54c) оставил превосходную улику: права можно снять навсегда, а витрина еще некоторое время будет клясться, что чеканщик бессмертен.

## ВЕРДИКТ СЫЩИКА

[ton-blockchain/acton-contracts](https://github.com/ton-blockchain/acton-contracts) пойман на деле о монете с лживой справкой. В родительском коммите [`5f131ea`](https://github.com/ton-blockchain/acton-contracts/commit/5f131ea6d3cd005ee0adba73b6670b4fe406f54c) файл [`jetton-v2.1/contracts/JettonMinter.tolk`](https://github.com/ton-blockchain/acton-contracts/blob/5f131ea6d3cd005ee0adba73b6670b4fe406f54c/jetton-v2.1/contracts/JettonMinter.tolk) на строках [161–167](https://github.com/ton-blockchain/acton-contracts/blob/5f131ea6d3cd005ee0adba73b6670b4fe406f54c/jetton-v2.1/contracts/JettonMinter.tolk#L161-L167) снимал администратора, но getter на строке [197](https://github.com/ton-blockchain/acton-contracts/blob/5f131ea6d3cd005ee0adba73b6670b4fe406f54c/jetton-v2.1/contracts/JettonMinter.tolk#L197) все равно говорил `true`.

Потом коммит [`92225ba`](https://github.com/ton-blockchain/acton-contracts/commit/92225ba47468edde6c86e0a68218b27f4b721890) на строке [197](https://github.com/ton-blockchain/acton-contracts/blob/92225ba47468edde6c86e0a68218b27f4b721890/jetton-v2.1/contracts/JettonMinter.tolk#L197) поставил на место простую проверку `storage.adminAddress != null`. Приговор прост: если ключ от монетного двора выброшен в Темзу, справка не должна обещать рабочий станок.

*Сыщик еще раз сверил строки [161–167](https://github.com/ton-blockchain/acton-contracts/blob/5f131ea6d3cd005ee0adba73b6670b4fe406f54c/jetton-v2.1/contracts/JettonMinter.tolk#L161-L167) и [191–201](https://github.com/ton-blockchain/acton-contracts/blob/92225ba47468edde6c86e0a68218b27f4b721890/jetton-v2.1/contracts/JettonMinter.tolk#L191-L201), погас газовый рожок и растворился в тумане. В Лондоне вывеска может пережить хозяина. Но у блокчейн-контракта хотя бы справка обязана знать, кто держит штемпель.*

🐀
