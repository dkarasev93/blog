+++
title = "№ 111 — Газовый запас, который считали на глаз"
date = 2026-10-26T16:59:00+03:00
tags = ["ton-payment-network"]
+++

# ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

**№ 111.** *Лондон. Над Темзой висит туман, газовый рожок кашляет в переулке, а сыщик получает из [xssnick/ton-payment-network](https://github.com/xssnick/ton-payment-network) счет на активацию канала. Счет старый, круглый и слишком уверенный в себе.*

## ДЕЛО О ЧЕТВЕРТИ МОНЕТЫ

След ведет в коммит [f7e6b34](https://github.com/xssnick/ton-payment-network/commit/f7e6b34d4efa64f1adac49b9c81a8364927ab021) репозитория [xssnick/ton-payment-network](https://github.com/xssnick/ton-payment-network). В его папке лежит файл [tonpayments/api.go](https://github.com/xssnick/ton-payment-network/blob/f7e6b34d4efa64f1adac49b9c81a8364927ab021/tonpayments/api.go), а в строках [2193–2198](https://github.com/xssnick/ton-payment-network/blob/f7e6b34d4efa64f1adac49b9c81a8364927ab021/tonpayments/api.go#L2193-L2198) — расчет платы перед пополнением канала.

Код выглядит как честная бухгалтерия:

```go
activationAmount := big.NewInt(0)
if !channel.Our.ActiveOnchain {
	activationAmount.Mul(tlb.MustFromTON(s.cfg.ReplicationMessageAttachAmount).Nano(), big.NewInt(4))
	activationAmount.Add(activationAmount, tlb.MustFromTON("0.05").Nano())
}
amtTonFeeToCheck := tlb.FromNanoTON(new(big.Int).Add(tlb.MustFromTON("0.09").Nano(), activationAmount))
```

Цитата дословная, из [строк 2193–2198](https://github.com/xssnick/ton-payment-network/blob/f7e6b34d4efa64f1adac49b9c81a8364927ab021/tonpayments/api.go#L2193-L2198). Сначала контора заводит нулевой мешок для газа. Потом, если канал еще не живет в сети, умножает настроечную сумму на четыре и прибавляет еще `0.05`. После этого к результату пристегивается `0.09`.

## СТАРЫЙ СЛЕД В ТУМАНЕ

В предыдущем расчете, который этот коммит выносит из дела, активация была записана как плоские `0.25`, а проверочная плата начиналась с `0.09`. Теперь четыре умножения отданы конфигурации, но сыщик не может подменить дословную улику догадкой: на указанном коммите перед ним именно `big.NewInt(0)`, четыре и `0.05`.

Суть находки не в том, что число четыре само по себе преступно. Суть в театре чисел. Сумма для активации живет в настройке, затем ее усиливают в четыре раза, затем сверху кладут еще запас, а рядом остается отдельная девятисоттысячная? Нет, `0.09` TON — простая плата за проверку. Газовый счетчик так долго менял вывески, что посетитель уже не понимает, где тариф, где запас, а где плата за то, что его вообще впустили в контору.

## ГДЕ СЫЩИК НАШЕЛ ИГОЛКУ

Проверка кошелька для TON берет сумму перевода и добавляет к ней рассчитанную плату — [строки 2200–2207](https://github.com/xssnick/ton-payment-network/blob/f7e6b34d4efa64f1adac49b9c81a8364927ab021/tonpayments/api.go#L2200-L2207). Для другого актива код отдельно проверяет газовый запас и баланс самого актива — [строки 2209–2216](https://github.com/xssnick/ton-payment-network/blob/f7e6b34d4efa64f1adac49b9c81a8364927ab021/tonpayments/api.go#L2209-L2216).

А затем, в [строках 2239–2243](https://github.com/xssnick/ton-payment-network/blob/f7e6b34d4efa64f1adac49b9c81a8364927ab021/tonpayments/api.go#L2239-L2243), та же сумма отправляется на развертывание или активацию:

```go
// deploy a contract or activate it
messages = append(messages, WalletMessage{
	Amount:    tlb.FromNanoTON(activationAmount),
	Body:      channel.InitMessageBody,
	StateInit: state,
})
```

Комментарий обещает простую сцену: «развернуть контракт или активировать его». Но за кулисами уже прошли четыре умножения и две надбавки. Сыщик не считает это доказанным ограблением: точный размер `ReplicationMessageAttachAmount` зависит от настроек, а корректность суммы требует запуска сценариев. Зато бухгалтерский почерк сочный: жесткий `0.25` ушел, а вместо него появился маленький каскад, который надо распутывать по конфигурации.

## ПРИГОВОР СЫЩИКА

Репозиторий [xssnick/ton-payment-network](https://github.com/xssnick/ton-payment-network) в [коммите f7e6b34](https://github.com/xssnick/ton-payment-network/commit/f7e6b34d4efa64f1adac49b9c81a8364927ab021) пойман не на красивой формуле, а на ее смене. В [tonpayments/api.go](https://github.com/xssnick/ton-payment-network/blob/f7e6b34d4efa64f1adac49b9c81a8364927ab021/tonpayments/api.go) один счетчик обслуживает и проверку кошелька, и реальное сообщение активации, а его величина собирается из четырехкратной настройки, `0.05` и `0.09`.

Приговор мягкий, но звонкий: газовый запас нельзя оценивать по силуэту числа. Если сумма берется из настройки, умножается и получает еще одну надбавку, каждая часть должна иметь имя, смысл и тест на границе. Иначе в тумане легко принять запас за тариф, тариф за гарантию, а четверть монеты — за вечный закон природы.

*Цитаты дословные, строки сверены с указанным коммитом.*

🐀
