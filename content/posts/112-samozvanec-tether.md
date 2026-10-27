---
title: "Выпуск № 112: Тетер, которого не было"
date: 2026-10-27T11:00:03+03:00
tags: [scam_backoffice_rules]
---

# Вечерний Валидатор

## Тетер, которого не было

Лондон просыпался нехотя. Туман полз по мостовой, газовый рожок сипел над дверью редакции, а из переулка донесся запах свежего кринжа. Сыщик «Вечернего Валидатора» накинул плащ и отправился в контору [tonkeeper/scam_backoffice_rules](https://github.com/tonkeeper/scam_backoffice_rules). Там хранился список подозрительных жетонов. Казалось бы, обычное дело: поймал самозванца, поставил метку, закрыл папку.

Но в тестовой комнате лежал вещественный доказатель номер один. В [коммите 52e5132](https://github.com/tonkeeper/scam_backoffice_rules/commit/52e5132d92ebdd365bf9595159b68862e771faf5) в [jetton_verifier_test.go](https://github.com/tonkeeper/scam_backoffice_rules/blob/52e5132d92ebdd365bf9595159b68862e771faf5/jetton_verifier_test.go) на строках [13–40](https://github.com/tonkeeper/scam_backoffice_rules/blob/52e5132d92ebdd365bf9595159b68862e771faf5/jetton_verifier_test.go#L13-L40) собрана витрина «известных» жетонов. Среди них красуется Cock Fights Token, а рядом — безымянный адрес, наряженный в имя Tether USD и символ USDT.

> `// this is a fake token added to the well-known jettons list to ban scam tokens`
>
> — [строка 35](https://github.com/tonkeeper/scam_backoffice_rules/blob/52e5132d92ebdd365bf9595159b68862e771faf5/jetton_verifier_test.go#L35), [коммит 52e5132](https://github.com/tonkeeper/scam_backoffice_rules/commit/52e5132d92ebdd365bf9595159b68862e771faf5)

Вот где преступник оставил визитную карточку: чтобы поймать фальшивый USDT, контора сначала внесла фальшивый USDT в список улик. В полицейском участке это звучало бы так: «Мы вычисляем самозванцев, временно назначая одного из них старшим инспектором».

Дальше дело стало еще гуще. В [тесте `TestJettonVerifier_IsSimilarToWellKnownSymbol`](https://github.com/tonkeeper/scam_backoffice_rules/blob/52e5132d92ebdd365bf9595159b68862e771faf5/jetton_verifier_test.go#L43-L50) настоящий адрес USDT получает оправдание на строках [133–137](https://github.com/tonkeeper/scam_backoffice_rules/blob/52e5132d92ebdd365bf9595159b68862e771faf5/jetton_verifier_test.go#L133-L137), а тот же символ с чужим адресом отправляется в черный список на строках [145–150](https://github.com/tonkeeper/scam_backoffice_rules/blob/52e5132d92ebdd365bf9595159b68862e771faf5). Сыщик записал: имя не спасает, адрес решает. Но для теста пришлось завести декоративного самозванца прямо в музей оригиналов.

### Место происшествия

- Репозиторий: [tonkeeper/scam_backoffice_rules](https://github.com/tonkeeper/scam_backoffice_rules)
- Коммит: [52e5132d92ebdd365bf9595159b68862e771faf5](https://github.com/tonkeeper/scam_backoffice_rules/commit/52e5132d92ebdd365bf9595159b68862e771faf5)
- Файл: [jetton_verifier_test.go](https://github.com/tonkeeper/scam_backoffice_rules/blob/52e5132d92ebdd365bf9595159b68862e771faf5/jetton_verifier_test.go)
- Улика: комментарий на строке [35](https://github.com/tonkeeper/scam_backoffice_rules/blob/52e5132d92ebdd365bf9595159b68862e771faf5/jetton_verifier_test.go#L35) и подставной Tether на строках [36–39](https://github.com/tonkeeper/scam_backoffice_rules/blob/52e5132d92ebdd365bf9595159b68862e771faf5/jetton_verifier_test.go#L36-L39)

В тумане еще долго было видно, как газовый рожок моргает над дверью. На табличке значилось: «Вход для настоящих жетонов. Фальшивые — по предварительной записи».
