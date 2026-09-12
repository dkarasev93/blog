---
title: "Выпуск № 58: Длина, которая врала в тоннеле"
date: 2026-09-12T16:58:00+03:00
tags: [tonnet-relayer]
---

# 📰 ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

## ДЕЛО О ПОСЫЛКЕ, КОТОРАЯ ВРАЛА О СВОЕМ ВЕСЕ

Лондон тонул в сером тумане. Мостовая блестела, газовый рожок у редакции сипел, а сыщик «Вечернего Валидатора» получил папку из [TONresistor/tonnet-relayer](https://github.com/TONresistor/tonnet-relayer) — узла, который гоняет HTTP через релейный тоннель TONNET.

На обложке стояла деловая надпись: коммит [`605733d`](https://github.com/TONresistor/tonnet-relayer/commit/605733de08a21a3b6747ba650355e25ecdb22019), `fix: correct Content-Length mismatch on POST responses`. Внутри лежало признание куда сочнее обычной починки: библиотека [tonutils-go](https://github.com/xssnick/tonutils-go) переносила длину тела запроса в ответ. Получался HTTP-посыльный, который выдавал письмо и тут же называл его чужим весом.

## МЕСТО ПРОИСШЕСТВИЯ

Репозиторий [TONresistor/tonnet-relayer](https://github.com/TONresistor/tonnet-relayer), коммит [`605733d`](https://github.com/TONresistor/tonnet-relayer/commit/605733de08a21a3b6747ba650355e25ecdb22019), файл [`internal/exit/exit.go`](https://github.com/TONresistor/tonnet-relayer/blob/605733de08a21a3b6747ba650355e25ecdb22019/internal/exit/exit.go), строки [249–262](https://github.com/TONresistor/tonnet-relayer/blob/605733de08a21a3b6747ba650355e25ecdb22019/internal/exit/exit.go#L249-L262). Цитата сверена с содержимым файла на этом коммите.

Протокол, строки [249–262](https://github.com/TONresistor/tonnet-relayer/blob/605733de08a21a3b6747ba650355e25ecdb22019/internal/exit/exit.go#L249-L262):

```go
// Read full body to fix Content-Length mismatch from tonutils-go bug
// (tonutils copies request Content-Length to response Content-Length)
body, err := io.ReadAll(resp.Body)
if err != nil {
	return fmt.Errorf("read response body: %w", err)
}

// Reset response metadata and set correct Content-Length
resp.Body = io.NopCloser(bytes.NewReader(body))
resp.ContentLength = int64(len(body))
resp.TransferEncoding = nil
resp.Header.Del("Content-Length")
resp.Header.Del("Transfer-Encoding")
resp.Header.Set("Content-Length", fmt.Sprintf("%d", len(body)))
```

## ПЕРВЫЙ СЛЕД: КУРЬЕР ПЕРЕПУТАЛ КОНВЕРТЫ

Сцена начинается после HTTP-вызова. На строке [234](https://github.com/TONresistor/tonnet-relayer/blob/605733de08a21a3b6747ba650355e25ecdb22019/internal/exit/exit.go#L234) релейный узел получает `resp`, а на строке [242](https://github.com/TONresistor/tonnet-relayer/blob/605733de08a21a3b6747ba650355e25ecdb22019/internal/exit/exit.go#L242) обещает закрыть тело ответа. Казалось бы, письмо принято и можно отправлять его дальше по тоннелю.

Но коммит [`605733d`](https://github.com/TONresistor/tonnet-relayer/commit/605733de08a21a3b6747ba650355e25ecdb22019) прямо сообщает: в [tonutils-go](https://github.com/xssnick/tonutils-go) версии 1.10.2 длина из заголовка запроса переезжала в заголовок ответа. Это особенно больно для POST с большим телом: у ответа оказывается размер, который принадлежит входящему письму, а не тому, что вернул внешний сервер.

Иными словами, посыльный смотрит на конверт, который принес клиент, а затем приклеивает его вес к ответу сервера. Если размеры различаются, следующий почтальон верит печати и начинает искать в ответе байты, которых там нет, или считает письмо оборванным.

## ВТОРОЙ СЛЕД: ПЕРЕДАЧА ПОД НАБЛЮДЕНИЕМ

Строка [251](https://github.com/TONresistor/tonnet-relayer/blob/605733de08a21a3b6747ba650355e25ecdb22019/internal/exit/exit.go#L251) читает все тело через `io.ReadAll`. Это не просто жадность архивариуса: релею надо знать настоящий размер ответа до его упаковки.

Дальше строки [257–262](https://github.com/TONresistor/tonnet-relayer/blob/605733de08a21a3b6747ba650355e25ecdb22019/internal/exit/exit.go#L257-L262) устраивают ответу переодевание. Тело кладут в новый `io.NopCloser`, длину пересчитывают через `len(body)`, перенос чанков выключают, старые заголовки удаляют, а затем печатают честный `Content-Length`.

Газовый рожок кашлянул на строке [260](https://github.com/TONresistor/tonnet-relayer/blob/605733de08a21a3b6747ba650355e25ecdb22019/internal/exit/exit.go#L260): старую длину приходится вырвать из заголовков. На строке [262](https://github.com/TONresistor/tonnet-relayer/blob/605733de08a21a3b6747ba650355e25ecdb22019/internal/exit/exit.go#L262) ставится новая, вычисленная по настоящему телу. Вежливый HTTP-ответ возвращается из морга метаданных с верным паспортом.

## ТРЕТИЙ СЛЕД: ПОЧЕМУ ЭТО СОЧНО

После починки ответ сериализуется на строках [264–268](https://github.com/TONresistor/tonnet-relayer/blob/605733de08a21a3b6747ba650355e25ecdb22019/internal/exit/exit.go#L264-L268), а затем упаковывается в `StreamData` на строках [270–280](https://github.com/TONresistor/tonnet-relayer/blob/605733de08a21a3b6747ba650355e25ecdb22019/internal/exit/exit.go#L270-L280). Значит, неверная длина была не декоративной царапиной: она стояла прямо перед передачей ответа через TONNET.

Сочный контраст вот где. Заголовок коммита [`605733d`](https://github.com/TONresistor/tonnet-relayer/commit/605733de08a21a3b6747ba650355e25ecdb22019) говорит о несовпадении длины, а комментарий в файле [`internal/exit/exit.go`](https://github.com/TONresistor/tonnet-relayer/blob/605733de08a21a3b6747ba650355e25ecdb22019/internal/exit/exit.go) на строках [249–250](https://github.com/TONresistor/tonnet-relayer/blob/605733de08a21a3b6747ba650355e25ecdb22019/internal/exit/exit.go#L249-L250) без стыда называет виновника: `tonutils-go bug`.

Редакция не станет считать весь [TONresistor/tonnet-relayer](https://github.com/TONresistor/tonnet-relayer) негодной конторой. Наоборот, коммит [`605733d`](https://github.com/TONresistor/tonnet-relayer/commit/605733de08a21a3b6747ba650355e25ecdb22019) показывает точный ремонт на границе двух библиотек: внешний клиент принес одну длину, внутренний ответ получил другую, а релейный узел перестал верить первой попавшейся цифре.

## ВЕРДИКТ СЫЩИКА

[TONresistor/tonnet-relayer](https://github.com/TONresistor/tonnet-relayer) пойман не на краже тела, а на паспорте, который пришлось переписать перед отправкой. В коммите [`605733d`](https://github.com/TONresistor/tonnet-relayer/commit/605733de08a21a3b6747ba650355e25ecdb22019) файл [`internal/exit/exit.go`](https://github.com/TONresistor/tonnet-relayer/blob/605733de08a21a3b6747ba650355e25ecdb22019/internal/exit/exit.go) читает ответ целиком, меняет длину на строках [257–262](https://github.com/TONresistor/tonnet-relayer/blob/605733de08a21a3b6747ba650355e25ecdb22019/internal/exit/exit.go#L257-L262) и только затем пускает его в сериализацию.

Приговор прост: HTTP-посредник не должен переносить размер входящего письма на исходящее. Иначе крупный POST превращает тоннель в почтовую драму, где тело ответа уже доставлено, а его паспорт все еще живет в чужом конверте.

*Сыщик закрыл папку, еще раз сверил строки [249–262](https://github.com/TONresistor/tonnet-relayer/blob/605733de08a21a3b6747ba650355e25ecdb22019/internal/exit/exit.go#L249-L262) и погас лампу. В тумане Лондона можно спрятать целый тоннель. Но неверная длина все равно выдает курьера.*

🐀
