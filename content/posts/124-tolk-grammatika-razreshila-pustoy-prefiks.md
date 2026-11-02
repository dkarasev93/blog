---
title: "№ 124 — Грамматика, которая признала 0x числом"
date: 2026-11-02T16:59:00+03:00
tags: [ton-language-server]
---

# ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

## ДЕЛО О ЧИСЛЕ БЕЗ ЦИФР

Лондон тонул в тумане, когда сыщик «Вечернего Валидатора» получил папку из [ton-blockchain/ton-language-server](https://github.com/ton-blockchain/ton-language-server). Внутри лежала грамматика Tolk для редактора: маленькая контора, которая должна лишь подсвечивать код, но решила смотреть на числа с удивительной широтой души.

Газовый рожок кашлянул. В подвале нашли тест, где `0x`, `0b` и `123_` проходят как вполне приличные числовые граждане. Цифры, правда, не явились на опознание.

## МЕСТО ПРОИСШЕСТВИЯ

След ведет в [коммит 342d87c](https://github.com/ton-blockchain/ton-language-server/commit/342d87c26b5fab1e300015bf2831cafa59fda457), в файл [editors/code/src/languages/syntaxes/tolk.tmLanguage.test.ts](https://github.com/ton-blockchain/ton-language-server/blob/342d87c26b5fab1e300015bf2831cafa59fda457), строки [18–36](https://github.com/ton-blockchain/ton-language-server/blob/342d87c26b5fab1e300015bf2831cafa59fda457). Цитата дословная:

```ts
describe("Tolk TextMate grammar", () => {
    it("scopes integer literals with compiler-compatible separators", () => {
        const grammar = readTolkGrammar()
        const numericPattern = grammar.patterns.find(pattern => pattern.name === "constant.numeric")
        const numericRegex = new RegExp(numericPattern?.match ?? "", "u")

        expect(numericRegex.exec("100_000")?.[0]).toBe("100_000")
        expect(numericRegex.exec("0xFF_FF")?.[0]).toBe("0xFF_FF")
        expect(numericRegex.exec("0b1010_0011")?.[0]).toBe("0b1010_0011")
        expect(numericRegex.exec("123_")?.[0]).toBe("123_")
        expect(numericRegex.exec("0b0_____1")?.[0]).toBe("0b0_____1")
        expect(numericRegex.exec("0b_0____1")?.[0]).toBe("0b_0____1")
        expect(numericRegex.exec("0x")?.[0]).toBe("0x")
        expect(numericRegex.exec("0b")?.[0]).toBe("0b")
        expect(numericRegex.exec("0x_FF")?.[0]).toBe("0x_FF")
        expect(numericRegex.exec("0b_")?.[0]).toBe("0b_")

        expect(numericRegex.exec("_100")).toBeNull()
    })
```

На [строках 24–26](https://github.com/ton-blockchain/ton-language-server/blob/342d87c26b5fab1e300015bf2831cafa59fda457/editors/code/src/languages/syntaxes/tolk.tmLanguage.test.ts#L24-L26) сыщик не спорит: разделители вроде `100_000` и `0xFF_FF` действительно выглядят разумно. Но [строки 27–33](https://github.com/ton-blockchain/ton-language-server/blob/342d87c26b5fab1e300015bf2831cafa59fda457/editors/code/src/languages/syntaxes/tolk.tmLanguage.test.ts#L27-L33) распахивают дверь перед `123_`, `0b0_____1`, `0b_0____1`, голым `0x`, голым `0b`, а также `0x_FF` и `0b_`.

Это не догадка сыщика о намерениях авторов. Тест сам требует, чтобы регулярное выражение нашло в каждой строке именно весь подозрительный фрагмент. Для `_100` на [строке 35](https://github.com/ton-blockchain/ton-language-server/blob/342d87c26b5fab1e300015bf2831cafa59fda457/editors/code/src/languages/syntaxes/tolk.tmLanguage.test.ts#L35) выставлен шлагбаум, но перед числом без цифр он почему то не опускается.

## КАК УСТРОЕНА ЛОВУШКА

Вещественное правило живет в файле [syntaxes/tolk.tmLanguage.json](https://github.com/ton-blockchain/ton-language-server/blob/342d87c26b5fab1e300015bf2831cafa59fda457/syntaxes/tolk.tmLanguage.json), коммит [342d87c](https://github.com/ton-blockchain/ton-language-server/commit/342d87c26b5fab1e300015bf2831cafa59fda457), строки [40–43](https://github.com/ton-blockchain/ton-language-server/blob/342d87c26b5fab1e300015bf2831cafa59fda457/syntaxes/tolk.tmLanguage.json#L40-L43):

```json
{
    "name": "constant.numeric",
    "match": "\\b(-?(0x[0-9a-fA-F_]*|0b[01_]*|[0-9][0-9_]*))\\b"
}
```

Звездочка после набора символов разрешает ноль совпадений. Поэтому `0x` подходит под ветку `0x[0-9a-fA-F_]*`, `0b` — под ветку `0b[01_]*`, а `123_` — под десятичную ветку. Подсветчик не компилирует контракт и не подписывает транзакцию, так что сеть от этого не падает. Но редактор выдает подозрительному обрывку парадный мундир числа, еще не проверив, есть ли в нем хоть одна цифра.

Особенно викторианский штрих: тест называется `compiler-compatible separators` на [строке 19](https://github.com/ton-blockchain/ton-language-server/blob/342d87c26b5fab1e300015bf2831cafa59fda457/editors/code/src/languages/syntaxes/tolk.tmLanguage.test.ts#L19), а рядом стоят конструкции, которые выглядят не как разделители, а как недостроенные номера. В Лондоне это назвали бы не совместимостью, а слишком гостеприимной дверью.

## СВИДЕТЕЛЬ ИЗ СОСЕДНЕЙ КАМЕРЫ

Тот же [коммит 342d87c](https://github.com/ton-blockchain/ton-language-server/commit/342d87c26b5fab1e300015bf2831cafa59fda457) добавил в тест и более строгий осмотр строковых побегов. В [строках 48–54](https://github.com/ton-blockchain/ton-language-server/blob/342d87c26b5fab1e300015bf2831cafa59fda457/editors/code/src/languages/syntaxes/tolk.tmLanguage.test.ts#L48-L54) `\\n`, `\\r` и `\\t` признаются, а `\\0`, `\\u` и `\\u1234` получают отказ.

Сыщик не смешивает эти улики. Там стоит аккуратный фильтр, а в числовом коридоре — тест на почти все, что начинается с правильной вывески. Контраст и делает дело сочным: для побегов устроили паспортный контроль, для основания числа хватило двух букв и доброго расположения духа.

## ПРИГОВОР СЫЩИКА

[ton-blockchain/ton-language-server](https://github.com/ton-blockchain/ton-language-server) в [коммите 342d87c](https://github.com/ton-blockchain/ton-language-server/commit/342d87c26b5fab1e300015bf2831cafa59fda457) пойман не на краже монет, а на редакторской галантности. Файл [syntaxes/tolk.tmLanguage.json](https://github.com/ton-blockchain/ton-language-server/blob/342d87c26b5fab1e300015bf2831cafa59fda457/syntaxes/tolk.tmLanguage.json) строками [40–43](https://github.com/ton-blockchain/ton-language-server/blob/342d87c26b5fab1e300015bf2831cafa59fda457/syntaxes/tolk.tmLanguage.json#L40-L43) позволяет нулевое число символов после `0x` и `0b`, а тест [editors/code/src/languages/syntaxes/tolk.tmLanguage.test.ts](https://github.com/ton-blockchain/ton-language-server/blob/342d87c26b5fab1e300015bf2831cafa59fda457/editors/code/src/languages/syntaxes/tolk.tmLanguage.test.ts) строками [27–33](https://github.com/ton-blockchain/ton-language-server/blob/342d87c26b5fab1e300015bf2831cafa59fda457/editors/code/src/languages/syntaxes/tolk.tmLanguage.test.ts#L27-L33) это торжественно закрепляет.

Туман рассеялся. В подвале остались пустые префиксы, хвостовой знак и регулярное выражение с хорошими манерами. Газовый рожок выдал короткий сигнал: **если число еще не началось, не выдавайте ему орден за завершенную арифметику**.

🐀