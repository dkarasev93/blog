+++
title = "№ 35 — Пансион «У чужого чертежа»: пятьдесят три ссылки на соседний дом и кандидат, которого «нода бы C++» приняла"
date = 2026-08-31T11:01:00+03:00
description = "Тридцать пятый выпуск «Вечернего Валидатора»: сыщик постучал в новый дом xssnick/gton — цельную ноду TON на языке Го — и обнаружил пансион, живущий по чертежам соседа: домовые правила AGENTS.md велят сверять поведение с cppnode, пятьдесят три комментария начинаются словами «C++ делает так», константы пиринга скопированы с точностью до тысячи двадцати четырёх байт, а ради блоков, которые «нода бы C++» приняла, гошная нода пересериализует граф заново — с опечаткой «a candidate a C++ node» на память."
tags = ["gton"]
+++

# 📰 Вечерний Валидатор

**№ 35.** *Лондон. На окраине города, за кузницей mytonctrl (спецвыпуск) и чертёжным бюро (выпуск № 29), вырос новый дом — большой, свежий, пахнущий краской: [xssnick/gton](https://github.com/xssnick/gton), цельная нода TON на языке Го, с лайтсервером, хранилищем на Pebble и табличкой «валидаторских дел не делаем, зато кормим АПИ быстро». Сыщик вошёл с уважением: дом новый, жильцы работают. А вышел с блокнотом полным чужого почерка: весь пансион живёт по чертежам соседнего дома — того самого, каменного, сиплюпласного.*

*Место осмотра: репозиторий [xssnick/gton](https://github.com/xssnick/gton), коммит [`d9dfd03`](https://github.com/xssnick/gton/commit/d9dfd0399b435dc74c73fbd26b1b677713ba097e) (master от 26.08.2026). Файлы: [`AGENTS.md`](https://github.com/xssnick/gton/blob/d9dfd0399b435dc74c73fbd26b1b677713ba097e/AGENTS.md), [`service/p2p/types.go`](https://github.com/xssnick/gton/blob/d9dfd0399b435dc74c73fbd26b1b677713ba097e/service/p2p/types.go), [`service/p2p/broadcast_block.go`](https://github.com/xssnick/gton/blob/d9dfd0399b435dc74c73fbd26b1b677713ba097e/service/p2p/broadcast_block.go), [`service/masterchain_ref.go`](https://github.com/xssnick/gton/blob/d9dfd0399b435dc74c73fbd26b1b677713ba097e/service/masterchain_ref.go), [`service/blockproof/validator_set.go`](https://github.com/xssnick/gton/blob/d9dfd0399b435dc74c73fbd26b1b677713ba097e/service/blockproof/validator_set.go). Все цитаты дословные.*

---

## УЛИКА ПЕРВАЯ: ДОМОВЫЕ ПРАВИЛА, НАПИСАННЫЕ СОСЕДОМ

В прихожей пансиона висит домовая книга — [`AGENTS.md`](https://github.com/xssnick/gton/blob/d9dfd0399b435dc74c73fbd26b1b677713ba097e/AGENTS.md), устав для механических помощников. Строки [5–6](https://github.com/xssnick/gton/blob/d9dfd0399b435dc74c73fbd26b1b677713ba097e/AGENTS.md#L5-L6):

```
- `cppnode` contains the C++ node implementation.
- Use it for research and to verify that our behavior matches the reference implementation.
```

Переведём с канцелярского: «в соседнем доме стоит настоящая машина; ходите туда, смотрите и делайте так же». Это не стыдно, читатель, — это здраво: протокол сети один, и жить по нему надо одинаково. Но сыщик завёл счётную книгу и прошёлся по комнатам: пометок «C++ делает вот так» и «как у cppnode» в гошных фолиантах ровно **пятьдесят три**. Пятьдесят три раза дом смотрит в окно на соседа, прежде чем повесить полку.

## УЛИКА ВТОРАЯ: КОНСТАНТЫ С ЧУЖОГО ПЛЕЧА

Комната пиринга, [`service/p2p/types.go`](https://github.com/xssnick/gton/blob/d9dfd0399b435dc74c73fbd26b1b677713ba097e/service/p2p/types.go), строки [20–42](https://github.com/xssnick/gton/blob/d9dfd0399b435dc74c73fbd26b1b677713ba097e/service/p2p/types.go#L20-L42) — образец жанра:

```go
// maxPeersPerOverlay matches the C++ reference OverlayOptions::max_peers_
// (300 since the plumtree rework; the historical 20 starved broadcast
// reception — with so few roster edges, losing a couple of active senders
// dropped the node out of everyone's fanout sets).
maxPeersPerOverlay = 300
```

Триста — «как в справочном», и честная история болезни: когда-то стояло двадцать, и нода голодала, выпадая из всех вееров рассылки. Дальше арифметика величин, снятая с соседнего станка: строки [25–29](https://github.com/xssnick/gton/blob/d9dfd0399b435dc74c73fbd26b1b677713ba097e/service/p2p/types.go#L25-L29) — «C++ runs a live working set of ~30-46 per overlay», отсюда потолок в 64 живых транспорта (16 соседей запросов + 20 соседей сливового дерева + 24 транзитных = 60, остаток — «на ротацию»). А на строках [41–42](https://github.com/xssnick/gton/blob/d9dfd0399b435dc74c73fbd26b1b677713ba097e/service/p2p/types.go#L41-L42) — швейная точность:

```go
// C++ private overlays set the RLDP2 peer MTU to max broadcast size + 1024.
maxRLDPTwoStepTransferSize = maxOverlayPayloadSize + 1024
```

Шестнадцать мегабайт **плюс тысяча двадцать четыре байта** — потому что у соседа так. Не 1000, не 2048 — ровно 1024, как вышито на чужой манжете. Сыщик не осуждает: протокол — барышня строгая, и приёмный размер RLDP2 в приватных оверлеях либо совпадает до байта, либо тебя не пускают на танец. Но занести в протокол «пансион живёт чужими мерками» — обязан.

Есть там и живая казуистика, строки [38–39](https://github.com/xssnick/gton/blob/d9dfd0399b435dc74c73fbd26b1b677713ba097e/service/p2p/types.go#L38-L39): потолок простаивающих пиров был 2048, «was never reached, so nothing was ever swept» — сторож нанят на 2048 мест, ни разу не заполнил и восьмой части, и потому ни разу никого не выметал. Теперь мест 256. Прогресс, читатель: метла наконец-то имеет шанс коснуться пола.

## УЛИКА ТРЕТЬЯ: КАНДИДАТ, КОТОРОГО «НОДА БЫ C++» ПРИНЯЛА

Жемчужина осмотра — приёмная блоков, [`service/p2p/broadcast_block.go`](https://github.com/xssnick/gton/blob/d9dfd0399b435dc74c73fbd26b1b677713ba097e/service/p2p/broadcast_block.go). По проводу блок-кандидат приезжает пересериализованным в режиме 2 (строка [153](https://github.com/xssnick/gton/blob/d9dfd0399b435dc74c73fbd26b1b677713ba097e/service/p2p/broadcast_block.go#L153): «the wire payload is a mode-2 re-serialization (see the C++ sender)»), а хеш файла считается по каноническим байтам режима 31. Чужая причудливая упаковка — наша головная боль: быстрый разбор графа сохраняет дубли ячеек, соседская машина их сливает, и хеш не сходится. Что делает гошная нода? Строки [165–168](https://github.com/xssnick/gton/blob/d9dfd0399b435dc74c73fbd26b1b677713ba097e/service/p2p/broadcast_block.go#L165-L168):

```go
// the graph fast path keeps sender-side duplicate cells while the
// reference node deduplicates on re-serialization; retry with the
// canonical serializer before rejecting, so a candidate a C++ node
// would accept is not dropped here
```

Прежде чем вышвырнуть кандидата за порог, нода пересобирает его каноническим сериализатором — заново, целиком, — лишь бы «a candidate a C++ node would accept» не пропал. Остановитесь, читатель, и перечитайте: **«a candidate a C++ node»** — две частицы «a» подряд, писарь заикнулся от усердия, дословно переписывая повадки соседа. Смысл безупречен, грамматика — нет. Сыщик отметил в блокноте: даже опечатки в этом доме появляются от слишком пристального взгляда в чужое окно.

И это не единичный поклон. В [`service/masterchain_ref.go`](https://github.com/xssnick/gton/blob/d9dfd0399b435dc74c73fbd26b1b677713ba097e/service/masterchain_ref.go) дважды, на строках [14](https://github.com/xssnick/gton/blob/d9dfd0399b435dc74c73fbd26b1b677713ba097e/service/masterchain_ref.go#L14) и [25](https://github.com/xssnick/gton/blob/d9dfd0399b435dc74c73fbd26b1b677713ba097e/service/masterchain_ref.go#L25), записано: «cppnode sets BlockHandle::masterchain_ref_block…» — делаем как он. В [`service/blockproof/validator_set.go`](https://github.com/xssnick/gton/blob/d9dfd0399b435dc74c73fbd26b1b677713ba097e/service/blockproof/validator_set.go), строка [102](https://github.com/xssnick/gton/blob/d9dfd0399b435dc74c73fbd26b1b677713ba097e/service/blockproof/validator_set.go#L102): «C++ uses cur_temp_validators as active when present, then falls back to cur_validators» — и гошная нода покорно повторяет этот фокус с временными валидаторами, потому что истина — не в спецификации, а в соседнем доме.

## РЕЗЮМЕ СЫЩИКА

Дело не криминальное, читатель, а поучительное. Пансион gton — честный перенос взрослой сиплюпласной ноды на язык Го, и пятьдесят три пометки «как у C++» — это не плагиат, а дисциплина: сеть простит тебе молодость, но не простит иное поведение на проводе. Хозяин (тот самый xssnick, что держит сыскное бюро tonutils-go из выпуска № 32) ведёт себя как добросовестный управляющий: копирует с пристрастием, комментирует с историей болезни, исправляет голодные двадцатки на три сотни. Единственное, за что дом получает выговор от редакции, — заикание «a candidate a C++ node» на строке [167](https://github.com/xssnick/gton/blob/d9dfd0399b435dc74c73fbd26b1b677713ba097e/service/p2p/broadcast_block.go#L167). Впрочем, когда гоняешься за чужой машиной три года, начинаешь говорить с её акцентом.

🐀
