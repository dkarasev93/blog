---
title: "№ 130 — Шард, который вышел за дверь и оставил догоняющих в тумане"
date: 2026-11-05T16:59:00+03:00
tags: [gton]
---

# ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

## ДЕЛО О ПОЕЗДЕ, КОТОРЫЙ СМЕНИЛ ВЕТКУ

Лондон тонул в ноябрьском тумане, когда сыщик «Вечернего Валидатора» получил свежую папку из [xssnick/gton](https://github.com/xssnick/gton). Внутри лежал узел TON на Go, служба синхронизации и маленькая железная табличка: если следующий блок уехал на другую шардовую цепь, догонять его пока не обучили.

Газовый рожок кашлянул. Сеть, где шарды умеют расходиться и соединяться, встретила переход с видом кондуктора, который увидел незнакомую платформу и решил закончить смену.

## МЕСТО ПРОИСШЕСТВИЯ

Улика найдена в [коммите d1cbea5](https://github.com/xssnick/gton/commit/d1cbea5c33b5beaa751ffb87d564c586fe28060b) репозитория [xssnick/gton](https://github.com/xssnick/gton), в файле [service/blocksync/service.go](https://github.com/xssnick/gton/blob/d1cbea5c33b5beaa751ffb87d564c586fe28060b/service/blocksync/service.go), строки [599–607](https://github.com/xssnick/gton/blob/d1cbea5c33b5beaa751ffb87d564c586fe28060b/service/blocksync/service.go#L599-L607). Цитата дословная:

```go
			if chainKey(downloaded.ID) != c.key {
				c.service.log.Warn().
					Str("from", storage.FormatBlockRef(prev)).
					Str("got", downloaded.BlockRef()).
					Str("target", ev.BlockRef()).
					Str("overlay", ev.Overlay).
					Str("kind", ev.Kind).
					Msg("next block moved to another shard chain, catch-up is not implemented for this transition")
				return
			}
```

На строке [599](https://github.com/xssnick/gton/blob/d1cbea5c33b5beaa751ffb87d564c586fe28060b/service/blocksync/service.go#L599) служба сверяет цепочку скачанного блока с собственной. Если ключи не совпали, это не мелкая перемена адреса: следующий блок оказался в другой шардовой цепи.

А на строке [606](https://github.com/xssnick/gton/blob/d1cbea5c33b5beaa751ffb87d564c586fe28060b/service/blocksync/service.go#L606) сторож выдает признание: `catch-up is not implemented for this transition`. После этого на строке [607](https://github.com/xssnick/gton/blob/d1cbea5c33b5beaa751ffb87d564c586fe28060b/service/blocksync/service.go#L607) следует обычный `return`. Не новый маршрут, не вызов переходного механизма, не повторная попытка. Просто конец патруля.

## СНАЧАЛА ПРЫЖОК, ПОТОМ ЗАПИСКА В ЖУРНАЛЕ

Сцена особенно хороша тем, что до отказа код успевает получить следующий блок. На строке [583](https://github.com/xssnick/gton/blob/d1cbea5c33b5beaa751ffb87d564c586fe28060b/service/blocksync/service.go#L583) вызывается `waitForNextBlock`, а на строке [589](https://github.com/xssnick/gton/blob/d1cbea5c33b5beaa751ffb87d564c586fe28060b/service/blocksync/service.go#L589) проверяется, что номер последовательности вообще движется вперед.

То есть поезд прибыл. Проверка на [строках 599–606](https://github.com/xssnick/gton/blob/d1cbea5c33b5beaa751ffb87d564c586fe28060b/service/blocksync/service.go#L599-L606) замечает, что он прибыл на соседнюю ветку, и вместо пересадки оставляет сообщение в журнале. Служба синхронизации умеет увидеть перемену шардовой цепи, умеет предупредить о ней, но не умеет продолжить путь.

Дальше в [строках 610–615](https://github.com/xssnick/gton/blob/d1cbea5c33b5beaa751ffb87d564c586fe28060b/service/blocksync/service.go#L610-L615) лежит нормальный путь успеха: блок отправляется наружу как `SyncedBlock`, ему ставят `CatchUp: true`, а время скачивания записывают в отчет. Но до этой комнаты подозрительный пассажир не доходит. Проверка цепочки закрывает дверь раньше, чем блок получает право стать частью догонки.

## НЕ ВРЕМЕННАЯ НЕПРИЯТНОСТЬ, А ТАБЛИЧКА НА ДВЕРИ

Редакция не станет называть это дырой в консенсусе. Из фрагмента следует более точный приговор: конкретный путь catch-up останавливается, если очередной блок сменил шардовую цепь. Это может быть честным ограничением текущей реализации, но для службы синхронизации ограничение выглядит сурово.

Шардинг не обещает, что вся история будет идти по одной прямой. Поэтому проверка на [строке 599](https://github.com/xssnick/gton/blob/d1cbea5c33b5beaa751ffb87d564c586fe28060b/service/blocksync/service.go#L599) сама по себе разумна: нельзя бездумно притворяться, что блок из другой цепи продолжает прежний маршрут. Кринж начинается на [строке 606](https://github.com/xssnick/gton/blob/d1cbea5c33b5beaa751ffb87d564c586fe28060b/service/blocksync/service.go#L606), где отсутствие перехода не скрыто за сложной схемой, а честно напечатано в лог и завершено возвратом.

Для оператора картина проста и мрачна. Синхронизация шла, получила следующий блок, увидела новую шардовую цепь и прекратила догонку. В журнале осталась ясная фраза, но сам поезд уже не движется. Хороший сторож, плохой перевозчик.

## ВЕРДИКТ СЫЩИКА

[xssnick/gton](https://github.com/xssnick/gton) в [коммите d1cbea5](https://github.com/xssnick/gton/commit/d1cbea5c33b5beaa751ffb87d564c586fe28060b) оставил в [service/blocksync/service.go](https://github.com/xssnick/gton/blob/d1cbea5c33b5beaa751ffb87d564c586fe28060b/service/blocksync/service.go) редкую честность: на строке [606](https://github.com/xssnick/gton/blob/d1cbea5c33b5beaa751ffb87d564c586fe28060b/service/blocksync/service.go#L606) прямо сказано, что догонка при переходе между шардовыми цепями не готова, а на строке [607](https://github.com/xssnick/gton/blob/d1cbea5c33b5beaa751ffb87d564c586fe28060b/service/blocksync/service.go#L607) караул расходится.

Сыщик выносит мягкий, но звонкий вердикт: если поезд пересел на другую ветку, одной записи в журнале мало. Пассажир ждет маршрут, оператор ждет синхронизацию, а код ждет, когда кто-то наконец реализует переход.

Туман сомкнулся над рельсами. Газовый рожок выдал последний сигнал: **в блокчейне можно потерять нить не потому, что блок пропал, а потому, что он приехал не на ту платформу**.

🐀
