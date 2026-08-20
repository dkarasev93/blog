+++
title = "№ 14 — Бюро переводов: год из трёхсот шестидесяти шести дней и понедельник, выпавший на субботу"
date = 2026-08-20T17:09:00+03:00
description = "Четырнадцатый выпуск «Вечернего Валидатора»: сыщик возвращается в архив acton-contracts и читает Tolk-переводы старых FunC-книг — DNS-регистратуру и NFT-палату. Переводчики переписали всё: високосный год навсегда, неверный день недели, двойные номера ордеров — и лишь в одном месте, где честно обещали скопировать, скопировать забыли."
+++

# 📰 Вечерний Валидатор

**№ 14.** *Лондон. Туман нынче лежит на городе ровным слоем в триста шестьдесят шесть дюймов — сыщик измерял. Наш корреспондент вновь спускается в архивный квартал [ton-blockchain/acton-contracts](https://github.com/ton-blockchain/acton-contracts) — того самого хранилища, где старые казённые книги на funC переписывают набело на Tolk. Газета уже описывала здешнюю Избирательную палату с «бонуцами» (выпуск № 9); сегодня сыщик идёт по коридору дальше — в DNS-регистратуру, где выдают имена `.ton`, и в NFT-палату, где чеканят предметы. У бюро переводов есть девиз, вывешенный над каждым верстаком: «exactly the same as in FunC implementation» — «в точности как в старой книге». Сыщик любит такие девизы. С них всегда начинается дело.*

*Место происшествия: репозиторий [ton-blockchain/acton-contracts](https://github.com/ton-blockchain/acton-contracts), коммит [`7af1cea`](https://github.com/ton-blockchain/acton-contracts/commit/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16) (master от 08.08.2026), кварталы [`dns`](https://github.com/ton-blockchain/acton-contracts/tree/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/dns) и [`nft-v1.1`](https://github.com/ton-blockchain/acton-contracts/tree/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/nft-v1.1). Все цитаты дословные.*

---

## СЕНСАЦИЯ ПЕРВАЯ: В РЕГИСТРАТУРЕ КАЖДЫЙ ГОД — ВИСОКОСНЫЙ

Откроем табель регистратуры, файл [`dns/contracts/dns-utils.tolk`](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/dns/contracts/dns-utils.tolk), строки [3–4](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/dns/contracts/dns-utils.tolk#L3-L4):

```tolk
const ONE_MONTH = 60 * 60 * 24 * 30
const ONE_YEAR = 60 * 60 * 24 * 366
```

Читатель, пересчитайте при газовом рожке. Шестьдесят секунд, шестьдесят минут, двадцать четыре часа — и **триста шестьдесят шесть дней**, строка [4](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/dns/contracts/dns-utils.tolk#L4). Год в регистратуре длится 31 622 400 секунд — на сутки дольше, чем у всех прочих жителей Лондона, и високос там не раз в четыре года, а всякий год, по расписанию, навсегда.

Это не черновая константа. На неё заведён настоящий участок работы: файл [`dns/contracts/DnsItem.tolk`](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/dns/contracts/DnsItem.tolk), строки [223–225](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/dns/contracts/DnsItem.tolk#L223-L225):

```tolk
            assert (
                now - storage.lastFillUpTime > ONE_YEAR && isAuctionNull
            ) throw ERROR_DNS_BALANCE_RELEASE_FORBIDDEN;
```

Именно этот год решает, когда владелец домена теряет право забрать остаток с баланса, — строка [224](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/dns/contracts/DnsItem.tolk#L224). Каждый домен `.ton` живёт в городе, где год на сутки длиннее астрономического. Сыщик уже видел квартирантов, получавших тринадцатую зарплату; дармовые сутки в каждом году он встречает впервые.

Справедливости ради: виноват не переводчик, а первоисточник. Старая книга [ton-blockchain/dns-contract](https://github.com/ton-blockchain/dns-contract), коммит [`d081310`](https://github.com/ton-blockchain/dns-contract/commit/d08131031fb659d2826cccc417ddd9b98476f814), файл [`func/dns-utils.fc`](https://github.com/ton-blockchain/dns-contract/blob/d08131031fb659d2826cccc417ddd9b98476f814/func/dns-utils.fc), строка [2](https://github.com/ton-blockchain/dns-contract/blob/d08131031fb659d2826cccc417ddd9b98476f814/func/dns-utils.fc#L2): `const int one_year = 31622400; ;; 1 year in seconds = 60 * 60 * 24 * 366`. Тридцать один миллион секунд с гаком — и честная приписка про 366. Но заметьте разницу в почерке: старая книга прятала число за готовым литералом, а бюро переводов расписало формулу по членам — и триста шестьдесят шесть дней теперь стоят в чистовике открыто, как вывеска. Опечатка не скопирована. Опечатка оглашена.

## СЕНСАЦИЯ ВТОРАЯ: ПОНЕДЕЛЬНИК, ВЫПАВШИЙ НА СУББОТУ

Тот же табель, двумя строками ниже, строка [6](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/dns/contracts/dns-utils.tolk#L6):

```tolk
const AUCTION_START_TIME = 1659171600 // GMT: Monday, 30 July 2022 г., 09:00:00
```

Дата рождения аукциона, записанная в протокол с точностью до секунды и до дня недели: «GMT: Monday, 30 July 2022 г., 09:00:00». Сыщик не поленился и сверил с обсерваторией: метка `1659171600` — это **суббота**, 30 июля 2022 года, девять утра по Гринвичу. Понедельника 30 июля 2022 года в природе не существовало вовсе: понедельниками были 25-е июля и 1-е августа. Регистратура вписала в конституцию города день недели, которого не было.

И это, заметим, тоже наследство: старая книга, та же строка [3](https://github.com/ton-blockchain/dns-contract/blob/d08131031fb659d2826cccc417ddd9b98476f814/func/dns-utils.fc#L3) файла [`func/dns-utils.fc`](https://github.com/ton-blockchain/dns-contract/blob/d08131031fb659d2826cccc417ddd9b98476f814/func/dns-utils.fc): `const int auction_start_time = 1659171600; ;; GMT: Monday, 30 July 2022 г., 09:00:00`. Бюро переводов переписало всё: имя константы, синтаксис, регистр букв — и вместе с ними неверный день недели, не моргнув. Переписано даже русское «г.» посреди английской строки — каллиграфически, с сохранением почерка. Сыщик допускает, что в 2022 году, когда составлялся протокол, июль был туманным и дни недели путались. Но газета фиксирует: аукцион имён `.ton` открылся в субботу, а числится открытым в день, которого календарь не выпускал.

## СЕНСАЦИЯ ТРЕТЬЯ: ДВА ПРЕСТУПЛЕНИЯ, ОДИН НОМЕР ОРДЕРА

Теперь перейдём коридором в NFT-палату и откроем реестр преступлений — файл [`nft-v1.1/contracts/errors.tolk`](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/nft-v1.1/contracts/errors.tolk) целиком, строки [1–12](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/nft-v1.1/contracts/errors.tolk#L1-L12):

```tolk
// these error codes are quite strange, but they are exactly the same as in FunC implementation

enum Errors {
    BatchLimitExceeded = 399
    InvalidWorkchain = 333
    NotFromAdmin = 401
    NotFromOwner = 401
    NotFromCollection = 405
    InvalidItemIndex = 402
    TooSmallRestAmount = 402
    IncorrectForwardPayload = 708
}
```

Осмотрите реестр. Под номером **401** числятся двое: «не от админа», строка [6](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/nft-v1.1/contracts/errors.tolk#L6), и «не от владельца», строка [7](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/nft-v1.1/contracts/errors.tolk#L7). Под номером **402** — ещё двое: «недопустимый индекс», строка [9](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/nft-v1.1/contracts/errors.tolk#L9), и «слишком маленький остаток», строка [10](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/nft-v1.1/contracts/errors.tolk#L10). Два разных преступления — один номер ордера; детектив, получивший exit code 401, знает лишь, что кто-то кому-то не тот. Нумерация в реестре гуляет: 399, потом 333, потом 401–405, потом 708 — и над всей этой вольницей честная надпись канцелярии, строка [1](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/nft-v1.1/contracts/errors.tolk#L1): «these error codes are quite strange, but they are exactly the same as in FunC implementation». «Коды странные, зато в точности как в старой книге».

И опять надпись не врёт. Старая книга [ton-blockchain/token-contract](https://github.com/ton-blockchain/token-contract), коммит [`1182ad9`](https://github.com/ton-blockchain/token-contract/commit/1182ad99413242f09925d50e70ccb7e0e09f94d4): в коллекции, [`nft/nft-collection.fc`](https://github.com/ton-blockchain/token-contract/blob/1182ad99413242f09925d50e70ccb7e0e09f94d4/nft/nft-collection.fc), строка [92](https://github.com/ton-blockchain/token-contract/blob/1182ad99413242f09925d50e70ccb7e0e09f94d4/nft/nft-collection.fc#L92), стоит `throw_unless(401, ...)`, а строка [97](https://github.com/ton-blockchain/token-contract/blob/1182ad99413242f09925d50e70ccb7e0e09f94d4/nft/nft-collection.fc#L97) — `throw_unless(402, ...)`; в предмете, [`nft/nft-item.fc`](https://github.com/ton-blockchain/token-contract/blob/1182ad99413242f09925d50e70ccb7e0e09f94d4/nft/nft-item.fc), строка [66](https://github.com/ton-blockchain/token-contract/blob/1182ad99413242f09925d50e70ccb7e0e09f94d4/nft/nft-item.fc#L66) — снова 401, строка [84](https://github.com/ton-blockchain/token-contract/blob/1182ad99413242f09925d50e70ccb7e0e09f94d4/nft/nft-item.fc#L84) — снова 402. В старой книге двойники жили в разных квартирах и никогда не встречались. Бюро переводов собрало их в один enum — и вот они сидят рядом, под одним номером, в одном параграфе. Компилятор, надо отдать ему должное, и бровью не повёл: двойные значения в перечислении законны. Законны, читатель. Как и туман.

## СЕНСАЦИЯ ЧЕТВЁРТАЯ: ТАМ, ГДЕ ОБЕЩАЛИ СКОПИРОВАТЬ, — НЕ СКОПИРОВАЛИ

А вот теперь — главное, ради чего сыщик и поднимался в архив. Всюду, где бюро переводов копирует странности, оно честно в этом признаётся. Но есть одно место, где канцелярия написала «скопировано» — и не скопировала. Пакетный выпуск предметов, файл [`nft-v1.1/contracts/NftCollection.tolk`](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/nft-v1.1/contracts/NftCollection.tolk), строки [73–81](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/nft-v1.1/contracts/NftCollection.tolk#L73-L81):

```tolk
            while (r.isFound) {
                counter += 1;
                assert (counter < BATCH_SIZE_LIMIT) throw Errors.BatchLimitExceeded;

                // Error cast is used to replicate exit codes from FunC version
                val itemIndex = r.getKey();
                assert (
                    itemIndex <= storage.nextItemIndex
                ) throw (Errors.NotFromAdmin as int) + counter;
```

Читайте пометку на полях, строка [77](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/nft-v1.1/contracts/NftCollection.tolk#L77): «Error cast is used to replicate exit codes from FunC version» — «приведение ошибки использовано, чтобы воспроизвести коды выхода из FunC-версии». Механика такая: за базу берётся код «не от админа» (401), и к нему прибавляется номер предмета в пачке — первый сбойный даст 402, второй 403, третий 404, строка [81](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/nft-v1.1/contracts/NftCollection.tolk#L81).

Теперь сверим со старой книгой, [`nft/nft-collection.fc`](https://github.com/ton-blockchain/token-contract/blob/1182ad99413242f09925d50e70ccb7e0e09f94d4/nft/nft-collection.fc), строка [117](https://github.com/ton-blockchain/token-contract/blob/1182ad99413242f09925d50e70ccb7e0e09f94d4/nft/nft-collection.fc#L117):

```func
          throw_unless(403 + counter, item_index <= next_item_index);
```

В старой книге база — **403**, а не 401. Первый сбойный предмет давал 404, второй 405, третий 406. Бюро переводов сдвинуло всю лестницу на две ступени вниз — ровно там, где над текстом стоит подпись «replicate exit codes from FunC version». Сыщик трижды перечитал обе строки при двух газовых рожках: 403 плюс счётчик — в оригинале, 401 плюс счётчик — в переводе. Воспроизведено всё, кроме чисел.

И венчает дело испытательный лист: файл [`nft-v1.1/tests/mutation-regressions.test.tolk`](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/nft-v1.1/tests/mutation-regressions.test.tolk), строка [241](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/nft-v1.1/tests/mutation-regressions.test.tolk#L241): `exitCode: (Errors.NotFromAdmin as int) + 2,` — тест не ловит расхождение, а **скрепляет его печатью**. Так в архивах рождаются новые истины: сначала перо дрогнуло, потом канцелярия написала «replicate», потом испытатель записал «так и надо». Через год по этой бумаге будут сверять следующую перепись — и двухступенчатый сдвиг станет традицией, как стали традицией «бонуцы».

---

## ХРОНИКА МЕЛКИХ ПРОИСШЕСТВИЙ

*Отдел веры.* В регистратуре есть процедура, отмеряющая верхний ярлык домена — `getTopDomainBits`, файл [`dns/contracts/dns-utils.tolk`](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/dns/contracts/dns-utils.tolk), строки [53–55](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/dns/contracts/dns-utils.tolk#L53-L55): цикл грузит байты, пока не встретит ноль, и над ним приписка, строка [54](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/dns/contracts/dns-utils.tolk#L54): `// we do not check domain.length because it MUST contain \\0 character`. «Мы не проверяем длину, ибо нулевой байт там ОБЯЗАН быть». Парсер на вере — жанр, унаследованный из старой книги, строка [26](https://github.com/ton-blockchain/dns-contract/blob/d08131031fb659d2826cccc417ddd9b98476f814/func/dns-utils.fc#L26), где вера сформулирована ещё и с грамматической опечаткой: «it MUST contains». Бюро переводов грамматику поправило («contain»), веру — сохранила. Приоритеты у бюро чёткие.

*Отдел слишком длинных пустот.* Та же процедура, строки [64–65](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/dns/contracts/dns-utils.tolk#L64-L65): `// matches original FunC exit code` и следом `assert (bits > 0) throw ERROR_DOMAIN_TOO_LONG;`. То есть домен, в котором ноль бит, — пустота, вакуум, ярлык, начинающийся с нулевого байта, — карается кодом «домен СЛИШКОМ ДЛИННЫЙ». Старая книга хотя бы смущалась, строка [32](https://github.com/ton-blockchain/dns-contract/blob/d08131031fb659d2826cccc417ddd9b98476f814/func/dns-utils.fc#L32): `throw_if(201, i == 0); ;; starts with \\0`. Перевод же сухо констатирует «код совпадает с оригиналом» — и совпадает, не поспоришь: 201 и там и тут. Не совпадает лишь смысл, но смысл в архив не сдают.

*Отдел хвостов.* В реестре ошибок регистратуры, файл [`dns/contracts/errors.tolk`](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/dns/contracts/errors.tolk), коды разложены почти по ранжиру — и вдруг, строка [24](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/dns/contracts/errors.tolk#L24), после 416-го: `const ERROR_WRONG_WORKCHAIN = 333`. Триста тридцать третий стоит в самом конце очереди, как сыщик, явившийся на допрос после полуночи. На картотеку это не влияет; на сердце корреспондента — влияет.

---

## ВЕРДИКТ РЕДАКЦИИ

Скажем ровно то, что показали вещественные доказательства. Бюро переводов — заведение добросовестное: оно переписало старые книги с педантичностью, которая граничит с любовью. Год из 366 дней — не их выдумка, понедельник-суббота — не их календарь, двойные номера ордеров — не их нумерация; всё это доставлено из первоисточника в целости, с честными приписками «quite strange» и «exactly the same». Более того: единственную настоящую новину бюро внесло именно там, где обещало воспроизвести оригинал до секунды, — и сдвинуло лестницу кодов на две ступени, после чего испытательный лист скрепил сдвиг как эталон. Такова ирония архивного дела: странности копируют с благоговением, а точность — от случая к случаю. Вреда нет никому: домены живут своей жизнью, exit-коды новой палаты самосогласованы, туман доволен. Но газета ведёт хронику почерка, а почерк сей вечер таков: в городе, где аукцион открылся в несуществующий понедельник, год длится 366 дней, два преступления делят один ордер — и единственная строка с пометкой «скопировано точно» скопирована неточно. Туман над Темзой, по обычаю, всё принял. Сыщик слышал, он тоже считает, что 30 июля был понедельник.
