---
title: "Выпуск № 66: Конвертер, который честно сказал some shit"
date: 2026-09-16T16:59:00+03:00
tags: [opentonapi]
---

# 📰 ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

## ДЕЛО О КОНВЕРТЕРЕ, КОТОРЫЙ ПОТЕРЯЛ СЛОВАРЬ

Лондон стоял в тумане, когда сыщик «Вечернего Валидатора» получил депешу из конторы [tonkeeper/opentonapi](https://github.com/tonkeeper/opentonapi). Бумага пришла из коммита [`7375d82`](https://github.com/tonkeeper/opentonapi/commit/7375d828bf610898e63ba9642f014ecebe740826), носившего строгую вывеску: `fix: remove 9000 cap on offset for GET /v2/jettons/{account_id}/holders`.

Под такой вывеской ожидался чинный ремонт пагинации. Но в подвале [конвертера](https://github.com/tonkeeper/opentonapi/blob/7375d828bf610898e63ba9642f014ecebe740826/pkg/api/converters.go) сыщик нашел другой протокол: если JSON вдруг не пожелает собираться, служба не сообщит, какое поле стало подозрительным. Она просто скажет всему городу: `some shit`.

## МЕСТО ПРОИСШЕСТВИЯ

Репозиторий [tonkeeper/opentonapi](https://github.com/tonkeeper/opentonapi), коммит [`7375d82`](https://github.com/tonkeeper/opentonapi/commit/7375d828bf610898e63ba9642f014ecebe740826), файл [`pkg/api/converters.go`](https://github.com/tonkeeper/opentonapi/blob/7375d828bf610898e63ba9642f014ecebe740826/pkg/api/converters.go), строки [113–146](https://github.com/tonkeeper/opentonapi/blob/7375d828bf610898e63ba9642f014ecebe740826/pkg/api/converters.go#L113-L146). Цитата сверена по содержимому файла на этом коммите.

Вот дословный протокол из кабинета, где всякая структура должна стать картой JSON:

```go
func anyToJSONRawMap(a any) map[string]jx.Raw { //todo: переписать этот ужас
	var m = map[string]jx.Raw{}
	if am, ok := a.(map[string]any); ok {
		for k, v := range am {
			m[k], _ = json.Marshal(v)
		}
		return m
	}
	t := reflect.ValueOf(a)
	switch t.Kind() {
	case reflect.Struct:
		for i := 0; i < t.NumField(); i++ {
			var b []byte
			var err error
			if aj, ok := t.Field(i).Interface().(json.Marshaler); ok {
				b, err = aj.MarshalJSON()
			} else if t.Field(i).Kind() == reflect.Struct {
				m := anyToJSONRawMap(t.Field(i).Interface())
				m2 := make(map[string]json.RawMessage)
				for k, v := range m {
					m2[k] = json.RawMessage(v)
				}
				b, err = json.Marshal(m2)
			} else {
				b, err = json.Marshal(t.Field(i).Interface())
			}
			if err != nil {
				panic("some shit")
			}
			name := t.Type().Field(i).Name
			m[name] = b
		}
	default:
		panic(fmt.Sprintf("some shit %v", t.Kind()))
	}
	return m
}
```

## ПЕРВЫЙ СЛЕД: УЖАС ПОД ПЕЧАТЬЮ TODO

На строке [113](https://github.com/tonkeeper/opentonapi/blob/7375d828bf610898e63ba9642f014ecebe740826/pkg/api/converters.go#L113) функция получает самое широкое имя из возможных: `anyToJSONRawMap`. На вход можно подать карту или структуру, а на выходе обещан словарь сырых JSON-фрагментов.

В ту же строку вмонтирована служебная записка: `//todo: переписать этот ужас`. Не «упростить», не «добавить тест», не «уточнить формат». Ужас уже признан, внесен в официальный протокол и поставлен на дежурство. Газовый рожок кашлянул: конвертер сам оставил на двери табличку с оценкой своей работы.

Дальше, на строках [115–119](https://github.com/tonkeeper/opentonapi/blob/7375d828bf610898e63ba9642f014ecebe740826/pkg/api/converters.go#L115-L119), карта полей проходит без особой драмы. Значения отправляются в `json.Marshal`, а ошибки этой операции даже не поднимаются наружу: результат присваивается через `m[k], _`.

## ВТОРОЙ СЛЕД: СЛОВАРЬ ИЗ ЗЕРКАЛА

Если вход не карта, служба берет его отражение через `reflect.ValueOf` на строке [121](https://github.com/tonkeeper/opentonapi/blob/7375d828bf610898e63ba9642f014ecebe740826/pkg/api/converters.go#L121). Затем на строках [122–123](https://github.com/tonkeeper/opentonapi/blob/7375d828bf610898e63ba9642f014ecebe740826/pkg/api/converters.go#L122-L123) указывает: законный посетитель здесь только `reflect.Struct`.

Структуру разбирают по полям. Если поле само умеет быть `json.Marshaler`, ему дают слово на строках [127–128](https://github.com/tonkeeper/opentonapi/blob/7375d828bf610898e63ba9642f014ecebe740826/pkg/api/converters.go#L127-L128). Если внутри сидит другая структура, конвертер снова зовет сам себя на строках [129–135](https://github.com/tonkeeper/opentonapi/blob/7375d828bf610898e63ba9642f014ecebe740826/pkg/api/converters.go#L129-L135). В иных случаях он старается честно упаковать поле на строках [136–138](https://github.com/tonkeeper/opentonapi/blob/7375d828bf610898e63ba9642f014ecebe740826/pkg/api/converters.go#L136-L138).

На этом месте сцена еще напоминает инженерную работу. Есть отражение типов, рекурсия, сериализация, даже попытка учесть собственный маршалер. Но затем ошибка получает не имя поля, не тип и не исходную причину. Она получает характер.

## ТРЕТИЙ СЛЕД: ПРИГОВОР «SOME SHIT»

Строки [139–141](https://github.com/tonkeeper/opentonapi/blob/7375d828bf610898e63ba9642f014ecebe740826/pkg/api/converters.go#L139-L141) ловят провал упаковки и вызывают:

```go
panic("some shit")
```

Дословно. Если маршалинг поля отказал, процесс не получает хотя бы названия виновника. В журнал не попадает `err`, хотя переменная `err` прямо стоит в соседнем протоколе. Вместо этого дежурный клерк разбивает стекло, нажимает тревожную кнопку и кричит формулу, которую приличная контора обычно оставляет за дверью.

Сыщик не станет утверждать, что любой экземпляр структуры неизбежно доведет службу до этой паники. Для такого вывода нужна трасса конкретного входа. Но сам маршрут написан ясно: ошибка сериализации превращается в панику, а сообщение скрывает и причину, и поле, и тип.

Еще сочнее выглядит второй выход. На строках [145–146](https://github.com/tonkeeper/opentonapi/blob/7375d828bf610898e63ba9642f014ecebe740826/pkg/api/converters.go#L145-L146) любой вход, который не оказался структурой или картой, получает новый приговор:

```go
panic(fmt.Sprintf("some shit %v", t.Kind()))
```

Теперь к ругательству прикручен вид значения. Если пришел указатель, срез или еще один неожиданный жилец, город узнает не «ожидалась структура», а `some shit ptr` или `some shit slice`. Это уже не сообщение об ошибке, а визитная карточка раздраженного сторожа.

## ПОЧЕМУ ЭТО СОЧНО

Кринж здесь не в крепком английском слове самом по себе. В личном черновике разработчика такая реплика могла бы пережить одну бессонную ночь. Но перед нами публичный Go-код из большого TON-сервиса, и фраза стоит в исполняемой ветке, куда ведет обобщенный конвертер.

Особенно выразителен контраст с именем функции на строке [113](https://github.com/tonkeeper/opentonapi/blob/7375d828bf610898e63ba9642f014ecebe740826/pkg/api/converters.go#L113). Вывеска обещает универсальный переход из `any` в карту JSON. Под вывеской лежит отражение типов, рекурсия и несколько мест, где неудачный вход не возвращает ошибку, а вызывает панику. А над всей конторой висит честное `//todo: переписать этот ужас`.

Коммит [`7375d82`](https://github.com/tonkeeper/opentonapi/commit/7375d828bf610898e63ba9642f014ecebe740826) сам по себе посвящен пределу пагинации для держателей jetton. В его описании говорится об ограничении `9000`, спецификации и сгенерированной проверке запроса. Но в тот же коммит добавлен весь файл [pkg/api/converters.go](https://github.com/tonkeeper/opentonapi/blob/7375d828bf610898e63ba9642f014ecebe740826/pkg/api/converters.go), где рядом с деловой починкой поселилась эта маленькая словесная катастрофа.

Редакция не станет считать [tonkeeper/opentonapi](https://github.com/tonkeeper/opentonapi) темной лавкой. Наоборот, такой участок легко сделать полезнее: возвращать исходную ошибку вместе с именем поля, ограничивать поддержанные типы заранее и не превращать неожиданный вид входа в панику. Но старые строки [139–146](https://github.com/tonkeeper/opentonapi/blob/7375d828bf610898e63ba9642f014ecebe740826/pkg/api/converters.go#L139-L146) уже заслужили рамку в редакции.

## ВЕРДИКТ СЫЩИКА

[tonkeeper/opentonapi](https://github.com/tonkeeper/opentonapi) поймано на деле о конвертере с дурным нравом. В коммите [`7375d82`](https://github.com/tonkeeper/opentonapi/commit/7375d828bf610898e63ba9642f014ecebe740826) файл [`pkg/api/converters.go`](https://github.com/tonkeeper/opentonapi/blob/7375d828bf610898e63ba9642f014ecebe740826/pkg/api/converters.go) на строке [113](https://github.com/tonkeeper/opentonapi/blob/7375d828bf610898e63ba9642f014ecebe740826/pkg/api/converters.go#L113) называет собственную работу ужасом, на строке [140](https://github.com/tonkeeper/opentonapi/blob/7375d828bf610898e63ba9642f014ecebe740826/pkg/api/converters.go#L140) превращает ошибку JSON в `panic("some shit")`, а на строке [146](https://github.com/tonkeeper/opentonapi/blob/7375d828bf610898e63ba9642f014ecebe740826/pkg/api/converters.go#L146) выдает разновидности этого диагноза по имени типа.

Приговор прост: если код не знает, как упаковать поле, пусть назовет поле и вернет причину, а не шлет в туман ругательство с трассой паники. Сыщик обвел строки [139–146](https://github.com/tonkeeper/opentonapi/blob/7375d828bf610898e63ba9642f014ecebe740826/pkg/api/converters.go#L139-L146), погас газовый рожок и удалился по мокрой мостовой.

В Лондоне даже самый унылый конвертер может стать свидетелем. Но если свидетель говорит только `some shit`, допрос придется начинать с самого начала.

🐀
