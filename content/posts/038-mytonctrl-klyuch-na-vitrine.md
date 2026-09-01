+++
title = "№ 38 — Витрина с ключом: MyTonCtrl экспортирует секрет и печатает его в газовый рожок"
date = 2026-09-01T16:58:00+03:00
description = "Тридцать восьмой выпуск «Вечернего Валидатора»: в MyTonCtrl команда экспорта читает приватный файл кошелька, кодирует его в Base64 и печатает строкой Secret key; рядом резервная служба отдает все приватные ключи валидатора одной командой."
tags = ["mytonctrl"]
+++

# 📰 ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

**№ 38 · Вторник, 1 сентября 2026 г. · Цена: 0.05 TON (ключ в комплект не входит)**

---

## ВИТРИНА С КЛЮЧОМ: КОГДА СЕКРЕТ ВЫХОДИТ НА ПРОГУЛКУ

Туман над Лондоном в этот вечер был густ, как лог после ночного обновления. Сыщик шел вдоль домов TON и сверял вывески. На одной из них значилось: [ton-blockchain/mytonctrl](https://github.com/ton-blockchain/mytonctrl) — контора, где валидатору помогают жить, голосовать, обновляться и не падать в канаву.

Место происшествия: [ton-blockchain/mytonctrl](https://github.com/ton-blockchain/mytonctrl), коммит [`92b2a87`](https://github.com/ton-blockchain/mytonctrl/commit/92b2a87fe8cbbec6430ee868e7c82c7322eb9f50) ([полный SHA](https://github.com/ton-blockchain/mytonctrl/commit/92b2a87fe8cbbec6430ee868e7c82c7322eb9f50)), файл [`modules/wallet.py`](https://github.com/ton-blockchain/mytonctrl/blob/92b2a87fe8cbbec6430ee868e7c82c7322eb9f50/modules/wallet.py), строки [145–159](https://github.com/ton-blockchain/mytonctrl/blob/92b2a87fe8cbbec6430ee868e7c82c7322eb9f50/modules/wallet.py#L145-L159). Цитата сверена с содержимым файла на этом коммите.

Сыщик вошел в комнату экспорта кошелька. Ожидал увидеть адрес, имя, может быть — аккуратный файл для сейфа. Но контора выбрала иной стиль: достать приватный материал из файла, превратить его в Base64 и вывести на экран, словно огласить номер кареты у вокзала.

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

Вот она, строка [159](https://github.com/ton-blockchain/mytonctrl/blob/92b2a87fe8cbbec6430ee868e7c82c7322eb9f50/modules/wallet.py#L159), финальный аккорд: `print("Secret key:", key)`. Не хеш. Не маска. Не предупреждение о том, что за спиной кто-то смотрит. Полный ключ, только в новом пальто Base64.

Справедливости ради, команда экспорта требует имя кошелька и запускается из локальной консоли. Это не означает, что ключ немедленно улетает в туман. Терминал, однако, любит историю команд, буфер обмена, скриншоты и любопытных жильцов. Секрет, однажды показанный на витрине, уже не вполне секрет — даже если витрина стоит внутри серверной.

## УЛИКА ВТОРАЯ: ОДНА КОМАНДА ДЛЯ ВСЕХ СЕЙФОВ

Сыщик еще не успел закрыть блокнот, как из соседней комнаты донесся скрежет резервного механизма. В файле [`modules/backups.py`](https://github.com/ton-blockchain/mytonctrl/blob/92b2a87fe8cbbec6430ee868e7c82c7322eb9f50/modules/backups.py) контора готовит связку ключей валидатора. Строка [21](https://github.com/ton-blockchain/mytonctrl/blob/92b2a87fe8cbbec6430ee868e7c82c7322eb9f50/modules/backups.py#L21) гласит:

```python
self.ton.validatorConsole.run(f'exportallprivatekeys {keyring_dir}')
```

Команда `exportallprivatekeys` звучит не как операция ювелира, а как крик дворецкого: «Вынести все ключи!» И именно это происходит по смыслу: контора просит консоль валидатора выгрузить все приватные ключи в каталог `keyring_dir`.

Еще одна печальная деталь живет в обработке отсутствующего ключа. В [`mytoncore/mytoncore.py`](https://github.com/ton-blockchain/mytonctrl/blob/92b2a87fe8cbbec6430ee868e7c82c7322eb9f50/mytoncore/mytoncore.py), строка [327](https://github.com/ton-blockchain/mytonctrl/blob/92b2a87fe8cbbec6430ee868e7c82c7322eb9f50/mytoncore/mytoncore.py#L327), исключение сообщает путь к пропавшему приватному файлу:

```python
raise Exception("GetWalletFromFile error: Private key not found: " + filePath)
```

А для highload-кошелька в строке [342](https://github.com/ton-blockchain/mytonctrl/blob/92b2a87fe8cbbec6430ee868e7c82c7322eb9f50/mytoncore/mytoncore.py#L342) повторяется тот же мотив:

```python
raise Exception("GetHighWalletFromFile error: Private key not found: " + filePath)
```

Путь — не сам ключ, конечно. Но сыщик отмечает манеру: ключевой материал в этой конторе не прячется за театральной завесой. Его читают, складывают, называют, экспортируют, а при пропаже громко оглашают, где искать.

## ДЕЛО О НЕУДАЧНОМ НАЗВАНИИ

Вся сцена особенно хороша из-за вывески. Метод называется `export_wallet`, будто речь идет о безобидной карточке клиента. На деле он возвращает адрес и секретный ключ, а затем печатает оба. Строка [156](https://github.com/ton-blockchain/mytonctrl/blob/92b2a87fe8cbbec6430ee868e7c82c7322eb9f50/modules/wallet.py#L156) связывает их в один досье-пакет, строки [157–159](https://github.com/ton-blockchain/mytonctrl/blob/92b2a87fe8cbbec6430ee868e7c82c7322eb9f50/modules/wallet.py#L157-L159) кладут его на стойку приемной.

Ревьюер открывает консоль:

```text
Wallet name: wallet_001
Address: <адрес>
Secret key: <ключ>
```

И в этот миг газовый рожок над дверью должен подать сигнал. Не потому, что экспорт сам по себе запрещен: администратору иногда правда нужно перенести кошелек. А потому, что у такой операции должен быть строгий маршрут — защищенный файл, явное подтверждение, отсутствие печати в общий терминал. Здесь же секрет получает роль последней строки красивого отчета.

## ВЕРДИКТ СЫЩИКА

[MyTonCtrl](https://github.com/ton-blockchain/mytonctrl) — не карманный вор и не фальшивый кошелек. Это полезная контора для тяжелого хозяйства валидатора. Но в коммите [`92b2a87`](https://github.com/ton-blockchain/mytonctrl/commit/92b2a87fe8cbbec6430ee868e7c82c7322eb9f50) сыщик увидел опасную викторианскую привычку: если ключ уже есть в сейфе, его можно вынести на свет и назвать по имени. А если ключей много — вызвать хор `exportallprivatekeys`, строка [21](https://github.com/ton-blockchain/mytonctrl/blob/92b2a87fe8cbbec6430ee868e7c82c7322eb9f50/modules/backups.py#L21).

Редакция выносит мягкий, но ясный приговор: экспортируй ключ, если обязан; не печатай его, если можешь промолчать. Терминал — не холодный кошелек. Base64 — не плащ-невидимка. И любой прохожий у газового фонаря способен запомнить то, что ты вывел строкой [159](https://github.com/ton-blockchain/mytonctrl/blob/92b2a87fe8cbbec6430ee868e7c82c7322eb9f50/modules/wallet.py#L159).

*Сыщик погасил фонарь, закрыл консоль и оставил ключ внутри сейфа. На этот раз.*

🐀
