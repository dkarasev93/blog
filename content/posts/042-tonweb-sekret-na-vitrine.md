+++
title = "№ 42 — Секрет на витрине: tonweb учит кошелек ходить с чужим ключом"
date = 2026-09-03T16:59:00+03:00
description = "Сорок второй выпуск «Вечернего Валидатора»: в примере tonweb кошелек сначала получает случайную пару, а затем без церемоний получает жестко заданный секретный ключ и готовится к отправке TON."
tags = ["tonweb"]
+++

# 📰 ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

**№ 42 · Четверг, 3 сентября 2026 г. · Цена: 0.05 TON (ключ в витрину не входит)**

---

## ДЕЛО О КЛЮЧЕ, КОТОРЫЙ ПРИШЕЛ ИЗ НИОТКУДА

Лондон в тот час был сер, как непросмотренный pull request. Туман полз по мостовой, газовый рожок у редакции сипел, а сыщик «Вечернего Валидатора» получил записку из мастерской [toncenter/tonweb](https://github.com/toncenter/tonweb). Контора обещает JavaScript-инструменты для кошельков TON. Внутри, однако, нашлась сцена с реквизитом, который слишком легко принять за настоящий ключ от чужого сейфа.

Место происшествия: репозиторий [toncenter/tonweb](https://github.com/toncenter/tonweb), коммит [`76dfd07`](https://github.com/toncenter/tonweb/commit/76dfd0701714c0a316aee503c2962840acaf74ef), файл [`test/wallet-example.html`](https://github.com/toncenter/tonweb/blob/76dfd0701714c0a316aee503c2962840acaf74ef/test/wallet-example.html), строки [14–22](https://github.com/toncenter/tonweb/blob/76dfd0701714c0a316aee503c2962840acaf74ef/test/wallet-example.html#L14-L22). Цитата сверена с содержимым файла на этом коммите.

Сыщик раскрыл пример. Сперва все выглядело безупречно: строка [14](https://github.com/toncenter/tonweb/blob/76dfd0701714c0a316aee503c2962840acaf74ef/test/wallet-example.html#L14) создает новую случайную пару, строка [15](https://github.com/toncenter/tonweb/blob/76dfd0701714c0a316aee503c2962840acaf74ef/test/wallet-example.html#L15) берет ее секретную часть. Кошелек рождается из свежего материала. Но затем в комнату входит второй свидетель: заранее вписанная длинная hex-строка.

Протокол, строки [14–22](https://github.com/toncenter/tonweb/blob/76dfd0701714c0a316aee503c2962840acaf74ef/test/wallet-example.html#L14-L22):

```html
        const keyPair = nacl.sign.keyPair(); // create new random key pair
        let secretKey = keyPair.secretKey;

        let wallet = tonweb.wallet.create({publicKey: keyPair.publicKey}); // create interface to wallet smart contract (wallet v3 by default)

        // OR

        wallet = tonweb.wallet.create({address: 'EQDjVXa_oltdBP64Nc__p397xLCvGm2IcZ1ba7anSW0NAkeP'}); // if your know only address at this moment
        secretKey = TonWeb.utils.hexToBytes('cdd624b8c960fc419d207689dd4c3bcadca7a0df53b664f97ac06454efe90c4b1dc1391e4affae5fa96b194b97de179926d791107846d80dacf700a9db1e8f7c');
```

Строка [22](https://github.com/toncenter/tonweb/blob/76dfd0701714c0a316aee503c2962840acaf74ef/test/wallet-example.html#L22) не просит пароль, не читает защищенный файл и не говорит «только для теста». Она превращает жестко заданные байты в `secretKey`. Так случайный ключ из строки [15](https://github.com/toncenter/tonweb/blob/76dfd0701714c0a316aee503c2962840acaf74ef/test/wallet-example.html#L15) отправляется на пенсию, а его место занимает постоялец, которого кто-то оставил прямо в исходнике.

## КОШЕЛЕК, КОТОРОМУ СНАЧАЛА ДАЛИ ЛИЦО, А ПОТОМ — КЛЮЧ

Самая театральная деталь живет в адресе на строке [21](https://github.com/toncenter/tonweb/blob/76dfd0701714c0a316aee503c2962840acaf74ef/test/wallet-example.html#L21). Пример говорит: если известен только адрес, создайте интерфейс кошелька. Это разумная часть урока: публичный адрес можно показать на доске. Но следующая строка подкладывает к нему секретный материал, причем как готовую константу.

В досье [toncenter/tonweb](https://github.com/toncenter/tonweb) адрес и ключ не просто лежат рядом для красоты. На строке [32](https://github.com/toncenter/tonweb/blob/76dfd0701714c0a316aee503c2962840acaf74ef/test/wallet-example.html#L32) этот `secretKey` передается в `wallet.deploy`. Цитата из того же файла и коммита [`76dfd07`](https://github.com/toncenter/tonweb/commit/76dfd0701714c0a316aee503c2962840acaf74ef):

```js
        const deploy = wallet.deploy(secretKey); // deploy method
```

То есть это не мертвый пример вывода байтов. Рядом стоит действие, которое использует ключ для операции кошелька. В учебной витрине посетителю дают адрес, затем незаметно выдают ключ и предлагают попробовать открыть дверь.

Редакция не утверждает, что этот hex-материал дает доступ к средствам прямо сейчас. Репозиторий не сообщает, чей это ключ и пополнялся ли соответствующий кошелек. Но для примера, который могут копировать без чтения каждой строки, это опасная постановка. Секретные данные в публичном HTML легко уходят в форки, сниппеты, сборки и чужие учебные проекты.

## ВТОРАЯ ДВЕРЬ ВЕДЕТ К ПЕРЕВОДУ

Сыщик поднял папку выше. На строках [45–53](https://github.com/toncenter/tonweb/blob/76dfd0701714c0a316aee503c2962840acaf74ef/test/wallet-example.html#L45-L53) пример готовит перевод. Вызов снова принимает `keyPair.secretKey`, полученный из случайной пары в строке [14](https://github.com/toncenter/tonweb/blob/76dfd0701714c0a316aee503c2962840acaf74ef/test/wallet-example.html#L14):

```js
        const transfer = wallet.methods.transfer(
            {
                secretKey: keyPair.secretKey,
                toAddress: 'EQDjVXa_oltdBP64Nc__p397xLCvGm2IcZ1ba7anSW0NAkeP',
                amount: TonWeb.utils.toNano(0.01), // 0.01 TON
                seqno: seqno,
                payload: 'Hello',
                sendMode: 3,
            }
        );
```

Получилась почти комедия ошибок. Для развертывания выбран вручную вписанный секрет из строки [22](https://github.com/toncenter/tonweb/blob/76dfd0701714c0a316aee503c2962840acaf74ef/test/wallet-example.html#L22), а для перевода — случайный ключ из строки [47](https://github.com/toncenter/tonweb/blob/76dfd0701714c0a316aee503c2962840acaf74ef/test/wallet-example.html#L47). Один кошелек получил адрес из одной истории, ключ из другой, а перевод явился с третьим удостоверением. Газовый рожок мог бы крикнуть: «Господа, у нас тут две пары документов на одну дверь!»

Здесь есть и техническая, и методическая дичь. Читатель видит знакомый адрес на строках [21](https://github.com/toncenter/tonweb/blob/76dfd0701714c0a316aee503c2962840acaf74ef/test/wallet-example.html#L21) и [48](https://github.com/toncenter/tonweb/blob/76dfd0701714c0a316aee503c2962840acaf74ef/test/wallet-example.html#L48), видит секрет на строке [22](https://github.com/toncenter/tonweb/blob/76dfd0701714c0a316aee503c2962840acaf74ef/test/wallet-example.html#L22), а затем может решить, что все это единый рабочий сценарий. На деле код смешивает случайную пару и заранее заданный ключ, как два показания свидетелей, записанные в один протокол.

## ВЕРДИКТ СЫЩИКА

[toncenter/tonweb](https://github.com/toncenter/tonweb) — не карманный вор, а полезная библиотека. Именно поэтому коммит [`76dfd07`](https://github.com/toncenter/tonweb/commit/76dfd0701714c0a316aee503c2962840acaf74ef) выглядит так сочно: файл [`test/wallet-example.html`](https://github.com/toncenter/tonweb/blob/76dfd0701714c0a316aee503c2962840acaf74ef/test/wallet-example.html) показывает секретный ключ на строке [22](https://github.com/toncenter/tonweb/blob/76dfd0701714c0a316aee503c2962840acaf74ef/test/wallet-example.html#L22), использует его для deploy на строке [32](https://github.com/toncenter/tonweb/blob/76dfd0701714c0a316aee503c2962840acaf74ef/test/wallet-example.html#L32), а перевод на строках [45–53](https://github.com/toncenter/tonweb/blob/76dfd0701714c0a316aee503c2962840acaf74ef/test/wallet-example.html#L45-L53) внезапно берет иную пару.

Приговор редакции: публичный пример кошелька не должен содержать готовый секретный материал. Для демонстрации лучше сгенерировать ключ на месте и явно разделить сценарии адреса, deploy и transfer. А если адрес чужой или тестовый, это следует написать крупными буквами, не прятать под комментарий `// OR` на строке [19](https://github.com/toncenter/tonweb/blob/76dfd0701714c0a316aee503c2962840acaf74ef/test/wallet-example.html#L19).

*Сыщик закрыл HTML, накрыл hex-строку черной бумагой и погас газовую лампу. В Лондоне еще можно оставить адрес на витрине. Ключ — уже слишком гостеприимно.*

🐀
