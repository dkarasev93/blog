+++
title = "№ 105 — Верификатор просит заплатить спамом"
date = 2026-10-23T16:59:00+03:00
tags = ["acton"]
+++

# ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

**№ 105.** *Лондон. Туман ползет по мостовой, когда сыщик получает папку из [ton-blockchain/acton](https://github.com/ton-blockchain/acton). На папке печать коммита [10c3e6a](https://github.com/ton-blockchain/acton/commit/10c3e6addc81a1db5f417c92d01067f0fd585f53): `feat(verifier): add testnet payment for verification (#1222)`. Внутри — не взятка и не шантаж, а официальный способ проверить контракт: отправить тестовую монету с особой запиской. Канцелярия называет это прямо: «on-chain spam payment».*

## ДЕЛО О ПЛАТЕЖЕ, КОТОРЫЙ САМ СЕБЯ НАЗВАЛ СПАМОМ

Сыщик открыл [файл `src/bin/acton.rs`](https://github.com/ton-blockchain/acton/blob/10c3e6addc81a1db5f417c92d01067f0fd585f53/src/bin/acton.rs) и поднял газовый рожок над [строками 857–861](https://github.com/ton-blockchain/acton/blob/10c3e6addc81a1db5f417c92d01067f0fd585f53/src/bin/acton.rs#L857-L861):

```rust
        #[arg(
            long = "new",
            help = "Use the testnet Acton verifier with an on-chain spam payment"
        )]
        new_verifier: bool,
```

Цитата дословная, господа присяжные. Не редакция окрестила перевод монет спамом: так его описала сама кнопка командной строки. Пользователь запускает `acton verify --new`, а программа обещает ему не просто проверку исходников, а тестнетный ритуал с платежом, который официально числится спамом. Викторианский почтамт хотя бы печатал на письмах слово «реклама» до того, как отправить их по адресам.

Но шутка не заканчивается на вывеске. Под ней лежит полноценная платежная палата.

## ПЕРВЫЙ СЛЕД: КАЗНА ДЛЯ ОДНОЙ ЗАПИСКИ

В том же коммите новый [файл `apps/verifier/src/payment.rs`](https://github.com/ton-blockchain/acton/blob/10c3e6addc81a1db5f417c92d01067f0fd585f53/apps/verifier/src/payment.rs) начинает с [строк 24–35](https://github.com/ton-blockchain/acton/blob/10c3e6addc81a1db5f417c92d01067f0fd585f53/apps/verifier/src/payment.rs#L24-L35):

```rust
pub const PAYMENT_COMMENT_PREFIX: &str = "acton-verify:v1:";
const HISTORY_PAGE_SIZE: usize = 1_000;
const PROCESSING_LEASE: Duration = Duration::from_mins(5);
const PROVIDER_REQUEST_TIMEOUT: Duration = Duration::from_secs(30);
const MAX_PAYMENT_ATTEMPTS: u64 = 3;

#[derive(Clone, Debug, Serialize)]
pub struct PaymentQuote {
    pub payment_address: String,
    pub amount_nano: String,
    pub comment: String,
}
```

Пять минут на аренду обработки, тридцать секунд на провайдера, тысяча записей за страницу, три попытки и отдельный префикс для платежной записки. Пахнет не случайной монетой, а аккуратно заведенной бухгалтерией для спама. У каждой проверки будет адрес, сумма и комментарий вида `acton-verify:v1:<code_hash>`.

В [строках 143–170](https://github.com/ton-blockchain/acton/blob/10c3e6addc81a1db5f417c92d01067f0fd585f53/apps/verifier/src/payment.rs#L143-L170) контора честно ограничивает нововведение тестнетом:

```rust
    /// Opens the payment ledger and configures the testnet payment verifier.
    pub fn from_config(config: &Config) -> Result<Self, PaymentError> {
        if config.network() != TonNetwork::Testnet {
            return Err(PaymentError::UnsupportedNetwork);
        }
        let payment_address = config
            .payment_address()
            .ok_or(PaymentError::MissingConfiguration("payment.address"))?;
        validate_payment_address(payment_address)?;
        let min_amount_nano = config
            .payment_min_amount_nano()
            .filter(|amount| *amount > 0)
            .ok_or(PaymentError::MissingConfiguration(
                "payment.min_amount_nano",
            ))?;
```

Редакция отмечает: тестнет — это не основная сеть, а значит, перед нами не доказательство кражи и не повод кричать о катастрофе. Это доказательство более странное и более сочное: для проверки исходников завели специальный платный проход, а сам проход назвали спамом.

## ВТОРОЙ СЛЕД: ПЛАТЕЖНАЯ ИСТОРИЯ СТАНОВИТСЯ ОРУЖИЕМ ПАМЯТИ

В [строках 189–243](https://github.com/ton-blockchain/acton/blob/10c3e6addc81a1db5f417c92d01067f0fd585f53/apps/verifier/src/payment.rs#L189-L243) верификатор при старте поднимает историю кошелька. Сначала он берет самую новую транзакцию как вершину, затем идет по истории в обратную сторону страницами по тысяче записей, пока не встретит ту же вершину. Если порядок логических времен нарушился или вершина не найдена, палата считает восстановление неудачным.

```rust
        while !reached_tip {
            let page = self
                .client
                .transactions(
                    &self.payment_address,
                    HISTORY_PAGE_SIZE,
                    offset,
                    HistorySort::Ascending,
                )
                .await?;
            let page_len = page.len();

            for transaction in page {
                let transaction_key = (transaction.lt, transaction.hash.clone());
                if previous_transaction.as_ref().is_some_and(|previous| {
                    transaction_key.0 < previous.0
                        || (transaction_key.0 == previous.0
                            && transaction_key.1.as_str() <= previous.1.as_str())
                }) {
                    return Err(PaymentError::HistoryChangedDuringRecovery);
                }
```

Сцена достойна Лондона: перед тем как принять новую монету, контора перечитывает весь журнал платежного кошелька и проверяет, не сдвинулся ли архив под ногами. Платежная система не доверяет ни памяти процесса, ни одному ответу провайдера. Она доверяет только истории, которая сама себя не перепутала.

Это уже не «кинул монету и получил справку». Это маленький отдел по борьбе с двойным использованием квитанций. В [строках 476–501](https://github.com/ton-blockchain/acton/blob/10c3e6addc81a1db5f417c92d01067f0fd585f53/apps/verifier/src/payment.rs#L476-L501) ledger различает `processing`, `retryable` и `consumed`, а после трех попыток отправляет повторного просителя восвояси.

## ТРЕТИЙ СЛЕД: СПАМ С ПАСПОРТОМ

Самое важное правило спрятано в [строках 246–281](https://github.com/ton-blockchain/acton/blob/10c3e6addc81a1db5f417c92d01067f0fd585f53/apps/verifier/src/payment.rs#L246-L281): транзакция должна быть входящей, завершенной, адрес должен совпасть, сумма — не быть меньше минимума, а комментарий — точно соответствовать хешу кода.

```rust
    fn recovered_payment(&self, transaction: PaymentTransaction) -> Option<RecoveredPayment> {
        let incoming = valid_incoming_message(&transaction, &self.payment_address)?;
        let comment = incoming.comment.as_deref()?;
        let code_hash = code_hash_from_comment(comment)?;
        let amount_nano = incoming.value?;
        if amount_nano < self.min_amount_nano {
            return None;
        }
```

А при личной подаче квитанки [строки 269–281](https://github.com/ton-blockchain/acton/blob/10c3e6addc81a1db5f417c92d01067f0fd585f53/apps/verifier/src/payment.rs#L269-L281) повторяют досмотр: сумма, адрес и записка сверяются еще раз. Спам, как выяснилось, обязан иметь паспорт, маршрут и правильную фамилию на конверте.

В этом и состоит абсурд дела. Снаружи — строгая защита от поддельной или повторной оплаты. Внутри — вывеска, которая без всякой стыдливости сообщает пользователю: отправьте on-chain spam payment. Газовый рожок кашляет, но протокол составлен аккуратно.

## ПРИГОВОР СЫЩИКА

Репозиторий [ton-blockchain/acton](https://github.com/ton-blockchain/acton) пойман в коммите [10c3e6a](https://github.com/ton-blockchain/acton/commit/10c3e6addc81a1db5f417c92d01067f0fd585f53) на редком признании: платный доступ к верификатору сам называет собственный платеж спамом. В [строках 857–861](https://github.com/ton-blockchain/acton/blob/10c3e6addc81a1db5f417c92d01067f0fd585f53/src/bin/acton.rs#L857-L861) это стоит в справке CLI, а [apps/verifier/src/payment.rs](https://github.com/ton-blockchain/acton/blob/10c3e6addc81a1db5f417c92d01067f0fd585f53/apps/verifier/src/payment.rs) показывает, что за шуткой стоит целая платежная машина: история, лимит суммы, комментарий с хешем кода и защита от повторной подачи.

Приговор без скидки: технически это тестнетная плата, а не доказанное зло. Но назвать собственную транзакцию спамом — все равно что повесить на входе в банк табличку «внутри мелкое мошенничество», а затем удивляться, почему сыщик достал лупу. Газовый рожок пробил сто пятый удар, квитанка ушла в архив, а корреспондент растворился в лондонском тумане.

🐀
