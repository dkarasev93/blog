+++
title = "№ 81 — Таймер и задача, которые одновременно вынесли приговор"
date = 2026-09-24T17:00:35+03:00
description = "Восемьдесят первый выпуск «Вечернего Валидатора»: в ядре TON таймер и настоящая задача могли первыми вынести разный вердикт одной Promise, пока atomic_flag не поставил их в очередь."
tags = ["ton"]
+++

# ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

**№ 81.** *Лондон. Туман ползет по мостовой, газовый рожок у редакции сипит, а сыщик получил свежую папку из машинного отделения [ton-blockchain/ton](https://github.com/ton-blockchain/ton). На обложке написано: «ждать с таймером». Внутри два посыльных — сама задача и ее сторожевой будильник — бегут к одной двери с разными приговорами. Один приносит результат, другой через срок приносит ошибку. Кто первым схватится за ручку, тот и прав.*

## МЕСТО ПРОИСШЕСТВИЯ

Улика найдена в репозитории [ton-blockchain/ton](https://github.com/ton-blockchain/ton), в коммите [`8180ecf`](https://github.com/ton-blockchain/ton/commit/8180ecf3bfe3249fcd8fa25114bebd56b3ee029c), в файле [`tdactor/td/actor/SharedFuture.h`](https://github.com/ton-blockchain/ton/blob/8180ecf3bfe3249fcd8fa25114bebd56b3ee029c/tdactor/td/actor/SharedFuture.h). Сам коммит называется `Fix race in await_with_timeout`: газета цитирует уже поставленный замок, а его форма честно показывает, от какой гонки спасались.

## СЕНСАЦИЯ: ДВА ПОСЫЛЬНЫХ, ОДНА PROMISE

Функция [`await_with_timeout`](https://github.com/ton-blockchain/ton/blob/8180ecf3bfe3249fcd8fa25114bebd56b3ee029c/tdactor/td/actor/SharedFuture.h#L60-L87) получает задачу и срок ожидания. На строке [62](https://github.com/ton-blockchain/ton/blob/8180ecf3bfe3249fcd8fa25114bebd56b3ee029c/tdactor/td/actor/SharedFuture.h#L62) она создает мост к результату, а на строке [63](https://github.com/ton-blockchain/ton/blob/8180ecf3bfe3249fcd8fa25114bebd56b3ee029c/tdactor/td/actor/SharedFuture.h#L63) складывает Promise вместе с флагом в общий ящик:

```cpp
auto [task_result, promise] = StartedTask<T>::make_bridge();
auto promise_ptr = std::make_shared<std::pair<Promise<T>, std::atomic_flag>>(std::move(promise), false);
```

Затем из ящика выходят два работника. Первый следит за часами, второй ждет настоящую задачу. Оба имеют доступ к одной Promise. В старой сцене это была бы прекрасная канцелярская дуэль: будильник мог первым вписать `await timeout`, а задача почти в тот же миг — настоящий результат. Два пера, одна строка, ни одного свободного места для мирного протокола.

## ПЕРВЫЙ ПОСЫЛЬНЫЙ: СРОК ПРИШЕЛ С ПЕЧАТЬЮ ОШИБКИ

Сторожевой работник записан в строках [65–74](https://github.com/ton-blockchain/ton/blob/8180ecf3bfe3249fcd8fa25114bebd56b3ee029c/tdactor/td/actor/SharedFuture.h#L65-L74). Он отсоединяется от актора, спит до срока, а потом пробует занять флаг:

```cpp
    auto worker_timeout = [](Timestamp timeout,
                             std::shared_ptr<std::pair<Promise<T>, std::atomic_flag>> promise_ptr) -> Task<> {
      co_await detach_from_actor();
      co_await coro_sleep(timeout);
      if (!promise_ptr->second.test_and_set()) {
        promise_ptr->first.set_error(Status::Error(AWAIT_TIMEOUT_CODE, "await timeout"));
      }
      co_return {};
    };
```

Строка [69](https://github.com/ton-blockchain/ton/blob/8180ecf3bfe3249fcd8fa25114bebd56b3ee029c/tdactor/td/actor/SharedFuture.h#L69) — новая дверь с одним пропуском. Если флаг еще свободен, сторож его занимает и строкой [70](https://github.com/ton-blockchain/ton/blob/8180ecf3bfe3249fcd8fa25114bebd56b3ee029c/tdactor/td/actor/SharedFuture.h#L70) выдает ошибку тайм-аута. Если задача уже успела первой, сторож молча уходит. Викторианская полиция назвала бы это строгим правилом: первый протокол принят, второй отправлен в архив.

## ВТОРОЙ ПОСЫЛЬНЫЙ: РЕЗУЛЬТАТ СТУЧИТСЯ ПОЗЖЕ

Настоящая задача идет по соседнему коридору, строки [76–85](https://github.com/ton-blockchain/ton/blob/8180ecf3bfe3249fcd8fa25114bebd56b3ee029c/tdactor/td/actor/SharedFuture.h#L76-L85):

```cpp
    auto worker_wait = [](StartedTask<T> task,
                          std::shared_ptr<std::pair<Promise<T>, std::atomic_flag>> promise_ptr) -> Task<> {
      co_await detach_from_actor();
      auto result = co_await std::move(task).wrap();
      if (!promise_ptr->second.test_and_set()) {
        promise_ptr->first.set_result(std::move(result));
      }
      co_return {};
    };
```

После ожидания результата работник тоже проверяет тот же флаг на строке [80](https://github.com/ton-blockchain/ton/blob/8180ecf3bfe3249fcd8fa25114bebd56b3ee029c/tdactor/td/actor/SharedFuture.h#L80). Если он первый — результат отправляется в Promise на строке [81](https://github.com/ton-blockchain/ton/blob/8180ecf3bfe3249fcd8fa25114bebd56b3ee029c/tdactor/td/actor/SharedFuture.h#L81). Если опоздал — письмо не вскрывается второй раз.

## КАКОЙ КРИК БЫЛ В ПОДВАЛЕ

Сыщик не станет приписывать старому варианту то, чего один фрагмент не доказывает. Но сам коммит [8180ecf](https://github.com/ton-blockchain/ton/commit/8180ecf3bfe3249fcd8fa25114bebd56b3ee029c) прямо называет происшествие гонкой. И исправление выстроено вокруг одного факта: таймер и завершение задачи — независимые пути к одной Promise.

До нового порядка оба пути могли добраться до финального действия каждый со своей стороны. В лучшем случае второй вызов был бы отвергнут внутренностями Promise. В худшем — ранний тайм-аут победил бы почти готовый результат или порядок зависел бы от случайного движения планировщика. Не баг с кричащей вывеской, а мелкая дуэль за последнюю строку, где исход меняется от нескольких микросекунд.

## ХРОНИКА МЕЛКОГО КРИНЖА

*Отдел упаковки.* На строке [63](https://github.com/ton-blockchain/ton/blob/8180ecf3bfe3249fcd8fa25114bebd56b3ee029c/tdactor/td/actor/SharedFuture.h#L63) Promise и `atomic_flag` живут бок о бок в `pair`. Суровый, но выразительный сейф: один флаг на два независимых пера.

*Отдел тишины.* Оба работника запускаются как отсоединенные задачи: таймер на строке [74](https://github.com/ton-blockchain/ton/blob/8180ecf3bfe3249fcd8fa25114bebd56b3ee029c/tdactor/td/actor/SharedFuture.h#L74), ожидание на строке [85](https://github.com/ton-blockchain/ton/blob/8180ecf3bfe3249fcd8fa25114bebd56b3ee029c/tdactor/td/actor/SharedFuture.h#L85). Значит, оба посыльных не стоят в очереди у одного клерка. Они разбегаются по городу, а встречаются уже у общего замка.

*Отдел последнего слова.* Только победитель вызывает `set_error` на строке [70](https://github.com/ton-blockchain/ton/blob/8180ecf3bfe3249fcd8fa25114bebd56b3ee029c/tdactor/td/actor/SharedFuture.h#L70) или `set_result` на строке [81](https://github.com/ton-blockchain/ton/blob/8180ecf3bfe3249fcd8fa25114bebd56b3ee029c/tdactor/td/actor/SharedFuture.h#L81). Остальным остается честное молчание.

## ВЕРДИКТ СЫЩИКА

Репозиторий [ton-blockchain/ton](https://github.com/ton-blockchain/ton) пойман не на том, что умеет ждать, а на том, что раньше два разных исхода могли одновременно претендовать на одну Promise. Коммит [`8180ecf`](https://github.com/ton-blockchain/ton/commit/8180ecf3bfe3249fcd8fa25114bebd56b3ee029c) в файле [`tdactor/td/actor/SharedFuture.h`](https://github.com/ton-blockchain/ton/blob/8180ecf3bfe3249fcd8fa25114bebd56b3ee029c/tdactor/td/actor/SharedFuture.h) ставит atomic-флажок между сроком и результатом: кто первым вызвал `test_and_set`, тот и пишет в протокол.

На строках [65–85](https://github.com/ton-blockchain/ton/blob/8180ecf3bfe3249fcd8fa25114bebd56b3ee029c/tdactor/td/actor/SharedFuture.h#L65-L85) лежит вся викторианская мораль: не важно, пришел ли посыльный с хорошей вестью или с тайм-аутом. Важно, чтобы дверь открылась ровно одному. Газовый рожок дернулся, туман сгустился. В машинном отделении наконец оставили место только для одного последнего слова.

🐀
