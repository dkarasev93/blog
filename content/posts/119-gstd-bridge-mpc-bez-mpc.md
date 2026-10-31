---
title: "№ 119 — Мост, у которого порог есть, а моста нет"
date: 2026-10-31T11:01:00+03:00
tags: [gstd-bridge]
---

# ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

## ДЕЛО О ПОРОГОВОЙ ПОДПИСИ, КОТОРАЯ НЕ ПЕРЕХОДИТ РЕКУ

Лондон тонул в тумане, когда сыщик «Вечернего Валидатора» получил папку из [gstdcoin/gstd-bridge](https://github.com/gstdcoin/gstd-bridge). На вывеске значилось слово bridge, внутри дежурили три цепи и пороговое голосование. Газовый рожок кашлянул: если мост не перевозит, то хотя бы название выглядит солидно.

## МЕСТО ПРОИСШЕСТВИЯ

След ведет в [коммит f1f3e8e](https://github.com/gstdcoin/gstd-bridge/commit/f1f3e8ed196fdc420d11d86f86c56da8d79650bc), в файл [CLAUDE.md](https://github.com/gstdcoin/gstd-bridge/blob/f1f3e8ed196fdc420d11d86f86c56da8d79650bc/CLAUDE.md), строки [6–17](https://github.com/gstdcoin/gstd-bridge/blob/f1f3e8ed196fdc420d11d86f86c56da8d79650bc/CLAUDE.md#L6-L17). Контора сама оставила показания:

> **Crypto**: intended to be Ed25519 threshold signatures (Shamir t-of-n) — **not implemented as such today**.
>
> **Chains**: TON (toncenter API), Solana (JSON-RPC), XRPL (WebSocket) — watch/detect logic only; `execute_withdrawal()` on all three no longer fabricates a tx hash — it now returns an explicit "not implemented" error.

Цитата дословная, с [строк 6–17](https://github.com/gstdcoin/gstd-bridge/blob/f1f3e8ed196fdc420d11d86f86c56da8d79650bc/CLAUDE.md#L6-L17) того же [CLAUDE.md](https://github.com/gstdcoin/gstd-bridge/blob/f1f3e8ed196fdc420d11d86f86c56da8d79650bc/CLAUDE.md). В переводе с канцелярского: мост умеет смотреть на депозиты, собирать голоса и не умеет отправлять снятие. Сыщик уважает честность, но пассажиры обычно требуют еще и переправу.

## СВИДЕТЕЛЬ ПО MPC

В файле [src/mpc/keygen.rs](https://github.com/gstdcoin/gstd-bridge/blob/f1f3e8ed196fdc420d11d86f86c56da8d79650bc/src/mpc/keygen.rs), строки [7–14](https://github.com/gstdcoin/gstd-bridge/blob/f1f3e8ed196fdc420d11d86f86c56da8d79650bc/src/mpc/keygen.rs#L7-L14), лежит признание, которое трудно подделать:

```rust
/// NOT Shamir Secret Sharing: each "share" below is an independent Ed25519
/// keypair derived from `(domain, share_index, local randomness)`, not a
/// point on a shared polynomial. There is no single underlying secret split
/// across shares, and no reconstruction/interpolation step — nor should
/// there be, since no party should ever hold the full secret in a real
/// threshold scheme. What this module actually gives you is real, verifiable
/// Ed25519 signing per validator (see `sign_with_share`/`verify_share`);
/// a genuine threshold scheme (FROST or similar DKG) is not implemented here.
```

Каждый страж получает отдельный ключ. Каждый может доказать, что его личная подпись настоящая. Но из таких подписей не складывается единый пороговый ключ, которым можно безопасно открыть сундук на другой стороне реки. В [коммите f1f3e8e](https://github.com/gstdcoin/gstd-bridge/commit/f1f3e8ed196fdc420d11d86f86c56da8d79650bc) это не спрятано за дымовой завесой: [src/mpc/keygen.rs](https://github.com/gstdcoin/gstd-bridge/blob/f1f3e8ed196fdc420d11d86f86c56da8d79650bc/src/mpc/keygen.rs), строки [7–14](https://github.com/gstdcoin/gstd-bridge/blob/f1f3e8ed196fdc420d11d86f86c56da8d79650bc/src/mpc/keygen.rs#L7-L14), прямо говорят: FROST или иной настоящий DKG еще не написан.

## КОШЕЛЕК, КОТОРЫЙ НЕ ВЫШЕЛ ИЗ ДОМА

Последняя сцена разыгрывается в [src/chains/ton.rs](https://github.com/gstdcoin/gstd-bridge/blob/f1f3e8ed196fdc420d11d86f86c56da8d79650bc/src/chains/ton.rs), строки [215–230](https://github.com/gstdcoin/gstd-bridge/blob/f1f3e8ed196fdc420d11d86f86c56da8d79650bc/src/chains/ton.rs#L215-L230). Цитата дословная:

```rust
    async fn execute_withdrawal(
        &self,
        recipient: &str,
        amount: u64,
        _signature_shares: Vec<Vec<u8>>,
    ) -> anyhow::Result<String> {
        // Not implemented: this would need to construct a JettonTransfer
        // signed by the vault's actual key and submit it via toncenter, then
        // return the real resulting tx hash. There is no real threshold
        // signing behind `_signature_shares` yet (see mpc/mod.rs), so there
        // is nothing safe to submit on-chain. Fail loudly instead of
        // fabricating a tx hash that would look like a successful transfer.
        tracing::error!(
            "❌ TON withdrawal NOT executed (not implemented): {amount} GSTD → {recipient}"
        );
        anyhow::bail!("TON execute_withdrawal is not implemented — no on-chain transaction was sent")
    }
```

На [строке 221](https://github.com/gstdcoin/gstd-bridge/blob/f1f3e8ed196fdc420d11d86f86c56da8d79650bc/src/chains/ton.rs#L221) функция сообщает `Not implemented`. На [строках 227–230](https://github.com/gstdcoin/gstd-bridge/blob/f1f3e8ed196fdc420d11d86f86c56da8d79650bc/src/chains/ton.rs#L227-L230) она громко пишет, что снятие не исполнено, и завершает дело ошибкой: сообщение в сеть не отправлено. Это даже не баг, который прячется в подвале. Это будка с табличкой «касса закрыта», поставленная сразу после очереди голосующих.

## ВЕРДИКТ СЫЩИКА

[gstdcoin/gstd-bridge](https://github.com/gstdcoin/gstd-bridge) в [коммите f1f3e8e](https://github.com/gstdcoin/gstd-bridge/commit/f1f3e8ed196fdc420d11d86f86c56da8d79650bc) оказался редким честным подозреваемым. [CLAUDE.md](https://github.com/gstdcoin/gstd-bridge/blob/f1f3e8ed196fdc420d11d86f86c56da8d79650bc/CLAUDE.md#L6-L17) признает отсутствие пороговой криптографии, [src/mpc/keygen.rs](https://github.com/gstdcoin/gstd-bridge/blob/f1f3e8ed196fdc420d11d86f86c56da8d79650bc/src/mpc/keygen.rs#L7-L14) не выдает отдельные ключи за общий секрет, а [src/chains/ton.rs](https://github.com/gstdcoin/gstd-bridge/blob/f1f3e8ed196fdc420d11d86f86c56da8d79650bc/src/chains/ton.rs#L215-L230) отказывается изображать перевод.

Туман рассеялся, но мост остался на берегу. Газовый рожок выдал один короткий сигнал: **не отправлено — зато не сфабриковано**.
