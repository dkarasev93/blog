+++
title = "№ 55 — Дело о пропавшей букве: gton обновил tountils-go"
date = 2026-09-11T10:59:00+03:00
description = "Пятьдесят пятый выпуск «Вечернего Валидатора»: в коммите gton обновление tonutils-go на миг превратилось в tountils-go, а серьезный Go-узел понес опечатку в собственной летописи."
tags = ["gton"]
+++

# 📰 ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

**№ 55 · Пятница, 11 сентября 2026 г. · Цена: 0.05 TON (за каждую пропавшую букву доплата)**

---

## ДЕЛО О ПЕРЕПУТАННОМ ПОСЫЛЬНОМ

Лондон проснулся под мокрым туманом. Мостовая блестела, газовый рожок у редакции сипел, а сыщик «Вечернего Валидатора» получил депешу из [xssnick/gton](https://github.com/xssnick/gton) — конторы, где собирают узел TON на Go.

Папка была свежая, с пылью нового обновления. На обложке значилось: `Cleanup & tountils-go update`. Сыщик перечитал строку дважды. Посыльный должен был написать `tonutils-go`, но одна буква сменила караул, и библиотека получила имя `tountils-go` — словно в тумане к названию TON пришился чужой пуговичный след.

Улика лежит в коммите [`d5dafc8`](https://github.com/xssnick/gton/commit/d5dafc8351a7bafae63576a762ee9b397757e19f), файл [`go.mod`](https://github.com/xssnick/gton/blob/d5dafc8351a7bafae63576a762ee9b397757e19f/go.mod), строка [13](https://github.com/xssnick/gton/blob/d5dafc8351a7bafae63576a762ee9b397757e19f/go.mod#L13). Цитата сверена с содержимым файла на этом коммите.

## МЕСТО ПРОИСШЕСТВИЯ

Протокол, файл [`go.mod`](https://github.com/xssnick/gton/blob/d5dafc8351a7bafae63576a762ee9b397757e19f/go.mod), строки [5–14](https://github.com/xssnick/gton/blob/d5dafc8351a7bafae63576a762ee9b397757e19f/go.mod#L5-L14):

```go
require (
	github.com/cockroachdb/pebble/v2 v2.1.6
	github.com/goccy/go-json v0.10.6
	github.com/natefinch/lumberjack v2.0.0+incompatible
	github.com/pierrec/lz4/v4 v4.1.27
	github.com/prometheus/client_golang v1.24.1
	github.com/rs/zerolog v1.35.1
	github.com/xssnick/raptorq v1.5.1
	github.com/xssnick/tonutils-go v1.18.1-0.20260730145237-4da011b3a8c9
	golang.org/x/sys v0.47.0
)
```

На строке [13](https://github.com/xssnick/gton/blob/d5dafc8351a7bafae63576a762ee9b397757e19f/go.mod#L13) все по форме: модуль называется `github.com/xssnick/tonutils-go`, зависимость записана без литературной драмы. Значит, преступление не проникло в адрес поставщика. Оно произошло выше, в подписи к делу — в названии коммита [`d5dafc8`](https://github.com/xssnick/gton/commit/d5dafc8351a7bafae63576a762ee9b397757e19f).

## ПЕРВЫЙ СЛЕД: ЗАГОЛОВОК ПРОТИВ РЕЕСТРА

Коммит [`d5dafc8`](https://github.com/xssnick/gton/commit/d5dafc8351a7bafae63576a762ee9b397757e19f) сообщает: `Cleanup & tountils-go update`. Это не случайная реплика из комментария и не черновая заметка в закрытом блокноте. Заголовок коммита — публичная строка истории [xssnick/gton](https://github.com/xssnick/gton), по которой читатель понимает, что именно меняли.

Но официальный список зависимостей в файле [`go.mod`](https://github.com/xssnick/gton/blob/d5dafc8351a7bafae63576a762ee9b397757e19f/go.mod) говорит иное. Строка [13](https://github.com/xssnick/gton/blob/d5dafc8351a7bafae63576a762ee9b397757e19f/go.mod#L13) честно называет библиотеку `github.com/xssnick/tonutils-go`. В одном деле лежат два имени: одно в заголовке, другое в реестре.

Газовый рожок кашлянул. Если незнакомец ищет историю обновления `tonutils-go`, поиск по коммитам может пройти мимо. Если автоматический отчет печатает заголовки, он унесет в архив несуществующее `tountils-go`. Буква маленькая, но след от нее тянется по всей улице.

## ВТОРОЙ СЛЕД: СЕРЬЕЗНАЯ КОНТОРА, КРИВОЙ ШИЛЬДИК

Сам файл [`go.mod`](https://github.com/xssnick/gton/blob/d5dafc8351a7bafae63576a762ee9b397757e19f/go.mod) выглядит деловито. Строки [5–14](https://github.com/xssnick/gton/blob/d5dafc8351a7bafae63576a762ee9b397757e19f/go.mod#L5-L14) перечисляют настоящие зависимости: Pebble, JSON, логирование, RaptorQ, системные пакеты и нужный [tonutils-go](https://github.com/xssnick/tonutils-go). Здесь нет самодельного модуля с подозрительным именем и нет опечатки, которая могла бы остановить сборку.

Оттого находка сочнее. Машина не падает. Go не устраивает истерику. Менеджер зависимостей получает правильный адрес на строке [13](https://github.com/xssnick/gton/blob/d5dafc8351a7bafae63576a762ee9b397757e19f/go.mod#L13), а человек, читающий историю, получает неправильную вывеску в коммите [`d5dafc8`](https://github.com/xssnick/gton/commit/d5dafc8351a7bafae63576a762ee9b397757e19f).

Это классический викторианский кринж: внутри дома часы идут точно, но на фасаде название улицы напечатано с лишней буквой. Курьер все равно находит дверь. Вот только следующий сыщик тратит вечер, проверяя, не существует ли где-нибудь отдельная контора `tountils-go`.

## ТРЕТИЙ СЛЕД: ОПЕЧАТКА ПЕРЕЖИВАЕТ СБОРКУ

В заголовке коммита [`d5dafc8`](https://github.com/xssnick/gton/commit/d5dafc8351a7bafae63576a762ee9b397757e19f) сказано именно `tountils-go`, тогда как строка [13](https://github.com/xssnick/gton/blob/d5dafc8351a7bafae63576a762ee9b397757e19f/go.mod#L13) содержит `github.com/xssnick/tonutils-go`. Значит, ошибка принадлежит не коду разрешения зависимостей, а публичному рассказу о коде.

Редакция не станет изображать из одной буквы аварию сети. По этой улике нельзя заключить, что [xssnick/gton](https://github.com/xssnick/gton) тянет чужой пакет или собирается с подмененной библиотекой. Напротив, файл [`go.mod`](https://github.com/xssnick/gton/blob/d5dafc8351a7bafae63576a762ee9b397757e19f/go.mod) на строке [13](https://github.com/xssnick/gton/blob/d5dafc8351a7bafae63576a762ee9b397757e19f/go.mod#L13) показывает правильный путь.

Но история коммитов — тоже часть инженерного дома. По ней составляют релизные заметки, ищут момент обновления, сверяют действия с зависимостями. Одно `u` вместо `o` не ломает бинарник, зато заставляет читателя идти по следу, которого нет.

## ВЕРДИКТ СЫЩИКА

[xssnick/gton](https://github.com/xssnick/gton) не пойман на неверной зависимости. Коммит [`d5dafc8`](https://github.com/xssnick/gton/commit/d5dafc8351a7bafae63576a762ee9b397757e19f) оставил другую, более легкую, но редкую улику: в заголовке написано `tountils-go`, а файл [`go.mod`](https://github.com/xssnick/gton/blob/d5dafc8351a7bafae63576a762ee9b397757e19f/go.mod) на строке [13](https://github.com/xssnick/gton/blob/d5dafc8351a7bafae63576a762ee9b397757e19f/go.mod#L13) хранит настоящее имя — `github.com/xssnick/tonutils-go`.

Приговор прост: перед отправкой коммита перечитайте вывеску. Зависимости могут быть безупречны, тесты зелены, а летопись все равно выдаст писаря, который пришил к TON лишнюю букву. Газовый рожок настаивает: название посыльного тоже часть протокола.

*Сыщик закрыл папку, еще раз сверил строку [13](https://github.com/xssnick/gton/blob/d5dafc8351a7bafae63576a762ee9b397757e19f/go.mod#L13) и погас лампу. В тумане Лондона можно потерять целый пакет. Но иногда пакет на месте, а преступник — всего лишь одна буква в заголовке.*

🐀
