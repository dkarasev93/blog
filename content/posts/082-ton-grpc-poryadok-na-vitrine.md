+++
title = "№ 82 — Порядок на витрине, а в подвале его нет"
date = 2026-09-25T11:00:00+03:00
description = "Восемьдесят второй выпуск «Вечернего Валидатора»: gRPC-метод принимает order, честно читает его, а затем отправляет все транзакции одним и тем же строем."
tags = ["ton-grpc"]
+++

# ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

**№ 82.** *Лондон. Туман ползет по мостовой, газовый рожок у редакции хрипит, а сыщик получил папку из свежей конторы [getgems-io/ton-grpc](https://github.com/getgems-io/ton-grpc). На обложке написано `GetTransactions`: клиент просит порядок, сервер кивает и выдает ему один и тот же порядок при любом запросе.*

## МЕСТО ПРОИСШЕСТВИЯ

Улика найдена в репозитории [getgems-io/ton-grpc](https://github.com/getgems-io/ton-grpc), в коммите [`ff10ed5`](https://github.com/getgems-io/ton-grpc/commit/ff10ed522aba0c8bb3776ae23d7270e6ad8bf080), в файле [`crates/ton-grpc/src/block.rs`](https://github.com/getgems-io/ton-grpc/blob/ff10ed522aba0c8bb3776ae23d7270e6ad8bf080/crates/ton-grpc/src/block.rs). Коммит носит чинное имя `feat: ton-emulator (#1724)`; цитаты сняты с этого снимка.

В контракте протокола для `GetTransactionsRequest` предусмотрен выбор строя. В [`proto/ton.proto`](https://github.com/getgems-io/ton-grpc/blob/ff10ed522aba0c8bb3776ae23d7270e6ad8bf080/proto/ton.proto) строки [227–235](https://github.com/getgems-io/ton-grpc/blob/ff10ed522aba0c8bb3776ae23d7270e6ad8bf080/proto/ton.proto#L227-L235) говорят буквально:

```protobuf
message GetTransactionsRequest {
  enum Order {
    UNORDERED = 0;
    ASC = 1;
//    DESC = 2;
  }
  BlockId block_id = 1;
  Order order = 2;
}
```

Покупателю обещаны как минимум два режима: `UNORDERED` и `ASC`. В проекте даже оставлена черновая строка для `DESC`, будто третий строй уже стучится в дверь.

## СЕНСАЦИЯ: ПАРАД, КОТОРЫЙ НЕ СЛЫШИТ КОМАНДУ

Метод `get_transactions` находится в файле [`crates/ton-grpc/src/block.rs`](https://github.com/getgems-io/ton-grpc/blob/ff10ed522aba0c8bb3776ae23d7270e6ad8bf080/crates/ton-grpc/src/block.rs), строки [142–166](https://github.com/getgems-io/ton-grpc/blob/ff10ed522aba0c8bb3776ae23d7270e6ad8bf080/crates/ton-grpc/src/block.rs#L142-L166). Улика дословно:



```rust
    async fn get_transactions(
        &self,
        request: Request<GetTransactionsRequest>,
    ) -> Result<Response<Self::GetTransactionsStream>, Status> {
        let msg = request.into_inner();

        // TODO[akostylev0]
        let _order = msg.order();
        let block_id = msg
            .block_id
            .context("block id is required")
            .map_err(|e| Status::internal(e.to_string()))?;

        let mut client = self.client.clone();
        let block_id = extend_block_id(&mut client, &block_id)
            .await
            .map_err(|e: anyhow::Error| Status::internal(e.to_string()))?;

        let stream = client
            .get_block_tx_stream(&block_id, false)
            .map_ok(|tx| tx.into())
            .map_err(|e| Status::internal(e.to_string()))
            .boxed();

        Ok(Response::new(stream))
    }
```

Надпись `let _order = msg.order();` на строке [149](https://github.com/getgems-io/ton-grpc/blob/ff10ed522aba0c8bb3776ae23d7270e6ad8bf080/crates/ton-grpc/src/block.rs#L149) выглядит как допрос, но это лишь регистрация показаний. Значение тут же превращают в переменную с нижним подчеркиванием, а дальше не передают никуда.

Финальная команда, строки [160–164](https://github.com/getgems-io/ton-grpc/blob/ff10ed522aba0c8bb3776ae23d7270e6ad8bf080/crates/ton-grpc/src/block.rs#L160-L164), всегда одна. В ней аргумент `false` не спрашивает, чего желал клиент:

```rust
        let stream = client
            .get_block_tx_stream(&block_id, false)
            .map_ok(|tx| tx.into())
            .map_err(|e| Status::internal(e.to_string()))
            .boxed();
```

Аргумент `false` просто приказывает получить поток в одном фиксированном режиме. `order` принят на входе, прочитан, назван и оставлен в прихожей.

## ВТОРОЙ СЛЕД: СОСЕДНИЙ МЕТОД УМЕЕТ СЛУШАТЬ

Сцена особенно неловка потому, что в том же файле соседний метод `get_transaction_ids` работает иначе. Он получает порядок на строке [91](https://github.com/getgems-io/ton-grpc/blob/ff10ed522aba0c8bb3776ae23d7270e6ad8bf080/crates/ton-grpc/src/block.rs#L91), а затем на строках [102–106](https://github.com/getgems-io/ton-grpc/blob/ff10ed522aba0c8bb3776ae23d7270e6ad8bf080/crates/ton-grpc/src/block.rs#L102-L106) выбирает разный вызов:

```rust
        let stream = match order {
            Order::Unordered => client.get_block_tx_stream_unordered(&block_id).boxed(),
            Order::Asc => client.get_block_tx_id_stream(&block_id, false).boxed(),
            Order::Desc => client.get_block_tx_id_stream(&block_id, true).boxed(),
        };
```

Иными словами, дверь не заколочена по технической необходимости. Один служащий умеет читать приказ `Order`, соседний служащий делает вид, что приказа не было. В одной комнате клиент получает выбор, в другой — парад по расписанию.

## ВЕРДИКТ СЫЩИКА

Репозиторий [getgems-io/ton-grpc](https://github.com/getgems-io/ton-grpc) пойман на очень чистой сцене: схема обещает параметр `order`, метод его извлекает, а затем вызывает `get_block_tx_stream` с постоянным `false`. Коммит [`ff10ed5`](https://github.com/getgems-io/ton-grpc/commit/ff10ed522aba0c8bb3776ae23d7270e6ad8bf080) и файл [`crates/ton-grpc/src/block.rs`](https://github.com/getgems-io/ton-grpc/blob/ff10ed522aba0c8bb3776ae23d7270e6ad8bf080/crates/ton-grpc/src/block.rs) оставили улику на виду: [`let _order = msg.order();`](https://github.com/getgems-io/ton-grpc/blob/ff10ed522aba0c8bb3776ae23d7270e6ad8bf080/crates/ton-grpc/src/block.rs#L149).

Редакция не утверждает, что каждый клиент уже получил неверный результат: точный смысл `false` задает библиотека ниже по стеку. Но одно видно без лупы: параметр порядка в этом gRPC-методе не влияет на выбранный вызов. Клиент просит строй, сервер берет один и тот же строй из шкафа.

Газовый рожок дернулся, туман сгустился. На двери осталась надпись из протокола: `// TODO[akostylev0]`. Сыщик поставил рядом свою: **«Приказ принят. Приказ не исполнен»**.

🐀
