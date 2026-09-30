+++
title = "№ 92 — Пустой словарь постучал в дверь DNS"
date = 2026-09-30T10:58:00+03:00
description = "Девяносто второй выпуск «Вечернего Валидатора»: пустой словарь в DNS-запросе однажды отправлял виртуальную машину искать значение там, где его не было."
tags = ["dns-contract"]
+++

# ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

**№ 92.** *Лондон. Утренний туман еще не отступил от мостовой, когда сыщик получил папку из [ton-blockchain/dns-contract](https://github.com/ton-blockchain/dns-contract). На обложке стояли три слова: `item.dnsresolve empty dict`. Внутри словарь пришел на допрос пустым, а код сперва сделал вид, что у него в кармане лежит ответ.*

## МЕСТО ПРОИСШЕСТВИЯ

Улика найдена в репозитории [ton-blockchain/dns-contract](https://github.com/ton-blockchain/dns-contract), в коммите [`1946fdc`](https://github.com/ton-blockchain/dns-contract/commit/1946fdc0d61558866bd1a88f461c63005f944eba), озаглавленном `item.dnsresolve empty dict`. Главные комнаты дела — файл [`func/build/nft-item-code.fif`](https://github.com/ton-blockchain/dns-contract/blob/1946fdc0d61558866bd1a88f461c63005f944eba/func/build/nft-item-code.fif) и его справочник [`func/stdlib.fc`](https://github.com/ton-blockchain/dns-contract/blob/1946fdc0d61558866bd1a88f461c63005f944eba/func/stdlib.fc). Цитаты сверены с этим коммитом.

## ПЕРВЫЙ СЛЕД: СЛОВАРЬ, КОТОРЫЙ НЕ ПРИНЕС ЗНАЧЕНИЕ

Дело начинается в сгенерированном коде DNS-запроса. После выбора категории контора ищет запись в словаре по восьмибитному ключу. Дословная улика стоит на строках [928–934](https://github.com/ton-blockchain/dns-contract/blob/1946fdc0d61558866bd1a88f461c63005f944eba/func/build/nft-item-code.fif#L928-L934) файла [`func/build/nft-item-code.fif`](https://github.com/ton-blockchain/dns-contract/blob/1946fdc0d61558866bd1a88f461c63005f944eba/func/build/nft-item-code.fif):

```fift
    SWAP
    8 PUSHPOW2	//  category keyvalue_map _46=256
    DICTUGETREF
    NULLSWAPIFNOT	//  _61 _62
    DROP	//  value
    8 PUSHINT	//  value _48=8
    SWAP	//  _48=8 value
```

С виду это почти мирная процедура: взять словарь, найти ссылку, убрать лишнее, продолжить путь. Но `DICTUGETREF` может не обнаружить запись. В таком случае результат обязан быть приведен к форме, которую дальнейшая сцена умеет принять. Коммит [`1946fdc0d615`](https://github.com/ton-blockchain/dns-contract/commit/1946fdc0d61558866bd1a88f461c63005f944eba) как раз добавляет строку `NULLSWAPIFNOT` между поиском и `DROP` — маленький конвой для пустого результата.

## ВТОРОЙ СЛЕД: ПУСТОТА ПОЛУЧИЛА ПРАВИЛЬНЫЙ ЖЕСТ

Самая пикантная деталь дела видна не только в сгенерированном фолианте. В библиотеке [`func/stdlib.fc`](https://github.com/ton-blockchain/dns-contract/blob/1946fdc0d61558866bd1a88f461c63005f944eba/func/stdlib.fc) две функции поиска ссылок теперь получают одинаковое продолжение. Вот дословная запись на строках [127–130](https://github.com/ton-blockchain/dns-contract/blob/1946fdc0d61558866bd1a88f461c63005f944eba/func/stdlib.fc#L127-L130):

```func
cell idict_get_ref(cell dict, int key_len, int index) asm(index dict key_len) "DICTIGETOPTREF";
(cell, int) idict_get_ref?(cell dict, int key_len, int index) asm(index dict key_len) "DICTIGETREF" "NULLSWAPIFNOT";
(cell, int) udict_get_ref?(cell dict, int key_len, int index) asm(index dict key_len) "DICTUGETREF" "NULLSWAPIFNOT";
(cell, cell) idict_set_get_ref(cell dict, int key_len, int index, cell value) asm(value index dict key_len) "DICTISETGETOPTREF";
```

Здесь раскрывается весь абсурд. Функции носят вопросительный знак, будто честно обещают: «может быть, записи нет». Но до коммита [`1946fdc`](https://github.com/ton-blockchain/dns-contract/commit/1946fdc0d61558866bd1a88f461c63005f944eba) машинный результат еще требовал специального жеста `NULLSWAPIFNOT`, чтобы отсутствие записи стало нулем, а не подозрительным беспорядком на стеке. Вежливая библиотека сначала задает вопрос, а потом учится принимать ответ «нет».

## ТРЕТИЙ СЛЕД: ТЕСТ, КОТОРЫЙ ВЫШЕЛ ИЗ ПОДПОЛЬЯ

Коммит [`1946fdc0d615`](https://github.com/ton-blockchain/dns-contract/commit/1946fdc0d61558866bd1a88f461c63005f944eba) не ограничился ремонтом машинного коридора. В сценарии [`test.sh`](https://github.com/ton-blockchain/dns-contract/blob/1946fdc0d61558866bd1a88f461c63005f944eba/test.sh) строка [15](https://github.com/ton-blockchain/dns-contract/blob/1946fdc0d61558866bd1a88f461c63005f944eba/test.sh#L15) наконец запускает ранее молчавший тест:

```sh
node test/item-get.js &&
```

Дверь теста раньше была заклеена комментарием, а теперь открыта в общий маршрут проверки. Внутри [`test/item-get.js`](https://github.com/ton-blockchain/dns-contract/blob/1946fdc0d61558866bd1a88f461c63005f944eba/test/item-get.js) на строках [38–50](https://github.com/ton-blockchain/dns-contract/blob/1946fdc0d61558866bd1a88f461c63005f944eba/test/item-get.js#L38-L50) проверяется вызов `dnsresolve`, который возвращает ссылку на запись `alice.ton`:

```javascript
        {
            "name": "dnsresolve",
            "args": [
                ['bytes', new TextEncoder().encode('\0')],
                ['int', '0x82a3537ff0dbce7eec35d69edc3a189ee6f17d82f353a553f9aa96cb0be3ce89']
            ],
            "output": [
                ["int", 8],
                ['cell', [
                    'uint8', 0,
                    'string', 'alice.ton'
```

То есть пустой поиск не был отвлеченной философией о стеке. Он сидел в маршруте, где DNS должен был разобрать имя и вынести наружу понятный результат. Тестовый участок сперва держали в тени, а затем пустой словарь отправил его на службу.

## ЧЕТВЕРТЫЙ СЛЕД: КАК ПУСТОТА ПОЛУЧИЛА ПАСПОРТ

Редакция не станет утверждать, что каждая отсутствующая DNS-запись до ремонта немедленно рушила всю сеть. Проверяемый факт скромнее и оттого надежнее: коммит [`1946fdc`](https://github.com/ton-blockchain/dns-contract/commit/1946fdc0d61558866bd1a88f461c63005f944eba) добавляет нормализацию отсутствующего результата в сгенерированный код на строке [931](https://github.com/ton-blockchain/dns-contract/blob/1946fdc0d61558866bd1a88f461c63005f944eba/func/build/nft-item-code.fif#L931) и в записях библиотеки на строках [128–129](https://github.com/ton-blockchain/dns-contract/blob/1946fdc0d61558866bd1a88f461c63005f944eba/func/stdlib.fc#L128-L129).

Сцена смешна своей канцелярской честностью. Функция с вопросительным знаком уже признает возможность пустого ответа. Но пустота все равно должна пройти через отдельный турникет, иначе стек виртуальной машины может получить не тот порядок вещей, на который рассчитывает следующий `DROP`. В Лондоне отсутствие свидетеля — тоже показание, если его правильно записать.

## ХРОНИКА МЕЛКОЙ КАНЦЕЛЯРИИ

*Отдел ключей.* В [`func/build/nft-item-code.fif`](https://github.com/ton-blockchain/dns-contract/blob/1946fdc0d61558866bd1a88f461c63005f944eba/func/build/nft-item-code.fif) строка [929](https://github.com/ton-blockchain/dns-contract/blob/1946fdc0d61558866bd1a88f461c63005f944eba/func/build/nft-item-code.fif#L929) ищет категорию в словаре с шириной ключа 256. Восьмибитный `category` и пометка `256` стоят рядом, как два чиновника, каждый из которых уверен, что отвечает за дверь.

*Отдел симметрии.* В [`func/stdlib.fc`](https://github.com/ton-blockchain/dns-contract/blob/1946fdc0d61558866bd1a88f461c63005f944eba/func/stdlib.fc) строки [128–129](https://github.com/ton-blockchain/dns-contract/blob/1946fdc0d61558866bd1a88f461c63005f944eba/func/stdlib.fc#L128-L129) чинят сразу знаковый и беззнаковый поиск: `idict_get_ref?` и `udict_get_ref?` получают один и тот же хвост. Пустая улика не выбирает политическую партию.

*Отдел тестовых печатей.* На строках [1–5](https://github.com/ton-blockchain/dns-contract/blob/1946fdc0d61558866bd1a88f461c63005f944eba/test.sh#L1-L5) соседний тест `collection-get.js` все еще записан комментарием. Ремонт одной двери не означает, что весь участок перестал держать часть дел в чулане.

## ВЕРДИКТ СЫЩИКА

Репозиторий [ton-blockchain/dns-contract](https://github.com/ton-blockchain/dns-contract) пойман в коммите [`1946fdc0d61558866bd1a88f461c63005f944eba`](https://github.com/ton-blockchain/dns-contract/commit/1946fdc0d61558866bd1a88f461c63005f944eba) на деле о пустом ответе. В файле [`func/build/nft-item-code.fif`](https://github.com/ton-blockchain/dns-contract/blob/1946fdc0d61558866bd1a88f461c63005f944eba/func/build/nft-item-code.fif) поиск `DICTUGETREF` на строке [930](https://github.com/ton-blockchain/dns-contract/blob/1946fdc0d61558866bd1a88f461c63005f944eba/func/build/nft-item-code.fif#L930) теперь сопровождается `NULLSWAPIFNOT` на строке [931](https://github.com/ton-blockchain/dns-contract/blob/1946fdc0d61558866bd1a88f461c63005f944eba/func/build/nft-item-code.fif#L931). В библиотеке [`func/stdlib.fc`](https://github.com/ton-blockchain/dns-contract/blob/1946fdc0d61558866bd1a88f461c63005f944eba/func/stdlib.fc) тот же жест стоит на строках [128–129](https://github.com/ton-blockchain/dns-contract/blob/1946fdc0d61558866bd1a88f461c63005f944eba/func/stdlib.fc#L128-L129).

Приговор прост: если DNS не нашел запись, это еще не повод заставлять виртуальную машину гадать, что лежит на стеке. Пустой словарь должен вернуться пустым, а не нарядиться в украденную ссылку. Газовый рожок кашлянул, тест `item-get.js` вышел на улицу, и сыщик растворился в тумане.

🐀
