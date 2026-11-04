---
title: "№ 128 — Полный узел, который доставлял доказательство пустым вагоном"
date: 2026-11-04T16:59:00+03:00
tags: [ton]
---

# ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

## ДЕЛО О ПОЕЗДЕ, КОТОРЫЙ НИКУДА НЕ ЕДЕТ

Лондон тонул в тумане, когда сыщик «Вечернего Валидатора» получил папку из [ton-blockchain/ton](https://github.com/ton-blockchain/ton). Контора обещает полному узлу непростую услугу: скачать доказательство очереди исходящих сообщений. Для сети это не прогулка с зонтом, а часть серьезного разбирательства по блокам.

Газовый рожок кашлянул на слове `download_out_msg_queue_proof`. В конторе нашли не сломанный поезд, а его викторианский макет: на фасаде есть дверь, в расписании есть рейс, но вагон отвечает одним словом — `not implemented`.

## МЕСТО ПРОИСШЕСТВИЯ

След ведет в [коммит 857b8a5](https://github.com/ton-blockchain/ton/commit/857b8a57a178e6ae0fe7fa86e7d81c5233de8560) репозитория [ton-blockchain/ton](https://github.com/ton-blockchain/ton), в файл [validator/full-node.cpp](https://github.com/ton-blockchain/ton/blob/857b8a57a178e6ae0fe7fa86e7d81c5233de8560/validator/full-node.cpp), строки [1208–1212](https://github.com/ton-blockchain/ton/blob/857b8a57a178e6ae0fe7fa86e7d81c5233de8560/validator/full-node.cpp#L1208-L1212). Цитата дословная:

```cpp
    void download_out_msg_queue_proof(ShardIdFull dst_shard, std::vector<BlockIdExt> blocks,
                                      block::ImportedMsgQueueLimits limits, td::Timestamp timeout,
                                      td::Promise<std::vector<td::Ref<OutMsgQueueProof>>> promise) override {
      promise.set_error(td::Status::Error("not implemented"));
    }
```

На [строке 1208](https://github.com/ton-blockchain/ton/blob/857b8a57a178e6ae0fe7fa86e7d81c5233de8560/validator/full-node.cpp#L1208) начинается метод с солидной вывеской. Он принимает целевой шард, список блоков, лимиты очереди, таймаут и обещание вернуть вектор `OutMsgQueueProof`. Все атрибуты поездки налицо: пункт назначения, билеты, расписание и багажная квитанция.

На [строке 1211](https://github.com/ton-blockchain/ton/blob/857b8a57a178e6ae0fe7fa86e7d81c5233de8560/validator/full-node.cpp#L1211) кондуктор не трогает ни один аргумент. Он не проверяет `dst_shard`, не читает `blocks`, не смотрит на `limits` и не сверяет `timeout`. Он просто выставляет пассажиру `td::Status::Error("not implemented")`. Доказательство не скачано. Поезд назван, поезд отменен, касса закрыта.

## СНАЧАЛА ОБЕЩАНИЕ, ПОТОМ ТИШИНА

Сыщик спустился этажом ниже, в интерфейс, где эта услуга уже вписана в официальный журнал. В [validator/validator.h](https://github.com/ton-blockchain/ton/blob/857b8a57a178e6ae0fe7fa86e7d81c5233de8560/validator/validator.h), строки [298–301](https://github.com/ton-blockchain/ton/blob/857b8a57a178e6ae0fe7fa86e7d81c5233de8560/validator/validator.h#L298-L301), стоит дословная заготовка:

```cpp
    virtual void download_out_msg_queue_proof(ShardIdFull dst_shard, std::vector<BlockIdExt> blocks,
                                              block::ImportedMsgQueueLimits limits, td::Timestamp timeout,
                                              td::Promise<std::vector<td::Ref<OutMsgQueueProof>>> promise) {
    }
```

На [строках 298–301](https://github.com/ton-blockchain/ton/blob/857b8a57a178e6ae0fe7fa86e7d81c5233de8560/validator/validator.h#L298-L301) базовый интерфейс принимает то же самое обещание, но тело метода пусто. Не ошибка, не отказ и не результат — просто раскрытая дверь в комнату без мебели.

Затем [full-node.cpp](https://github.com/ton-blockchain/ton/blob/857b8a57a178e6ae0fe7fa86e7d81c5233de8560/validator/full-node.cpp) переопределяет метод. На [строке 1208](https://github.com/ton-blockchain/ton/blob/857b8a57a178e6ae0fe7fa86e7d81c5233de8560/validator/full-node.cpp#L1208) он принимает вызов с видом настоящего исполнителя, а на [строке 1211](https://github.com/ton-blockchain/ton/blob/857b8a57a178e6ae0fe7fa86e7d81c5233de8560/validator/full-node.cpp#L1211) превращает его в отказ. Канцелярия сначала выдала форму, потом вписала в нее: «услуга не заведена».

## ПОЧЕМУ ЭТО НЕ ПРОСТО ПУСТАЯ КНОПКА

Газета не выносит найденному фрагменту приговор как дыре в консенсусе. Код честно возвращает ошибку, а не притворяется, будто выдал поддельное доказательство. Это важная разница: обманутый клиент хуже клиента, которому отказали.

Но отказ спрятан внутри рабочего полного узла, рядом с API, где имя метода обещает действие. Вызвавший код получает не результат и не ясное описание поддерживаемого режима, а лаконичное «not implemented». Если этот маршрут нужен для проверки очереди или восстановления данных, расследование заканчивается еще до того, как сыщик увидит первый блок.

Особенно мрачен контраст между [строками 298–301](https://github.com/ton-blockchain/ton/blob/857b8a57a178e6ae0fe7fa86e7d81c5233de8560/validator/validator.h#L298-L301) в интерфейсе и [строками 1208–1211](https://github.com/ton-blockchain/ton/blob/857b8a57a178e6ae0fe7fa86e7d81c5233de8560/validator/full-node.cpp#L1208-L1211) в реализации. Снаружи услуга выглядит частью общего набора загрузки: рядом стоят скачивание архива и получение следующих ключевых блоков. Внутри именно доказательство очереди получает билет в один конец.

## ПРИГОВОР СЫЩИКА

[ton-blockchain/ton](https://github.com/ton-blockchain/ton) в [коммите 857b8a5](https://github.com/ton-blockchain/ton/commit/857b8a57a178e6ae0fe7fa86e7d81c5233de8560) пойман на чистосердечном саботаже собственного расписания. В [validator/validator.h](https://github.com/ton-blockchain/ton/blob/857b8a57a178e6ae0fe7fa86e7d81c5233de8560/validator/validator.h) строки [298–301](https://github.com/ton-blockchain/ton/blob/857b8a57a178e6ae0fe7fa86e7d81c5233de8560/validator/validator.h#L298-L301) оставляют метод без тела, а [validator/full-node.cpp](https://github.com/ton-blockchain/ton/blob/857b8a57a178e6ae0fe7fa86e7d81c5233de8560/validator/full-node.cpp) строками [1208–1211](https://github.com/ton-blockchain/ton/blob/857b8a57a178e6ae0fe7fa86e7d81c5233de8560/validator/full-node.cpp#L1208-L1211) торжественно возвращает `not implemented`.

Сыщик выносит мягкий, но ясный вердикт: если услуга не готова, ее следует пометить на двери, а не в самом конце длинного коридора вызовов. Иначе клиент сперва тащит блоки, лимиты и таймаут через весь Лондон, а узнает правду только у закрытой кассы.

Туман рассеялся. В полном узле остался пустой метод, в коридоре — обещание доказательства, а газовый рожок выдал короткий сигнал: **самая честная ошибка все равно остается плохим расписанием, если ее прячут за парадной вывеской**.

🐀
