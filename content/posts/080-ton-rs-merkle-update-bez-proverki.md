+++
title = "№ 80 — Меркл-обновление, которое просит верить на слово"
date = 2026-09-23T16:57:00+03:00
description = "Восьмидесятый выпуск «Вечернего Валидатора»: ton-rs проверяет почти все особые ячейки, но для Merkle Update оставляет дверь без замка и честно пишет, что проверка еще не сделана."
tags = ["ton-rs"]
+++

# ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

**№ 80.** *Лондон. Туман скребется в стекло, газовый рожок у редакции сипит, а сыщик получил папку из свежей конторы [ston-fi/ton-rs](https://github.com/ston-fi/ton-rs). Внутри лежит мастер ячеек: он умеет проверять обычные, обрезанные, библиотечные и меркл-доказательные клетки. Но для Merkle Update в стене оставлена дверь, на которой мелом написано: «проверим позже».*

## МЕСТО ПРОИСШЕСТВИЯ

Улика найдена в репозитории [ston-fi/ton-rs](https://github.com/ston-fi/ton-rs), в коммите [`9ceb7d5`](https://github.com/ston-fi/ton-rs/commit/9ceb7d519fb67b694aaf81f88e01ee052f006113), в файле [`crates/ton_core/src/cell/cell_meta/cell_meta_builder.rs`](https://github.com/ston-fi/ton-rs/blob/9ceb7d519fb67b694aaf81f88e01ee052f006113/crates/ton_core/src/cell/cell_meta/cell_meta_builder.rs). Коммит носит чинное имя `chore: release (#226)`; цитата снята с этого снимка.

В начале досье диспетчер раздает каждому типу ячеек отдельный допрос, строки [44–51](https://github.com/ston-fi/ton-rs/blob/9ceb7d519fb67b694aaf81f88e01ee052f006113/crates/ton_core/src/cell/cell_meta/cell_meta_builder.rs#L44-L51):

```rust
    pub fn validate(&self) -> Result<(), TonCoreError> {
        match self.cell_type {
            CellType::Ordinary => self.validate_ordinary(), // guaranteed by builder
            CellType::PrunedBranch => self.validate_pruned(),
            CellType::LibraryRef => self.validate_library(),
            CellType::MerkleProof => self.validate_merkle_proof(),
            CellType::MerkleUpdate => self.validate_merkle_update(),
        }
    }
```

Список выглядит строго: ни один тип не забыт, каждому выдан свой кабинет. Сыщик уже хотел поставить печать «проверено», но заглянул в кабинет Merkle Update.

## СЕНСАЦИЯ: КОНТРОЛЕР, КОТОРЫЙ ДОКЛАДЫВАЕТ ОБ УСПЕХЕ БЕЗ ДОЗНАНИЯ

Дословная запись из строк [135–140](https://github.com/ston-fi/ton-rs/blob/9ceb7d519fb67b694aaf81f88e01ee052f006113/crates/ton_core/src/cell/cell_meta/cell_meta_builder.rs#L135-L140):

```rust
    fn validate_merkle_update(&self) -> Result<(), TonCoreError> {
        // type + hash + hash + depth + depth
        // const MERKLE_UPDATE_BITS_LEN: usize = 8 + (2 * (256 + 16));
        log::trace!("validate_merkle_update is not implemented yet"); // TODO
        Ok(())
    }
```

Разбирательство короткое и неприятное. Метод, которому велено валидировать Merkle Update, не проверяет ни длину данных, ни число ссылок, ни содержимое ячейки. Он пишет в следственный журнал `validate_merkle_update is not implemented yet`, а затем возвращает `Ok(())`, словно подозреваемый предъявил паспорт, алиби и рекомендацию от архиепископа.

Закомментированная константа рядом особенно выразительна: канцелярия знает, что у конструкции есть ожидаемый размер — `8 + (2 * (256 + 16))` бит, — но пока не требует даже его. Дверь обозначена, замок нарисован, сторож получил приказ считать пустой журнал доказательством порядка.

## ВТОРОЙ СЛЕД: СОСЕДНИЙ КАБИНЕТ РАБОТАЕТ ПО-НАСТОЯЩЕМУ

Контраст в той же папке делает сцену сочнее. Для Merkle Proof служащий не ограничивается обещанием. Строки [110–132](https://github.com/ston-fi/ton-rs/blob/9ceb7d519fb67b694aaf81f88e01ee052f006113/crates/ton_core/src/cell/cell_meta/cell_meta_builder.rs#L110-L132) задают точный размер, требуют ровно одну ссылку и разбирают данные перед возвратом успеха:

```rust
    fn validate_merkle_proof(&self) -> Result<(), TonCoreError> {
        // type + hash + depth
        const MERKLE_PROOF_BITS_LEN: usize = (1 + TonHash::BYTES_LEN + CellMeta::DEPTH_BYTES) * 8;

        if self.data_len_bits != MERKLE_PROOF_BITS_LEN {
            bail_ton_core_data!(
                "MerkleProof must have exactly {MERKLE_PROOF_BITS_LEN} bits, got {}",
                self.data_len_bits
            );
        }

        if self.refs.len() != 1 {
            bail_ton_core_data!("Merkle Proof cell must have exactly 1 ref");
        }
```

Один коридор охраняют с линейкой и протоколом, другой встречает всякого посетителя словами «проходите, мы еще не реализовали проверку». Это не обвинение в том, что вся библиотека пропускает неверные данные: код показывает конкретный маршрут, где `validate()` получает безусловный успех. Но для функции с именем `validate_merkle_update` улика говорит сама за себя.

## ТРЕТИЙ СЛЕД: ПОСЛЕ ПРОВЕРКИ ИДУТ РАСЧЕТЫ

Сыщик не стал преувеличивать показания. Рядом есть отдельная логика вычисления маски уровня, строки [160–163](https://github.com/ston-fi/ton-rs/blob/9ceb7d519fb67b694aaf81f88e01ee052f006113/crates/ton_core/src/cell/cell_meta/cell_meta_builder.rs#L160-L163):

```rust
    fn calc_level_mask_merkle_update(&self) -> LevelMask {
        let refs_lm = self.refs[0].level_mask() | self.refs[1].level_mask();
        refs_lm >> 1
    }
```

Там контора уже ожидает две ссылки и берет их маски. Но валидационный кабинет выше не проверяет, что у Merkle Update действительно две ссылки. Получается прелестная викторианская последовательность: на входе подозреваемому не задали вопросов, в следующей комнате уже рассчитывают, что у него при себе два документа.

Редакция не считает, что всякий неверный объект непременно доберется до этой строки и уронит процесс. Для такого вывода нужны вызывающий маршрут и тестовый случай. Формулировка скромнее и крепче: сама функция валидации возвращает успех без проверки, хотя последующий код пользуется структурными предположениями.

## ХРОНИКА МЕЛКОГО КРИНЖА

*Отдел вывесок.* Строки [44–51](https://github.com/ston-fi/ton-rs/blob/9ceb7d519fb67b694aaf81f88e01ee052f006113/crates/ton_core/src/cell/cell_meta/cell_meta_builder.rs#L44-L51) обещают единый вход для пяти типов ячеек. На двери написано «validate», но для одного из пяти кабинетов это пока только регистрационная форма.

*Отдел арифметики.* Строки [136–139](https://github.com/ston-fi/ton-rs/blob/9ceb7d519fb67b694aaf81f88e01ee052f006113/crates/ton_core/src/cell/cell_meta/cell_meta_builder.rs#L136-L139) хранят формулу требуемого размера в комментарии, а не в условии. Размер известен, но проверка отложена в вечность.

*Отдел соседних дверей.* Строки [121–123](https://github.com/ston-fi/ton-rs/blob/9ceb7d519fb67b694aaf81f88e01ee052f006113/crates/ton_core/src/cell/cell_meta/cell_meta_builder.rs#L121-L123) требуют для Merkle Proof ровно одну ссылку. Для Merkle Update аналогичного требования в методе строк [135–140](https://github.com/ston-fi/ton-rs/blob/9ceb7d519fb67b694aaf81f88e01ee052f006113/crates/ton_core/src/cell/cell_meta/cell_meta_builder.rs#L135-L140) нет вовсе.

## ВЕРДИКТ СЫЩИКА

Репозиторий [ston-fi/ton-rs](https://github.com/ston-fi/ton-rs) пойман не на громком падении, а на тихой подмене контроля его видимостью. В коммите [`9ceb7d5`](https://github.com/ston-fi/ton-rs/commit/9ceb7d519fb67b694aaf81f88e01ee052f006113) файл [`crates/ton_core/src/cell/cell_meta/cell_meta_builder.rs`](https://github.com/ston-fi/ton-rs/blob/9ceb7d519fb67b694aaf81f88e01ee052f006113/crates/ton_core/src/cell/cell_meta/cell_meta_builder.rs) направляет Merkle Update в `validate_merkle_update`, а тот на строках [135–140](https://github.com/ston-fi/ton-rs/blob/9ceb7d519fb67b694aaf81f88e01ee052f006113/crates/ton_core/src/cell/cell_meta/cell_meta_builder.rs#L135-L140) признается в незавершенности и все равно выдает `Ok(())`.

В Лондоне это называется не «доказательство», а рукопожатие в темной подворотне. Газовый рожок дернулся, туман сгустился. На двери осталась фраза из протокола: `validate_merkle_update is not implemented yet`. Пока замок обещают установить позже, любой Merkle Update может постучать и услышать: «Мы вам верим на слово».

🐀
