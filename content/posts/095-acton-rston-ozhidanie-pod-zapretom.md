---
title: "№ 95 — Ожидание, которому запретили падать"
date: 2026-10-01T16:58:00+03:00
description: "Девяносто пятый выпуск «Вечернего Валидатора»: rston выставил expect_used = deny, заменил несколько паник на ошибки и заставил отсутствующую ячейку помнить все четыре уровня TON."
tags: [acton]
---

# ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

**№ 95.** *Лондон. Туман прижался к окнам редакции, когда сыщик получил папку из [ton-blockchain/acton](https://github.com/ton-blockchain/acton). На обложке стояла суровая надпись: `eliminate expect panics and deny expect calls`. Внутри библиотека rston сперва запретила слово `expect`, затем призналась, что отдельная ячейка помнит не все уровни хеша, а меркловское дознание иногда возвращает пустой карман без ясного рапорта. Газовый рожок кашлянул: даже паника теперь должна выдать документ.*

## МЕСТО ПРОИСШЕСТВИЯ

Улика найдена в репозитории [ton-blockchain/acton](https://github.com/ton-blockchain/acton), в коммите [`eab40ee`](https://github.com/ton-blockchain/acton/commit/eab40eebcb534150eb856d33040c74d0b63afe4c), названном `fix(rston): eliminate expect panics and deny expect calls`. Главные комнаты дела — файлы [`libs/rston/src/cell/cell_impl/mod.rs`](https://github.com/ton-blockchain/acton/blob/eab40eebcb534150eb856d33040c74d0b63afe4c/libs/rston/src/cell/cell_impl/mod.rs), [`libs/rston/src/cell/mod.rs`](https://github.com/ton-blockchain/acton/blob/eab40eebcb534150eb856d33040c74d0b63afe4c/libs/rston/src/cell/mod.rs), [`libs/rston/src/merkle/proof.rs`](https://github.com/ton-blockchain/acton/blob/eab40eebcb534150eb856d33040c74d0b63afe4c/libs/rston/src/merkle/proof.rs) и [`libs/rston/Cargo.toml`](https://github.com/ton-blockchain/acton/blob/eab40eebcb534150eb856d33040c74d0b63afe4c/Cargo.toml). Цитаты сверены с этим коммитом.

## ПЕРВЫЙ СЛЕД: СЛОВО `EXPECT` ВЫЗВАЛИ НА ДОПРОС

Самая короткая, но самая безжалостная записка лежит в [`libs/rston/Cargo.toml`](https://github.com/ton-blockchain/acton/blob/eab40eebcb534150eb856d33040c74d0b63afe4c/Cargo.toml), строки [147–151](https://github.com/ton-blockchain/acton/blob/eab40eebcb534150eb856d33040c74d0b63afe4c/libs/rston/Cargo.toml#L147-L151):

```toml
[lints.rust]
unexpected_cfgs = { level = "warn", check-cfg = ['cfg(fuzzing)'] }

[lints.clippy]
expect_used = "deny"
```

`expect_used = "deny"` — не совет и не пожелание дежурному программисту. Это приказ линтеру: любой новый `expect` в библиотеке становится нарушением дисциплины. Коммит [`eab40ee`](https://github.com/ton-blockchain/acton/commit/eab40eebcb534150eb856d33040c74d0b63afe4c) поставил запрет рядом с проверкой конфигурации, словно в участке открыли отдел внутренней безопасности.

Пикантность в том, что запрет пришлось вводить не в пустом доме. В той же папке старый код уже оставлял места, где отсутствие результата превращалось в панику. Теперь сыщик видит не обещание «никогда не падаем», а строгий маршрут: вместо крика в темноте библиотека должна вернуть ошибку, которую можно передать дальше.

## ВТОРОЙ СЛЕД: ХЕШ, КОТОРЫЙ РАНЬШЕ БРОСАЛСЯ С ВЫСОТЫ

В [`libs/rston/src/cell/mod.rs`](https://github.com/ton-blockchain/acton/blob/eab40eebcb534150eb856d33040c74d0b63afe4c/libs/rston/src/cell/mod.rs) коммит меняет контракт у `HashBytes::from_slice`. На строках [710–716](https://github.com/ton-blockchain/acton/blob/eab40eebcb534150eb856d33040c74d0b63afe4c/libs/rston/src/cell/mod.rs#L710-L716) лежит дословная запись:

```rust
/// Copies a 32-byte slice into a hash.
///
/// Returns an error if the slice does not contain exactly 32 bytes.
#[inline]
pub fn from_slice(slice: &[u8]) -> Result<Self, std::array::TryFromSliceError> {
    slice.try_into().map(Self)
}
```

Раньше неправильная длина была поводом для `expect`: срез не на 32 байта мог превратить обычную проверку входа в аварию. Теперь паспорт хеша оформляют как `Result`. В [`libs/rston/CHANGELOG.md`](https://github.com/ton-blockchain/acton/blob/eab40eebcb534150eb856d33040c74d0b63afe4c/libs/rston/CHANGELOG.md) строки [5–8](https://github.com/ton-blockchain/acton/blob/eab40eebcb534150eb856d33040c74d0b63afe4c/libs/rston/CHANGELOG.md#L5-L8) без дипломатии сообщают: длина не 32 байта возвращает ошибку, а не панику.

Картина смешна своей простотой. Чтобы двадцать девять или тридцать три байта не устроили пожар, контора переписала сам входной билет: прежде билет рвался в руках стражи, теперь его можно официально отклонить. Вежливость к плохому свидетелю победила `expect`.

## ТРЕТИЙ СЛЕД: ОТСУТСТВУЮЩАЯ ЯЧЕЙКА ПОЛУЧИЛА ЧЕТЫРЕ ПАМЯТИ

В [`libs/rston/src/cell/cell_impl/mod.rs`](https://github.com/ton-blockchain/acton/blob/eab40eebcb534150eb856d33040c74d0b63afe4c/libs/rston/src/cell/cell_impl/mod.rs) хранится более редкое признание. `AbsentCell` теперь носит массив ровно из четырех пар на строках [431–434](https://github.com/ton-blockchain/acton/blob/eab40eebcb534150eb856d33040c74d0b63afe4c/libs/rston/src/cell/cell_impl/mod.rs#L431-L434), а заполняется он так, на строках [441–449](https://github.com/ton-blockchain/acton/blob/eab40eebcb534150eb856d33040c74d0b63afe4c/libs/rston/src/cell/cell_impl/mod.rs#L441-L449):

```rust
pub struct AbsentCell {
    d1: u8,
    hashes: [(HashBytes, u16); 4],
}

impl AbsentCell {
    pub fn new<T: AsRef<DynCell>>(cell: T) -> Cell {
        let cell = cell.as_ref();
        let desc = cell.descriptor();
        // Copy only level mask bits and fill the rest with absent mask.
        let d1 = (desc.d1 & CellDescriptor::LEVEL_MASK) | CellDescriptor::ABSENT_MASK;

        let hashes = [0, 1, 2, 3].map(|level| (*cell.hash(level), cell.depth(level)));

        Cell(CellInner::new(Self { d1, hashes }))
    }
```

Отсутствующая ячейка — уже звучит как свидетель, которого нет в комнате, но его отпечатки нужны делу. Прежняя версия хранила только набор, выведенный из маски уровня. Новый коммит кладет в карман все четыре уровня TON, даже если потом часть уровней не понадобится.

А затем стража перестает гадать по индексу. На строках [452–460](https://github.com/ton-blockchain/acton/blob/eab40eebcb534150eb856d33040c74d0b63afe4c/libs/rston/src/cell/cell_impl/mod.rs#L452-L460) уровень выше третьего получает значение третьего:

```rust
/// Every TON level has a cached value; higher levels use level 3.
fn hash_and_depth(&self, level: u8) -> &(HashBytes, u16) {
    let [level0, level1, level2, level3] = &self.hashes;
    match level {
        0 => level0,
        1 => level1,
        2 => level2,
        _ => level3,
    }
}
```

Дальше `hash` и `depth` обращаются к этому общему шкафу на строках [498–504](https://github.com/ton-blockchain/acton/blob/eab40eebcb534150eb856d33040c74d0b63afe4c/libs/rston/src/cell/cell_impl/mod.rs#L498-L504). Сыщик не станет утверждать, что всякая старая маска уже падала. Факт прозаичнее и сочнее: в тестах теперь перебираются уровни от нулевого до 255, а ячейка обязана ответить без выхода за полку.

## ЧЕТВЕРТЫЙ СЛЕД: МЕРКЛОВСКАЯ ЛЕСТНИЦА БОЛЬШЕ НЕ ПРИКИДЫВАЕТСЯ ЦЕЛОЙ

В [`libs/rston/src/merkle/proof.rs`](https://github.com/ton-blockchain/acton/blob/eab40eebcb534150eb856d33040c74d0b63afe4c/libs/rston/src/merkle/proof.rs) коммит проходит сразу по двум маршрутам. В последовательном построении доказательства на строках [500–506](https://github.com/ton-blockchain/acton/blob/eab40eebcb534150eb856d33040c74d0b63afe4c/libs/rston/src/merkle/proof.rs#L500-L506) отсутствующая ссылка получает `CellUnderflow`:

```rust
// Check if child is in a tree
match self.filter.check(child_repr_hash) {
    // Included subtrees are used as is
    FilterAction::IncludeSubtree => last
        .references
        .peek_prev_cloned()
        .ok_or(Error::CellUnderflow)?,
```

Параллельный маршрут повторяет признание на строках [650–657](https://github.com/ton-blockchain/acton/blob/eab40eebcb534150eb856d33040c74d0b63afe4c/libs/rston/src/merkle/proof.rs#L650-L657):

```rust
// Check if child is in a tree
match self.filter.check(child_repr_hash) {
    // Included subtrees are used as is
    FilterAction::IncludeSubtree => ExtCell::Ordinary(
        last.references
            .peek_prev_cloned()
            .ok_or(Error::CellUnderflow)?,
    ),
```

Раньше здесь стояло `expect("mut not fail")` — фраза, похожая на записку преступника, который уверяет, что дверь точно не заклинит. Коммит заменил уверенность на `CellUnderflow`. Если ссылка пропала, программа теперь может сообщить о нехватке ячейки, а не свалиться с видом человека, который только что обнаружил отсутствие пола.

Особенно хороша повторная улика: один ремонт сделан в обычной лестнице, второй — в параллельной. Ошибка не получила права исчезнуть при смене маршрута. В [журнале изменений rston](https://github.com/ton-blockchain/acton/blob/eab40eebcb534150eb856d33040c74d0b63afe4c/libs/rston/CHANGELOG.md) строки [36–42](https://github.com/ton-blockchain/acton/blob/eab40eebcb534150eb856d33040c74d0b63afe4c/libs/rston/CHANGELOG.md#L36-L42) сухо перечисляют три перемены: запрет `expect`, четыре уровня `AbsentCell` и `CellUnderflow` для двух видов меркловского дознания. Сухо — но за каждой строкой слышен скрип падающей лестницы.

## ПЯТЫЙ СЛЕД: КОМИТТ САМ ОСТАВИЛ ПЕЧАТЬ

Название коммита [`eab40ee`](https://github.com/ton-blockchain/acton/commit/eab40eebcb534150eb856d33040c74d0b63afe4c) не прячется за словами «рефакторинг» или «обновление зависимостей». В нем прямо записано: `eliminate expect panics and deny expect calls`. Контора одновременно закрыла вход для новых `expect` и вымела несколько старых паник из комнат, где обрабатываются хеши, отсутствующие ячейки и доказательства.

Это не доказательство, что вся библиотека раньше рушилась при каждом неверном байте. Редакция держит шляпу: по приведенным файлам можно честно утверждать только масштаб ремонта. Коммит изменил тип результата у хеша, добавил безопасный ответ для недостающей ссылки и сделал хранение уровней явным. Сыщик не выдумывает пожар, когда у него есть вполне настоящий дым.

## ХРОНИКА МЕЛКОЙ КАНЦЕЛЯРИИ

*Отдел кошельков.* В [журнале изменений rston](https://github.com/ton-blockchain/acton/blob/eab40eebcb534150eb856d33040c74d0b63afe4c/libs/rston/CHANGELOG.md) строки [18–21](https://github.com/ton-blockchain/acton/blob/eab40eebcb534150eb856d33040c74d0b63afe4c/libs/rston/CHANGELOG.md#L18-L21) обещают, что запросы сверх лимита кошелька возвращают `TooManyMessages` с фактическим числом и пределом. Даже очередь писем теперь старается показывать цифры, а не только хлопать дверью.

*Отдел старых имен.* В [том же файле журнала](https://github.com/ton-blockchain/acton/blob/eab40eebcb534150eb856d33040c74d0b63afe4c/libs/rston/CHANGELOG.md) строки [46–49](https://github.com/ton-blockchain/acton/blob/eab40eebcb534150eb856d33040c74d0b63afe4c/libs/rston/CHANGELOG.md#L46-L49) сообщают о переименовании `tycho-types` в `rston` и о переходе на общий формат. Палата меняет вывеску, пока стража проверяет замки.

*Отдел паники.* В [файле ячейки](https://github.com/ton-blockchain/acton/blob/eab40eebcb534150eb856d33040c74d0b63afe4c/libs/rston/src/cell/cell_impl/mod.rs) строки [474–479](https://github.com/ton-blockchain/acton/blob/eab40eebcb534150eb856d33040c74d0b63afe4c/libs/rston/src/cell/cell_impl/mod.rs#L474-L479) все еще честно паникуют при попытке попросить у отсутствующей ячейки данные или длину битов. Ремонт был точечным: хеши и глубины вытащили из опасного шкафа, а не сделали бессмертными все методы подряд.

## ВЕРДИКТ СЫЩИКА

Репозиторий [ton-blockchain/acton](https://github.com/ton-blockchain/acton) пойман в коммите [`eab40ee`](https://github.com/ton-blockchain/acton/commit/eab40eebcb534150eb856d33040c74d0b63afe4c) на деле о запретной панике. В [`libs/rston/Cargo.toml`](https://github.com/ton-blockchain/acton/blob/eab40eebcb534150eb856d33040c74d0b63afe4c/Cargo.toml) строки [147–151](https://github.com/ton-blockchain/acton/blob/eab40eebcb534150eb856d33040c74d0b63afe4c/Cargo.toml#L147-L151) запрещают `expect`. В [`libs/rston/src/cell/mod.rs`](https://github.com/ton-blockchain/acton/blob/eab40eebcb534150eb856d33040c74d0b63afe4c/libs/rston/src/cell/mod.rs) строки [710–716](https://github.com/ton-blockchain/acton/blob/eab40eebcb534150eb856d33040c74d0b63afe4c/libs/rston/src/cell/mod.rs#L710-L716) превращают плохую длину хеша в `Result`. В [`libs/rston/src/merkle/proof.rs`](https://github.com/ton-blockchain/acton/blob/eab40eebcb534150eb856d33040c74d0b63afe4c/libs/rston/src/merkle/proof.rs) строки [503–506](https://github.com/ton-blockchain/acton/blob/eab40eebcb534150eb856d33040c74d0b63afe4c/libs/rston/src/merkle/proof.rs#L503-L506) и [653–657](https://github.com/ton-blockchain/acton/blob/eab40eebcb534150eb856d33040c74d0b63afe4c/libs/rston/src/merkle/proof.rs#L653-L657) недостающая ссылка получает имя `CellUnderflow`.

Приговор таков: если библиотека знает, что дверь может быть пуста, не ставь у порога табличку `mut not fail`. Верни понятную ошибку, запасись всеми четырьмя уровнями и запрети новым служащим повторять старый фокус. Газовый рожок пробил один удар, `expect` отправился в архив, а сыщик растворился в тумане.

🐀
