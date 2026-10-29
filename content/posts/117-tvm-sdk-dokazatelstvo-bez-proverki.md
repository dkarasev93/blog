---
title: "№ 117 — Доказательство, которому забыли задать последний вопрос"
date: 2026-10-29T16:59:00+03:00
tags: [tvm-sdk]
---

# ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

## ДЕЛО О ПРОВЕРКЕ, КОТОРАЯ ОСТАНОВИЛАСЬ НА ПОЛПУТИ

Лондон тонул в тумане, когда сыщик «Вечернего Валидатора» получил папку из [tvmlabs/tvm-sdk](https://github.com/tvmlabs/tvm-sdk). Внутри лежал не просто новый опкод, а признание, от которого газовый рожок поперхнулся: рекурсивное доказательство могло пройти первую проверку, не доказав, что свернутые внутренние доказательства вообще верны.

## МЕСТО ПРОИСШЕСТВИЯ

Улика отмечена в [коммите fd76a8a](https://github.com/tvmlabs/tvm-sdk/commit/fd76a8a928f6affcfc1aeddb7cc42f6cd11297a2), в файле [tvm_vm/halo2_test_data/rotate_light_client/README.md](https://github.com/tvmlabs/tvm-sdk/blob/fd76a8a928f6affcfc1aeddb7cc42f6cd11297a2/tvm_vm/halo2_test_data/rotate_light_client/README.md), строки [19–28](https://github.com/tvmlabs/tvm-sdk/blob/fd76a8a928f6affcfc1aeddb7cc42f6cd11297a2/tvm_vm/halo2_test_data/rotate_light_client/README.md#L19-L28). Цитата дословная:

> `ZKHALO2VERIFYWITHVK` performs a plain SHPLONK `verify_proof` (`SingleStrategy`,
> Blake2b). It does **not** pair `instances[0..12]` (the KZG accumulator) against
> the embedded `[s]·G2`, so it accepts the aggregation proof without checking that
> the folded inner shard/step proofs are valid.

Переводить тут почти нечего. Опкод видел агрегированную оболочку, ставил печать «проверено» и не заглядывал в карманы, где лежали двенадцать элементов KZG-аккумулятора. Внутренние доказательства могли быть неверны, но наружная дверь уже открывалась.

## ПЕРВЫЙ СЛЕД: ПРИЗНАНИЕ В СОБСТВЕННОМ ДЕЛЕ

В [строках 21–28](https://github.com/tvmlabs/tvm-sdk/blob/fd76a8a928f6affcfc1aeddb7cc42f6cd11297a2/tvm_vm/halo2_test_data/rotate_light_client/README.md#L21-L28) контора сама называет эту проверку недостаточной и требует отдельный decider extension. Это не жалоба постороннего наблюдателя, а инструкция к тестовому набору для ETH light-client rotate.

Схема была особенно нарядной: корень — дерево рекурсии над восемью shard-доказательствами и одним step-доказательством. Однако обычный SHPLONK подтверждал только отношение агрегатора. Связь аккумулятора с SRS оставалась за дверью. Получался лондонский нотариус, который удостоверяет конверт, не вскрывая письмо.

## ВТОРОЙ СЛЕД: ПОЧИНКА С ПОМЕТКОЙ «ЛОМАЕТ КОНСЕНСУС»

В [tvm_vm/CHANGELOG.md](https://github.com/tvmlabs/tvm-sdk/blob/fd76a8a928f6affcfc1aeddb7cc42f6cd11297a2/tvm_vm/CHANGELOG.md) [строка 16](https://github.com/tvmlabs/tvm-sdk/blob/fd76a8a928f6affcfc1aeddb7cc42f6cd11297a2/tvm_vm/CHANGELOG.md#L16) сообщает о новом спаривании `e(lhs, [1]G2) == e(rhs, [s]G2)`. А [строка 8](https://github.com/tvmlabs/tvm-sdk/blob/fd76a8a928f6affcfc1aeddb7cc42f6cd11297a2/tvm_vm/CHANGELOG.md#L8) предупреждает: прежнее поле заголовка было частью reserved-региона, а теперь принимает только `0` или `12` и при другом значении бросает `FatalError`.

Еще прямее звучит [корневой CHANGELOG](https://github.com/tvmlabs/tvm-sdk/blob/fd76a8a928f6affcfc1aeddb7cc42f6cd11297a2/CHANGELOG.md), строки [9–15](https://github.com/tvmlabs/tvm-sdk/blob/fd76a8a928f6affcfc1aeddb7cc42f6cd11297a2/CHANGELOG.md#L9-L15): узлы на версиях 3.0.x и 3.1.0 могут получить разный результат транзакции, а прежняя схема принимала агрегат без проверки вложенных доказательств.

Цитата дословная:

> - Consensus: VM behaviour changes on `gosh`-feature builds — a new opcode and a stricter `ZKHALO2VERIFYWITHVK`. Nodes on 3.0.x and 3.1.0 can disagree on the result of a transaction that uses either, so every validator of a network has to move to 3.1.0 together.
> - `ZKHALO2VERIFYWITHVK`: recursive-aggregation (snark-verifier) proofs are now checked in full. A VkBlob with `accumulator_limbs = 12` makes the opcode also run the KZG accumulator pairing check over `instances[0..12]`; until now such a proof was accepted without verifying the inner proofs it folds.

Это уже не просто неловкая строчка в тесте. Это смена поведения VM, где разъехавшиеся версии валидаторов могут по-разному ответить на один и тот же вызов.

## ТРЕТИЙ СЛЕД: ПАТЧ В ТРИДЦАТЬ СТРОК

Сыщик заглянул в [коммит fd76a8a](https://github.com/tvmlabs/tvm-sdk/commit/fd76a8a928f6affcfc1aeddb7cc42f6cd11297a2), файл [tvm_vm/src/executor/zk_halo2_with_vk.rs](https://github.com/tvmlabs/tvm-sdk/blob/fd76a8a928f6affcfc1aeddb7cc42f6cd11297a2/tvm_vm/src/executor/zk_halo2_with_vk.rs), строки [327–336](https://github.com/tvmlabs/tvm-sdk/blob/fd76a8a928f6affcfc1aeddb7cc42f6cd11297a2/tvm_vm/src/executor/zk_halo2_with_vk.rs#L327-L336). Там причина описана почти как показания свидетеля:

```rust
// A snark-verifier aggregation proof defers its inner KZG opening checks into a
// 12-limb accumulator exposed as `instances[0..12]`. A plain SHPLONK
// `verify_proof` proves the AGGREGATION relation but NOT that accumulator, so
// the folded inner proofs go unchecked.
```

Дальше [строки 474–486](https://github.com/tvmlabs/tvm-sdk/blob/fd76a8a928f6affcfc1aeddb7cc42f6cd11297a2/tvm_vm/src/executor/zk_halo2_with_vk.rs#L474-L486) требуют сразу два условия: обычный `verify_proof` и KZG decider, если заголовок VkBlob сообщает `12`. Раньше было достаточно первого. Теперь второй замок наконец поставили на дверь.

## ПРИГОВОР СЫЩИКА

[Тестовый набор rotate](https://github.com/tvmlabs/tvm-sdk/blob/fd76a8a928f6affcfc1aeddb7cc42f6cd11297a2/tvm_vm/halo2_test_data/rotate_light_client/README.md), [коммит fd76a8a](https://github.com/tvmlabs/tvm-sdk/commit/fd76a8a928f6affcfc1aeddb7cc42f6cd11297a2), строки [19–28](https://github.com/tvmlabs/tvm-sdk/blob/fd76a8a928f6affcfc1aeddb7cc42f6cd11297a2/tvm_vm/halo2_test_data/rotate_light_client/README.md#L19-L28): доказательство считали, но не доводили до последнего вопроса. А последний вопрос в криптографии обычно и есть тот, ради которого сыщик не спит.

Вердикт без театра: старый путь проверял агрегат, но не внутреннюю криптографическую обязанность, которую тот обещал выполнить. Исправление добавило спаривание и одновременно изменило правила VM для `gosh`. Версия валидатора здесь не косметика, а часть показаний.

Газовый рожок пробил сто семнадцатый удар. В тумане остался один совет: если релиз сам пишет «до сих пор принималось без проверки», папку надо закрывать не после чтения changelog, а после проверки всех узлов.

🐀
