+++
title = "№ 107 — Метод, который хотел быть шестидесятичетырехбитным"
date = 2026-10-24T17:00:00+03:00
tags = ["ton-indexer"]
+++

# ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

**№ 107.** *Лондон. Туман лежит на мостовой, когда сыщик получает папку из [toncenter/ton-indexer](https://github.com/toncenter/ton-indexer). На папке печать коммита [930a04b](https://github.com/toncenter/ton-indexer/commit/930a04b8edccffcadb668a39153cfcc355ff4a42): `Reject method dicts with keys wider than 32 bits`. Внутри — короткое дело о числе, которое сначала разрешили до шестидесяти четырех, а потом поймали на том, что жить оно должно было в тридцати двух.*

## ДЕЛО О ШИРИНЕ, КОТОРАЯ НЕ ВЛЕЗЛА В ЧЕРТЕЖ

След ведет в файл [ton-index-worker/tondb-scanner/src/parse_contract_methods.cpp](https://github.com/toncenter/ton-indexer/blob/930a04b8edccffcadb668a39153cfcc355ff4a42/ton-index-worker/tondb-scanner/src/parse_contract_methods.cpp). Функция `parse_contract_methods` разбирает код контракта, выискивает в нем словарь get-методов и выписывает их идентификаторы. Занятие важное: индексатор должен понять, какие двери у контракта вообще есть.

В старой редакции того же файла, в родительском коммите [adf35af](https://github.com/toncenter/ton-indexer/commit/adf35af6547b1bf6b9b9868d3f32a32da9bf1286), канцелярия ставила верхнюю планку так:

```cpp
    if (methods_dict_key_len > 64) {
      return td::Status::Error("Method id is wider than 64 bits");
    }
```

Цитата снята с [строк 32–34](https://github.com/toncenter/ton-indexer/blob/adf35af6547b1bf6b9b9868d3f32a32da9bf1286/ton-index-worker/tondb-scanner/src/parse_contract_methods.cpp#L32-L34) родительского коммита. С виду все чинно: шестьдесят четыре — число солидное, почти полный этаж. Но внутри обработчика ключ превращается в идентификатор через `key.get_uint(key_len)`, а весь фолиант называет результат как `std::vector<unsigned long long>`. Сыщик не стал обвинять старый код в катастрофе без экспертизы. Он лишь отметил: контора дала словарю право говорить шире, чем ее дело должно было позволять.

## СЕНСАЦИЯ: ШЕСТЬДЕСЯТЬ ЧЕТЫРЕ СТАЛИ ТРИДЦАТЬЮ ДВУМЯ

Свежий приговор записан в коммите [930a04b](https://github.com/toncenter/ton-indexer/commit/930a04b8edccffcadb668a39153cfcc355ff4a42), файл [parse_contract_methods.cpp](https://github.com/toncenter/ton-indexer/blob/930a04b8edccffcadb668a39153cfcc355ff4a42/ton-index-worker/tondb-scanner/src/parse_contract_methods.cpp), строки [32–34](https://github.com/toncenter/ton-indexer/blob/930a04b8edccffcadb668a39153cfcc355ff4a42/ton-index-worker/tondb-scanner/src/parse_contract_methods.cpp#L32-L34):

```cpp
    auto methods_dict_key_len = static_cast<int>(cs.fetch_ulong(10));
    if (methods_dict_key_len > 32) {
      return td::Status::Error("Method id is wider than 32 bits");
    }
```

Вот так выглядит признание. Раньше швейцар принимал ключи до 64 бит, теперь на двери висит табличка: после 32 — разворот. Ошибка сообщает не «неверный словарь», не «неизвестный формат», а совершенно честное: `Method id is wider than 32 bits`. В Лондоне редко встретишь сторожа, который прямо признает: «ваш номер слишком широк для нашей конторы».

Но подлинная прелесть дела в том, что десять бит длины читаются из ячейки без всякого сомнения, а затем у подозреваемого внезапно появляется потолок в 32. Словарь может принести ключ длиной 33, 48 или 64 бита; индексатор не пытается его сократить, не делает вид, что понял, и не выдает усеченный адрес. Он ставит печать отказа. Старый сторож мерил лестницу шестьюдесятью четырьмя ступенями, новый признал: лестница в конторе короче.

## ВТОРОЙ СЛЕД: ПРОТОКОЛ С НОМЕРОМ, КОТОРЫЙ ПЕЧАТАЮТ МАГИЕЙ

До самой ширины сыщик добрался не сразу. Сначала контора требует, чтобы первые байты выглядели как правильное начало кода. Строки [16–27](https://github.com/toncenter/ton-indexer/blob/930a04b8edccffcadb668a39153cfcc355ff4a42/ton-index-worker/tondb-scanner/src/parse_contract_methods.cpp#L16-L27) из того же фолианта:

```cpp
    auto first_op = cs.fetch_ulong(8);
    auto expectedCodePage = cs.fetch_ulong(8);
    if (first_op != 0xFF || expectedCodePage != 0) {
      return td::Status::Error("Failed to parse 1. SETCP or codepage is not 0");
    }

    auto second_op = cs.fetch_ulong(13);
    auto is_push_op_flag = cs.fetch_ulong(1);
    if (second_op != 0b1111010010100 || is_push_op_flag != 1) {
      return td::Status::Error("Failed to parse 2. DICTPUSHCONST");
    }
```

Сыщик прочел показания при свете газового рожка. Сначала контора достает восемь бит, потом еще восемь, потом тринадцать и один. Если числа не совпали с `0xFF`, нулевой страницей и двоичной молитвой `0b1111010010100`, дело получает диагноз «Failed to parse 1» или «Failed to parse 2». У протокола нет именованных печатей, только порядковые номера: первая дверь, вторая дверь, третья дверь — ищите сами, где именно скрипнул пол.

Отдельно отметим писарскую смесь: `expectedCodePage` живет рядом с `first_op`, `second_op` и `is_push_op_flag`, будто одна переменная приехала из другой канцелярии. Компилятор доволен, сыщик — нет. В машинном отделении, где каждый бит может изменить смысл кода, имя выглядит как визитная карточка человека, который пришел на заседание в чужом сюртуке.

## ТРЕТИЙ СЛЕД: ЧЕТЫРЕ ТЫСЯЧИ ДВЕРЕЙ ДЛЯ ОДНОГО КОНТРАКТА

После проверки ширины индексатор открывает словарь. В строках [36–40](https://github.com/toncenter/ton-indexer/blob/930a04b8edccffcadb668a39153cfcc355ff4a42/ton-index-worker/tondb-scanner/src/parse_contract_methods.cpp#L36-L40) он одновременно ставит еще одну ограду:

```cpp
    auto methods_dict_cell = cs.fetch_ref();
    std::vector<unsigned long long> method_ids;
    TRY_STATUS(safe_parse::for_each_dict_entry(methods_dict_cell, methods_dict_key_len, kMaxContractMethods,
        [&](td::ConstBitPtr key, int key_len, td::Ref<vm::CellSlice>) -> td::Status {
          method_ids.push_back(key.get_uint(key_len));
          return td::Status::OK();
        }));
```

А величина ограды стоит выше, в строке [9](https://github.com/toncenter/ton-indexer/blob/930a04b8edccffcadb668a39153cfcc355ff4a42/ton-index-worker/tondb-scanner/src/parse_contract_methods.cpp#L9): `constexpr std::size_t kMaxContractMethods = 4096;`.

Итак, контракту разрешено иметь не больше четырех тысяч методов, но каждый метод обязан уместиться в тридцати двух битах. Контора не просто считает двери, она проверяет ширину табличек на каждой двери. Редакция не спорит с защитой: бесконечный словарь — плохая мебель для индексатора. Но зрелище забавно: один сторож с линейкой на 4096 дверей и другой с ножницами, отрезающими все, что шире 32 бит.

## ПРИГОВОР СЫЩИКА

Репозиторий [toncenter/ton-indexer](https://github.com/toncenter/ton-indexer) в коммите [930a04b](https://github.com/toncenter/ton-indexer/commit/930a04b8edccffcadb668a39153cfcc355ff4a42) пойман на исправлении, которое само рассказывает старую историю: проверка ширины ключа была поставлена на 64 бита, а затем снижена до 32. В файле [parse_contract_methods.cpp](https://github.com/toncenter/ton-indexer/blob/930a04b8edccffcadb668a39153cfcc355ff4a42/ton-index-worker/tondb-scanner/src/parse_contract_methods.cpp) рядом живут магические опкоды, порядковые диагнозы `Failed to parse 1` и `Failed to parse 2`, а также потолок в 4096 записей.

Это не доказательство того, что каждый широкий ключ уже обрушил город. Это доказательство более точное: ширина формата и ширина приемной однажды разошлись, и конторе пришлось перекрасить табличку прямо в рабочем фолианте. Старый сторож пропускал слишком широких посетителей. Новый хотя бы знает, где заканчивается дверь.

*Газовый рожок пробил сто седьмой удар. Сыщик свернул папку, запер словарь и растворился в тумане.* 🕵️
