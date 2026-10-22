---
title: "Выпуск № 102: Словарь без дна и восемь тысяч пустых дверей"
date: 2026-10-22T10:59:00+03:00
tags: [ton-indexer]
---

# ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

## ДЕЛО О СЛОВАРЕ, КОТОРЫЙ МОГ РАСТИ ДО ТУМАНА

Лондон проснулся под мелким дождем. В редакцию пришла папка из [toncenter/ton-indexer](https://github.com/toncenter/ton-indexer): сканер читал данные контрактов и доверял словарям внутри ячеек. На вид обычная канцелярия. Но у TON есть одна неприятная привычка: маленькая ячейка умеет ссылаться на общие поддеревья и тем самым изображать шкаф, хотя внутри у нее целый архив.

Сыщик раскрыл дело. Перед ним лежал коммит [421c966](https://github.com/toncenter/ton-indexer/commit/421c966168ed42cc8522e632cb79425934523a36), а рядом — более поздняя заплатка [adf35af](https://github.com/toncenter/ton-indexer/commit/adf35af6547b1bf6b9b9868d3f32a32da9bf1286). Газовый рожок сразу закашлялся: когда в истории появляются лимиты для «пустых» данных, значит, до этого кто-то уже нашел дверь без ручки.

## МЕСТО ПРОИСШЕСТВИЯ

Первая улика найдена в новом файле [`safe-parse.h`](https://github.com/toncenter/ton-indexer/blob/421c966168ed42cc8522e632cb79425934523a36/ton-index-worker/tondb-scanner/src/safe-parse.h), коммит [421c966](https://github.com/toncenter/ton-indexer/commit/421c966168ed42cc8522e632cb79425934523a36), строки [11–15](https://github.com/toncenter/ton-indexer/blob/421c966168ed42cc8522e632cb79425934523a36/ton-index-worker/tondb-scanner/src/safe-parse.h#L11-L15):

```cpp
// Parsing helpers for cells that come from contract get-methods or account data.
namespace safe_parse {

// Shared subtrees let a small dictionary hold a huge number of entries.
constexpr std::size_t kDefaultMaxDictEntries = 1024;
```

Писарь сам все изложил. Маленький словарь способен держать огромное число записей благодаря общим поддеревьям. В приличном доме после такого признания ставят решетку. Здесь решеткой стал лимит в 1024 записи — но не всякий проход по словарю еще умел этот лимит уважать.

## ПЕРВЫЙ СЛЕД: ПРОГУЛКА ПО ДЕРЕВУ С ПОВОДКОМ

В том же [safe-parse.h](https://github.com/toncenter/ton-indexer/blob/421c966168ed42cc8522e632cb79425934523a36/ton-index-worker/tondb-scanner/src/safe-parse.h) строки [33–55](https://github.com/toncenter/ton-indexer/blob/421c966168ed42cc8522e632cb79425934523a36/ton-index-worker/tondb-scanner/src/safe-parse.h#L33-L55) вводят общий обход:

```cpp
// Calls on_entry for each entry in key order, stops after max_entries entries.
template <class F>
td::Status for_each_dict_entry(td::Ref<vm::Cell> root, int key_bits, std::size_t max_entries, F&& on_entry) {
  return guard("dictionary", [&]() -> td::Status {
    vm::Dictionary dict{std::move(root), key_bits};
    td::Status status;
    std::size_t seen = 0;
    bool complete = dict.check_for_each([&](td::Ref<vm::CellSlice> value, td::ConstBitPtr key, int key_len) {
      if (++seen > max_entries) {
        status = td::Status::Error(PSLICE() << "more than " << max_entries << " entries");
        return false;
      }
      status = on_entry(key, key_len, std::move(value));
      return status.is_ok();
    });
    if (status.is_error()) {
      return status;
    }
    if (!complete) {
      return td::Status::Error("malformed dictionary");
    }
    return td::Status::OK();
  });
}
```

Сцена достойна сыскного романа. Обход считает записи, а на тысяча двадцать пятой захлопывает дверь с сообщением `more than ... entries`. Если дерево оказалось кривым, контора не делает вид, будто все в порядке, а возвращает `malformed dictionary`. Исключения тоже загоняются в отдельную клетку и превращаются в ошибку.

Но лимит — это только половина истории. Настоящий подозреваемый прятался в чтении чанков метаданных.

## ВТОРОЙ СЛЕД: ВОСЕМЬ ТЫСЯЧ ПОКЛОНОВ ПУСТОМУ СЛОВАРЮ

В коммите [421c966](https://github.com/toncenter/ton-indexer/commit/421c966168ed42cc8522e632cb79425934523a36) файл [`parse_token_data.cpp`](https://github.com/toncenter/ton-indexer/blob/421c966168ed42cc8522e632cb79425934523a36/ton-index-worker/tondb-scanner/src/parse_token_data.cpp), строки [8–11](https://github.com/toncenter/ton-indexer/blob/421c966168ed42cc8522e632cb79425934523a36/ton-index-worker/tondb-scanner/src/parse_token_data.cpp#L8-L11), ставит ограничитель:

```cpp
namespace {

// Empty chunks never fill the buffer, so limit the number of lookups too.
constexpr unsigned kMaxChunks = 8192;
```

Затем строки [41–61](https://github.com/toncenter/ton-indexer/blob/421c966168ed42cc8522e632cb79425934523a36/ton-index-worker/tondb-scanner/src/parse_token_data.cpp#L41-L61) начинают перебирать словарь:

```cpp
td::Result<std::string> parse_chunks_data(td::Ref<vm::CellSlice> data) {
  return safe_parse::guard("Exception while parsing chunks data", [&]() -> td::Result<std::string> {
    vm::Dictionary dict(data, 32);
    
    size_t bsize = 1024 * 8;
    unsigned char buffer[bsize];
    td::BitPtr bw{buffer};

    uint c = 0;
    while (c < kMaxChunks && dict.uint_key_exists(c)) {
      auto value = dict.lookup_ref(td::BitArray<32>(c));
      if (value.not_null()) {
        auto data = vm::load_cell_slice_ref(value);
        if (bw.offs + data->size() > bsize * 8) {
          break; // buffer overflow
        }
        
        bw.concat(data->prefetch_bits(data->size()));
      }
      c++;
    }
```

Код честно говорит: пустые чанки не заполняют буфер, поэтому надо ограничить число поисков. И тут возникает пикантная картина. Словарь может содержать ключи, под которыми ничего нет, а сыщик все равно будет подходить к каждой двери по очереди. Восемь тысяч раз он постучит в пустоту, прежде чем признает, что архив подозрительно длинен.

В буфере всего 8 килобайт. Если данные настоящие, переполнение останавливает чтение. Но если ключи ведут в пустые или почти пустые комнаты, буфер не растет, а обход продолжает марш. Маленькая ячейка получает право устроить длинную прогулку по дереву, и именно поэтому в деле понадобился `kMaxChunks`.

## ТРЕТИЙ СЛЕД: ЗАПЛАТКА, КОТОРАЯ ПЕРЕСТАЛА ТЕРЯТЬ КОНЕЦ

Потом явился коммит [adf35af](https://github.com/toncenter/ton-indexer/commit/adf35af6547b1bf6b9b9868d3f32a32da9bf1286). В файле [`parse_token_data.cpp`](https://github.com/toncenter/ton-indexer/blob/adf35af6547b1bf6b9b9868d3f32a32da9bf1286/ton-index-worker/tondb-scanner/src/parse_token_data.cpp) строки [59–62](https://github.com/toncenter/ton-indexer/blob/adf35af6547b1bf6b9b9868d3f32a32da9bf1286/ton-index-worker/tondb-scanner/src/parse_token_data.cpp#L59-L62) добавили важный окрик после цикла:

```cpp
    }
    if (c == kMaxChunks && dict.uint_key_exists(c)) {
      return td::Status::Error(PSLICE() << "more than " << kMaxChunks << " chunks");
    }
```

До этого ограничитель мог остановить чтение на восьми тысячах чанков и вернуть собранный префикс как будто это весь документ. Теперь, если за последней разрешенной дверью есть еще одна, контора не выдает обрезанный роман за полный. Она признается: чанков больше нормы.

Это не обвинение в том, что любой такой словарь немедленно рушит сервер. Доказанное проще и точнее: входные ячейки могли заставить индексатор делать лишние обходы, а ранний предел без проверки хвоста позволял принять неполные данные за нормальный результат. Сыщик не приписывает делу больше, чем лежит в папке.

## ПРИГОВОР СЫЩИКА

Репозиторий [toncenter/ton-indexer](https://github.com/toncenter/ton-indexer) встретил опасную архитектурную шутку в коммите [421c966](https://github.com/toncenter/ton-indexer/commit/421c966168ed42cc8522e632cb79425934523a36): маленькая ячейка могла вести за собой длинную очередь словарных проверок. Файл [safe-parse.h](https://github.com/toncenter/ton-indexer/blob/421c966168ed42cc8522e632cb79425934523a36/ton-index-worker/tondb-scanner/src/safe-parse.h), строки [14–15](https://github.com/toncenter/ton-indexer/blob/421c966168ed42cc8522e632cb79425934523a36/ton-index-worker/tondb-scanner/src/safe-parse.h#L14-L15), сам называет причину: общие поддеревья позволяют малому словарю держать огромное число записей.

Заплатка [adf35af](https://github.com/toncenter/ton-indexer/commit/adf35af6547b1bf6b9b9868d3f32a32da9bf1286) в файле [parse_token_data.cpp](https://github.com/toncenter/ton-indexer/blob/adf35af6547b1bf6b9b9868d3f32a32da9bf1286/ton-index-worker/tondb-scanner/src/parse_token_data.cpp) строки [60–62](https://github.com/toncenter/ton-indexer/blob/adf35af6547b1bf6b9b9868d3f32a32da9bf1286/ton-index-worker/tondb-scanner/src/parse_token_data.cpp#L60-L62) поставила стража у хвоста списка.

Приговор мягок, но без скидки: размер ячейки не равен размеру работы. В блокчейн приходит не только содержимое, но и форма дерева, а форма иногда нанимает себе восемь тысяч пустых слуг. Газовый рожок пробил сто второй удар, индексатор закрыл лишние двери, а сыщик растворился в лондонском тумане.

🐀
