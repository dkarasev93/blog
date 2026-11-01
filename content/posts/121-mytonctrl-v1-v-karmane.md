+++
title = "№ 121 — Кошелек, который принял hv1 за v1"
date = 2026-11-01T11:01:00+03:00
tags = ["mytonctrl"]
+++

# ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

## ДЕЛО О КОШЕЛЬКЕ, КОТОРЫЙ ПЕРЕПУТАЛ БУКВУ С ПРИГОВОРОМ

Лондон тонул в тумане, когда сыщик получил папку из [ton-blockchain/mytonctrl](https://github.com/ton-blockchain/mytonctrl). На обложке стоял коммит [2568c631](https://github.com/ton-blockchain/mytonctrl/commit/2568c6319f7027cfbe66bee2b7e3b242ff47d3b8) с сухим приказом `drop highload wallet support`. Газовый рожок кашлянул: в подвале конторы нашли не просто выброшенную функцию, а проверку, которая однажды могла принять highload-кошелек за обычный v1.

## МЕСТО ПРОИСШЕСТВИЯ

Главная улика лежит в коммите [2568c631](https://github.com/ton-blockchain/mytonctrl/commit/2568c6319f7027cfbe66bee2b7e3b242ff47d3b8), в файле [tests/unit/test_wallet.py](https://github.com/ton-blockchain/mytonctrl/blob/2568c6319f7027cfbe66bee2b7e3b242ff47d3b8/tests/unit/test_wallet.py), строки [273–279](https://github.com/ton-blockchain/mytonctrl/blob/2568c6319f7027cfbe66bee2b7e3b242ff47d3b8/tests/unit/test_wallet.py#L273-L279). Цитата дословная:

```python
def test_do_move_coins_highload_wallet_is_not_supported(ton: MyTonCore, module: WalletModule, monkeypatch):
    # "hv1" contains "v1": a substring match used to pick wallet.fif for a highload wallet
    wallet = _transfer_wallet('hv1')
    calls = _patch_transfer(ton, monkeypatch, _account())

    with pytest.raises(Exception, match="Wallet version 'hv1' is not supported"):
        module.do_move_coins(wallet, _DEST, 1.0)
```

Показание короткое, как выстрел в туман. Строка [274](https://github.com/ton-blockchain/mytonctrl/blob/2568c6319f7027cfbe66bee2b7e3b242ff47d3b8/tests/unit/test_wallet.py#L274) сама признает: `"hv1" contains "v1"`. Ранее проверка искала подстроку, а не точную версию. Поэтому highload-ярлык `hv1` мог пройти в дверь обычного `v1` и получить скрипт `wallet.fif`.

## ПЕРЕПУТАННЫЕ ВЫВЕСКИ

В том же коммите контора перевела распознавание на начало строки. В файле [modules/wallet.py](https://github.com/ton-blockchain/mytonctrl/blob/2568c6319f7027cfbe66bee2b7e3b242ff47d3b8/modules/wallet.py) строки [234–246](https://github.com/ton-blockchain/mytonctrl/blob/2568c6319f7027cfbe66bee2b7e3b242ff47d3b8/modules/wallet.py#L234-L246) механизм уже раскладывает версии по дверям:

```python
        if version.startswith("v1"):
            fift_script = "wallet.fif"
            args = [fift_script, wallet.path, dest, seqno, str(coins), "-m", str(mode), result_file_path]
        elif version.startswith("v2"):
            fift_script = "wallet-v2.fif"
            args = [fift_script, wallet.path, dest, seqno, str(coins), "-m", str(mode), result_file_path]
        elif version.startswith("v3"):
            fift_script = "wallet-v3.fif"
            args = [fift_script, wallet.path, dest, str(subwallet), seqno, str(coins), "-m", str(mode), result_file_path]
        else:
            raise Exception(f"MoveCoins error: Wallet version {wallet.version!r} is not supported")
```

Цитата снята дословно со строк [236–246](https://github.com/ton-blockchain/mytonctrl/blob/2568c6319f7027cfbe66bee2b7e3b242ff47d3b8/modules/wallet.py#L236-L246). Теперь `hv1` не начинается с `v1`, а значит, добирается до строки [246](https://github.com/ton-blockchain/mytonctrl/blob/2568c6319f7027cfbe66bee2b7e3b242ff47d3b8/modules/wallet.py#L246), где его выдворяют без запуска Fift.

## КАРМАННЫЙ СВИДЕТЕЛЬ

Тест не ограничился формальным исключением. В строках [275–279](https://github.com/ton-blockchain/mytonctrl/blob/2568c6319f7027cfbe66bee2b7e3b242ff47d3b8/tests/unit/test_wallet.py#L275-L279) кошелек получает имя `hv1`, а затем контора требует, чтобы исключение появилось до отправки монет. Иными словами, исправление проверяет не только надпись на двери, но и отсутствие визита в Fift.

Сыщик не станет считать каждую похожую строку кражей TON. Доказано другое: проверка по подстроке стирала границу между `v1` и `hv1`, а тест коммита [2568c631](https://github.com/ton-blockchain/mytonctrl/commit/2568c6319f7027cfbe66bee2b7e3b242ff47d3b8) оставил это признание на бумаге. В Лондоне иногда достаточно одной буквы, чтобы чужой кошелек получил чужую форму.

## ПРИГОВОР СЫЩИКА

[ton-blockchain/mytonctrl](https://github.com/ton-blockchain/mytonctrl) в коммите [2568c631](https://github.com/ton-blockchain/mytonctrl/commit/2568c6319f7027cfbe66bee2b7e3b242ff47d3b8) пойман на старой привычке читать версию как случайный фрагмент слова. Файл [modules/wallet.py](https://github.com/ton-blockchain/mytonctrl/blob/2568c6319f7027cfbe66bee2b7e3b242ff47d3b8/modules/wallet.py) теперь требует начало `v1`, `v2` или `v3`, а тест [tests/unit/test_wallet.py](https://github.com/ton-blockchain/mytonctrl/blob/2568c6319f7027cfbe66bee2b7e3b242ff47d3b8/tests/unit/test_wallet.py) караулит highload-ярлык `hv1` у черного входа.

Газовый рожок пробил полдень. Вывески поправлены, но сыщик оставил в протоколе главное: в кошельковой конторе подстрока не должна получать полномочия паспорта.

🐀
