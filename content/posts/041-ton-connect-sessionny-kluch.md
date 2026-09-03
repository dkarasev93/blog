+++
title = "№ 41 — Дело о признании: TON Connect выдает секретный ключ на допросе"
date = 2026-09-03T11:00:22+03:00
description = "Сорок первый выпуск «Вечернего Валидатора»: в протоколе ton-connect/sdk ошибка расшифровки включает приватный ключ сессии в текст исключения и отправляет его прямо в логи."
tags = ["ton-connect"]
+++

# 📰 ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

**№ 41 · Четверг, 3 сентября 2026 г. · Цена: 0.05 TON (за признание доплата не взимается)**

---

## ДЕЛО О ПРИЗНАНИИ, КОТОРОЕ НИКТО НЕ ПРОСИЛ

Лондон тонул в тумане, а газовый рожок у редакции кашлял так, будто сам получил сообщение от моста. Сыщик «Вечернего Валидатора» шел по следу [ton-connect/sdk](https://github.com/ton-connect/sdk) — солидной мастерской, где dApp и кошельки обмениваются зашифрованными посланиями. Внутри у нее живет NaCl, ключи и строгий протокол. Все, казалось, было заперто.

Место происшествия: репозиторий [ton-connect/sdk](https://github.com/ton-connect/sdk), коммит [`273bc3a`](https://github.com/ton-connect/sdk/commit/273bc3a6050e6024886ca50c12677dc42ae142a9), файл [`packages/protocol/src/crypto/session-crypto.ts`](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/protocol/src/crypto/session-crypto.ts), строки [105–108](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/protocol/src/crypto/session-crypto.ts#L105-L108). Цитата сверена с содержимым файла на этом коммите.

Сыщик вошел в комнату расшифровки. Здесь входящее сообщение проверяют через `nacl.box.open`. Если послание подделано, испорчено или пришло не с тем ключом, страж должен поднять тревогу. Но вместо короткого «расшифровка не удалась» контора решила провести полный допрос — с предоставлением всех улик, включая ту, которую не следовало показывать никому.

Протокол, строки [105–108](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/protocol/src/crypto/session-crypto.ts#L105-L108):

```ts
        if (!decrypted) {
            throw new Error(
                `Decryption error: \n message: ${message.toString()} \n sender pubkey: ${senderPublicKey.toString()} \n keypair pubkey: ${this.keyPair.publicKey.toString()} \n keypair secretkey: ${this.keyPair.secretKey.toString()}`
            );
        }
```

На строке [107](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/protocol/src/crypto/session-crypto.ts#L107) подозреваемый произносит вслух: `keypair secretkey`. Не хеш, не отпечаток, не скромная метка для дежурного. Сам секретный ключ превращается в часть текста ошибки.

## КАБИНЕТ, ГДЕ СЕКРЕТЫ ЛЮБЯТ ПУБЛИКУ

Чтобы оценить размах сцены, сыщик поднялся по лестнице к месту хранения ключей. В том же файле [packages/protocol/src/crypto/session-crypto.ts](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/protocol/src/crypto/session-crypto.ts) класс `SessionCrypto` держит пару ключей в приватном поле. Его описание находится на строке [40](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/protocol/src/crypto/session-crypto.ts#L40): там `keyPair` указано как внутреннее свойство класса.

Сама операция расшифровки проходит через [`decrypt`](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/protocol/src/crypto/session-crypto.ts#L95-L112), строки [98–103](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/protocol/src/crypto/session-crypto.ts#L98-L103): библиотека передает в `nacl.box.open` сообщение, nonce, публичный ключ отправителя и `this.keyPair.secretKey`. То есть ключ нужен для дела, но при неудаче его еще и приглашают на печать.

На строке [106](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/protocol/src/crypto/session-crypto.ts#L106) рождается исключение. Дальше оно может попасть в консоль браузера, систему сбора ошибок, отчет разработчика или любой иной карман, куда приложение складывает детали падений. Никто не обязан злонамеренно взламывать сейф: достаточно обычного лога, отправленного в привычный сервис наблюдения.

Особенно театральна соседняя улика. На строках [100–102](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/protocol/src/crypto/session-crypto.ts#L100-L102) присутствует публичный ключ отправителя и секретный ключ текущей пары. Первый можно обсуждать в гостиной. Второй следует держать за семью замками. Но строка [107](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/protocol/src/crypto/session-crypto.ts#L107) усаживает их рядом за один стол, будто они равноправные джентльмены.

## ДВА КЛЮЧА, ОДНА ПЕЧАЛЬНАЯ СТРОКА

В [ton-connect/sdk](https://github.com/ton-connect/sdk) есть и штатный путь экспорта пары для продолжения сессии. Метод [`stringifyKeypair`](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/protocol/src/crypto/session-crypto.ts#L118-L123) возвращает публичный и секретный ключи как hex-строки, строки [118–122](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/protocol/src/crypto/session-crypto.ts#L118-L122). Это уже осознанная операция хранения или восстановления сессии.

Но при обычной ошибке расшифровки никто не ожидает получить такой подарок. Пользователь не нажимает «экспортировать ключ». Кошелек не просит «покажите секрет». Просто пришло сообщение, которое не удалось открыть, и аварийный механизм печатает на свет то, что должен был защищать.

Сыщик не утверждает, что каждая ошибка немедленно уносит ключ в руки первого карманника. Но риск здесь не литературный. Если исключение окажется в логе, снимке состояния или системе аналитики, секрет покинет границы криптографической комнаты. Для сессионного ключа это уже не мелкая неловкость, а приглашение к чтению сообщений, адресованных этой сессии.

Ирония в том, что комментарий над [`decrypt`](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/protocol/src/crypto/session-crypto.ts#L90-L94) сам описывает приличное поведение: метод «Throws if `nacl.box.open` rejects the message». Ошибка должна сообщить о провале проверки. Она не обязана сдавать весь гардероб свидетеля, включая ключ от задней двери.

## ВЕРДИКТ СЫЩИКА

[ton-connect/sdk](https://github.com/ton-connect/sdk) — не темная лавка, а важная часть связки кошельков и dApp. Тем сочнее улика в коммите [`273bc3a`](https://github.com/ton-connect/sdk/commit/273bc3a6050e6024886ca50c12677dc42ae142a9): в файле [`packages/protocol/src/crypto/session-crypto.ts`](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/protocol/src/crypto/session-crypto.ts) строки [98–103](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/protocol/src/crypto/session-crypto.ts#L98-L103) используют секрет для расшифровки, а строка [107](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/protocol/src/crypto/session-crypto.ts#L107) отправляет его в исключение.

Приговор прост: сообщение об ошибке может содержать тип сбоя, длину послания или безопасные отпечатки ключей, но не сам секрет. Газовый рожок редакции требует убрать `keypair secretkey` из текста и проверить, куда уходят старые ошибки. В криптографии плохой допрос опаснее молчаливого подозрения.

*Сыщик закрыл папку, запер ключ в ящике и велел логам держать язык за зубами. За окном Лондон продолжал тонуть в тумане, но на этот раз туман был безопаснее отладочного сообщения.*

🐀
