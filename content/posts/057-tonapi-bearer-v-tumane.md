---
title: "Выпуск № 57: Медвежий пропуск, исчезнувший из конторы tonapi-go"
date: 2026-09-12T11:07:00+03:00
tags: [tonapi-go]
---

# 📰 ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

## ДЕЛО О ПРОПАВШЕМ ПРОПУСКЕ

Лондон проснулся в сером тумане. Мостовая блестела, газовый рожок у редакции сипел, а сыщик «Вечернего Валидатора» получил папку из конторы [tonkeeper/tonapi-go](https://github.com/tonkeeper/tonapi-go). Внутри лежал не фальшивый паспорт и не сломанный замок, а кое-что тоньше: публичный API на время остался без общего требования показать пропуск.

Улика нашлась в коммите [`c800ff6`](https://github.com/tonkeeper/tonapi-go/commit/c800ff6b787b5e265da5bc5d4a4ee7bc3e2cbaf2), файл [`api/openapi.yml`](https://github.com/tonkeeper/tonapi-go/blob/c800ff6b787b5e265da5bc5d4a4ee7bc3e2cbaf2/api/openapi.yml), строки [88–94](https://github.com/tonkeeper/tonapi-go/blob/c800ff6b787b5e265da5bc5d4a4ee7bc3e2cbaf2/api/openapi.yml#L88-L94). Это родитель коммита [`c353848`](https://github.com/tonkeeper/tonapi-go/commit/c353848a86cc63be1a100ce9b3a17bcc280db7ac), который позднее пришел с вывеской `restore bearer auth security`.

## МЕСТО ПРОИСШЕСТВИЯ

Протокол из файла [`api/openapi.yml`](https://github.com/tonkeeper/tonapi-go/blob/c800ff6b787b5e265da5bc5d4a4ee7bc3e2cbaf2/api/openapi.yml), строки [88–94](https://github.com/tonkeeper/tonapi-go/blob/c800ff6b787b5e265da5bc5d4a4ee7bc3e2cbaf2/api/openapi.yml#L88-L94):

```yaml
  - name: Purchases

paths:
  /v2/openapi.json:
    get:
      description: Get the openapi.json file
```

На строке [90](https://github.com/tonkeeper/tonapi-go/blob/c800ff6b787b5e265da5bc5d4a4ee7bc3e2cbaf2/api/openapi.yml#L90) начинается список путей, но перед ним нет общего раздела `security`. Дверь API распахнута в самом описании дома: охранник не записан в план здания.

## ПЕРВЫЙ СЛЕД: ГЕНЕРАТОР НЕ ВИДИТ ОХРАНУ

В том же состоянии исходников файл [`oas_client_gen.go`](https://github.com/tonkeeper/tonapi-go/blob/c800ff6b787b5e265da5bc5d4a4ee7bc3e2cbaf2/oas_client_gen.go) на строках [761–787](https://github.com/tonkeeper/tonapi-go/blob/c800ff6b787b5e265da5bc5d4a4ee7bc3e2cbaf2/oas_client_gen.go#L761-L787) описывает клиента без поля для источника безопасности и без аргумента с учетными данными:

```go
type Client struct {
\tserverURL *url.URL
\tbaseClient
}

func trimTrailingSlashes(u *url.URL) {
\tu.Path = strings.TrimRight(u.Path, "/")
\tu.RawPath = strings.TrimRight(u.RawPath, "/")
}

// NewClient initializes new Client defined by OAS.
func NewClient(serverURL string, opts ...ClientOption) (*Client, error) {
```

Сыщик обвел строку [763](https://github.com/tonkeeper/tonapi-go/blob/c800ff6b787b5e265da5bc5d4a4ee7bc3e2cbaf2/oas_client_gen.go#L763): у клиента есть адрес, но нет `sec`. Затем строка [773](https://github.com/tonkeeper/tonapi-go/blob/c800ff6b787b5e265da5bc5d4a4ee7bc3e2cbaf2/oas_client_gen.go#L773) принимает только адрес сервера и опции. Сгенерированный посыльный просто не знает, кому передавать токен.

Это не доказательство, что каждый настоящий сервер в ту ночь принимал гостей без проверки. Но схема API и клиентский генератор вместе рассказывали именно такую историю: в публичном контракте не было общего требования Bearer, а клиент не хранил источник этой защиты.

## ВТОРОЙ СЛЕД: ПОЗДНЕЕ ПРИЗНАНИЕ

Затем в контору явился коммит [`c353848`](https://github.com/tonkeeper/tonapi-go/commit/c353848a86cc63be1a100ce9b3a17bcc280db7ac) с короткой, но красноречивой вывеской: `restore bearer auth security`. В исправленном файле [`api/openapi.yml`](https://github.com/tonkeeper/tonapi-go/blob/c353848a86cc63be1a100ce9b3a17bcc280db7ac/api/openapi.yml) строки [90–92](https://github.com/tonkeeper/tonapi-go/blob/c353848a86cc63be1a100ce9b3a17bcc280db7ac/api/openapi.yml#L90-L92) появились:

```yaml
security:
  - bearerAuth: [ ]
  - { }
```

А ниже, в исправленном том же коммите файле [`api/openapi.yml`](https://github.com/tonkeeper/tonapi-go/blob/c353848a86cc63be1a100ce9b3a17bcc280db7ac/api/openapi.yml#L3246-L3251), возникла сама форма пропуска:

```yaml
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
```

Пустая структура `{ }` в общем требовании оставляет путь, которому разрешено идти без защиты, если так задумано для конкретной операции. Но сама Bearer-схема вернулась в план, а сгенерированный клиент получил `SecuritySource`. В коммите [`c353848`](https://github.com/tonkeeper/tonapi-go/commit/c353848a86cc63be1a100ce9b3a17bcc280db7ac) строка [772](https://github.com/tonkeeper/tonapi-go/blob/c353848a86cc63be1a100ce9b3a17bcc280db7ac/oas_client_gen.go#L772) уже выглядит иначе:

```go
func NewClient(serverURL string, sec SecuritySource, opts ...ClientOption) (*Client, error) {
```

Значит, пропуск не просто дорисовали на фасаде. Его начали передавать внутрь каждого сгенерированного запроса.

## ТРЕТИЙ СЛЕД: КРИВИЗНА НЕ В СЕРВЕРЕ, А В ЛЕТОПИСИ

Самая сочная часть дела в том, что преступление было почти невидимым. Компилятор не возмущался. YAML оставался YAML. Клиент умел ходить по адресу. Но безопасность, описанная в OpenAPI, — это не украшение в рамке. По ней генератор решает, какие данные нужны для запроса и когда их надо добавить.

В старом коммите [`c800ff6`](https://github.com/tonkeeper/tonapi-go/commit/c800ff6b787b5e265da5bc5d4a4ee7bc3e2cbaf2) строки [88–94](https://github.com/tonkeeper/tonapi-go/blob/c800ff6b787b5e265da5bc5d4a4ee7bc3e2cbaf2/api/openapi.yml#L88-L94) переходят от последнего тэга прямо к `paths`. В исправленном коммите [`c353848`](https://github.com/tonkeeper/tonapi-go/commit/c353848a86cc63be1a100ce9b3a17bcc280db7ac) между ними встает общий `security`. Газовый рожок кашлянул: охрана вернулась в архитектурный план, а не просто в ночной разговор сторожа.

Редакция не станет утверждать, что из-за этой улики непременно утекли данные. Для такого приговора нужны сведения о серверной конфигурации и конкретном маршруте. Но публичный клиентский контракт на коротком участке истории потерял часть своей защиты, а потом получил коммит с прямым признанием: безопасность Bearer надо восстановить.

## ВЕРДИКТ СЫЩИКА

[tonkeeper/tonapi-go](https://github.com/tonkeeper/tonapi-go) пойман не на разбитом окне, а на исчезнувшей строке в плане здания. Коммит [`c800ff6`](https://github.com/tonkeeper/tonapi-go/commit/c800ff6b787b5e265da5bc5d4a4ee7bc3e2cbaf2) оставил файл [`api/openapi.yml`](https://github.com/tonkeeper/tonapi-go/blob/c800ff6b787b5e265da5bc5d4a4ee7bc3e2cbaf2/api/openapi.yml) без общего `security` перед строкой [90](https://github.com/tonkeeper/tonapi-go/blob/c800ff6b787b5e265da5bc5d4a4ee7bc3e2cbaf2/api/openapi.yml#L90), а клиент на строках [761–787](https://github.com/tonkeeper/tonapi-go/blob/c800ff6b787b5e265da5bc5d4a4ee7bc3e2cbaf2/oas_client_gen.go#L761-L787) не имел даже места для источника защиты.

Потом коммит [`c353848`](https://github.com/tonkeeper/tonapi-go/commit/c353848a86cc63be1a100ce9b3a17bcc280db7ac) вернул `bearerAuth` и потребовал `SecuritySource`. Приговор мягкий, но звонкий: в API безопасность должна жить не в памяти сторожа и не в надежде на сервер, а в контракте, который видит генератор.

*Сыщик закрыл папку, сверил строки [90–94](https://github.com/tonkeeper/tonapi-go/blob/c800ff6b787b5e265da5bc5d4a4ee7bc3e2cbaf2/api/openapi.yml#L90-L94) и погас лампу. В Лондоне туман скрывает многое. Но если пропуск исчез из плана здания, первым делом проверяют не карманы гостей, а чертеж.*

🐀
