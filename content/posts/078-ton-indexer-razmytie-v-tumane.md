+++
title = "№ 78 — Индексатор, который сначала принес картинку, а потом спрятал ее в тумане"
date = 2026-09-22T17:57:00+03:00
description = "Семьдесят восьмой выпуск «Вечернего Валидатора»: ton-indexer хранит для метаданных отдельные флаги NSFW и scam, заранее строит размытые копии изображений, а затем может заменить обычные ссылки на блюр или пустоту."
tags = ["ton-indexer"]
+++

# ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

**№ 78.** *Лондон. Вечерний туман полез по мостовой, газовый рожок у редакции сипит, а сыщик получил папку из конторы индексатора. Там обещают честно показывать метаданные жеттонов и прочих цифровых лиц. Но в подвале уже стоит особый фотограф: он делает три обычных снимка, три размытых, а при тревожном флаге меняет лицо героя на блюр или вообще оставляет пустую раму. Канцелярия не просто хранит портреты. Она заранее готовит им туман.*

## МЕСТО ПРОИСШЕСТВИЯ

Улика найдена в репозитории [toncenter/ton-indexer](https://github.com/toncenter/ton-indexer), в коммите [`1fe72ea`](https://github.com/toncenter/ton-indexer/commit/1fe72ea51d6a402d7ec38299d8c82606ff8066ad), в файлах [`ton-index-go/index/crud/crud.go`](https://github.com/toncenter/ton-indexer/blob/1fe72ea51d6a402d7ec38299d8c82606ff8066ad/ton-index-go/index/crud/crud.go), [`ton-metadata-fetcher/images.go`](https://github.com/toncenter/ton-indexer/blob/1fe72ea51d6a402d7ec38299d8c82606ff8066ad/ton-metadata-fetcher/images.go) и [`ton-metadata-fetcher/main.go`](https://github.com/toncenter/ton-indexer/blob/1fe72ea51d6a402d7ec38299d8c82606ff806/ton-metadata-fetcher/main.go). Коммит называется `Added nsfw and scam flags for metadata, added bluring of images (#418)`; цитаты сверены по этому снимку.

## ПЕРВЫЙ СЛЕД: КОНТОРА ЗАВОДИТ ДВА ЧЕРНЫХ СПИСКА

Сначала индексатор расширяет запись метаданных двумя флагами. В файле [`ton-index-go/index/crud/crud.go`](https://github.com/toncenter/ton-indexer/blob/1fe72ea51d6a402d7ec38299d8c82606ff8066ad/ton-index-go/index/crud/crud.go) строки [122–124](https://github.com/toncenter/ton-indexer/blob/1fe72ea51d6a402d7ec38299d8c82606ff8066ad/ton-index-go/index/crud/crud.go#L122-L124) считывают `IsNsfw`, а рядом приводят пустое значение к `false`. Второй флаг, `IsScam`, прибывает в той же распечатке.

Политическая вывеска ясна: картинка теперь приходит не одна. За ней идут два невидимых чиновника — один решает, не слишком ли она непристойна, второй проверяет, не пахнет ли она мошенничеством. В самом фрагменте нет доказательства, что `IsScam` уже меняет изображение. Но `IsNsfw` получает отдельный маршрут, и этот маршрут ведет прямо в мастерскую размытия.

Внутри метода на строках [185–198](https://github.com/toncenter/ton-indexer/blob/1fe72ea51d6a402d7ec38299d8c82606ff8066ad/ton-index-go/index/crud/crud.go#L185-L198) висит дословная инструкция:

```go
func applyNsfwTransform(row models.TokenInfo) models.TokenInfo {
	if row.IsNsfw != nil && *row.IsNsfw {
		fields := []string{"_image_small", "_image_medium", "_image_big"}
		for _, field := range fields {
			if val, exists := row.Extra[field]; exists {
				if img_url, ok := val.(string); ok && img_url != "" {
					if val_blur, exists_blur := row.Extra[field+"_blur"]; exists_blur {
						row.Extra[field] = val_blur
					} else {
						row.Extra[field] = ""
					}
					delete(row.Extra, field+"_blur")
				}
			}
```

Это не предупреждение для клиента и не серый значок поверх карточки. Если флаг `IsNsfw` истинен на строке [186](https://github.com/toncenter/ton-indexer/blob/1fe72ea51d6a402d7ec38299d8c82606ff8066ad/ton-index-go/index/crud/crud.go#L186), контора перебирает три размера изображения. Найдет готовую размытость — подменит ссылку. Не найдет — запишет пустую строку на строке [194](https://github.com/toncenter/ton-indexer/blob/1fe72ea51d6a402d7ec38299d8c82606ff8066ad/ton-index-go/index/crud/crud.go#L194). В Лондоне это называлось бы: «портрет подозрителен, поэтому лицо вырезано из дела».

## ВТОРОЙ СЛЕД: РАЗМЫТИЕ ПОЯВЛЯЕТСЯ ДО ПРЕСТУПЛЕНИЯ

Самая сочная деталь прячется не в подмене, а в подготовке улики. Файл [`ton-metadata-fetcher/images.go`](https://github.com/toncenter/ton-indexer/blob/1fe72ea51d6a402d7ec38299d8c82606ff8066ad/ton-metadata-fetcher/images.go) на строках [21–32](https://github.com/toncenter/ton-indexer/blob/1fe72ea51d6a402d7ec38299d8c82606ff8066ad/ton-metadata-fetcher/images.go#L21-L32) получает новый параметр `blur` и добавляет к адресу ImgProxy операцию `/bl:256`:

```go
func (b *ImgProxyUrlBuilder) BuildUrl(src string, preset string, blur bool) string {
	var path string
	if isIpfs(src) {
		src = strings.TrimPrefix(src, "ipfs://")
		src = fmt.Sprintf("%s/%s", b.ipfs_resolve_base_url, src)
	}
	encoded_url := base64.RawURLEncoding.EncodeToString([]byte(src))
	opts := fmt.Sprintf("pr:%s", preset)
	if blur {
		opts += "/bl:256"
	}
	path = fmt.Sprintf("/%s/%s", opts, encoded_url)
```

Функция на строке [21](https://github.com/toncenter/ton-indexer/blob/1fe72ea51d6a402d7ec38299d8c82606ff8066ad/ton-metadata-fetcher/images.go#L21) не ждет, пока читатель пожалуется на портрет. Она умеет строить и обычный адрес, и адрес с блюром. На строках [28–30](https://github.com/toncenter/ton-indexer/blob/1fe72ea51d6a402d7ec38299d8c82606ff8066ad/ton-metadata-fetcher/images.go#L28-L30) туман добавляется одним условием, почти буднично: `blur` равен истине — появляется `bl:256`.

А затем файл [`ton-metadata-fetcher/main.go`](https://github.com/toncenter/ton-indexer/blob/1fe72ea51d6a402d7ec38299d8c82606ff8066ad/ton-metadata-fetcher/main.go) на строках [442–447](https://github.com/toncenter/ton-indexer/blob/1fe72ea51d6a402d7ec38299d8c82606ff8066ad/ton-metadata-fetcher/main.go#L442-L447) выписывает сразу шесть адресов:

```go
		content.Extra["_image_small"] = img_url_builder.BuildUrl(*content.Image, "small", false)
		content.Extra["_image_medium"] = img_url_builder.BuildUrl(*content.Image, "medium", false)
		content.Extra["_image_big"] = img_url_builder.BuildUrl(*content.Image, "big", false)
		content.Extra["_image_small_blur"] = img_url_builder.BuildUrl(*content.Image, "small", true)
		content.Extra["_image_medium_blur"] = img_url_builder.BuildUrl(*content.Image, "medium", true)
		content.Extra["_image_big_blur"] = img_url_builder.BuildUrl(*content.Image, "big", true)
```

Три первых ссылки служат обычной витриной, три последних — запасным туманом. Приговор еще не вынесен, а размытый свидетель уже сидит в архиве. Газовый рожок кашлянул: в этой конторе цензура не носит ведро с краской по ночам. Она заранее просит фотографа сделать запасную версию каждого портрета.

## ТРЕТИЙ СЛЕД: ЕСЛИ РАЗМЫТОГО НЕТ, ОСТАНЕТСЯ НИЧЕГО

Вернемся к методу `applyNsfwTransform`. Список на строке [187](https://github.com/toncenter/ton-indexer/blob/1fe72ea51d6a402d7ec38299d8c82606ff8066ad/ton-index-go/index/crud/crud.go#L187) охватывает `_image_small`, `_image_medium` и `_image_big`. На строках [191–192](https://github.com/toncenter/ton-indexer/blob/1fe72ea51d6a402d7ec38299d8c82606ff8066ad/ton-index-go/index/crud/crud.go#L191-L192) обычная ссылка меняется на соответствующий ключ с суффиксом `_blur`.

Но запасной план выглядит суровее. Если такого ключа нет, ветка на строках [193–195](https://github.com/toncenter/ton-indexer/blob/1fe72ea51d6a402d7ec38299d8c82606ff8066ad/ton-index-go/index/crud/crud.go#L193-L195) подставляет `""`. После этого строка [196](https://github.com/toncenter/ton-indexer/blob/1fe72ea51d6a402d7ec38299d8c82606ff8066ad/ton-index-go/index/crud/crud.go#L196) удаляет саму размыто-ссылочную улику из `Extra`.

С точки зрения интерфейса сцена проста: либо читателю дают туман, либо ему не дают даже адреса. Код не доказывает ошибку и не говорит, что пустая строка обязательно приведет к падению клиента. Он показывает другое — политика видимости встроена прямо в преобразование ответа. Портрет не помечают как сомнительный, а переписывают перед выдачей.

## ХРОНИКА МЕЛКОГО КРИНЖА

*Отдел запасов.* Строки [442–447](https://github.com/toncenter/ton-indexer/blob/1fe72ea51d6a402d7ec38299d8c82606ff8066ad/ton-metadata-fetcher/main.go#L442-L447) создают по два варианта каждого из трех размеров. Один набор для света, второй для тумана.

*Отдел приговора.* Строка [186](https://github.com/toncenter/ton-indexer/blob/1fe72ea51d6a402d7ec38299d8c82606ff8066ad/ton-index-go/index/crud/crud.go#L186) включает превращение только по `IsNsfw`. Флаг `IsScam` считывается рядом, но в показанном маршруте не командует блюром. Два черных списка заведены, а ключ от камеры выдали только одному.

*Отдел пустых рам.* Строка [194](https://github.com/toncenter/ton-indexer/blob/1fe72ea51d6a402d7ec38299d8c82606ff8066ad/ton-index-go/index/crud/crud.go#L194) не спорит с отсутствием размытой копии. Она просто ставит пустоту. Это надежно скрывает исходный адрес, но для клиента превращает портрет в немую стену.

## ВЕРДИКТ СЫЩИКА

Репозиторий [toncenter/ton-indexer](https://github.com/toncenter/ton-indexer) пойман на выразительной фотоканцелярии. Коммит [`1fe72ea`](https://github.com/toncenter/ton-indexer/commit/1fe72ea51d6a402d7ec38299d8c82606ff8066ad) в файле [`ton-metadata-fetcher/main.go`](https://github.com/toncenter/ton-indexer/blob/1fe72ea51d6a402d7ec38299d8c82606ff8066ad/ton-metadata-fetcher/main.go) заранее строит три обычных и три размытых адреса на строках [442–447](https://github.com/toncenter/ton-indexer/blob/1fe72ea51d6a402d7ec38299d8c82606ff8066ad/ton-metadata-fetcher/main.go#L442-L447).

Потом файл [`ton-index-go/index/crud/crud.go`](https://github.com/toncenter/ton-indexer/blob/1fe72ea51d6a402d7ec38299d8c82606ff8066ad/ton-index-go/index/crud/crud.go) на строках [185–196](https://github.com/toncenter/ton-indexer/blob/1fe72ea51d6a402d7ec38299d8c82606ff8066ad/ton-index-go/index/crud/crud.go#L185-L196) решает судьбу витрины: при `IsNsfw` обычная ссылка заменяется на блюр, а при нехватке запасного адреса — на пустоту. Это не взлом и не обвинение в дурном умысле. Это просто редкая, очень наглядная сцена, где индексатор сначала делает запасную маску для каждого лица, а потом выдает ее за естественный облик.

Сыщик закрыл папку. Газовый рожок дернулся в вечернем тумане. На двери осталась строка [30](https://github.com/toncenter/ton-indexer/blob/1fe72ea51d6a402d7ec38299d8c82606ff8066ad/ton-metadata-fetcher/images.go#L30): `opts += "/bl:256"`. В Лондоне портрет можно не уничтожать. Достаточно заранее заказать ему туман.

🐀
