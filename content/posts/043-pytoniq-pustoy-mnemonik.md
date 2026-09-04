+++
title = "№ 43 — Дело о пустом мнемонике: pytoniq отправляет кошелек в туман с пустыми карманами"
date = 2026-09-04T10:59:00+03:00
description = "Сорок третий выпуск «Вечернего Валидатора»: в примерах pytoniq кошельку передают пустой список мнемоники, после чего тот должен развернуться и отправить TON, а в одном файле за кулисами готовится тысяча переводов."
tags = ["pytoniq"]
+++

# 📰 ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

**№ 43 · Пятница, 4 сентября 2026 г. · Цена: 0.05 TON (мнемоника не выдана)**

---

## ДЕЛО О ПУСТОМ МНЕМОНИКЕ

Лондон встал серым, как терминал перед запуском неизвестного скрипта. Туман полз между домами TON, газовый рожок у редакции сипел, а сыщик «Вечернего Валидатора» получил папку из мастерской [yungwine/pytoniq](https://github.com/yungwine/pytoniq) — библиотеки, которая умеет общаться с сетью, собирать сообщения и управлять кошельками.

На обложке все выглядело прилично: тестовая сеть, клиент, кошелек, перевод. Но внутри папки лежал свидетель, у которого не было ни одного слова мнемоники.

Место происшествия: репозиторий [yungwine/pytoniq](https://github.com/yungwine/pytoniq), коммит [`135fdbd`](https://github.com/yungwine/pytoniq/commit/135fdbd01177e247d7a78d346d3777745ab9ab9d), файл [`examples/wallets/wallet.py`](https://github.com/yungwine/pytoniq/blob/135fdbd01177e247d7a78d346d3777745ab9ab9d/examples/wallets/wallet.py), строки [7–18](https://github.com/yungwine/pytoniq/blob/135fdbd01177e247d7a78d346d3777745ab9ab9d/examples/wallets/wallet.py#L7-L18). Цитата сверена с содержимым файла на этом коммите.

Протокол, строки [7–18](https://github.com/yungwine/pytoniq/blob/135fdbd01177e247d7a78d346d3777745ab9ab9d/examples/wallets/wallet.py#L7-L18):

```python
async def main():
    async with LiteBalancer.from_testnet_config(trust_level=2) as client:
        mnemo = []
        wallet = await WalletV4R2.from_mnemonic(client, mnemo)

        # deploy wallet if needed
        await wallet.deploy_via_external()

        await wallet.transfer(
            destination='EQAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAM9c',
            body='comment',
            amount=1 * 10**8 # 0.1 TON
        )
```

На строке [9](https://github.com/yungwine/pytoniq/blob/135fdbd01177e247d7a78d346d3777745ab9ab9d/examples/wallets/wallet.py#L9) лежит весь запас памяти: `mnemo = []`. На следующей строке [10](https://github.com/yungwine/pytoniq/blob/135fdbd01177e247d7a78d346d3777745ab9ab9d/examples/wallets/wallet.py#L10) этот пустой список торжественно вручают `WalletV4R2.from_mnemonic`.

Кошелек, разумеется, не получает секретную фразу из воздуха. Но пример не останавливается у гардероба и не говорит читателю: «Вставьте свои слова». Он идет дальше к deploy на строке [13](https://github.com/yungwine/pytoniq/blob/135fdbd01177e247d7a78d346d3777745ab9ab9d/examples/wallets/wallet.py#L13), а затем готовит перевод на строках [15–18](https://github.com/yungwine/pytoniq/blob/135fdbd01177e247d7a78d346d3777745ab9ab9d/examples/wallets/wallet.py#L15-L18). Газовый рожок кашлянул: у джентльмена нет паспорта, но уже заказана карета.

## КОШЕЛЕК, КОТОРОМУ НЕЛЬЗЯ ВЫДАТЬ КЛЮЧ

Сыщик проверил соседние комнаты. В файле [`examples/wallets/wallet_v5.py`](https://github.com/yungwine/pytoniq/blob/135fdbd01177e247d7a78d346d3777745ab9ab9d/examples/wallets/wallet_v5.py) на строке [10](https://github.com/yungwine/pytoniq/blob/135fdbd01177e247d7a78d346d3777745ab9ab9d/examples/wallets/wallet_v5.py#L10) снова стоит `mnemo = []`, а строка [11](https://github.com/yungwine/pytoniq/blob/135fdbd01177e247d7a78d346d3777745ab9ab9d/examples/wallets/wallet_v5.py#L11) отправляет пустоту в кошелек пятого поколения. Дальше, на строках [13–19](https://github.com/yungwine/pytoniq/blob/135fdbd01177e247d7a78d346d3777745ab9ab9d/examples/wallets/wallet_v5.py#L13-L19), повторяется тот же маршрут: deploy, transfer и 0.1 TON.

Это не одинокая опечатка, затерявшаяся в старом черновике. В [`examples/extra_currency.py`](https://github.com/yungwine/pytoniq/blob/135fdbd01177e247d7a78d346d3777745ab9ab9d/examples/extra_currency.py) строка [9](https://github.com/yungwine/pytoniq/blob/135fdbd01177e247d7a78d346d3777745ab9ab9d/examples/extra_currency.py#L9) снова задает пустую мнемонику, а строка [10](https://github.com/yungwine/pytoniq/blob/135fdbd01177e247d7a78d346d3777745ab9ab9d/examples/extra_currency.py#L10) пытается создать кошелек для работы с дополнительной валютой.

Справедливости ради, библиотека внутри умеет проверять мнемонику. Значит, при настоящем запуске этот пример, вероятно, споткнется на проверке раньше, чем дойдет до перевода. Но именно в этом и состоит дичь постановки: файл выглядит как готовая инструкция, содержит настоящий `LiteBalancer`, настоящий адрес и настоящий вызов отправки, а единственный обязательный материал заменен пустым списком.

Учебный код обязан быть исполнимым или громко сообщать, где вставить входные данные. Иначе новичок получает не пример, а викторианскую дверь без ручки: толкает ее, слышит скрежет ошибки и долго думает, виновата ли сеть.

## ТЫСЯЧА ПИСЕМ В НИКУДА

Сыщик перелистнул папку [`examples/extra_currency.py`](https://github.com/yungwine/pytoniq/blob/135fdbd01177e247d7a78d346d3777745ab9ab9d/examples/extra_currency.py). На строке [56](https://github.com/yungwine/pytoniq/blob/135fdbd01177e247d7a78d346d3777745ab9ab9d/examples/extra_currency.py#L56) пустой `mnemo` встречается второй раз, а строка [57](https://github.com/yungwine/pytoniq/blob/135fdbd01177e247d7a78d346d3777745ab9ab9d/examples/extra_currency.py#L57) готовит highload-кошелек.

Затем на строке [55](https://github.com/yungwine/pytoniq/blob/135fdbd01177e247d7a78d346d3777745ab9ab9d/examples/extra_currency.py#L55) появляется пояснение: программа хочет послать **1000 сообщений** на случайные адреса. Строки [62–68](https://github.com/yungwine/pytoniq/blob/135fdbd01177e247d7a78d346d3777745ab9ab9d/examples/extra_currency.py#L62-L68) собирают эти письма и отправляют их одним вызовом.

```python
        # send 1000 messages to random addresses, each message has 1 nano EC and 0 ton attached with empty body
        mnemo = []
        wallet = await HighloadWalletV3.from_mnemonic(client, mnemo)

        currency_id = 100
        amount = 1

        value = CurrencyCollection(grams=0, other=ExtraCurrencyCollection({currency_id: amount}))
        msgs = []
        for i in range(1000):
            message = wallet.create_internal_msg(dest=Address((0, get_random(32))), value=value, body=Cell.empty())
            msg = WalletMessage(send_mode=3, message=message)
            msgs.append(msg)
        await wallet.raw_transfer(msgs=msgs)
```

Здесь комедия уже с оркестром. Пустой список на строке [56](https://github.com/yungwine/pytoniq/blob/135fdbd01177e247d7a78d346d3777745ab9ab9d/examples/extra_currency.py#L56) соседствует не с безобидным `print`, а с highload-операцией. Цикл на строке [64](https://github.com/yungwine/pytoniq/blob/135fdbd01177e247d7a78d346d3777745ab9ab9d/examples/extra_currency.py#L64) разводит тысячу адресов, выбранных случайно, а строка [68](https://github.com/yungwine/pytoniq/blob/135fdbd01177e247d7a78d346d3777745ab9ab9d/examples/extra_currency.py#L68) просит кошелек все это подписать и отправить.

Да, это пример с дополнительной валютой, не тайный кран для раздачи TON. Но новичок, увидев «пустую мнемонику» и «1000 сообщений», вряд ли назовет сцену образцом бережного обучения. Сначала кошелек без слов, потом случайные адреса, затем тысяча писем в ночь. Лондонский почтальон уволился бы на первой сотне.

## ВЕРДИКТ СЫЩИКА

[pytoniq](https://github.com/yungwine/pytoniq) — полезная мастерская для TON, а не злодей из подземелья. Но коммит [`135fdbd`](https://github.com/yungwine/pytoniq/commit/135fdbd01177e247d7a78d346d3777745ab9ab9d) оставил сочную улику: в файле [`examples/wallets/wallet.py`](https://github.com/yungwine/pytoniq/blob/135fdbd01177e247d7a78d346d3777745ab9ab9d/examples/wallets/wallet.py) на строке [9](https://github.com/yungwine/pytoniq/blob/135fdbd01177e247d7a78d346d3777745ab9ab9d/examples/wallets/wallet.py#L9) мнемоника пуста, но строки [13–18](https://github.com/yungwine/pytoniq/blob/135fdbd01177e247d7a78d346d3777745ab9ab9d/examples/wallets/wallet.py#L13-L18) уже ведут кошелек к deploy и переводу. В [`examples/extra_currency.py`](https://github.com/yungwine/pytoniq/blob/135fdbd01177e247d7a78d346d3777745ab9ab9d/examples/extra_currency.py) та же пустота на строке [56](https://github.com/yungwine/pytoniq/blob/135fdbd01177e247d7a78d346d3777745ab9ab9d/examples/extra_currency.py#L56) стоит перед highload-рассылкой на тысячу адресов на строках [64–68](https://github.com/yungwine/pytoniq/blob/135fdbd01177e247d7a78d346d3777745ab9ab9d/examples/extra_currency.py#L64-L68).

Приговор редакции прост: пример кошелька должен принимать мнемонику из переменной окружения, явно помечать заглушку и не выглядеть готовым к отправке. Если демонстрация требует ручной вставки секрета, пусть она говорит об этом крупно, а не прячет отсутствие слов в двух символах `[]`.

*Сыщик закрыл папку, высыпал пустые карманы на стол и погасил газовую лампу. В тумане Лондона пропавшая мнемоника так и не нашлась. Зато тысяча писем уже стояла в очереди.*

🐀
