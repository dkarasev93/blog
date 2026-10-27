+++
title = "№ 113 — Витрина, где TON переоделся в GRAM"
date = 2026-10-27T16:59:00+03:00
description = "Вечерний Валидатор исследует коммит TON->GRAM: витрина меняет вывески, но оставляет монету TON под плащом."
tags = ["ton-console"]
+++

# 📰 ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

**№ 113 · Вторник, 27 октября 2026 г. · Цена: 0.05 TON за билет в театр переименований**

---

## ДЕЛО О МОНЕТЕ В ЧУЖОМ ПЛАЩЕ

Туман над Лондоном был густ, когда сыщик «Вечернего Валидатора» получил папку из [tonkeeper/ton-console](https://github.com/tonkeeper/ton-console). На обложке стоял коммит [b0633e7](https://github.com/tonkeeper/ton-console/commit/b0633e734998f573bb2bbcc1ee935f7861f9535c) с лаконичной пометкой `wip: TON->GRAM`. Газовый рожок кашлянул: если рабочая витрина меняет имя валюты, но не саму валюту, в подвале явно осталась улика.

## МЕСТО ПРОИСШЕСТВИЯ

Главная сцена найдена в коммите [b0633e7](https://github.com/tonkeeper/ton-console/commit/b0633e734998f573bb2bbcc1ee935f7861f9535c), в файле [src/shared/lib/currency/basic-currency-amount.ts](https://github.com/tonkeeper/ton-console/blob/b0633e734998f573bb2bbcc1ee935f7861f9535c/src/shared/lib/currency/basic-currency-amount.ts), строки [18–24](https://github.com/tonkeeper/ton-console/blob/b0633e734998f573bb2bbcc1ee935f7861f9535c/src/shared/lib/currency/basic-currency-amount.ts#L18-L24). Цитата дословная:

```ts
get stringCurrency(): string {
    if (this.currency === CURRENCY.TON) {
        return 'GRAM';
    }

    return this.currency;
}
```

Сыщик перечитал строку [19](https://github.com/tonkeeper/ton-console/blob/b0633e734998f573bb2bbcc1ee935f7861f9535c/src/shared/lib/currency/basic-currency-amount.ts#L19): внутри все еще стоит `CURRENCY.TON`. Но строка [20](https://github.com/tonkeeper/ton-console/blob/b0633e734998f573bb2bbcc1ee935f7861f9535c/src/shared/lib/currency/basic-currency-amount.ts#L20) уже выдает зрителю `GRAM`. Это не обмен, не конвертация и не новая сеть. Это старый постоялец, которому выдали новый бейдж.

## ПЕРВЫЙ СЛЕД: КОШЕЛЕК СТАЛ ГРАМОМ НА СЛОВАХ

Коммит [b0633e7](https://github.com/tonkeeper/ton-console/commit/b0633e734998f573bb2bbcc1ee935f7861f9535c) прошелся по интерфейсу широким малярным валиком: баланс, пополнение, тестовый кран и комиссии теперь говорят GRAM. Для витрины это звучит бодро. Для кода это значит, что логика по прежнему узнает TON, а человек видит GRAM.

Особенно выразителен файл [src/features/airdrop/ui/controls/FeeControl.tsx](https://github.com/tonkeeper/ton-console/blob/b0633e734998f573bb2bbcc1ee935f7861f9535c/src/features/airdrop/ui/controls/FeeControl.tsx), строки [29–55](https://github.com/tonkeeper/ton-console/blob/b0633e734998f573bb2bbcc1ee935f7861f9535c/src/features/airdrop/ui/controls/FeeControl.tsx#L29-L55): валидатор принимает число, а его стражевые таблички уже требуют GRAM.

```tsx
            if (parseFloat(value) < 0.15) {
                return 'Minimum 0.15 GRAM';
            }
            if (parseFloat(value) > 5) {
                return 'Maximum 5 GRAM';
            }
```

Эта цитата снята со строк [31–36](https://github.com/tonkeeper/ton-console/blob/b0633e734998f573bb2bbcc1ee935f7861f9535c/src/features/airdrop/ui/controls/FeeControl.tsx#L31-L36). Сам порог остался прежним: от 0.15 до 5. Изменилась только вывеска. На входе стоит число, а смысл ему выдает строка рядом. Если потребитель API еще живет в мире TON, кассир и покупатель уже держат разные словари.

## ВТОРОЙ СЛЕД: СЧЕТ ВЫСТАВИЛИ В GRAM

В файле [src/pages/invoices/dashboard/InvoicesApi.tsx](https://github.com/tonkeeper/ton-console/blob/b0633e734998f573bb2bbcc1ee935f7861f9535c/src/pages/invoices/dashboard/InvoicesApi.tsx) строки [37–46](https://github.com/tonkeeper/ton-console/blob/b0633e734998f573bb2bbcc1ee935f7861f9535c/src/pages/invoices/dashboard/InvoicesApi.tsx#L37-L46) пример счета тоже перекрасили:

```tsx
                            Creating a 1 GRAM invoice with life time of 30 minutes and description
                            &quot;Example description&quot; with API using curl:
                        </Text>
                        <CopyPad
                            whiteSpace="pre-wrap"
                            text={`curl -X POST https://tonconsole.com/api/v1/services/invoices/invoice \\
    -H 'Content-Type: application/json' \\
    -H 'Authorization: Bearer ***' \\
    -d '{"amount": "1000000", "life_time": 1800, "description": "Example description", "currency": "GRAM"}'`}
```

Рядом, на строках [53–55](https://github.com/tonkeeper/ton-console/blob/b0633e734998f573bb2bbcc1ee935f7861f9535c/src/pages/invoices/dashboard/InvoicesApi.tsx#L53-L55), документация говорит про `nanoGRAMs`. Но на строке [46](https://github.com/tonkeeper/ton-console/blob/b0633e734998f573bb2bbcc1ee935f7861f9535c/src/pages/invoices/dashboard/InvoicesApi.tsx#L46) это уже значение поля `currency`, а не просто косметика текста. Пример API теперь рассказывает клиенту одну историю, тогда как проверка суммы и внутренний тип еще не получили нового паспорта.

## ВЕРДИКТ СЫЩИКА

[tonkeeper/ton-console](https://github.com/tonkeeper/ton-console) пойман в коммите [b0633e7](https://github.com/tonkeeper/ton-console/commit/b0633e734998f573bb2bbcc1ee935f7861f9535c) на самом лондонском фокусе: TON не исчез, а просто вышел на сцену в шляпе GRAM. В файле [src/shared/lib/currency/basic-currency-amount.ts](https://github.com/tonkeeper/ton-console/blob/b0633e734998f573bb2bbcc1ee935f7861f9535c/src/shared/lib/currency/basic-currency-amount.ts) строки [18–24](https://github.com/tonkeeper/ton-console/blob/b0633e734998f573bb2bbcc1ee935f7861f9535c/src/shared/lib/currency/basic-currency-amount.ts#L18-L24) оставили нам чистую улику: условие ищет TON, а рот интерфейса говорит GRAM.

Редакция не утверждает, что этот черновой коммит уже ломает переводы. Но это сочная граница между названием и смыслом: кошелек может показывать GRAM, пока API, типы, курсы и партнеры продолжают считать TON. Газовый рожок гудит. Сыщик ставит печать: *переименование без переезда*.
