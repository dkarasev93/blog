+++
title = "№ 74 — Кошелек и seed-фраза на витрине"
date = 2026-09-20T16:58:00+03:00
description = "Семьдесят четвертый выпуск «Вечернего Валидатора»: TON-кошелек показывает в Quick Start готовую seed-фразу, а рядом обещает безопасное управление ключами."
tags = ["wdk-wallet-ton"]
+++

# ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

**№ 74.** *Лондон. Туман сполз с крыш, газовый рожок у редакции сипит, а сыщик получил рекламный листок из новой кошельковой конторы. На фасаде обещают безопасность, некастодиальность и даже очистку приватных ключей из памяти. Но в приемной, прямо под заголовком Quick Start, лежит готовая seed-фраза. Не в тестовом приложении, не в закрытом сейфе, а на первой странице для каждого прохожего.*

## МЕСТО ПРОИСШЕСТВИЯ

Репозиторий [tetherto/wdk-wallet-ton](https://github.com/tetherto/wdk-wallet-ton), коммит [`1589b9e`](https://github.com/tetherto/wdk-wallet-ton/commit/1589b9ea4d712294ab1ca2b21b4d01887d8c6ffe), файл [`README.md`](https://github.com/tetherto/wdk-wallet-ton/blob/1589b9ea4d712294ab1ca2b21b4d01887d8c6ffe/README.md), строки [24–43](https://github.com/tetherto/wdk-wallet-ton/blob/1589b9ea4d712294ab1ca2b21b4d01887d8c6ffe/README.md#L24-L43). Цитата сверена по содержимому файла на этом коммите.

Дословная инструкция с витрины:

```javascript
import WalletManagerTon from '@tetherto/wdk-wallet-ton'

const seedPhrase = 'abandon abandon abandon abandon abandon abandon abandon abandon abandon abandon abandon about'

const wallet = new WalletManagerTon(seedPhrase, {
  tonClient: {
    url: 'https://testnet.toncenter.com/api/v2/jsonRPC',
  },
  transferMaxFee: 1_000_000_000n,
})

const account = await wallet.getAccount(0)
const address = await account.getAddress()
console.log('Address:', address)

wallet.dispose()
```

## ПЕРВЫЙ СЛЕД: ДВЕНАДЦАТЬ ДВЕРЕЙ И ОДИН КЛЮЧ

На строке [29](https://github.com/tetherto/wdk-wallet-ton/blob/1589b9ea4d712294ab1ca2b21b4d01887d8c6ffe/README.md#L29) контора вписывает в переменную `seedPhrase` фразу из стандартного набора слов. Сыщик не станет изображать незнание: это знаменитая демонстрационная фраза BIP-39, пригодная для примеров. Но кошелек, который обещает работу с настоящими счетами, показывает ее в своем главном быстром старте без крупной таблички «только учебный макет».

В этот момент читатель уже получил материал, достаточный для воспроизведения кошелька. Строка [31](https://github.com/tetherto/wdk-wallet-ton/blob/1589b9ea4d712294ab1ca2b21b4d01887d8c6ffe/README.md#L31) передает фразу в конструктор `WalletManagerTon`, а строки [32–35](https://github.com/tetherto/wdk-wallet-ton/blob/1589b9ea4d712294ab1ca2b21b4d01887d8c6ffe/README.md#L32-L35) добавляют адрес тестовой сети и лимит комиссии. Все выглядит как готовая инструкция, а не как коробка с нарисованным замком.

Газовый рожок хрипнул. Любой читатель может скопировать пять строк, установить пакет и получить тот же детерминированный набор ключей. Если такой адрес где-то использован в учебном стенде, его секрет уже не секрет. Если кто-то по ошибке отправит туда средства, городские голуби узнают об этом раньше владельца.

## ВТОРОЙ СЛЕД: БЕЗОПАСНОСТЬ В СОСЕДНЕМ ОКНЕ

На странице того же [репозитория](https://github.com/tetherto/wdk-wallet-ton) список возможностей обещает «Secure Memory Disposal» и очистку приватных ключей из памяти. Это достойная мера, но она не спасает от другой двери: секрет уже напечатан до запуска программы.

В [README.md](https://github.com/tetherto/wdk-wallet-ton/blob/1589b9ea4d712294ab1ca2b21b4d01887d8c6ffe/README.md) после создания кошелька идут строки [38–42](https://github.com/tetherto/wdk-wallet-ton/blob/1589b9ea4d712294ab1ca2b21b4d01887d8c6ffe/README.md#L38-L42): приложение получает аккаунт, читает адрес и вызывает `wallet.dispose()`. Сцена поставлена с заботой о гигиене памяти. Только ключ от показанного кошелька уже выдан всему Лондону строкой [29](https://github.com/tetherto/wdk-wallet-ton/blob/1589b9ea4d712294ab1ca2b21b4d01887d8c6ffe/README.md#L29).

Получается странная викторианская охрана. Клерк в конце визита сжигает записку в камине, но перед этим приколачивает ее копию к двери. Очистка памяти работает внутри процесса, однако она не может отменить публикацию seed-фразы в документации.

## ТРЕТИЙ СЛЕД: ТЕСТОВАЯ ВЫВЕСКА НЕ ЗАКРЫВАЕТ КОШЕЛЕК

Есть важная деталь в пользу конторы: URL на строке [33](https://github.com/tetherto/wdk-wallet-ton/blob/1589b9ea4d712294ab1ca2b21b4d01887d8c6ffe/README.md#L33) ведет в testnet. Это снижает риск случайной потери настоящих средств именно в показанном примере. Но seed-фраза остается публичной, а код задает полноценный кошелек, получает из него аккаунт и выводит адрес на строках [38–40](https://github.com/tetherto/wdk-wallet-ton/blob/1589b9ea4d712294ab1ca2b21b4d01887d8c6ffe/README.md#L38-L40).

Кроме того, соседняя строка [10](https://github.com/tetherto/wdk-wallet-ton/blob/1589b9ea4d712294ab1ca2b21b4d01887d8c6ffe/README.md#L10) описывает пакет как инструмент для управления BIP-44-кошельками TON, а строка [8](https://github.com/tetherto/wdk-wallet-ton/blob/1589b9ea4d712294ab1ca2b21b4d01887d8c6ffe/README.md#L8) сообщает лишь, что пакет находится в beta. Ни одна из этих строк не превращает фразу из Quick Start в безопасный секрет.

Сыщик не станет кричать о краже средств: перед нами демонстрационные данные, и тестовая сеть указана открыто. Но газетная улика сочна по другой причине. Документация знакомит публику с API кошелька через секрет, который выглядит настолько настоящим, что его легко унести в первый же рабочий пример.

## ЧЕТВЕРТЫЙ СЛЕД: КОММИТ С УБРАННОЙ ПЫЛЬЮ

Коммит [`1589b9e`](https://github.com/tetherto/wdk-wallet-ton/commit/1589b9ea4d712294ab1ca2b21b4d01887d8c6ffe) носит мирное имя `docs: simplify README to Option A structure`. Бумаги в нем действительно стали короче: старую витрину убрали, Quick Start собрали заново, а вместе с новой простотой наружу вышла демонстрационная фраза.

Файл [`README.md`](https://github.com/tetherto/wdk-wallet-ton/blob/1589b9ea4d712294ab1ca2b21b4d01887d8c6ffe/README.md) на строках [24–35](https://github.com/tetherto/wdk-wallet-ton/blob/1589b9ea4d712294ab1ca2b21b4d01887d8c6ffe/README.md#L24-L35) устроен без ловушек и лишних церемоний: импорт, фраза, конструктор, testnet, лимит комиссии. Именно поэтому улика хорошо видна. Чем легче путь от чтения до копирования, тем важнее крупно пояснить, что ключ учебный и не должен жить ни в одном настоящем кошельке.

## ХРОНИКА МЕЛКОГО КРИНЖА

*Отдел ключей.* Строка [29](https://github.com/tetherto/wdk-wallet-ton/blob/1589b9ea4d712294ab1ca2b21b4d01887d8c6ffe/README.md#L29) называет секрет `seedPhrase`, но не добавляет ни комментария, ни предупреждения о демонстрационном характере значения.

*Отдел тестовой сети.* Строка [33](https://github.com/tetherto/wdk-wallet-ton/blob/1589b9ea4d712294ab1ca2b21b4d01887d8c6ffe/README.md#L33) честно указывает testnet. Это хороший щит от одной беды, но не лицензия переносить фразу в production.

*Отдел финального поклона.* Строка [42](https://github.com/tetherto/wdk-wallet-ton/blob/1589b9ea4d712294ab1ca2b21b4d01887d8c6ffe/README.md#L42) зовет `wallet.dispose()`. Кошелек закрывает за собой дверь, пока его запасной ключ уже висит на фасаде.

## ВЕРДИКТ СЫЩИКА

Репозиторий [tetherto/wdk-wallet-ton](https://github.com/tetherto/wdk-wallet-ton) пойман на редком контрасте. Коммит [`1589b9e`](https://github.com/tetherto/wdk-wallet-ton/commit/1589b9ea4d712294ab1ca2b21b4d01887d8c6ffe) добавил в файл [`README.md`](https://github.com/tetherto/wdk-wallet-ton/blob/1589b9ea4d712294ab1ca2b21b4d01887d8c6ffe/README.md) короткую и понятную сцену запуска: строки [29–35](https://github.com/tetherto/wdk-wallet-ton/blob/1589b9ea4d712294ab1ca2b21b4d01887d8c6ffe/README.md#L29-L35) показывают seed-фразу и создают кошелек, а строки [38–42](https://github.com/tetherto/wdk-wallet-ton/blob/1589b9ea4d712294ab1ca2b21b4d01887d8c6ffe/README.md#L38-L42) получают аккаунт, печатают адрес и вызывают очистку.

Это не доказательство уязвимости самого алгоритма и не обвинение testnet в преступлении. Это плохая привычка в парадном входе: секрет, пусть даже учебный, выглядит как готовый рецепт, а оговорка о его безвредности отсутствует. Достаточно заменить фразу на явно выдуманную, написать `DEMO_ONLY` и пояснить назначение прямо возле строки [29](https://github.com/tetherto/wdk-wallet-ton/blob/1589b9ea4d712294ab1ca2b21b4d01887d8c6ffe/README.md#L29) — и газовый рожок уже не будет кашлять.

Сыщик закрыл папку. На стекле [README.md](https://github.com/tetherto/wdk-wallet-ton/blob/1589b9ea4d712294ab1ca2b21b4d01887d8c6ffe/README.md) осталась надпись из строки [29](https://github.com/tetherto/wdk-wallet-ton/blob/1589b9ea4d712294ab1ca2b21b4d01887d8c6ffe/README.md#L29): `abandon abandon abandon abandon abandon abandon abandon abandon abandon abandon abandon about`. В Лондоне ключи принято держать в кармане. Даже если дверь ведет в тестовую сеть.

🐀
