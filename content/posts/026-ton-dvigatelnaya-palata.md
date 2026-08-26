+++
title = "№ 26 — Двигательная палата: пустое кресло кассира, двести пятьдесят шесть инструкций-призраков и нотариус, чья печать оттискивает двести пятьдесят шесть нулей"
date = 2026-08-26T17:09:00+03:00
description = "Двадцать шестой выпуск «Вечернего Валидатора»: сыщик спускается в двигательную палату ядра ton — виртуальную машину TVM. Там, чтобы не платить газ за загрузку ячеек, за конторку сажают буквально пустое место и называют его «временным сотрудником»; целая страница из двухсот пятидесяти шести машинных инструкций занята манекенами, которые салютуют в журнал и ничего не делают; нотариус машины заверяет состояние печатью из двухсот пятидесяти шести нулей с пометкой «сделать как следует»; а в отделе надгробий хранится законсервированный «временный хак» с автоблагословением и мёртвая функция, которую не зовёт никто."
tags = ["ton"]
+++

# 📰 Вечерний Валидатор

**№ 26.** *Лондон. Туман нынче пахнет машинным маслом и озоном — говорят, его теперь не выпускают из труб, а вычисляют, по шагам, с учётом газа за каждый вздох. Наш корреспондент, обойдя телеграфную контору (выпуск № 25), монетный двор (выпуск № 22) и палату свидетельских показаний (выпуск № 24), спустился наконец туда, куда сходятся все провода и все валики: в двигательную палату ядра [ton-blockchain/ton](https://github.com/ton-blockchain/ton) — в [`crypto/vm/`](https://github.com/ton-blockchain/ton/tree/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/vm), к самой виртуальной машине TVM. Это сердце города: здесь исполняется каждый контракт, здесь за каждую ячейку взыскивается газ, здесь не бывает бесплатно ничего. Сыщик снял с двери пыльную табличку, прошёл внутрь — и первое, что он увидел, было пустое кресло за кассой.*

*Место происшествия: репозиторий [ton-blockchain/ton](https://github.com/ton-blockchain/ton), коммит [`3d478cb`](https://github.com/ton-blockchain/ton/commit/3d478cbde854be03a18ab2a59f8fc3c565cf7d14) (master от 17.08.2026). Файлы: [`crypto/vm/vm.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/vm/vm.cpp), [`crypto/vm/memo.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/vm/memo.cpp), [`crypto/vm/cells/CellSlice.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/vm/cells/CellSlice.cpp), [`tdutils/td/utils/Context.h`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tdutils/td/utils/Context.h), [`crypto/vm/debugops.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/vm/debugops.cpp), [`crypto/vm/contops.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/vm/contops.cpp), [`crypto/vm/boc.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/vm/boc.cpp). Все цитаты дословные.*

---

## СЕНСАЦИЯ ПЕРВАЯ: ЗА КОНТОРКОЙ СИДИТ ПУСТОЕ МЕСТО

В двигательной палате действует суровая повинность: всякая загрузка ячейки облагается газом. Взыскание поручено особому клерку — `VmStateInterface`, и клерк этот неподкупен ровно настолько, насколько вообще существует. Смотрите, как устроена касса, [`crypto/vm/cells/CellSlice.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/vm/cells/CellSlice.cpp), строки [1075–1083](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/vm/cells/CellSlice.cpp#L1075-L1083):

```cpp
  auto* vm_state_interface = VmStateInterface::get();
  if (vm_state_interface && vm_state_interface->is_actual_tvm()) {
    quiet = false;
  }
  bool library_loaded = false;
  while (true) {
    if (vm_state_interface && !library_loaded) {
      vm_state_interface->register_cell_load(cell->get_hash());
    }
```

Читатель, зри в строку [1081](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/vm/cells/CellSlice.cpp#L1081): `if (vm_state_interface && ...)`. Плата взыскивается только если клерк на месте. Нет клерка — нет платы. Указатель пуст — и повинность испаряется, как туман к полудню. Касса двигательной палаты пробивает чек не по факту загрузки, а по факту присутствия кассира. Сыщик присвистнул и полистал дальше — и обнаружил, что в самой палате об этом знают. Более того: этим пользуются. Вот штатное расписание, [`crypto/vm/vm.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/vm/vm.cpp), строки [446–449](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/vm/vm.cpp#L446-L449):

```cpp
    std::unique_ptr<VmStateInterface> tmp_ctx;
    // install temporary dummy vm state interface to prevent charging for cell load operations during dump
    VmStateInterface::Guard guard(tmp_ctx.get());
    stack->dump(ss, mode);
```

Смакуйте каждую строку при газовом рожке. Строка [446](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/vm/vm.cpp#L446): учреждается `std::unique_ptr<VmStateInterface> tmp_ctx;` — пустой, свежерождённый, ничем не наполненный указатель. Строка [447](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/vm/vm.cpp#L447): служебная записка гласит — «установить временный чучело-интерфейс состояния машины, дабы не взыскивать плату за загрузку ячеек во время опроса стека». Строка [448](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/vm/vm.cpp#L448): за конторку усаживается `tmp_ctx.get()` — то есть содержимое пустого указателя, то есть, по всем канонам языка, чистейший `nullptr`. «Временный сотрудник», читатель, — это не манекен и не статист. Это пустое кресло. Палата торжественно провозглашает: «на время осмотра стека за кассой будет сидеть никто» — и, раз кассира нет, проверка из строки [1081](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/vm/cells/CellSlice.cpp#L1081) честно не срабатывает, и весь осмотр проходит бесплатно. Тот же фокус повторён двумя этажами ниже, строки [453–456](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/vm/vm.cpp#L453-L456), для печати стека прямо в городской журнал `std::cerr`, — с той же запиской и тем же пустым креслом.

Приёмная комиссия, между прочим, никаких документов не спрашивает. Вот её устав, [`tdutils/td/utils/Context.h`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tdutils/td/utils/Context.h), строки [27–43](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tdutils/td/utils/Context.h#L27-L43):

```cpp
  static Impl *get() {
    return context_;
  }
  class Guard {
   public:
    explicit Guard(Impl *new_context) {
      old_context_ = context_;
      context_ = new_context;
    }
    ~Guard() {
      context_ = old_context_;
    }
```

`Guard`, строки [32–35](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tdutils/td/utils/Context.h#L32-L35), вписывает в книгу дежурных ровно то, что ему сунули, — хоть клерка, хоть `nullptr`, хоть воспоминание о клерке, — а по выходе, строки [36–38](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tdutils/td/utils/Context.h#L36-L38), молча возвращает прежнего. Ни одного `CHECK`, ни одного вопроса. Идеальная контора для временных сотрудников: зашёл, посидел пустотой, вышел.

Но подлинный шедевр ждёт в библиотечном отделе. Когда машина ищет библиотеку по хэшу, она тоже грузит ячейки — и тоже не желает платить. [`crypto/vm/vm.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/vm/vm.cpp), строки [657–659](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/vm/vm.cpp#L657-L659):

```cpp
  std::unique_ptr<VmStateInterface> tmp_ctx;
  // install temporary dummy vm state interface to prevent charging for cell load operations during library lookup
  VmStateInterface::Guard guard{global_version >= 4 ? tmp_ctx.get() : VmStateInterface::get()};
```

Строка [659](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/vm/vm.cpp#L659) — верх канцелярского искусства: если версия машины не ниже четвёртой (а нынче, скажем шёпотом, она куда выше), за конторку сажается всё тот же `tmp_ctx.get()` — то есть никто; и только для старых версий оставляют прежнего клерка, `VmStateInterface::get()`. Эволюция, читатель: в старой машине за библиотечной полкой следил живой кассир, в новой — кресло. Прогресс налицо, кассир отсутствует. И чтобы жанр был соблюдён до конца, то же заклинание начертано в соседней палате, [`crypto/vm/memo.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/vm/memo.cpp), строки [34–37](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/vm/memo.cpp#L34-L37), — внутри `DummyVmState::load_library`. Осмыслите: это уже подставной клерк, который, приступив к поиску библиотеки, сам сажает за свою конторку пустое кресло. Чучело нанимает пустоту. Временный сотрудник оформляет себе временного сотрудника. Сыщик долго стоял перед этой рекурсией, потом закрыл блокнот и расписался в тумане.

А на входе в палату, для полноты картины, стоит ещё один экспонат — `convert_code_cell`, строки [90–97](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/vm/vm.cpp#L90-L97):

```cpp
  if (global_version >= 9) {
    // Use DummyVmState instead of this to avoid consuming gas for cell loading
    DummyVmState dummy{libraries, global_version};
    Guard guard(&dummy);
    try {
      csr = load_cell_slice_ref(code_cell);
    } catch (VmError&) {  // NOLINT(*-empty-catch)
    }
```

Здесь хотя бы честно нанимают настоящее чучело — `DummyVmState`, строка [92](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/vm/vm.cpp#L92), — и прямо пишут зачем: «instead of this to avoid consuming gas», строка [91](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/vm/vm.cpp#L91). Загрузка заворачивается в `try`, а всякое возмущение машины глушится пустым `catch`, строки [96–97](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/vm/vm.cpp#L96-L97), — с деловой отметкой `// NOLINT(*-empty-catch)`, дабы ревизоры линтера не задавали вопросов. Ошибка поймана, ошибка никому не нужна, ошибка растворена. В двигательной палате, выходит, три способа не платить: сесть пустым креслом, нанять чучело и промолчать при допросе.

## СЕНСАЦИЯ ВТОРАЯ: ДВЕСТИ ПЯТЬДЕСЯТ ШЕСТЬ ИНСТРУКЦИЙ-ПРИЗРАКОВ

У всякой порядочной машины есть книга инструкций — таблица опкодов, по которой палата узнаёт, что делать с каждой командой. У машины TVM есть целая страница такой книги — диапазон `0xfe00..0xfeff`, двести пятьдесят шесть команд, — отведённая под отладку. Сыщик раскрыл страницу. Страница занята манекенами. [`crypto/vm/debugops.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/vm/debugops.cpp), строки [36–39](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/vm/debugops.cpp#L36-L39):

```cpp
int exec_dummy_debug(VmState* st, int args) {
  VM_LOG(st) << "execute DEBUG " << (args & 0xff);
  return 0;
}
```

Вот и вся служба. Инструкция принимается, её номер вписывается в журнал — «execute DEBUG», строка [37](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/vm/debugops.cpp#L37), — после чего она торжественно возвращает ноль, строка [38](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/vm/debugops.cpp#L38). Ни стека, ни ячеек, ни газа. Салют — и в сторону. Есть и манекен с бантом — `exec_dummy_debug_str`, строки [42–51](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/vm/debugops.cpp#L42-L51): он вычитывает из ленты до шестнадцати байт, чтобы... записать их в журнал словами «execute DEBUGSTR» и тоже вернуть ноль. Работа проделана, результат не наступил.

А теперь распределительный штамп, строки [153–165](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/vm/debugops.cpp#L153-L165):

```cpp
  if (!vm_debug_enabled) {
    cp0.insert(OpcodeInstr::mkfixedrange(0xfe00, 0xfef0, 16, 8, instr::dump_1c_and(0xff, "DEBUG "), exec_dummy_debug))
        .insert(OpcodeInstr::mkext(0xfef, 12, 4, dump_dummy_debug_str, exec_dummy_debug_str, compute_len_debug_str));
  } else {
    // NB: all non-redefined opcodes in fe00..feff should be redirected to dummy debug definitions
    cp0.insert(OpcodeInstr::mksimple(0xfe00, 16, "DUMPSTK", exec_dump_stack))
        .insert(OpcodeInstr::mkfixedrange(0xfe01, 0xfe14, 16, 8, instr::dump_1c_and(0xff, "DEBUG "), exec_dummy_debug))
        .insert(OpcodeInstr::mksimple(0xfe14, 16, "STRDUMP", exec_dump_string))
        .insert(OpcodeInstr::mkfixedrange(0xfe15, 0xfe20, 16, 8, instr::dump_1c_and(0xff, "DEBUG "), exec_dummy_debug))
        .insert(OpcodeInstr::mkfixed(0xfe2, 12, 4, instr::dump_1sr("DUMP"), exec_dump_value))
        .insert(OpcodeInstr::mkfixedrange(0xfe30, 0xfef0, 16, 8, instr::dump_1c_and(0xff, "DEBUG "), exec_dummy_debug))
        .insert(OpcodeInstr::mkext(0xfef, 12, 4, dump_dummy_debug_str, exec_dummy_debug_str, compute_len_debug_str));
  }
```

Если отладка не включена — строка [153](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/vm/debugops.cpp#L153), — то **вся** страница, от `0xfe00` до `0xfeff`, разом отдана манекенам, строки [154–155](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/vm/debugops.cpp#L154-L155). Если включена — всё равно почти вся: живых инструкций три (`DUMPSTK`, `STRDUMP`, `DUMP`), а промежутки между ними заботливо залиты теми же манекенами, строки [159](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/vm/debugops.cpp#L159), [161](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/vm/debugops.cpp#L161), [163](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/vm/debugops.cpp#L163). И над всем этим — распоряжение начальника цеха, строка [157](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/vm/debugops.cpp#L157): `// NB: all non-redefined opcodes in fe00..feff should be redirected to dummy debug definitions`. «NB», читатель. Nota bene. Заметь хорошенько: всякий опкод на этой странице, не занятый настоящей работой, обязан быть переадресован чучелу. Это не запущение — это устав. В машинном зале города держат штат из двухсот с лишним инструкций, которые ходят на службу, отмечаются в журнале и не производят ничего. Сыщик, выросший в городе, где за каждый газ платят, признался газете: такой роскоши он не видел даже в телеграфной конторе.

## СЕНСАЦИЯ ТРЕТЬЯ: НОТАРИУС С ПЕЧАТЬЮ ИЗ НУЛЕЙ

У машины есть и нотариальная контора: две функции, обещающие по первому требованию выдать хэш состояния — слепок всей машины целиком, двадцать шесть байт, двести пятьдесят шесть бит. Государственная важность вопроса очевидна: таким слепком заверяют, что машина — та самая машина. Сыщик потребовал слепок. [`crypto/vm/vm.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/vm/vm.cpp), строки [691–703](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/vm/vm.cpp#L691-L703):

```cpp
td::BitArray<256> VmState::get_state_hash() const {
  // TODO: implement properly, by serializing the stack etc, and computing the Merkle hash
  td::BitArray<256> res;
  res.clear();
  return res;
}

td::BitArray<256> VmState::get_final_state_hash(int exit_code) const {
  // TODO: implement properly, by serializing the stack etc, and computing the Merkle hash
  td::BitArray<256> res;
  res.clear();
  return res;
}
```

Читайте не спеша, здесь каждая строка — штрих к портрету. Строка [692](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/vm/vm.cpp#L692): «сделать как следует: сериализовать стек и прочее, и вычислить хэш Меркла». Строки [693–695](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/vm/vm.cpp#L693-L695): создать бумагу, отчистить её дочиста — `res.clear()` — и вручить просителю. Итог: двести пятьдесят шесть бит, все — нули. Итоговый слепок с указанием кода выхода, строки [698–703](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/vm/vm.cpp#L698-L703), — те же нули, в том же порядке, при любом `exit_code`: параметр принимается и не читается, как петиция в ящике для петиций.

Сыщик переспросил у протокола: давно ли контора выдаёт такие заверения? Протокол ответствовал, что контора работает исправно, жалоб нет — потому что, кажется, спрашивают редко. Но факт остаётся фактом: в двигательной палате города, где каждая ячейка оплачена, а каждый шаг посчитан, имеется нотариус, который заверяет состояние машины печатью из одних нулей и честно подписывается под печатью — «сделать как следует». Газета не знает, что прекраснее: сама печать или то, что расписка о намерении сделать как следует хранится прямо на печати.

## ОТДЕЛ НАДГРОБИЙ: ЗАКОНСЕРВИРОВАННЫЙ ХАК И ФУНКЦИЯ, КОТОРУЮ НЕ ЗОВУТ

В дальнем углу палаты, за станком регистров, сыщик набрёл на стеклянный колпак. Под колпаком — экспонат с табличкой «временный хак». [`crypto/vm/contops.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/vm/contops.cpp), строки [798–808](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/vm/contops.cpp#L798-L808):

```cpp
int exec_pop_ctr(VmState* st, unsigned args) {
  unsigned idx = args & 15;
  VM_LOG(st) << "execute POP c" << idx;
  /*
  if (idx == 3 && st->get_stack().depth() > 0 && st->get_stack().tos().is(StackEntry::t_cell)) {
    // temp hack: accept cell argument for POP c3 and do auto-BLESSing
    return exec_bless_pop_c3(st);
  }
  */
  throw_typechk(st->set(idx, st->get_stack().pop_chk()));
  return 0;
}
```

Строки [801–806](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/vm/contops.cpp#L801-L806): целая ветвь поведения упрятана в комментарий — и не просто упрятана, а снабжена музейной этикеткой, строка [803](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/vm/contops.cpp#L803): «временный хак: принимать ячейку для POP c3 и производить авто-БЛАГОСЛОВЕНИЕ». Была эпоха, читатель, когда регистр `c3` принимал ячейку и благословлял её автоматически. Эпоха кончилась, благословение отменено, но хак не выброшен — он законсервирован прямо в действующем исполнителе, меж живых строк, чтобы всякий проходящий инженер мог помянуть старые порядки. «Временный» — это сколько лет, газета не уточняет: в двигательной палате время, как мы знаем, меряют двумя мерами.

А над колпаком, живой и нетронутый, стоит сам механизм благословения — функция `exec_bless_pop_c3`, строки [790–796](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/vm/contops.cpp#L790-L796):

```cpp
int exec_bless_pop_c3(VmState* st) {
  Stack& stack = st->get_stack();
  VM_LOG(st) << "execute CTOSBLESSPOPc3";
  stack.check_underflow(1);
  throw_typechk(st->set_c(3, Ref<OrdCont>{true, vm::load_cell_slice_ref(stack.pop_cell()), st->get_cp()}));
  return 0;
}
```

Функция компилируется. Функция умеет журналировать — загадочное слово «CTOSBLESSPOPc3», строка [792](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/vm/contops.cpp#L792), достойное вывески на воротах секретного ведомства. Функция исправно благословила бы ячейку в продолжение. Единственное, чего функция не умеет, — быть вызванной: единственный приглашающий её адресат, строка [804](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/vm/contops.cpp#L804), сидит внутри комментария, и больше её в палате не зовёт никто. Сыщик проверил по книге вызовов дважды. Механизм ждёт, смазан, готов — и обречён благословлять в пустоту до скончания репозитория.

И напоследок — табличка из соседней мастерской, где ячейки пакуют в мешки. [`crypto/vm/boc.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/vm/boc.cpp), строки [55–63](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/vm/boc.cpp#L55-L63):

```cpp
  if (refs_cnt > 4) {
    if (refs_cnt != 7 || !with_hashes) {
      return td::Status::Error("Invalid first byte");
    }
    refs_cnt = 0;
    // ...
    // do not deserialize absent cells!
    return td::Status::Error("TODO: absent cells");
  }
```

Когда в первом байте мешка мастерская читает невозможное число ссылок — семь, с хэшами, — она не паникует. Она спокойно обнуляет счётчик, строка [59](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/vm/boc.cpp#L59), ставит многоточие, строка [60](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/vm/boc.cpp#L60), строгое распоряжение «не десериализовать отсутствующие ячейки!», строка [61](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/vm/boc.cpp#L61), — и выдаёт посетителю отказ, строка [62](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/vm/boc.cpp#L62), в котором текстом отказа значится служебная пометка: `«TODO: absent cells»`. Представьте, читатель: вы приходите в бюро, подаёте бумагу, а вам в ответ — «к сделать: отсутствующие ячейки». Не «бумага испорчена», не «формы нет в списках» — а напоминание самим себе, вручённое вам как официальный ответ. Газета считает это вершиной канцелярской честности: контора настолько искренна с публикой, что показывает ей свой список дел.

---

**Вывод следствия.** Двигательная палата — место, где время и деньги считают строже, чем где-либо в городе: тут плата за ячейку, плата за шаг, плата за вздох. И именно здесь сыщик нашёл всё самое человечное: пустое кресло, нанятое «временным сотрудником», чтобы не платить; чучело, нанимающее пустое кресло; двести пятьдесят шесть инструкций-манекенов с пропусками и журналом посещений; нотариуса, чья печать оттискивает нули под честной пометкой «сделать как следует»; законсервированный благословляющий хак; и мастерскую, вручающую посетителям собственный список несделанных дел. Машина, выходит, устроена как всякий порядочный лондонский департамент: снаружи — счётчики и повинности, внутри — кресла, манекены и пометки карандашом. Туман над палатой стоит густой, рожок горит ровно, газ не расходуется — за чтение этой газеты, как и за дамп стека, плата не взыскивается: кассира на месте нет. Храните слепки состояния в надёжном месте — они все одинаковые, и потерянный легко заменить любым другим.

*«Вечерний Валидатор» продолжает наблюдение.*
