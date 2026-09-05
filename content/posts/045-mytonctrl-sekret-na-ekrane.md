+++
title = "№ 45 — Дело о ключе на экране: mytonctrl экспортирует сейф в терминал"
date = 2026-09-05T10:58:00+03:00
description = "Сорок пятый выпуск «Вечернего Валидатора»: команда экспорта кошелька в mytonctrl читает файл с приватными данными, кодирует его в Base64 и печатает под вывеской Secret key."
tags = ["mytonctrl"]
+++

# 📰 ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

**№ 45 · Суббота, 5 сентября 2026 г. · Цена: 0.05 TON (сейф в терминал не входит)**

---

## ДЕЛО О КЛЮЧЕ, КОТОРЫЙ ВЫШЕЛ НА ПРОГУЛКУ

Лондон просыпался в тумане, мостовая блестела, а газовый рожок у редакции сипел, словно увидел счет за охрану. Сыщик «Вечернего Валидатора» получил папку из конторы [ton-blockchain/mytonctrl](https://github.com/ton-blockchain/mytonctrl) — панели управления валидатором и кошельками TON. Внутри лежала маленькая, но очень шумная улика: штатная команда экспорта берет данные из файла кошелька и выводит их в терминал как `Secret key`.

Место происшествия: репозиторий [ton-blockchain/mytonctrl](https://github.com/ton-blockchain/mytonctrl), коммит [`92b2a87`](https://github.com/ton-blockchain/mytonctrl/commit/92b2a87fe8cbbec6430ee868e7c82c7322eb9f50), файл [`modules/wallet.py`](https://github.com/ton-blockchain/mytonctrl/blob/92b2a87fe8cbbec6430ee868e7c82c7322eb9f50/modules/wallet.py), строки [145–159](https://github.com/ton-blockchain/mytonctrl/blob/92b2a87fe8cbbec6430ee868e7c82c7322eb9f50/modules/wallet.py#L145-L159). Цитата сверена с содержимым файла на этом коммите.

Протокол, строки [145–159](https://github.com/ton-blockchain/mytonctrl/blob/92b2a87fe8cbbec6430ee868e7c82c7322eb9f50/modules/wallet.py#L145-L159):

```python
    def do_export_wallet(self, wallet_name):
        wallet = self.ton.GetLocalWallet(wallet_name)
        with open(wallet.privFilePath, 'rb') as file:
            data = file.read()
        key = base64.b64encode(data).decode("utf-8")
        return wallet.addrB64, key

    def export_wallet(self, args):
        if not check_usage_one_arg("ew", args):
            return
        name = args[0]
        addr, key = self.do_export_wallet(name)
        print("Wallet name:", name)
        print("Address:", addr)
        print("Secret key:", key)
```

На строке [146](https://github.com/ton-blockchain/mytonctrl/blob/92b2a87fe8cbbec6430ee868e7c82c7322eb9f50/modules/wallet.py#L146) команда получает локальный кошелек. Строки [147–148](https://github.com/ton-blockchain/mytonctrl/blob/92b2a87fe8cbbec6430ee868e7c82c7322eb9f50/modules/wallet.py#L147-L148) открывают `wallet.privFilePath` в режиме чтения байтов и забирают все содержимое файла. Это не публичный адрес и не справочная карточка: название поля прямо намекает на приватный материал.

Затем строка [149](https://github.com/ton-blockchain/mytonctrl/blob/92b2a87fe8cbbec6430ee868e7c82c7322eb9f50/modules/wallet.py#L149) превращает прочитанные байты в Base64. Само кодирование не создает защиту, а лишь меняет вид данных, чтобы их было удобно носить по текстовым каналам. Газовый рожок кашлянул: сейф перекрасили в белый цвет и назвали его конвертом.

## ЭКСПОРТ, КОТОРЫЙ ПОКАЗЫВАЕТ ВСЕ

Сыщик проверил, куда уходит добыча. Строка [150](https://github.com/ton-blockchain/mytonctrl/blob/92b2a87fe8cbbec6430ee868e7c82c7322eb9f50/modules/wallet.py#L150) возвращает вместе адрес кошелька и строку с кодированными данными. Для внутреннего обмена это может быть полезная пара значений. Но дальше начинается публичное заседание.

Строки [152–156](https://github.com/ton-blockchain/mytonctrl/blob/92b2a87fe8cbbec6430ee868e7c82c7322eb9f50/modules/wallet.py#L152-L156) принимают один аргумент, вызывают `do_export_wallet` и получают `addr` вместе с `key`. Никакого отдельного режима для тихого вывода, никакого предупреждения перед печатью, никакого вопроса «точно ли вы хотите показать приватные данные» в этой сцене не видно.

На строке [157](https://github.com/ton-blockchain/mytonctrl/blob/92b2a87fe8cbbec6430ee868e7c82c7322eb9f50/modules/wallet.py#L157) терминал получает имя кошелька. Строка [158](https://github.com/ton-blockchain/mytonctrl/blob/92b2a87fe8cbbec6430ee868e7c82c7322eb9f50/modules/wallet.py#L158) печатает адрес. До этого места публика вполне спокойна: имя и адрес можно показывать без паники.

Но строка [159](https://github.com/ton-blockchain/mytonctrl/blob/92b2a87fe8cbbec6430ee868e7c82c7322eb9f50/modules/wallet.py#L159) торжественно произносит `Secret key:` и следом выводит весь `key`. То есть команда с коротким именем экспорта не просто готовит данные для передачи в другую процедуру. Она пишет приватный материал на экран, где его увидят история терминала, журнал сеанса, запись экрана или случайный свидетель, заглянувший через плечо.

Редакция не утверждает, что каждая машина с этой командой немедленно теряет средства. Мы также не видим здесь сам файл кошелька и не знаем, запускалась ли команда на рабочем сервере. Но архитектура жеста предельно ясна: секретное содержимое читают целиком, кодируют обратимым способом и называют своим настоящим именем прямо в выводе.

## БУМАЖНЫЙ ПЛАТЬЕР ДЛЯ ПРИВАТНОГО МАТЕРИАЛА

Особенно мрачно выглядит контраст между адресом и ключом. Адрес на строке [158](https://github.com/ton-blockchain/mytonctrl/blob/92b2a87fe8cbbec6430ee868e7c82c7322eb9f50/modules/wallet.py#L158) — визитная карточка, ее можно отправить собеседнику. Значение на строке [159](https://github.com/ton-blockchain/mytonctrl/blob/92b2a87fe8cbbec6430ee868e7c82c7322eb9f50/modules/wallet.py#L159) — уже отмычка. Но оба предмета выходят из одной команды и ложатся на одну строку терминала, без перегородки и красной ленты.

Проблема не только в злонамеренном администраторе. Секрет может остаться в scrollback, попасть в сбор логов, оказаться на скриншоте для отчета или угодить в историю оболочки, если вокруг команды работает неаккуратная обвязка. Base64 на строке [149](https://github.com/ton-blockchain/mytonctrl/blob/92b2a87fe8cbbec6430ee868e7c82c7322eb9f50/modules/wallet.py#L149) не меняет этот риск: любой, кто получил строку, может декодировать ее.

Сыщик признает смягчающее обстоятельство. Иногда оператору действительно нужно перенести зашифрованный файл или экспортировать кошелек для резервной копии. Но тогда безопаснее отдавать путь к файлу, писать данные в отдельный файл с понятными правами или требовать явный флаг вроде `--show-secret` с громким предупреждением. Команда по умолчанию не должна превращать терминал в витрину.

## ВЕРДИКТ СЫЩИКА

[ton-blockchain/mytonctrl](https://github.com/ton-blockchain/mytonctrl) — нужная контора для обслуживания TON, не злодей из подземелья. Но коммит [`92b2a87`](https://github.com/ton-blockchain/mytonctrl/commit/92b2a87fe8cbbec6430ee868e7c82c7322eb9f50) оставил сочную улику в файле [`modules/wallet.py`](https://github.com/ton-blockchain/mytonctrl/blob/92b2a87fe8cbbec6430ee868e7c82c7322eb9f50/modules/wallet.py): строки [147–149](https://github.com/ton-blockchain/mytonctrl/blob/92b2a87fe8cbbec6430ee868e7c82c7322eb9f50/modules/wallet.py#L147-L149) читают приватный файл и кодируют его в Base64, а строки [157–159](https://github.com/ton-blockchain/mytonctrl/blob/92b2a87fe8cbbec6430ee868e7c82c7322eb9f50/modules/wallet.py#L157-L159) печатают имя, адрес и `Secret key` в терминал.

Приговор редакции прост: экспорт приватного материала должен быть отдельной, явно опасной операцией. По умолчанию показывайте адрес и сохраняйте секрет в защищенный файл, а не отправляйте его гулять по журналам. Газовый рожок требует хотя бы предупреждения перед строкой [159](https://github.com/ton-blockchain/mytonctrl/blob/92b2a87fe8cbbec6430ee868e7c82c7322eb9f50/modules/wallet.py#L159). Иначе викторианский сейф служит лишь красивой коробкой с окошком.

*Сыщик закрыл папку, вытер с терминала отпечатки Base64 и погас газовую лампу. В Лондоне ключ можно вынести из сейфа. Но не стоит заодно прикреплять к нему табличку с надписью «смотрите все».*

🐀
