---
title: "Выпуск № 65: Два отказника и одна обещанная пауза"
date: 2026-09-16T10:58:00+03:00
tags: [ton]
---

# 📰 ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

## ДЕЛО О ДВУХ РАБОТНИКАХ, КОТОРЫЕ СОРЕВНОВАЛИСЬ ЗА ОДНУ КНОПКУ

Лондон проснулся под мокрым туманом. У редакции сипел газовый рожок, когда сыщик «Вечернего Валидатора» получил депешу из главной конторы [ton-blockchain/ton](https://github.com/ton-blockchain/ton). На конверте стояла сухая надпись исправляющего коммита [`8180ecf`](https://github.com/ton-blockchain/ton/commit/8180ecf3bfe3249fcd8fa25114bebd56b3ee029c): `Fix race in await_with_timeout`.

Сухость заголовка скрывала великолепную сцену. Служба `await_with_timeout` запускала двух работников: один ждал ответа, другой ждал таймаута. Оба в конце пытались вынести решение через одну и ту же кнопку `Promise`. Кто успеет первым, тот и назовет судьбу операции. Второй мог прийти следом с еще одним приговором.

## МЕСТО ПРОИСШЕСТВИЯ

Репозиторий [ton-blockchain/ton](https://github.com/ton-blockchain/ton), родительский коммит [`e23d281`](https://github.com/ton-blockchain/ton/commit/e23d2817f509da50fe7f4ae980905cbe4cb99900), файл [`tdactor/td/actor/SharedFuture.h`](https://github.com/ton-blockchain/ton/blob/e23d2817f509da50fe7f4ae980905cbe4cb99900/tdactor/td/actor/SharedFuture.h), строки [60–80](https://github.com/ton-blockchain/ton/blob/e23d2817f509da50fe7f4ae980905cbe4cb99900/tdactor/td/actor/SharedFuture.h#L60-L80). Цитата сверена по содержимому файла на этом коммите.

Вот дословный протокол старой службы:

```cpp
template <typename T>
Task<T> await_with_timeout(StartedTask<T> task, Timestamp timeout) {
  auto [task_result, promise] = StartedTask<T>::make_bridge();
  auto promise_ptr = std::make_shared<Promise<T>>(std::move(promise));
  if (timeout) {
    auto worker_timeout = [](Timestamp timeout, std::shared_ptr<Promise<T>> promise_ptr) -> Task<> {
      co_await td::actor::detach_from_actor();
      co_await coro_sleep(timeout);
      promise_ptr->set_error(Status::Error(AWAIT_TIMEOUT_CODE, "await timeout"));
      co_return {};
    };
    worker_timeout(timeout, promise_ptr).start().detach_silent();
  }
  auto worker_wait = [](StartedTask<T> task, std::shared_ptr<Promise<T>> promise_ptr) -> Task<> {
    co_await td::actor::detach_from_actor();
    promise_ptr->set_result(co_await std::move(task).wrap());
    co_return {};
  };
  worker_wait(std::move(task), std::move(promise_ptr)).start().detach_silent();
  co_return co_await std::move(task_result);
}
```

## ПЕРВЫЙ СЛЕД: СУДЬЯ С ДВУМЯ ПОСЫЛЬНЫМИ

На строке [63](https://github.com/ton-blockchain/ton/blob/e23d2817f509da50fe7f4ae980905cbe4cb99900/tdactor/td/actor/SharedFuture.h#L63) для обоих работников изготовлен общий конверт с одной `Promise`. Затем ветка таймаута на строках [65–71](https://github.com/ton-blockchain/ton/blob/e23d2817f509da50fe7f4ae980905cbe4cb99900/tdactor/td/actor/SharedFuture.h#L65-L71) засыпает на заданный срок и возвращается с ошибкой `await timeout`.

Но соседний работник на строках [73–78](https://github.com/ton-blockchain/ton/blob/e23d2817f509da50fe7f4ae980905cbe4cb99900/tdactor/td/actor/SharedFuture.h#L73-L78) в это же время ждет настоящую задачу. Если задача успела первой, он пишет результат. Если первым проснулся таймаут, он пишет ошибку. До сих пор все походило на обычную гонку за звонком. Беда в том, что код не ставил на звонок ни замок, ни флажок: оба работника напрямую звонили `set_error` или `set_result`.

## ВТОРОЙ СЛЕД: ОДНА PROMISE, ДВА ПРИГОВОРА

Сыщик не станет утверждать лишнего: из одного фрагмента не следует, что каждая такая сцена непременно роняет процесс. Но следует другое: порядок двух финальных записей определялся состязанием корутин, а вторая запись не была защищена общей атомарной отметкой победителя.

Особенно комична строка [75](https://github.com/ton-blockchain/ton/blob/e23d2817f509da50fe7f4ae980905cbe4cb99900/tdactor/td/actor/SharedFuture.h#L75): вызов `set_result` получает результат вложенного ожидания прямо из `co_await`. Пока один посыльный дожидается ответа, другой уже может выдать наружу отказ. Судья получает два конверта с разными показаниями, а канцелярия надеется, что второй никто не заметит.

Имя функции обещает аккуратный предел времени. Реальная механика обещала состязание: нормальный результат против искусственной ошибки, одна общая `Promise`, два независимых пути завершения. Викторианский дворецкий сказал бы, что это не расписание, а дуэль в гардеробной.

## ТРЕТИЙ СЛЕД: КОНТОРА ПРИЗНАЛА ВИНУ

Исправляющий коммит [`8180ecf`](https://github.com/ton-blockchain/ton/commit/8180ecf3bfe3249fcd8fa25114bebd56b3ee029c) сменил общую упаковку на пару из `Promise` и `std::atomic_flag`. В том же файле [`tdactor/td/actor/SharedFuture.h`](https://github.com/ton-blockchain/ton/blob/8180ecf3bfe3249fcd8fa25114bebd56b3ee029c/tdactor/td/actor/SharedFuture.h), строки [60–86](https://github.com/ton-blockchain/ton/blob/8180ecf3bfe3249fcd8fa25114bebd56b3ee029c/tdactor/td/actor/SharedFuture.h#L60-L86), теперь лежит новый протокол:

```cpp
auto promise_ptr = std::make_shared<std::pair<Promise<T>, std::atomic_flag>>(std::move(promise), false);
```

И дословная охрана у двери:

```cpp
if (!promise_ptr->second.test_and_set()) {
  promise_ptr->first.set_error(Status::Error(AWAIT_TIMEOUT_CODE, "await timeout"));
}
```

Та же проверка появилась для результата на строках [76–82](https://github.com/ton-blockchain/ton/blob/8180ecf3bfe3249fcd8fa25114bebd56b3ee029c/tdactor/td/actor/SharedFuture.h#L76-L82). Кто первым поставил флажок, тот и получил право вынести вердикт. Второй работник может спокойно уйти в туман: его рука уже не дотянется до `Promise`.

## ПОЧЕМУ ЭТО СОЧНО

Кринж здесь не в сложном шаблоне C++. Наоборот, он в простоте декораций. Функция с именем про таймаут нанимает двух корутин, выдает им один и тот же конверт и позволяет каждой самостоятельно подписать окончательный ответ. Снаружи виден спокойный `co_await`, а в подвале два клерка бегут к одному звонку.

Еще выразительнее контраст коммитов. В родительском коммите [`e23d281`](https://github.com/ton-blockchain/ton/commit/e23d2817f509da50fe7f4ae980905cbe4cb99900) строка [63](https://github.com/ton-blockchain/ton/blob/e23d2817f509da50fe7f4ae980905cbe4cb99900/tdactor/td/actor/SharedFuture.h#L63) отдает общую `Promise` без флажка. В исправляющем коммите [`8180ecf`](https://github.com/ton-blockchain/ton/commit/8180ecf3bfe3249fcd8fa25114bebd56b3ee029c) строки [69–71](https://github.com/ton-blockchain/ton/blob/8180ecf3bfe3249fcd8fa25114bebd56b3ee029c/tdactor/td/actor/SharedFuture.h#L69-L71) и [80–82](https://github.com/ton-blockchain/ton/blob/8180ecf3bfe3249fcd8fa25114bebd56b3ee029c/tdactor/td/actor/SharedFuture.h#L80-L82) уже требуют право подписи через `test_and_set`.

Редакция не станет клеймить [ton-blockchain/ton](https://github.com/ton-blockchain/ton) темной лавкой. Коммит [`8180ecf`](https://github.com/ton-blockchain/ton/commit/8180ecf3bfe3249fcd8fa25114bebd56b3ee029c) назван прямо и чинит именно гонку. Но старая улика в [`tdactor/td/actor/SharedFuture.h`](https://github.com/ton-blockchain/ton/blob/e23d2817f509da50fe7f4ae980905cbe4cb99900/tdactor/td/actor/SharedFuture.h) хороша тем, что два честных механизма — ответ и таймаут — превращали одну кнопку в маленький азартный стол.

## ВЕРДИКТ СЫЩИКА

[ton-blockchain/ton](https://github.com/ton-blockchain/ton) поймано на деле о двух отказниках. В родительском коммите [`e23d281`](https://github.com/ton-blockchain/ton/commit/e23d2817f509da50fe7f4ae980905cbe4cb99900) файл [`tdactor/td/actor/SharedFuture.h`](https://github.com/ton-blockchain/ton/blob/e23d2817f509da50fe7f4ae980905cbe4cb99900/tdactor/td/actor/SharedFuture.h) на строках [63–75](https://github.com/ton-blockchain/ton/blob/e23d2817f509da50fe7f4ae980905cbe4cb99900/tdactor/td/actor/SharedFuture.h#L63-L75) давал таймауту и настоящему ответу прямой доступ к одной `Promise`.

Исправляющий коммит [`8180ecf`](https://github.com/ton-blockchain/ton/commit/8180ecf3bfe3249fcd8fa25114bebd56b3ee029c) в том же файле [`tdactor/td/actor/SharedFuture.h`](https://github.com/ton-blockchain/ton/blob/8180ecf3bfe3249fcd8fa25114bebd56b3ee029c/tdactor/td/actor/SharedFuture.h) поставил между ними атомарный флажок: строки [69–71](https://github.com/ton-blockchain/ton/blob/8180ecf3bfe3249fcd8fa25114bebd56b3ee029c/tdactor/td/actor/SharedFuture.h#L69-L71) для ошибки и [80–82](https://github.com/ton-blockchain/ton/blob/8180ecf3bfe3249fcd8fa25114bebd56b3ee029c/tdactor/td/actor/SharedFuture.h#L80-L82) для результата.

Приговор прост: у одной операции должен быть один победитель. Сыщик еще раз сверил строки [60–80](https://github.com/ton-blockchain/ton/blob/e23d2817f509da50fe7f4ae980905cbe4cb99900/tdactor/td/actor/SharedFuture.h#L60-L80) и [60–86](https://github.com/ton-blockchain/ton/blob/8180ecf3bfe3249fcd8fa25114bebd56b3ee029c/tdactor/td/actor/SharedFuture.h#L60-L86), погас газовый рожок и растворился в тумане. В Лондоне два посыльных могут бежать быстро. Но на одной кнопке все равно должен остаться один отпечаток.

🐀
