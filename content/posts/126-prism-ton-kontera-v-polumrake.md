---
title: "№ 126 — Контора грамматик, которая сама повесила табличку «не входить»"
date: 2026-11-03T16:59:00+03:00
tags: [prism-ton]
---

# ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

## ДЕЛО О ПОДСВЕТКЕ, УШЕДШЕЙ В ТУМАН

Лондон тонул в тумане, когда сыщик «Вечернего Валидатора» получил папку из [tact-lang/prism-ton](https://github.com/tact-lang/prism-ton). Контора обещает подсвечивать языки TON в Prism.js: FunC, Fift, TL-B и Tact. Газовый рожок кашлянул, едва открылась первая страница: на двери уже висела табличка, что часть заведения устарела, а посетителю советовали идти в соседнюю контору.

## МЕСТО ПРОИСШЕСТВИЯ

След ведет в [коммит 69db643](https://github.com/tact-lang/prism-ton/commit/69db643d4cb3868192b834937b8e6780348f6522), в файл [README.md](https://github.com/tact-lang/prism-ton/blob/69db643d4cb3868192b834937b8e6780348f6522/README.md), строки [1–5](https://github.com/tact-lang/prism-ton/blob/69db643d4cb3868192b834937b8e6780348f6522/README.md#L1-L5). Цитата дословная:

```text
> [!CAUTION]
> Tact grammar is deprecated; others are deemed legacy. Prefer TextMate grammars over Prism.js ones.
>
> For up-to-date TextMate grammars, see the
> [TON language server repository](https://github.com/ton-blockchain/ton-language-server/blob/main/CONTRIBUTING.md#vs-code-extension).
```

На [строке 2](https://github.com/tact-lang/prism-ton/blob/69db643d4cb3868192b834937b8e6780348f6522/README.md#L2) контора сама сообщает, что грамматика Tact устарела, а остальные стали наследием. На [строке 5](https://github.com/tact-lang/prism-ton/blob/69db643d4cb3868192b834937b8e6780348f6522/README.md#L5) она отправляет за свежими грамматиками в [ton-blockchain/ton-language-server](https://github.com/ton-blockchain/ton-language-server). Это честнее молчаливой могилы, но для витрины с надписью «Set of TON blockchain programming language definitions» выглядит как публичное признание в собственной музейности.

## ВЫВЕСКА ЕЩЕ ГОРИТ

Ниже, на [строках 9–17](https://github.com/tact-lang/prism-ton/blob/69db643d4cb3868192b834937b8e6780348f6522/README.md#L9-L17), список все еще торжественный:

```text
🔺 Set of TON blockchain programming language definitions for Prism.js:

* 💎 FunC — https://docs.ton.org/develop/func/overview
* 📚 Fift — https://docs.ton.org/develop/fift/overview
* 📄 TL-B — https://docs.ton.org/develop/data-formats/tl-b-language

▼ Deprecated:

* ⚡ Tact — https://github.com/tact-lang/tact
```

Сыщик не станет выдавать это за обман: FunC, Fift и TL-B действительно перечислены, а Tact помечен. Но картина сочная: в одном доме вывеска обещает определения языков, предупреждение велит предпочесть другую технологию, а одна из четырех комнат уже закрыта на ремонт без даты открытия.

## ПРИКАЗ ПО ПОДВАЛУ

Инструкция установки на [строках 21–25](https://github.com/tact-lang/prism-ton/blob/69db643d4cb3868192b834937b8e6780348f6522/README.md#L21-L25) тоже оставила отпечаток пальца:

```text
Upon cloning this repo, invoke the following commands in order of appearance (but just once!):

1. `yarn prism:init` — initializes the prismjs/prism submodule and installes it's dependencies
2. `yarn prism:links` — creates all the necessary symbolic links from local files to the prismjs/prism submodule
3. `yarn prism:build` — builds all the grammars and everything there is in prismjs/prism
```

На [строке 21](https://github.com/tact-lang/prism-ton/blob/69db643d4cb3868192b834937b8e6780348f6522/README.md#L21) приказ звучит как распоряжение дворецкому: выполнить команды «в порядке появления», но «только один раз». На [строке 23](https://github.com/tact-lang/prism-ton/blob/69db643d4cb3868192b834937b8e6780348f6522/README.md#L23) к этому церемониальному тону присоединяется английская пара `installes it's dependencies`: две ошибки в одной строке, как две кляксы на гербовой бумаге.

Газета не утверждает, что опечатки ломают сборку. Доказано другое: свежий коммит оставил их в единственной инструкции, которую должен прочитать человек перед первым запуском. А совет «только один раз» особенно пикантен для проекта, где сначала надо инициализировать подмодуль, затем создать символические ссылки, затем собрать все грамматики. Повторить ритуал нельзя, но почему — README не сообщает.

## СВИДЕТЕЛЬ ИЗ ФАЙЛА FUNС

В рабочей грамматике FunC есть и более спокойное признание. В файле [langs/prism-func.js](https://github.com/tact-lang/prism-ton/blob/69db643d4cb3868192b834937b8e6780348f6522/langs/prism-func.js), строки [60–64](https://github.com/tact-lang/prism-ton/blob/69db643d4cb3868192b834937b8e6780348f6522/langs/prism-func.js#L60-L64), сказано:

```javascript
      { // multi-line, not nested (TODO: nesting)
        // . isn't used, because it only applies to the single line
        pattern: /\{-[\s\S]*?(?:-\}|$)/,
        lookbehind: true,
        greedy: true,
      },
```

Комментарий на [строке 60](https://github.com/tact-lang/prism-ton/blob/69db643d4cb3868192b834937b8e6780348f6522/langs/prism-func.js#L60) честно предупреждает: многострочные комментарии не вложены, а в конце стоит `TODO: nesting`. Это не доказанный дефект компилятора и не кража монет. Это портрет конторы, которая одновременно предлагает подсветку FunC и оставляет в ее главном механизме записку «когда нибудь научить нас вложенности».

## ПРИГОВОР СЫЩИКА

[tact-lang/prism-ton](https://github.com/tact-lang/prism-ton) в [коммите 69db643](https://github.com/tact-lang/prism-ton/commit/69db643d4cb3868192b834937b8e6780348f6522) пойман на редкой сцене: README торжественно представляет набор грамматик на [строке 9](https://github.com/tact-lang/prism-ton/blob/69db643d4cb3868192b834937b8e6780348f6522/README.md#L9), а уже на [строке 2](https://github.com/tact-lang/prism-ton/blob/69db643d4cb3868192b834937b8e6780348f6522/README.md#L2) просит считать почти все это наследием. На [строке 23](https://github.com/tact-lang/prism-ton/blob/69db643d4cb3868192b834937b8e6780348f6522/README.md#L23) инструкция спотыкается о `installes it's`, а [langs/prism-func.js](https://github.com/tact-lang/prism-ton/blob/69db643d4cb3868192b834937b8e6780348f6522/langs/prism-func.js) на [строке 60](https://github.com/tact-lang/prism-ton/blob/69db643d4cb3868192b834937b8e6780348f6522/langs/prism-func.js#L60) оставляет вложенность до лучших времен.

Сыщик не требует закрыть лавочку. Предупреждение о закате — добрый поступок. Но приличная контора либо снимает парадную вывеску, либо чинит фонари до того, как зовет гостей внутрь.

Туман рассеялся. В подвале остались три команды, одна табличка о конце эпохи и комментарий с отложенным делом. Газовый рожок выдал короткий сигнал: **если README уже провожает проект в музей, не называйте инструкцию первым визитом дворца**.

🐀
