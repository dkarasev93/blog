+++
title = "№ 32 — Сыскное бюро «Тонутилс и компания»: сид от настоящего сейфа в ящике стола, записка «Меня зовут Евгений» в качестве улики и заклинание 0b00010001 с пометкой «проверить бы эту магию»"
date = 2026-08-29T17:12:00+03:00
description = "Тридцать второй выпуск «Вечернего Валидатора»: сыщик впервые входит в контору xssnick/tonutils-go — главное Go-бюро города. В ящике стола найден настоящий сид от боевого кошелька, которым клерки на каждый коммит ходят в живой город и тратят настоящие деньги; в картотеке лежит расшифрованная чужая записка «Меня зовут Евгений», вшитая в тест как эталон; адресная палата чеканит байт 0b00010001 с честной пометкой «TODO check this magic...» четвёртый год; зал манекенов кричит «implement me» тридцатью голосами; а комната перекрёстного допроса Go-машины против эталонной TVM исключена из обхода коммитом «No cross-test tvm in CI»."
tags = ["tonutils-go"]
+++

# 📰 Вечерний Валидатор

**Выпуск № 32.** *Лондон. Туман нынче с привкусом кофе и чужих мнемоник. Сыщик нашей газеты, пройдя мимо спящей типографии (выпуск № 31), кузницы (спецвыпуск) и чертежного бюро (выпуск № 29), постучал в контору, в которой до сей поры не бывал, — в сыскное бюро [xssnick/tonutils-go](https://github.com/xssnick/tonutils-go), главное заведение города по розыску блоков, ячеек и транзакций средствами языка Go. Контора живая, клиентов у неё пол-Лондона: через её сыщиков ходят к сети биржи, боты и половина бэкендов. Дверь открыта, газовый рожок горит, вершина летописи — коммит [`749603a`](https://github.com/xssnick/tonutils-go/commit/749603ab237d058cac5be4c42d36a52064db8b58), июль сего года. Сыщик представился, попросил показать, как контора проверяет собственную работу, — и замер. Потому что проверяет контора свою работу... на настоящем городе. С настоящими деньгами.*

*Место осмотра: репозиторий [xssnick/tonutils-go](https://github.com/xssnick/tonutils-go), коммит [`749603a`](https://github.com/xssnick/tonutils-go/commit/749603ab237d058cac5be4c42d36a52064db8b58) (master). Файлы: [`.github/workflows/go.yml`](https://github.com/xssnick/tonutils-go/blob/749603ab237d058cac5be4c42d36a52064db8b58/.github/workflows/go.yml), [`ton/wallet/integration_test.go`](https://github.com/xssnick/tonutils-go/blob/749603ab237d058cac5be4c42d36a52064db8b58/ton/wallet/integration_test.go), [`ton/wallet/wallet_test.go`](https://github.com/xssnick/tonutils-go/blob/749603ab237d058cac5be4c42d36a52064db8b58/ton/wallet/wallet_test.go), [`address/addr.go`](https://github.com/xssnick/tonutils-go/blob/749603ab237d058cac5be4c42d36a52064db8b58/address/addr.go). Все цитаты дословные.*

---

## ДЕЛО ПЕРВОЕ: СИД В ЯЩИКЕ СТОЛА

У всякой порядочной конторы есть испытательный полигон — манекены, меловые контуры, муляжи. У этой конторы полигона нет. Есть сейф конторы GitHub Actions, а в сейфе — строки [33–37](https://github.com/xssnick/tonutils-go/blob/749603ab237d058cac5be4c42d36a52064db8b58/.github/workflows/go.yml#L33-L37) ведомости [`go.yml`](https://github.com/xssnick/tonutils-go/blob/749603ab237d058cac5be4c42d36a52064db8b58/.github/workflows/go.yml):

```yaml
    - name: Run Test
      env:
        WALLET_SEED: ${{ secrets.WALLET_SEED }}
      run: |
        go test -v -failfast $(go list ./... | grep -v /example/ | grep -v /tvm/vm/cross-emulate-test) -covermode=count -coverprofile=coverage.out
```

Читатель, оцените ситуацию. В секретах репозитория хранится **настоящая мнемоническая фраза от настоящего кошелька**. На каждый пуш и на каждый пул-реквест контора достаёт её из сейфа и отправляет клерков в город. Не в макет города — в настоящий: [`integration_test.go`](https://github.com/xssnick/tonutils-go/blob/749603ab237d058cac5be4c42d36a52064db8b58/ton/wallet/integration_test.go), строки [40–52](https://github.com/xssnick/tonutils-go/blob/749603ab237d058cac5be4c42d36a52064db8b58/ton/wallet/integration_test.go#L40-L52), поднимает клиент к мейннету по адресу `global.config.json`, а строки [26–38](https://github.com/xssnick/tonutils-go/blob/749603ab237d058cac5be4c42d36a52064db8b58/ton/wallet/integration_test.go#L26-L38) — к тестнету, и оба инициализатора завершаются `panic(err)` на случай, если город не отвечает: упал сайт с конфигом — упал весь испытательный корпус, ещё до первого допроса.

Дальше — бухгалтерия ужаса. Строка [54](https://github.com/xssnick/tonutils-go/blob/749603ab237d058cac5be4c42d36a52064db8b58/ton/wallet/integration_test.go#L54): `var _seed = os.Getenv("WALLET_SEED")`, а следом протокольная константа, строка [24](https://github.com/xssnick/tonutils-go/blob/749603ab237d058cac5be4c42d36a52064db8b58/ton/wallet/integration_test.go#L24):

```go
const emptyWalletSeedEnvFatalMsg = "WALLET_SEED not found in environment"
```

И начинается обход: [`Test_HighloadHeavyTransfer`](https://github.com/xssnick/tonutils-go/blob/749603ab237d058cac5be4c42d36a52064db8b58/ton/wallet/integration_test.go#L56), [`Test_V5HeavyTransfer`](https://github.com/xssnick/tonutils-go/blob/749603ab237d058cac5be4c42d36a52064db8b58/ton/wallet/integration_test.go#L91), [`Test_WalletTransfer`](https://github.com/xssnick/tonutils-go/blob/749603ab237d058cac5be4c42d36a52064db8b58/ton/wallet/integration_test.go#L122), [`TestWallet_TransferEncrypted`](https://github.com/xssnick/tonutils-go/blob/749603ab237d058cac5be4c42d36a52064db8b58/ton/wallet/integration_test.go#L402) — «тяжёлые переводы», «переводы с шифрованием» — всё это, читатель, не чтение с полок, а **настоящие исходящие транзакции с настоящего кошелька**, которые клерки подписывают сидом из сейфа и отправляют в живую сеть при каждом прогоне. Фраза появилась в конторе 23 августа 2024 года (коммит [`f7d939d`](https://github.com/xssnick/tonutils-go/commit/f7d939d34943a90531d049b6f17e5835fb7adce9)) и ходит по городу до сих пор. Газ, комиссии, баланс — всё настоящее. Единственная подставная вещь во всей конторе — манекены, но о них ниже.

## ДЕЛО ВТОРОЕ: ЗАПИСКА «МЕНЯ ЗОВУТ ЕВГЕНИЙ»

Теперь — кабинет тайнописи. [`wallet_test.go`](https://github.com/xssnick/tonutils-go/blob/749603ab237d058cac5be4c42d36a52064db8b58/ton/wallet/wallet_test.go), строки [698–733](https://github.com/xssnick/tonutils-go/blob/749603ab237d058cac5be4c42d36a52064db8b58/ton/wallet/wallet_test.go#L698-L733), дознание под именем `TestDecryptCommentCell`. Сыщик читает протокол и не верит глазам. Строка [712](https://github.com/xssnick/tonutils-go/blob/749603ab237d058cac5be4c42d36a52064db8b58/ton/wallet/wallet_test.go#L712) честно объявляет происхождение улики:

```go
	// boc generated by wallet app
	h, _ := hex.DecodeString("b5ee9c7201010201007900014e2167da4bd9c640a0f62d96154af8b62dd5385d2efb8cc954ca980f03ac4647b451e70001b574b701009ac76e8acd0445b634a43486667942fc0274d450a93bfebb7403be0ca9ebbe8a5d1027353c8e18675578cca00473665ccd0587f6bb883017170bda699894d543a692af35e99ce1ae47090bc40841")
```

Это, читатель, **настоящее зашифрованное сообщение**, вырезанное из настоящей транзакции, сгенерированное настоящим кошельком-приложением. Дознаватель берёт сид из сейфа, идёт в город за публичным ключом адресата (строка [719](https://github.com/xssnick/tonutils-go/blob/749603ab237d058cac5be4c42d36a52064db8b58/ton/wallet/wallet_test.go#L719), адрес `kQC1-MwWMs8i...` выписан в протоколе дословно), вскрывает записку — и сверяет содержимое с эталоном. Строки [731–733](https://github.com/xssnick/tonutils-go/blob/749603ab237d058cac5be4c42d36a52064db8b58/ton/wallet/wallet_test.go#L731-L733):

```go
	if string(data) != "Меня зовут Евгений" {
		t.Fatal("incorrect content")
	}
```

Остановитесь. Вдумайтесь. В картотеке публичной конторы, открытой всему Лондону, лежит **расшифрованное личное послание** — «Меня зовут Евгений» — вшитое в тест как контрольный ответ. Зашифрованный комментарий к чужому переводу стал учебным пособием: всякий, кто клонирует репозиторий, получает в придачу и шифртекст, и открытый текст, и адрес адресата. Записка занесена в протокол 4 декабря 2024 года (коммит [`54d1699`](https://github.com/xssnick/tonutils-go/commit/54d169926572eb0e27678fa5dbcc09eef3a46add), «Address normalization when encrypt comment + load keys as snake too») — и с тех пор Евгений представляется каждому, кто запускает тесты. Шифрование, конечно, исправно: без сида записку не прочесть. Но сам факт, читатель: контора проверяет замки, ежедневно вскрывая одно и то же письмо незнакомца. Евгений, если вы это читаете, — ваша записка в надёжных руках. В очень многих надёжных руках.

## ДЕЛО ТРЕТЬЕ: ЗАКЛИНАНИЕ 0b00010001

Адресная палата конторы — [`addr.go`](https://github.com/xssnick/tonutils-go/blob/749603ab237d058cac5be4c42d36a52064db8b58/address/addr.go) — чеканит байт-флаг, которым адреса отличают отскакивающие письма от неотскакивающих, тестовые от боевых. Строки [219–228](https://github.com/xssnick/tonutils-go/blob/749603ab237d058cac5be4c42d36a52064db8b58/address/addr.go#L219-L228):

```go
func (a *Address) FlagsToByte() (flags byte) {
	// TODO check this magic...
	flags = 0b00010001
	if !a.flags.bounceable {
		setBit(&flags, 6)
	}
	if a.flags.testnet {
		setBit(&flags, 7)
	}
	return flags
}
```

«TODO check this magic...» — «надо бы проверить эту магию». Пометка висит над заклинанием с **6 мая 2022 года** — редакция сняла дактилоскопию: коммит [`a2e0c4c`](https://github.com/xssnick/tonutils-go/commit/a2e0c4ca0db1fb35b41bb916333ff06386344a2f), чьё имя само по себе является признанием: «parse address from cell; parse address flags; address manipulation methods; address string(); **ugly address tests**;». Мастер сам назвал свои тесты уродливыми, вписал бинарные руны `0b00010001` с многоточием сомнения — и ушёл. Заклинание, по справедливости, действует: `0b00010001` плюс шестой бит дают `0x51`, неотскакивающий адрес, как велит стандарт. Но четыре года, читатель, четыре года магия ждёт проверки, а между тем этот байт чеканит каждый перевод половины Go-города. Заклинание работает — но никто так и не решился снять с него табличку «не проверено». Наш сыщик уже видел похожее суеверие в спящей типографии (выпуск № 31, заклинание «dont work»): у магии в этом городе долгая память и короткие руки проверяющих.

## ДЕЛО ЧЕТВЁРТОЕ: ЗАЛ МАНЕКЕНОВ

В подвале конторы обнаружен зал манекенов — `MockAPI` и `WaiterMock`, чучела клиентов сети, на которых клерки отрабатывают приёмы. Чучела устроены своеобразно: у каждого по тридцать с лишним методов, и почти каждый метод — капкан. [`wallet_test.go`](https://github.com/xssnick/tonutils-go/blob/749603ab237d058cac5be4c42d36a52064db8b58/ton/wallet/wallet_test.go), строки [40–42](https://github.com/xssnick/tonutils-go/blob/749603ab237d058cac5be4c42d36a52064db8b58/ton/wallet/wallet_test.go#L40-L42):

```go
func (m MockAPI) FindLastTransactionByInMsgHash(ctx context.Context, addr *address.Address, msgHash []byte, maxTxNumToScan ...int) (*tlb.Transaction, error) {
	//TODO implement me
	panic("implement me")
}
```

Сыщик прошёл зал с фонарём и насчитал три десятка одинаковых надписей: строки [518–572](https://github.com/xssnick/tonutils-go/blob/749603ab237d058cac5be4c42d36a52064db8b58/ton/wallet/wallet_test.go#L518-L572) — двенадцать капканов подряд с деловой пометкой «//TODO implement me», строки [651–695](https://github.com/xssnick/tonutils-go/blob/749603ab237d058cac5be4c42d36a52064db8b58/ton/wallet/wallet_test.go#L651-L695) — ещё десяток уже без пометок, голым `panic("implement me")`. Прикоснулся к манекену не с той стороны — зал взрывается паникой. Это, разумеется, заводской бракодел от станка GoLand: он штампует заглушки интерфейсов именно с этой эпитафией. Но в конторе, где полигоном служит настоящий город, а сейфом — настоящий сид, даже манекены обвешаны гремучими капканами с лаконичной надписью «воплоти меня». Картина маслом: живые деньги ходят по живому городу, а муляжи — кричат.

## ДЕЛО ПЯТОЕ: ЗАКРЫТАЯ КОМНАТА ПЕРЕКРЁСТНЫХ ДОПРОСОВ

И наконец — самое тихое крыло конторы. В нём стоит установка перекрёстного допроса: [`tvm/vm/cross-emulate-test`](https://github.com/xssnick/tonutils-go/tree/749603ab237d058cac5be4c42d36a52064db8b58/tvm/vm/cross-emulate-test), где собственная Go-машина виртуальных вычислений конторы сверяется с эталонной TVM — та самая машина, что исполняет контракты и считает газ. Установка на месте, приборы пылятся. А в ведомости [`go.yml`](https://github.com/xssnick/tonutils-go/blob/749603ab237d058cac5be4c42d36a52064db8b58/.github/workflows/go.yml), строки [24](https://github.com/xssnick/tonutils-go/blob/749603ab237d058cac5be4c42d36a52064db8b58/.github/workflows/go.yml#L24) и [37](https://github.com/xssnick/tonutils-go/blob/749603ab237d058cac5be4c42d36a52064db8b58/.github/workflows/go.yml#L37), обходному клерку предписано:

```yaml
      run: go build -v $(go list ./... | grep -v /tvm/vm/cross-emulate-test)
...
        go test -v -failfast $(go list ./... | grep -v /example/ | grep -v /tvm/vm/cross-emulate-test) -covermode=count -coverprofile=coverage.out
```

Двойное `grep -v` — деликатное, почти фамильярное исключение: комнату перекрёстных допросов не строить и в неё не входить. Приказ датирован 11 мая 2026 года и озаглавлен без околичностей: «No cross-test tvm in CI» (коммит [`b36e00e`](https://github.com/xssnick/tonutils-go/commit/b36e00e9406e479679f1b31c98004a6c40da2575)). Поймите правильно, читатель: настоящие переводы в настоящий город — на каждый пуш, пожалуйста, с настоящим сидом; а сверку собственной виртуальной машины с эталоном — не надо, уберите от греха. Контора тратит газ в мейннете, чтобы проверить кошелёк, но не проверяет в конвейере машину, которая этот газ считает. Приоритеты сыска, ничего не попишешь.

## РЕЗЮМЕ СЫЩИКА

Контора [xssnick/tonutils-go](https://github.com/xssnick/tonutils-go) — заведение полезное и почтенное, и сыщик уходит из него с уважением и блокнотом, полным записей:

1. Сид от настоящего кошелька хранится в секретах CI, и клерки тратят с него настоящие средства на каждый коммит — полигоном служит живой город.
2. Расшифрованное личное послание «Меня зовут Евгений» — вшито в тест как эталон, рядом с шифртекстом и адресом адресата.
3. Байт-заклинание `0b00010001` четвёртый год носит пометку «проверить бы эту магию» — и чеканит адреса половине экосистемы.
4. Тридцать манекенов в подвале кричат «implement me» — единственные в конторе, кто не работает по-настоящему.
5. Комната сверки с эталонной TVM исключена из обхода одним изящным `grep -v`.

Мораль, читатель, стара как Лондон: город любит конторы, где всё настоящее. Настоящие деньги, настоящие письма, настоящая магия. Проверяйте свои заклинания, храните сиды в сейфах понадёжнее — и если ваш тест однажды ответит «incorrect content», помните: где-то в тумане Евгений так и не представился заново.

🐀
