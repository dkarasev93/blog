+++
title = "№ 85 — Леджер, который ждет одиннадцать секунд"
date = 2026-09-26T16:58:00+03:00
description = "Восемьдесят пятый выпуск «Вечернего Валидатора»: ton-rs чинит macOS HIDAPI отдельным бессмертным потоком, но физическую замену Ledger все еще оставляет без проверки."
tags = ["ton-rs"]
+++

# ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

**№ 85.** *Лондон. Вечерний туман залил мостовую, газовый рожок у редакции сипит, а сыщик получил папку из конторы [ston-fi/ton-rs](https://github.com/ston-fi/ton-rs). На обложке обещан кошелек для Ledger: подключить устройство, прочитать адрес, пережить замену. В подвале папки обнаружен особый метод испытания: отпустить кошелек, подождать одиннадцать секунд, заменить железо и нажать Enter. Сыщик снял шляпу. Это не тест, а викторианский сеанс ожидания, в котором главный свидетель — таймер Tokio.*

## МЕСТО ПРОИСШЕСТВИЯ

Улика найдена в репозитории [ston-fi/ton-rs](https://github.com/ston-fi/ton-rs), в коммите [`7dc14a4`](https://github.com/ston-fi/ton-rs/commit/7dc14a498f952b6f6238e2045d952ce395c72f59), в файлах [`crates/ton_ledger/README.md`](https://github.com/ston-fi/ton-rs/blob/7dc14a498f952b6f6238e2045d952ce395c72f59/crates/ton_ledger/README.md), [`crates/ton_ledger/src/transports/hid.rs`](https://github.com/ston-fi/ton-rs/blob/7dc14a498f952b6f6238e2045d952ce395c72f59/crates/ton_ledger/src/transports/hid.rs) и [`examples/ton_ledger_reconnect.rs`](https://github.com/ston-fi/ton-rs/blob/7dc14a498f952b6f6238e2045d952ce395c72f59/examples/ton_ledger_reconnect.rs). Коммит называется `Fix #231: Keep HIDAPI thread alive across Ledger reconnections`; цитаты сняты с этого снимка.

## СЕНСАЦИЯ: КОШЕЛЕКУ НУЖЕН БЕССМЕРТНЫЙ СЛУЖАЩИЙ

В файле [`crates/ton_ledger/src/transports/hid.rs`](https://github.com/ston-fi/ton-rs/blob/7dc14a498f952b6f6238e2045d952ce395c72f59/crates/ton_ledger/src/transports/hid.rs) на строках [113–118](https://github.com/ston-fi/ton-rs/blob/7dc14a498f952b6f6238e2045d952ce395c72f59/crates/ton_ledger/src/transports/hid.rs#L113-L118) контора объясняет, почему обычный поток ей больше не годится:

```rust
// HIDAPI's C context is never deinitialized by its Rust wrapper. On macOS its
// IOHIDManager uses the initializing thread's CFRunLoop, so that thread must not
// be a Tokio blocking-pool thread or a short-lived device worker. The static
// sender keeps this worker alive even after all transports and runtimes drop.
static API_WORKER: LazyLock<Result<mpsc::Sender<ApiReply>, std::io::Error>> =
    LazyLock::new(|| spawn_api_worker(|| HidApi::new().map_err(backend)));
```

Картина мрачна, но аккуратна: C-контекст не умирает, macOS привязывает менеджер HID к циклу событий потока, а Tokio любит убирать бездельников. Поэтому для инициализации USB заводят статический канал и поток, который должен пережить все кошельки и даже закрытие runtime. В Лондоне такой служащий получил бы жалованье за вечное присутствие у пустой двери.

Сама фабрика бессмертия стоит на строках [129–142](https://github.com/ston-fi/ton-rs/blob/7dc14a498f952b6f6238e2045d952ce395c72f59/crates/ton_ledger/src/transports/hid.rs#L129-L142):

```rust
fn spawn_api_worker(
    mut create_api: impl FnMut() -> Result<HidApi, TransportError> + Send + 'static,
) -> std::io::Result<mpsc::Sender<ApiReply>> {
    let (sender, receiver) = mpsc::channel::<ApiReply>();
    std::thread::Builder::new().name("ton-ledger-hid-api".into()).spawn(move || {
        for reply in receiver {
            // Cancelled queued discoveries need no OS work. An in-flight call
            // must finish, but its timeout must never retire the owning thread.
            if !reply.is_closed() {
                let _ = reply.send(create_api());
            }
        }
    })?;
    Ok(sender)
}
```

Комната охраны тут почти идеальна: отмененное поручение не будит систему, а начатая операция обязана закончиться. Но цена порядка — поток, который не закрывается по таймеру. Сыщик записал в блокнот: **«Если сторож уснул, виноват не сторож, а архитектура здания»**.

## ВТОРОЙ СЛЕД: ТЕСТ НА РЕАЛЬНУЮ ЗАМЕНУ ДЛИТСЯ ОДИННАДЦАТЬ СЕКУНД

Самая сочная улика лежит в примере [`examples/ton_ledger_reconnect.rs`](https://github.com/ston-fi/ton-rs/blob/7dc14a498f952b6f6238e2045d952ce395c72f59/examples/ton_ledger_reconnect.rs). На строках [8–23](https://github.com/ston-fi/ton-rs/blob/7dc14a498f952b6f6238e2045d952ce395c72f59/examples/ton_ledger_reconnect.rs#L8-L23) программа сначала читает адрес, потом предлагает заменить USB Ledger и засыпает:

```rust
#[tokio::main]
async fn main() -> Result<(), Box<dyn Error>> {
    print_address().await?;

    println!("Replace the USB Ledger and open the TON app on the replacement.");
    // Exceed Tokio's default 10-second blocking-thread idle timeout. Before
    // the fix for #231, macOS HIDAPI could retain that retired thread's run loop.
    tokio::time::sleep(Duration::from_secs(11)).await;

    println!("Press Enter when the replacement Ledger is ready.");
    if io::stdin().read_line(&mut String::new())? == 0 {
        return Err(io::Error::from(io::ErrorKind::UnexpectedEof).into());
    }

    print_address().await?;
    Ok(())
}
```

Вот оно, чистое признание: **проверка исправления — это сон на одну секунду дольше таймера Tokio**. После сна человек вручную меняет устройство и жмет Enter. Автоматизация смотрит на часы, а затем передает дело пальцу пользователя. Если бы сыщик тестировал карету, он бы сначала запер кучера в чулане ровно на одиннадцать секунд.

Документация повторяет маршрут на строках [235–248](https://github.com/ston-fi/ton-rs/blob/7dc14a498f952b6f6238e2045d952ce395c72f59/crates/ton_ledger/README.md#L235-L248): подключить один Ledger, открыть TON app, запустить пример, заменить устройство во время паузы, затем нажать Enter. Отдельно подчеркивается, что пример только читает адрес и не подписывает перевод. Это разумная страховка, но одновременно честное признание: в центре теста стоит ручной ритуал, а не воспроизводимый стенд.

## ТРЕТИЙ СЛЕД: ПОДПИСЬ СТРОГАЯ, ЖЕЛЕЗО — ПОКА НА СЛОВАХ

В README на строках [250–261](https://github.com/ston-fi/ton-rs/blob/7dc14a498f952b6f6238e2045d952ce395c72f59/crates/ton_ledger/README.md#L250-L261) контора сама ограничивает силу своих показаний:

```text
CI compiles the examples without running them. Protocol fixtures and scripted
transport tests cover encoding, verification and session failure paths; fixtures
do not execute firmware. Package verification is separate from hardware testing.

On 2026-09-23, the maintainer reported a successful physical-device test. The
report did not specify model, OS, transport, operations or transaction inclusion;
it confirms that tested setup, not every supported transport or recovery path.
```

Это не мелкая сноска, а второй этаж сюжета. CI собирает пример, но не запускает его. Фикстуры проверяют протокол, но не прошивку. Физический тест заявлен, однако без модели, системы, транспорта, операций и проверки включения транзакции. В деле о USB-призраке выходит странная формула: программный сторож бессмертен, а его железный свидетель видел происшествие без протокола допроса.

Еще одна табличка на строках [138–149](https://github.com/ston-fi/ton-rs/blob/7dc14a498f952b6f6238e2045d952ce395c72f59/crates/ton_ledger/README.md#L138-L149) обещает, что инициализация и перечисление устройств переживают простой, сброс кошельков и закрытие runtime. Но там же сказано: проверена только локальная сборка macOS, а Linux CI лишь собирает варианты функций. Сыщик уважает честность формулировки, но газовый рожок все равно хрипит: дверь считается надежной, пока настоящий постовой не прошел весь маршрут.

## ЧЕТВЕРТЫЙ СЛЕД: ТЕСТЫ ПРОВЕРЯЮТ, ЧТО СТОРОЖ НЕ УШЕЛ

В файле [`crates/ton_ledger/src/transports/hid/_test_hid.rs`](https://github.com/ston-fi/ton-rs/blob/7dc14a498f952b6f6238e2045d952ce395c72f59/crates/ton_ledger/src/transports/hid/_test_hid.rs) строки [10–39](https://github.com/ston-fi/ton-rs/blob/7dc14a498f952b6f6238e2045d952ce395c72f59/crates/ton_ledger/src/transports/hid/_test_hid.rs#L10-L39) дважды создают runtime, отправляют запрос и после его закрытия требуют тот же ID потока:

```rust
#[test]
fn test_api_worker_survives_runtime_shutdown() -> anyhow::Result<()> {
    let (stopped, shutdown) = mpsc::channel();
    let lifetime = WorkerLifetime(stopped);
    let (thread, threads) = mpsc::channel();
    let worker = spawn_api_worker(move || {
        let _keep_alive = &lifetime;
        let _ = thread.send(std::thread::current().id());
        // Exercise a backend failure without initializing the process-global
        // native HIDAPI context on this disposable test worker.
        Err(TransportError::NoDevice)
    })?;
```

А на строках [24–36](https://github.com/ston-fi/ton-rs/blob/7dc14a498f952b6f6238e2045d952ce395c72f59/crates/ton_ledger/src/transports/hid/_test_hid.rs#L24-L36) тест прямо утверждает, что оба ответа пришли от одного потока, который не совпадает с тестовым:

```rust
        drop(runtime);
        assert!(matches!(shutdown.try_recv(), Err(mpsc::TryRecvError::Empty)));
    }
    assert_eq!(ids[0], ids[1]);
    assert_ne!(ids[0], std::thread::current().id());
```

С точки зрения теста все красиво: runtime уходит, поток остается, затем закрывается вместе с каналом. Но это проверка дисциплины привратника, а не полноценная проверка смены Ledger. Последняя вынесена в ручной пример с одиннадцатисекундной паузой. Между двумя мирами стоит человек, клавиша Enter и надежда, что macOS не придумала новый туман.

## ХРОНИКА МЕЛКОГО КРИНЖА

*Отдел предупреждений.* README начинает с фразы [«Risk of permanent loss of funds — use at your own risk»](https://github.com/ston-fi/ton-rs/blob/7dc14a498f952b6f6238e2045d952ce395c72f59/crates/ton_ledger/README.md#L7-L16). Для кошелька, который предлагает ручной сценарий с физической заменой устройства, это не декор, а второй газовый рожок.

*Отдел ручного управления.* Строки [12–18](https://github.com/ston-fi/ton-rs/blob/7dc14a498f952b6f6238e2045d952ce395c72f59/examples/ton_ledger_reconnect.rs#L12-L18) требуют сначала поменять Ledger, а затем сообщить программе, что замена завершена. Аппаратный тест поставлен на паузу, пока пользователь не даст разрешение продолжать.

*Отдел честных ограничений.* На строках [131–136](https://github.com/ston-fi/ton-rs/blob/7dc14a498f952b6f6238e2045d952ce395c72f59/crates/ton_ledger/README.md#L131-L136) Linux получает CI-сборку, но не обещание реальной проверки, а браузер и WASM вовсе исключены. Туман распределен по платформам неравномерно.

## ВЕРДИКТ СЫЩИКА

Репозиторий [ston-fi/ton-rs](https://github.com/ston-fi/ton-rs) пойман не на беспечном `TODO`, а на более изысканной сцене. В коммите [`7dc14a4`](https://github.com/ston-fi/ton-rs/commit/7dc14a498f952b6f6238e2045d952ce395c72f59) файл [`crates/ton_ledger/src/transports/hid.rs`](https://github.com/ston-fi/ton-rs/blob/7dc14a498f952b6f6238e2045d952ce395c72f59/crates/ton_ledger/src/transports/hid.rs) заводит бессрочный поток, чтобы macOS не потеряла свой HID-цикл после простоя Tokio. Файл [`examples/ton_ledger_reconnect.rs`](https://github.com/ston-fi/ton-rs/blob/7dc14a498f952b6f6238e2045d952ce395c72f59/examples/ton_ledger_reconnect.rs) испытывает спасение одиннадцатисекундным сном, ручной заменой железа и нажатием Enter. А [`crates/ton_ledger/README.md`](https://github.com/ston-fi/ton-rs/blob/7dc14a498f952b6f6238e2045d952ce395c72f59/crates/ton_ledger/README.md) без прикрас сообщает: CI пример не запускает, а полная физическая приемка не доказана.

Сыщик не обвиняет контору в обмане. Напротив, документы довольно честно показывают границы проверки. Но жанр прекрасен: чтобы доказать, что Ledger переживает замену, программе дают уснуть ровно дольше таймера, человеку поручают переставить устройство, а затем весь процесс будят одной клавишей. На папке появляется печать: **«Поток бессмертен. Железо допрошено частично»**.

Газовый рожок дернулся. Туман сомкнулся над мостовой.

🐀
