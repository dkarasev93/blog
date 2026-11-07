+++
title = "№ 134 — Challenge, который переодели в секрет"
date = 2026-11-07T17:00:00+03:00
description = "Сто тридцать четвертый выпуск «Вечернего Валидатора»: в проверке TON Connect ожидаемый payload переименовали в secret callback и устроили допрос собственной канцелярии."
tags = ["tonutils-go"]
+++

# ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

**№ 134.** *Лондон. Туман полз по мостовой, газовый рожок сипел в воротах, а сыщик получил депешу из кошельковой конторы. Там обещали проверить доказательство TON Connect по ожидаемому challenge. Обещание было напечатано крупным шрифтом. Мелким шрифтом выяснилось: проверяющий берет ожидаемый payload, выдает ему чужой бейджик `secret`, а затем торжественно сверяет записку с самой собой.*

## МЕСТО ПРОИСШЕСТВИЯ

Улика найдена в репозитории [xssnick/tonutils-go](https://github.com/xssnick/tonutils-go), в свежем коммите [`219b7aed`](https://github.com/xssnick/tonutils-go/commit/219b7aed1887395577de44277917e1f51a53e177), в файле [`ton/wallet/proof.go`](https://github.com/xssnick/tonutils-go/blob/219b7aed1887395577de44277917e1f51a53e177/ton/wallet/proof.go). Коммит датирован 14 октября 2026 года, а подозреваемая логика в нем все еще жива. Цитата с [строк 96–104](https://github.com/xssnick/tonutils-go/blob/219b7aed1887395577de44277917e1f51a53e177/ton/wallet/proof.go#L96-L104) дословна:

```go
// VerifyProof checks the proof against the expected login challenge.
// The caller must bind the challenge to the session and consume it once after success.
func (v *TonConnectVerifier) VerifyProof(ctx context.Context, addr *address.Address, proof TonConnectProof, expectedPayload string, stateInit []byte) error {
	return v.VerifyProofHandlePayload(ctx, addr, proof, stateInit, func(payload, secret string) error {
		if payload != secret {
			return errors.New("invalid payload in proof")
		}
		return nil
	}, expectedPayload)
}
```

## ПАРОЛЬ, КОТОРЫЙ ПРИШЕЛ ПОД ЧУЖИМ ИМЕНЕМ

Сыщик сперва осмотрел подпись. В [строках 80–93](https://github.com/xssnick/tonutils-go/blob/219b7aed1887395577de44277917e1f51a53e177/ton/wallet/proof.go#L80-L93) ключ достают, подпись проверяют, а затем вызывают `payloadVerifier`. На этой части фонарь светит исправно: криптографическая печать действительно сверяется.

Но затем начинается викторианская комедия имен. Функция принимает `expectedPayload` — тот самый challenge, который должен принадлежать текущему входу в систему. Однако в [строке 104](https://github.com/xssnick/tonutils-go/blob/219b7aed1887395577de44277917e1f51a53e177/ton/wallet/proof.go#L104) это значение передают как `secret`, а в [строках 99–103](https://github.com/xssnick/tonutils-go/blob/219b7aed1887395577de44277917e1f51a53e177/ton/wallet/proof.go#L99-L103) сравнивают `payload` с `secret`.

Иными словами, имя у конверта — `expectedPayload`, печать на конверте — `secret`, а сторож смотрит на содержимое и на печать одновременно. Проверка работает, но читателю кода приходится самому установить, что `secret` — это не второй секрет и не серверный token, а тот же challenge, переодетый на входе в callback.

## СЦЕНА С ПОДМЕНОЙ

Представим обычный вход. Сервер выдал challenge `expectedPayload`. Кошелек подписал payload, и подпись успешно прошла проверку в [строках 85–87](https://github.com/xssnick/tonutils-go/blob/219b7aed1887395577de44277917e1f51a53e177/ton/wallet/proof.go#L85-L87). После этого контора должна убедиться, что payload совпадает с challenge конкретной сессии.

Но внутренний клерк получает два конверта: `payload` и `secret`. А `secret` — это как раз прежнее значение `expectedPayload`, переданное последним аргументом в [строке 104](https://github.com/xssnick/tonutils-go/blob/219b7aed1887395577de44277917e1f51a53e177/ton/wallet/proof.go#L104). Поэтому преступление здесь не криптографическое, а канцелярское: проверка корректна, но ее интерфейс притворяется более сложным, чем есть.

Для метода с именем `VerifyProof` это особенно бодрый номер. Документальная строка обещает проверку «expected login challenge» на [строке 96](https://github.com/xssnick/tonutils-go/blob/219b7aed1887395577de44277917e1f51a53e177/ton/wallet/proof.go#L96), а тело метода передает это ожидание callback как секрет. Так в одном коридоре поселились правильный комментарий и совершенно лишняя маскарадная роль для параметра.

## ПРИГОВОР СЫЩИКА

Это не доказательство готового взлома и не баг проверки: при таком вызове `expectedPayload` доходит до сравнения без подмены. Но как улика качества интерфейса дело крепкое. Параметр с ясным именем обещает одно, callback получает его под другим именем, а комментарий заставляет читателя на секунду искать второй секрет там, где лежит тот же challenge.

Исправление выглядит почти неприлично простым: назвать аргумент callback `expectedPayload` или сравнить `payload` с `expectedPayload` прямо в `VerifyProof`. До тех пор сыщик советует не доверять одному имени параметра. В тумане Лондона даже правильный challenge может оказаться просто хорошо переименованной запиской.

Газовый рожок кашлянул. Дело закрыто: сторож сверяет конверт верно, но канцелярия выдала ему чужую печать.

🐀
