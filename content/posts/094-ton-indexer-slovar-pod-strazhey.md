+++
title = "№ 94 — Словарь адресов попал под стражу"
date = 2026-10-01T10:58:00+03:00
description = "Девяносто четвертый выпуск «Вечернего Валидатора»: toncenter/ton-indexer вынес разбор словаря адресов в отдельную функцию и впервые поставил ловушку виртуальной машины под try-catch."
tags = ["ton-indexer"]
+++

# ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

**№ 94.** *Лондон. Туман заползал под двери редакции, когда сыщик получил из [toncenter/ton-indexer](https://github.com/toncenter/ton-indexer) свежую папку. Внутри лежал словарь адресов, а рядом — признание: виртуальная машина умеет бросить ошибку прямо посреди обхода. На этот раз контора не стала делать вид, что словарь сам себя разберет.*

## МЕСТО ПРОИСШЕСТВИЯ

Улика найдена в репозитории [toncenter/ton-indexer](https://github.com/toncenter/ton-indexer), в коммите [`f4ce832`](https://github.com/toncenter/ton-indexer/commit/f4ce832782af9fc51fa87de29003fde56c8444c5), озаглавленном `Fix vm error`. Главная комната дела — файл [`ton-index-worker/tondb-scanner/src/smc-interfaces/Multisig.cpp`](https://github.com/toncenter/ton-indexer/blob/f4ce832782af9fc51fa87de29003fde56c8444c5/ton-index-worker/tondb-scanner/src/smc-interfaces/Multisig.cpp). Цитаты сверены с этим коммитом.

## ПЕРВЫЙ СЛЕД: СЛОВАРЬ В ЧЕРНОМ КАБИНЕТЕ

Коммит вынес разбор ячеечного словаря в отдельную функцию. На строках [8–24](https://github.com/toncenter/ton-indexer/blob/f4ce832782af9fc51fa87de29003fde56c8444c5/ton-index-worker/tondb-scanner/src/smc-interfaces/Multisig.cpp#L8-L24) появилась дословная улика из файла [`ton-index-worker/tondb-scanner/src/smc-interfaces/Multisig.cpp`](https://github.com/toncenter/ton-indexer/blob/f4ce832782af9fc51fa87de29003fde56c8444c5/ton-index-worker/tondb-scanner/src/smc-interfaces/Multisig.cpp):

```cpp
td::Result<std::vector<block::StdAddress>> parse_address_dict(td::Ref<vm::Cell> cell) {
  std::vector<block::StdAddress> result;
  try {
    vm::Dictionary dict{std::move(cell), 8};
    for (auto it = dict.begin(); !it.eof(); ++it) {
      block::StdAddress address;
      block::tlb::MsgAddressInt address_int{};
      if (!address_int.extract_std_address(it.cur_value(), address)) {
        return td::Status::Error("Unable to extract address");
      }
      result.push_back(address);
    }
  } catch (vm::VmError& e) {
    return td::Status::Error(PSLICE() << "Failed to parse address dict: " << e.get_msg());
  }
  return result;
}
```

Картина достаточно сочная: `vm::Dictionary` открывает дверь, цикл перебирает жильцов, а любой `vm::VmError` ловится за воротник и превращается в `td::Status::Error`. Раньше такой обход был размазан по двум комнатам и не имел общего поводка для исключения. Теперь у словаря есть отдельный конвой.

## ВТОРОЙ СЛЕД: ДВА СЛОВАРЯ, ОДИН КОНВОИР

В `MultisigContract::start_up` первый словарь приходит из стека на строке [70](https://github.com/toncenter/ton-indexer/blob/f4ce832782af9fc51fa87de29003fde56c8444c5/ton-index-worker/tondb-scanner/src/smc-interfaces/Multisig.cpp#L70). Контора вызывает `parse_address_dict`, проверяет результат и только после этого переносит адреса в `data.signers` на строке [77](https://github.com/toncenter/ton-indexer/blob/f4ce832782af9fc51fa87de29003fde56c8444c5/ton-index-worker/tondb-scanner/src/smc-interfaces/Multisig.cpp#L77).

Второй словарь идет следом. На строках [80–87](https://github.com/toncenter/ton-indexer/blob/f4ce832782af9fc51fa87de29003fde56c8444c5/ton-index-worker/tondb-scanner/src/smc-interfaces/Multisig.cpp#L80-L87) тот же клерк занимается `proposers`: ошибка передается в обещание, актер останавливается, а удачный список получает `data.proposers`.

```cpp
  if (stack[3].is_cell()) {
    auto proposers = parse_address_dict(stack[3].as_cell());
    if (proposers.is_error()) {
      promise_.set_error(proposers.move_as_error());
      stop();
      return;
    }
    data.proposers = proposers.move_as_ok();
  }
```

Здесь смешно не то, что код стал аккуратнее. Смешно, что два разных списка адресов — подписанты и предлагающие — нуждались в одной и той же службе спасения, словно в старом особняке обе лестницы вели в одну яму.

## ТРЕТИЙ СЛЕД: ОШИБКА ПОЛУЧИЛА ПАСПОРТ

Функция на строках [15–21](https://github.com/toncenter/ton-indexer/blob/f4ce832782af9fc51fa87de29003fde56c8444c5/ton-index-worker/tondb-scanner/src/smc-interfaces/Multisig.cpp#L15-L21) различает два вида неприятностей. Если значение словаря не удается превратить в адрес, возвращается `Unable to extract address`. Если сама виртуальная машина выбрасывает `vm::VmError`, наружу уходит `Failed to parse address dict: ` с сообщением ошибки.

Это важная разница. Неверный адрес — плохой документ конкретного жильца. Ошибка VM — пожар в самом архиве. Новый код не смешивает их в один безымянный крик, а хотя бы подписывает, на каком этаже случилась беда.

После этого `MultisigOrder::start_up` тоже пользуется общей функцией на строке [144](https://github.com/toncenter/ton-indexer/blob/f4ce832782af9fc51fa87de29003fde56c8444c5/ton-index-worker/tondb-scanner/src/smc-interfaces/Multisig.cpp#L144). Результат проходит ту же проверку на строках [145–150](https://github.com/toncenter/ton-indexer/blob/f4ce832782af9fc51fa87de29003fde56c8444c5/ton-index-worker/tondb-scanner/src/smc-interfaces/Multisig.cpp#L145-L150). Один стражник теперь обслуживает и контракт, и его ордер.

## ЧЕТВЕРТЫЙ СЛЕД: ПОДОЗРИТЕЛЬНЫЙ ПОРЯДОК В ДЕЛЕ

Сыщик не станет оглашать этот коммит доказательством полной неуязвимости. Но документ фиксирует любопытный контраст. Для `MultisigContract` перед разбором проверяется, что элементы стека являются ячейкой или пустотой, на строках [54–63](https://github.com/toncenter/ton-indexer/blob/f4ce832782af9fc51fa87de29003fde56c8444c5/ton-index-worker/tondb-scanner/src/smc-interfaces/Multisig.cpp#L54-L63). А в маршруте `MultisigOrder` словарь сразу извлекается как ячейка на строке [144](https://github.com/toncenter/ton-indexer/blob/f4ce832782af9fc51fa87de29003fde56c8444c5/ton-index-worker/tondb-scanner/src/smc-interfaces/Multisig.cpp#L144).

Редакция не утверждает, что это уже отдельный сбой: контракт ордера получил ожидаемый тип от `execute_smc_method`. Улика скромнее. В одном деле входной стек досматривают до порога, а в соседнем надеются, что `as_cell()` не станет плохим свидетелем. В большом доме даже полезный общий конвоир не отменяет разницу между двумя входами.

## ХРОНИКА МЕЛКОЙ КАНЦЕЛЯРИИ

*Отдел названий.* Коммит [`f4ce832`](https://github.com/toncenter/ton-indexer/commit/f4ce832782af9fc51fa87de29003fde56c8444c5) называется `Fix vm error`. Лаконичность почти подозрительна: внутри не просто исправлена надпись, а вынесен общий разбор двух словарей адресов.

*Отдел возвратов.* На строках [72–75](https://github.com/toncenter/ton-indexer/blob/f4ce832782af9fc51fa87de29003fde56c8444c5/ton-index-worker/tondb-scanner/src/smc-interfaces/Multisig.cpp#L72-L75) ошибка передается через `promise_`, затем актер вызывает `stop()`. Ошибка не растворяется в тумане и не оставляет контору ждать ответа от сломанного словаря.

*Отдел повторов.* В `MultisigOrder` прежний ручной обход также заменен вызовом общей функции на строках [144–150](https://github.com/toncenter/ton-indexer/blob/f4ce832782af9fc51fa87de29003fde56c8444c5/ton-index-worker/tondb-scanner/src/smc-interfaces/Multisig.cpp#L144-L150). Два маршрута, один почерк, одна клетка для `vm::VmError`.

## ВЕРДИКТ СЫЩИКА

Репозиторий [toncenter/ton-indexer](https://github.com/toncenter/ton-indexer) пойман в коммите [`f4ce832`](https://github.com/toncenter/ton-indexer/commit/f4ce832782af9fc51fa87de29003fde56c8444c5) на деле о словаре, который мог заговорить голосом виртуальной машины. В файле [`ton-index-worker/tondb-scanner/src/smc-interfaces/Multisig.cpp`](https://github.com/toncenter/ton-indexer/blob/f4ce832782af9fc51fa87de29003fde56c8444c5/ton-index-worker/tondb-scanner/src/smc-interfaces/Multisig.cpp) строки [8–24](https://github.com/toncenter/ton-indexer/blob/f4ce832782af9fc51fa87de29003fde56c8444c5/ton-index-worker/tondb-scanner/src/smc-interfaces/Multisig.cpp#L8-L24) теперь держат обход под `try-catch`, а строки [70–87](https://github.com/toncenter/ton-indexer/blob/f4ce832782af9fc51fa87de29003fde56c8444c5/ton-index-worker/tondb-scanner/src/smc-interfaces/Multisig.cpp#L70-L87) применяют его к двум спискам.

Приговор без громких обвинений: если словарь может выдать ошибку VM, не заставляй сыщика искать ее по обломкам стека. Дай беде имя, передай ее обещанию и останови актера. Газовый рожок хрипнул, словарь отправился под стражу, а сыщик исчез в тумане.

🐀
