+++
title = "№ 109 — Ноль, который едва не скрыл деление на ноль"
date = 2026-10-25T16:59:00+03:00
tags = ["ton"]
+++

# ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

**№ 109.** *Лондон. Туман ползет по мостовой, когда сыщик получает из [ton-blockchain/ton](https://github.com/ton-blockchain/ton) папку с необычной печатью: «ошибки найдены LLM». Газовый рожок кашляет. Искусственный разум пришел в цех Tolk и обнаружил, что ноль собирались выдавать за ответ даже тогда, когда знаменатель мог стать нулем.*

## ДЕЛО О НУЛЕ, КОТОРЫИ СЧИТАЛСЯ ОТВЕТОМ

След ведет в [коммит e6aeaef](https://github.com/ton-blockchain/ton/commit/e6aeaef681763d00b459a1712cc35592e81bcac1) репозитория [ton-blockchain/ton](https://github.com/ton-blockchain/ton). Заголовок дела не прячет свидетеля: ` [Tolk] Fix several bugs found by LLM`. Цитаты ниже сняты с этого коммита.

Главная улика лежит в [файле `tolk/builtins.cpp`](https://github.com/ton-blockchain/ton/blob/e6aeaef681763d00b459a1712cc35592e81bcac1/tolk/builtins.cpp), строки [826–846](https://github.com/ton-blockchain/ton/blob/e6aeaef681763d00b459a1712cc35592e81bcac1/tolk/builtins.cpp#L826-L846):

```cpp
static AsmOp compile_muldiv(std::vector<VarDescr>& res, std::vector<VarDescr>& args, AnyV origin,
                     int round_mode) {
  tolk_assert(res.size() == 1 && args.size() == 3);
  VarDescr &r = res[0], &x = args[0], &y = args[1], &z = args[2];
  if (x.is_int_const() && y.is_int_const() && z.is_int_const()) {
    r.set_const(muldiv(x.int_const, y.int_const, z.int_const, round_mode));
    if (!r.int_const->is_valid()) {
      err(*z.int_const == 0 ? "division by zero" : "integer overflow").fire(origin);
    }
    x.unused();
    y.unused();
    z.unused();
    return push_const(origin, r.int_const);
  }
  // 0 * y / z = 0 only when z is known non-zero; otherwise preserve potential div-by-zero at runtime
  if ((x.always_zero() || y.always_zero()) && z.always_nonzero()) {
```

Сыщик читает старую формулу по следам исправления. Если один множитель ноль, оптимизатор хотел немедленно выдать `0`, не утруждая себя вычислением деления. Для обычной арифметики соблазн понятен: ноль на что угодно — ноль. Но в подземелье компилятора знаменатель тоже имеет право на характер. Если `z` равен нулю, выражение `0 * y / z` не обязано становиться мирным нулем: сначала может случиться деление на ноль.

Прежняя стража смотрела только на нулевой множитель и уже готовила приговор. После дознания в условии появилась вторая печать — `z.always_nonzero()`. Теперь нулевой ответ выдается лишь тогда, когда знаменатель доказанно не ноль. В противном случае дело доходит до обычного пути, где [строки 848–853](https://github.com/ton-blockchain/ton/blob/e6aeaef681763d00b459a1712cc35592e81bcac1/tolk/builtins.cpp#L848-L853) прямо оставляют проверку:

```cpp
  r.val = emulate_div(emulate_mul(x.val, y.val), z.val);
  if (z.is_int_const()) {
    if (*z.int_const == 0) {
      err("division by zero").fire(origin);
    }
```

Вот где шутка становится опасной. Оптимизатор не ломал таблицу умножения; он вычеркивал сам факт преступления. Ноль должен был затушевать знаменатель, а LLM принесла в участок увеличительное стекло.

## ВТОРОЙ СЛЕД: В КОНТОРЕ НАШЛИ ДВЕ ДВЕРИ С ОДНИМ НОМЕРОМ

В том же [коммите e6aeaef](https://github.com/ton-blockchain/ton/commit/e6aeaef681763d00b459a1712cc35592e81bcac1) сыщик находит еще одну сочную починку. [Файл `tolk/pipe-generate-fif-output.cpp`](https://github.com/ton-blockchain/ton/blob/e6aeaef681763d00b459a1712cc35592e81bcac1/tolk/pipe-generate-fif-output.cpp), строки [35–43](https://github.com/ton-blockchain/ton/blob/e6aeaef681763d00b459a1712cc35592e81bcac1/tolk/pipe-generate-fif-output.cpp#L35-L43), вводит отдельного распорядителя столкновений:

```cpp
GNU_ATTRIBUTE_NOINLINE
static void fire_on_method_id_collision(FunctionPtr f1, FunctionPtr f2) {
  if (f1->is_entrypoint() && f2->is_entrypoint()) {
    err("both `main` and `onInternalMessage` are not allowed").fire(f1->ident_anchor);
  } else if (f1->is_contract_getter() && f2->is_contract_getter()) {
    err("GET methods hash collision: `{}` and `{}` produce the same method_id={}. Consider renaming one of these functions.", f2, f1, f1->tvm_method_id).fire(f1->ident_anchor);
  } else {
    err("method_id collision: `{}` and `{}` produce the same method_id={}", f2, f1, f1->tvm_method_id).fire(f1->ident_anchor);
```

А проверка теперь перебирает все методы с идентификаторами в [строках 390–400](https://github.com/ton-blockchain/ton/blob/e6aeaef681763d00b459a1712cc35592e81bcac1/tolk/pipe-generate-fif-output.cpp#L390-L400):

```cpp
    // check for collisions of method_id (must be unique: it's a key in a routing dict)
    if (fun_ref->has_tvm_method_id()) {
      has_fun_main |= fun_ref->name == "main";
      has_onInternalMessage |= fun_ref->name == "onInternalMessage";

      for (FunctionPtr other : all_methods_with_id) {
        if (other->tvm_method_id == fun_ref->tvm_method_id) {
          fire_on_method_id_collision(fun_ref, other);
        }
      }
      all_methods_with_id.push_back(fun_ref);
    }
```

Раньше столкновения отслеживали уже, но не всякую пару: свежая починка расширила картотеку до `all_methods_with_id` и отдельно различила входные двери, getter'ы и прочие методы. Один номер на двух дверях — не повод для философии, а повод остановить сборку.

## ТРЕТИЙ СЛЕД: ТРИДЦАТЬ ДВЕ ЯЧЕЙКИ И ОДНА ЛИШНЯЯ

В том же [файле `tolk/pipe-generate-fif-output.cpp`](https://github.com/ton-blockchain/ton/blob/e6aeaef681763d00b459a1712cc35592e81bcac1/tolk/pipe-generate-fif-output.cpp) строки [421–430](https://github.com/ton-blockchain/ton/blob/e6aeaef681763d00b459a1712cc35592e81bcac1/tolk/pipe-generate-fif-output.cpp#L421-L430) ставят предел глобальным переменным:

```cpp
  int n_used_globals = 0;
  for (GlobalVarPtr var_ref : G.all_global_vars) {
    if (!var_ref->is_really_used()) {
      if (G_settings.verbosity >= 2) {
        std::cerr << var_ref->name << ": variable not generated, it's unused\\n";
      }
      continue;
    }

    // Asm.fif short GETGLOB/SETGLOB accept only slots 1..31
```

Чуть ниже, на [строках 430–433](https://github.com/ton-blockchain/ton/blob/e6aeaef681763d00b459a1712cc35592e81bcac1/tolk/pipe-generate-fif-output.cpp#L430-L433), счетчик выдает приговор после тридцать первой ячейки. Компилятор не станет строить контракт, который просит шкафчик шире, чем допускает короткая форма `GETGLOB/SETGLOB`.

## ПРИГОВОР СЫЩИКА

Репозиторий [ton-blockchain/ton](https://github.com/ton-blockchain/ton) в [коммите e6aeaef](https://github.com/ton-blockchain/ton/commit/e6aeaef681763d00b459a1712cc35592e81bcac1) оставил редкую улику: в заголовке прямо сказано, что несколько ошибок нашла LLM. Самая сочная — оптимизация нуля, которая без проверки знаменателя могла спрятать деление на ноль. Лекарство простое и дословное: `z.always_nonzero()`.

Приговор без сенсационной истерики: это не доказательство падения каждой сети. Это доказательство того, что даже аккуратный оптимизатор должен отвечать на вопрос, не исчезла ли вместе с быстрым путем ошибка. Ноль — не алиби. Газовый рожок пробил сто девятый удар, а сыщик записал в протокол: иногда искусственный разум находит в машинном отделении то, что человеческий глаз давно принял за очевидность.

🐀
