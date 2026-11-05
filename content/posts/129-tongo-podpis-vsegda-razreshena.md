---
title: "№ 129 — Кошелек, которому навсегда разрешили подпись"
date: 2026-11-05T10:59:00+03:00
tags: [tongo]
---

# ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

## ДЕЛО О ВЕЧНОМ РАЗРЕШЕНИИ

Лондон просыпался под туманом, когда газовый рожок у редакции принес папку из [tonkeeper/tongo](https://github.com/tonkeeper/tongo). Контора добавила поддержку Wallet V5 R1 — кошелька, где у состояния есть отдельный флаг, разрешена ли подпись. Флаг выглядит как замок на двери: его можно открыть, можно закрыть, а иногда следует спросить владельца.

Сыщик открыл [коммит 0fe589d](https://github.com/tonkeeper/tongo/commit/0fe589df866d50e8a732aadc44a5ca4d5d83c6f6) и прошел в [wallet/wallet_v5.go](https://github.com/tonkeeper/tongo/blob/0fe589df866d50e8a732aadc44a5ca4d5d83c6f6/wallet/wallet_v5.go). На двери висела короткая записка: замок есть, но ключ у клерка всегда повернут в одну сторону.

## МЕСТО ПРОИСШЕСТВИЯ

В [wallet/wallet_v5.go](https://github.com/tonkeeper/tongo/blob/0fe589df866d50e8a732aadc44a5ca4d5d83c6f6/wallet/wallet_v5.go) строки [12–17](https://github.com/tonkeeper/tongo/blob/0fe589df866d50e8a732aadc44a5ca4d5d83c6f6/wallet/wallet_v5.go#L12-L17) описывают состояние кошелька:

```go
type DataV5R1 struct {
	IsSignatureAllowed bool
	Seqno              uint32
	WalletID           uint32
	PublicKey          tlb.Bits256
	Extensions         tlb.HashmapE[tlb.Bits256, tlb.Uint1]
}
```

Поле `IsSignatureAllowed` не декоративно: оно записывается в данные кошелька рядом с номером, идентификатором и ключом. По смыслу это настоящий переключатель режима, а не подпись на полях дела.

Но в конструкторе на [строках 49–61](https://github.com/tonkeeper/tongo/blob/0fe589df866d50e8a732aadc44a5ca4d5d83c6f6/wallet/wallet_v5.go#L49-L61) сыщик нашел следующее:

```go
func NewWalletV5R1(publicKey ed25519.PublicKey, opts Options) *walletV5R1 {
	workchain := defaultOr(opts.Workchain, 0)
	subWalletID := defaultOr(opts.SubWalletID, 0)
	networkGlobalID := int64(defaultOr[int32](opts.NetworkGlobalID, MainnetGlobalID))
	contextID := int64(genContextID(uint32(workchain), subWalletID))
	walletID := contextID ^ networkGlobalID

	return &walletV5R1{
		publicKey:          publicKey,
		workchain:          workchain,
		walletID:           uint32(walletID),
		isSignatureAllowed: true, // todo: add option to disable signature
	}
}
```

Цитата дословная. Состояние умеет хранить запрет, но новый кошелек получает `true` без вопроса, без параметра и без запасного пути. Комментарий `todo: add option to disable signature` честно признается: ручку запрета еще не установили, зато табличка «разрешено» уже прибита к двери.

## ЗАМОК, КОТОРЫЙ НЕ СПРАШИВАЕТ

Дальше флаг не остается в записной книжке. В [wallet/wallet_v5.go](https://github.com/tonkeeper/tongo/blob/0fe589df866d50e8a732aadc44a5ca4d5d83c6f6/wallet/wallet_v5.go) строки [71–78](https://github.com/tonkeeper/tongo/blob/0fe589df866d50e8a732aadc44a5ca4d5d83c6f6/wallet/wallet_v5.go#L71-L78) кладут его в состояние:

```go
func (w *walletV5R1) generateStateInit() (*tlb.StateInit, error) {
	data := DataV5R1{
		IsSignatureAllowed: w.isSignatureAllowed,
		Seqno:              0,
		WalletID:           w.walletID,
		PublicKey:          publicKeyToBits(w.publicKey),
	}
	return generateStateInit(V5R1, data)
}
```

На [строке 73](https://github.com/tonkeeper/tongo/blob/0fe589df866d50e8a732aadc44a5ca4d5d83c6f6/wallet/wallet_v5.go#L73) значение из конструктора отправляется прямо в состояние. Следовательно, это не забытая переменная и не мертвый комментарий: каждый кошелек, созданный через этот конструктор, получает разрешение на подпись как часть своего адресообразующего состояния.

В [wallet/wallet_v5.go](https://github.com/tonkeeper/tongo/blob/0fe589df866d50e8a732aadc44a5ca4d5d83c6f6/wallet/wallet_v5.go) строки [93–116](https://github.com/tonkeeper/tongo/blob/0fe589df866d50e8a732aadc44a5ca4d5d83c6f6/wallet/wallet_v5.go#L93-L116) контора уже создает и подписывает сообщение. Сам по себе этот метод не доказывает обход проверки: он лишь показывает, что новый экземпляр способен выпускать подпись. Доказано другое и только другое: параметр, который должен решать судьбу флага, пока отсутствует, а значение в состоянии всегда выставлено в `true`.

## НЕ ПРИГОВОР, А ОТПЕЧАТОК ПАЛЬЦА

Газета не станет называть это доказанной дырой в контракте. В коде нет строки, где `false` отвергается, и нет проверки сети, которую сыщик мог бы назвать сломанной. Возможно, текущая версия Wallet V5 R1 намеренно стартует только с разрешенной подписью.

Но тогда в деле остается странная декорация: тип состояния уже содержит `IsSignatureAllowed`, а публичный конструктор не дает его настроить. Комментарий на [строке 60](https://github.com/tonkeeper/tongo/blob/0fe589df866d50e8a732aadc44a5ca4d5d83c6f6/wallet/wallet_v5.go#L60) обещает будущую возможность отключения, пока API делает вид, будто выбора не существует. Это не приговор кошельку, а точный отпечаток незавершенного интерфейса.

## ПРИГОВОР СЫЩИКА

В [коммите 0fe589d](https://github.com/tonkeeper/tongo/commit/0fe589df866d50e8a732aadc44a5ca4d5d83c6f6) контора [tonkeeper/tongo](https://github.com/tonkeeper/tongo) принесла кошелек с флагом разрешения подписи, но выдала ему вечное «да». В [wallet/wallet_v5.go](https://github.com/tonkeeper/tongo/blob/0fe589df866d50e8a732aadc44a5ca4d5d83c6f6/wallet/wallet_v5.go) строки [49–61](https://github.com/tonkeeper/tongo/blob/0fe589df866d50e8a732aadc44a5ca4d5d83c6f6/wallet/wallet_v5.go#L49-L61) создают значение `true`, а комментарий тут же просит когда нибудь добавить возможность сказать `false`.

Туман поднялся над Темзой. Газовый рожок выдал короткий сигнал: **если замок уже вписан в паспорт кошелька, не оставляйте ключ повернутым навсегда и не прячьте запасной ключ в комментарии**.

🐀
