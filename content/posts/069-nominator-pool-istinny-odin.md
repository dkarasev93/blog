+++
title = "№ 69 — Пул номинаторов: доказательство для галочки и режим, который все еще в черновике"
date = 2026-09-18T11:04:00+03:00
description = "Шестьдесят девятый выпуск «Вечернего Валидатора»: сыщик входит в пул номинаторов и находит проверку, которая всегда права по типу, режим, оставленный в комментарии, и возврат, который делает вид, что потери не было."
tags = ["nominator-pool-v2"]
+++

# 📰 Вечерний Валидатор

**№ 69.** *Лондон. Туман нынче лег на Темзу так плотно, что даже газовый рожок светит не вперед, а внутрь себя. Сыщик получил свежую депешу из нового учреждения — пула номинаторов, где ставки валидаторов и деньги вкладчиков ходят по кругу выборов. Вывеска строгая, двери тяжелые, а внутри пахнет свежей краской и старой формальностью: один сторож признается, что проверяет невозможное, другой оставляет настоящий режим на потом, а третий при возврате средств старается сделать вид, что происшествия не было.*

*Место происшествия: репозиторий [ton-blockchain/nominator-pool-v2](https://github.com/ton-blockchain/nominator-pool-v2), коммит [`450443e`](https://github.com/ton-blockchain/nominator-pool-v2/commit/450443e7b2e2ceddd2637b6402e2f2a055dcb458) (master от 14.09.2026), файл [`contracts/Pool.tolk`](https://github.com/ton-blockchain/nominator-pool-v2/blob/450443e7b2e2ceddd2637b6402e2f2a055dcb458/contracts/Pool.tolk). Все цитаты дословные.*

---

## СЕНСАЦИЯ ПЕРВАЯ: ПРОВЕРКА, КОТОРАЯ НЕ МОЖЕТ ПРОВАЛИТЬСЯ

Сначала сыщик поднялся к окну, где принимают новое сообщение о ставке. Перед тем как отправить сумму дальше, палата разбирает подпись и достает из нее `stakeAt`. На бумаге это выглядит как проверка времени. В деле — как проверка, заранее лишенная всякой интриги. Вот протокол, строки [359–364](https://github.com/ton-blockchain/nominator-pool-v2/blob/450443e7b2e2ceddd2637b6402e2f2a055dcb458/contracts/Pool.tolk#L359-L364):

```tolk
                // Validates the NewStake body
                val newStake = NewStakeBody.fromSlice(msg.signedBody);
                // stakeAt is uint32, so this assert is always true.
                // Its real purpose is to use newStake, preventing the compiler
                // from optimizing out the fromSlice call above.
                assert(newStake.stakeAt >= 0, ErrorsPool.InvalidMessage);
```

Читатель, не спешите звать адвоката арифметики. Канцелярия сама все растолковала: `stakeAt is uint32, so this assert is always true`, строка [361](https://github.com/ton-blockchain/nominator-pool-v2/blob/450443e7b2e2ceddd2637b6402e2f2a055dcb458/contracts/Pool.tolk#L361). Число без знака не может быть меньше нуля. Значит, сторож не проверяет, что метка времени верна, свежа или хотя бы имеет отношение к текущему раунду. Он лишь заставляет бумагу пройти через `fromSlice`, чтобы компилятор не выкинул разбор как лишний, строки [362–364](https://github.com/ton-blockchain/nominator-pool-v2/blob/450443e7b2e2ceddd2637b6402e2f2a055dcb458/contracts/Pool.tolk#L362-L364).

Это не баг уровня «одна запятая уехала в туман». Это прекрасная сцена: функция читает тело сообщения, находит в нем время, потом делает вид, что допросила время, хотя единственный вопрос звучит так: «Ты не отрицательное?» Время отвечает: «Я `uint32`». Палата ставит печать `InvalidMessage` на случай, который не наступит, и отправляет депешу дальше с чувством исполненного долга.

Сыщик проверил, что настоящая проверка суммы все же стоит рядом. В строках [366–375](https://github.com/ton-blockchain/nominator-pool-v2/blob/450443e7b2e2ceddd2637b6402e2f2a055dcb458/contracts/Pool.tolk#L366-L375) сверяются лимиты ставки и доступный размер. Так что деньги не оставлены без всякой охраны. Но поле, которому приписали роль метки времени, живет в протоколе как театральный свидетель: присутствует, раскрывает рот, не сообщает ничего.

## СЕНСАЦИЯ ВТОРАЯ: ЕДИНЫЙ НОМИНАТОР, КОТОРОГО ОТЛОЖИЛИ НА ПОТОМ

Далее сыщик прошел в зал, где распределяют ставку по четному и нечетному кругу. Над дверью висела табличка: «режим нескольких номинаторов». Внутри все именно так и устроено: палата считает текущий раунд, выбирает прокси, ведет `users`, учитывает лимиты. А где-то за занавеской, среди старых чертежей, лежит другой режим — для одного номинатора. Он не включен. Он даже не написан до конца.

Вот место, строки [377–399](https://github.com/ton-blockchain/nominator-pool-v2/blob/450443e7b2e2ceddd2637b6402e2f2a055dcb458/contracts/Pool.tolk#L377-L399):

```tolk
                val curVset = getCurVset();
                // For multi nominator mode
                val electionsConf = getElectionsTiming();
                val roundIsOdd = (storage.roundIndex % 2) > 0;
                val roundIndex = parityToMask(roundIsOdd);
                // if(storage.maxNominators > 0) {
                // True single nominator mode TODO
                assert(!storage.roundClosed, ErrorsPool.RoundIsClosed);
                assert((valData.usageState & roundIndex) == 0, ErrorsPool.AlreadyStakedInThatRound);
                assert((valData.roundParity as int & roundIndex) != 0, ErrorsPool.RoundNotAllowed);
                // + pendingWithdrawals because pending withdrawals are already part of nominatorsAmount
                val ownerShare = totalBalance - storage.nominatorsAmount + estPendingWithdrawals;
```

Прочтите строки [378–383](https://github.com/ton-blockchain/nominator-pool-v2/blob/450443e7b2e2ceddd2637b6402e2f2a055dcb458/contracts/Pool.tolk#L378-L383) медленно. Сначала честное «For multi nominator mode». Потом закомментированное условие. Потом приговор: `True single nominator mode TODO`. Не «режим включен при одном участнике», не «проверить число номинаторов», а просто заметка на полях. Самый важный режим для особого случая оставлен в состоянии привидения: его имя есть, его логика обещана, его скобки заколочены.

Дальше палата все равно ведет учет так, будто зал полон: проверяет занятость раунда в строках [384–386](https://github.com/ton-blockchain/nominator-pool-v2/blob/450443e7b2e2ceddd2637b6402e2f2a055dcb458/contracts/Pool.tolk#L384-L386), прибавляет слот и сумму в строках [388–393](https://github.com/ton-blockchain/nominator-pool-v2/blob/450443e7b2e2ceddd2637b6402e2f2a055dcb458/contracts/Pool.tolk#L388-L393), а затем отправляет ставку на прокси, строки [433–443](https://github.com/ton-blockchain/nominator-pool-v2/blob/450443e7b2e2ceddd2637b6402e2f2a055dcb458/contracts/Pool.tolk#L433-L443). Дом с одной квартирой уже построен по плану многоквартирного дома: если туда придет один житель, архитектура не изменится, только коридор будет звучать пустее.

Сыщик не утверждает, что это опасно само по себе. Пул может быть рассчитан именно на многономинаторскую схему, а один номинатор — будущая услуга. Но газета ведет хронику почерка. А почерк тут ясен: контракт принимает деньги сегодня, а особый режим обещает когда-нибудь. В Лондоне это называется не дорожной картой, а запиской, которую забыли снять с двери.

## СЕНСАЦИЯ ТРЕТЬЯ: ВОЗВРАТ, КОТОРЫЙ ДЕЛАЕТ ВИД, ЧТО НИЧЕГО НЕ БЫЛО

В третьем кабинете сыщик нашел процедуру обработки неудачного возврата. Если ставка вернулась почти целиком, палата решает, что это не настоящая потеря, удаляет запись о пользователе и уменьшает счетчик, будто неудачного события не было. Протокол, строки [68–80](https://github.com/ton-blockchain/nominator-pool-v2/blob/450443e7b2e2ceddd2637b6402e2f2a055dcb458/contracts/Pool.tolk#L68-L80):

```tolk
    // If return didn't have an effect on state, we can simply ignore
    if (stakeReturned != null) {
        var round = stakeReturned.round;
        val delta = abs(stakeReturned.data.tonUsed - value);
        round.users.delete(addrHash);
        if (delta <= ton("1")) {
            // If delta is withing newStake reserve
            // make it look like it never happen
            round.used = max(round.used - stakeReturned.data.tonUsed, 0);
        } else {
            // Otherwise book it as real loss.
            // practically looks impossible
            round.returned += value;
        }
```

Вещественные доказательства лежат прямо на столе. Сначала `delta` сравнивают с одним TON, строка [71](https://github.com/ton-blockchain/nominator-pool-v2/blob/450443e7b2e2ceddd2637b6402e2f2a055dcb458/contracts/Pool.tolk#L71). Потом запись пользователя удаляют, строка [72](https://github.com/ton-blockchain/nominator-pool-v2/blob/450443e7b2e2ceddd2637b6402e2f2a055dcb458/contracts/Pool.tolk#L72). Если разница мала, счетчик уменьшают на всю использованную сумму, строки [73–76](https://github.com/ton-blockchain/nominator-pool-v2/blob/450443e7b2e2ceddd2637b6402e2f2a055dcb458/contracts/Pool.tolk#L73-L76), и комментарий не прячет замысел: `make it look like it never happen`.

Орфография тут не единственная улика. Палата не говорит «событие было, но его стоимость мала». Она стирает запись, снимает сумму с раунда и оставляет город с чистым протоколом. Разница в один TON не считается потерей, а проходит как косметика. В большой финансовой системе один TON может быть платой за возврат, стоимостью действия или просто округлением. Здесь же он проходит через дверь архива как невиновный свидетель: его видели, но в деле его нет.

Если же дельта больше, начинается другая пьеса: сумма записывается в `round.returned`, строки [77–80](https://github.com/ton-blockchain/nominator-pool-v2/blob/450443e7b2e2ceddd2637b6402e2f2a055dcb458/contracts/Pool.tolk#L77-L80). То есть контора различает «почти вернулось» и «вернулось заметно», но граница проведена не по смыслу операции, а по одному круглому TON. В соседнем коридоре бухгалтерия старательно считает доли и штрафы, а тут сторож держит мерную линейку с одной отметкой.

---

## ХРОНИКА МЕЛКИХ ПРОИСШЕСТВИЙ

*Отдел честных признаний.* В той же процедуре стоит комментарий `// practically looks impossible`, строки [78–80](https://github.com/ton-blockchain/nominator-pool-v2/blob/450443e7b2e2ceddd2637b6402e2f2a055dcb458/contracts/Pool.tolk#L78-L80). «Практически кажется невозможным» — так палата описывает ветку, куда она сама положила учет реальной потери. Сыщик любит такие слова: если случай невозможен, его не надо удалять из кода. Достаточно положить рядом `round.returned += value` и ждать, пока невозможное придет с распиской.

*Отдел отрицательных итогов.* При обработке возврата ниже по протоколу прибыль считается как `returned - stakeReturned.tonUsed`, строка [556](https://github.com/ton-blockchain/nominator-pool-v2/blob/450443e7b2e2ceddd2637b6402e2f2a055dcb458/contracts/Pool.tolk#L556). В следующем коридоре палата уже отдельно разбирает случай `profit < 0`, строки [583–599](https://github.com/ton-blockchain/nominator-pool-v2/blob/450443e7b2e2ceddd2637b6402e2f2a055dcb458/contracts/Pool.tolk#L583-L599). Значит, отрицательная прибыль не привиделась сыщику: учреждение само знает, что возврат может быть меньше использованной суммы. На этом фоне фраза «practically looks impossible» звучит еще теплее: невозможное имеет отдельную бухгалтерскую процедуру.

*Отдел щедрого сообщения.* При новой ставке сумма для внутреннего учета равна `msg.value - ton("1")`, строка [339](https://github.com/ton-blockchain/nominator-pool-v2/blob/450443e7b2e2ceddd2637b6402e2f2a055dcb458/contracts/Pool.tolk#L339), тогда как лимиты выше сверяют с полным `msg.value`, строки [338–341](https://github.com/ton-blockchain/nominator-pool-v2/blob/450443e7b2e2ceddd2637b6402e2f2a055dcb458/contracts/Pool.tolk#L338-L341). Один TON назначен платой за путь, но в разных окнах его то считают частью ставки, то вычитают. Канцелярия не теряет деньги — она теряет единицу измерения между двумя дверями.

---

## ВЕРДИКТ РЕДАКЦИИ

Скажем по совести: [ton-blockchain/nominator-pool-v2](https://github.com/ton-blockchain/nominator-pool-v2) — не цирк, а серьезный пул с лимитами, раундами, прокси и защитой от неверных отправителей. Код не прячет свои спорные места: он сам сообщает, что проверка всегда истинна, режим одного номинатора еще не готов, а малая разница при возврате должна «сделать вид, что ничего не случилось». Такая честность лучше молчания, но газета все равно обязана поднять рожок.

Почерк выпуска таков: поле `stakeAt` разобрано ради побочного эффекта, а не проверено по смыслу; режим одного номинатора стоит в комментарии за закрытой скобкой; возврат до одного TON может стереть след участия из учета; а отрицательная прибыль, которую сперва назвали почти невозможной, уже имеет собственную комнату в бухгалтерии. В этом городе даже невозможное получает адрес, если сумма достаточно мала.

Туман над Темзой все принял. Сыщик закрыл дело, но оставил на двери записку: если сторож проверяет только тип числа, это еще не значит, что он проверил время.
