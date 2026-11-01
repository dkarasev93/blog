---
title: "№ 122 — Расширение, которое полностью умело исчезнуть"
date: 2026-11-01T17:00:00+03:00
tags: [tolk-vscode]
---

# ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

## ДЕЛО О ПОЛНОМ ПОКРЫТИИ, КОТОРОЕ УШЛО В ИСТОРИЮ

Лондон тонул в тумане, когда сыщик «Вечернего Валидатора» получил папку из [ton-blockchain/tolk-vscode](https://github.com/ton-blockchain/tolk-vscode). На обложке обещали поддержку Tolk, знакомый синтаксис и светлое будущее. Но газовый рожок кашлянул: будущее оказалось в другом расширении, а прежнее оставили как исторический экспонат.

## МЕСТО ПРОИСШЕСТВИЯ

След ведет в [коммит 6e1eeb2](https://github.com/ton-blockchain/tolk-vscode/commit/6e1eeb2ab5b098026923024ebc3d66a2ec7ada66), в файл [README.md](https://github.com/ton-blockchain/tolk-vscode/blob/6e1eeb2ab5b098026923024ebc3d66a2ec7ada66/README.md), строки [1–8](https://github.com/ton-blockchain/tolk-vscode/blob/6e1eeb2ab5b098026923024ebc3d66a2ec7ada66/README.md#L1-L8). Цитата дословная:

```text
## ⚠️ DEPRECATION NOTICE

TOLK support has been moved to "TON" VS Code extension.  
This extension (tolk-vscode) is unpublished and no longer maintained.

**→ Proceed to [ton-language-server](https://github.com/ton-blockchain/ton-language-server)**

*↓ The below is left for history only*
```

Сыщик отметил формулировку на строке [4](https://github.com/ton-blockchain/tolk-vscode/blob/6e1eeb2ab5b098026923024ebc3d66a2ec7ada66/README.md#L4): расширение не просто заброшено, оно unpublished — снято с витрины. А строка [8](https://github.com/ton-blockchain/tolk-vscode/blob/6e1eeb2ab5b098026923024ebc3d66a2ec7ada66/README.md#L8) оставляет под ним табличку «для истории». Так обычно выглядит не триумфальный релиз, а музейная дверь, которую забыли запереть.

## ОБЕЩАНИЕ НА СТАРОЙ ВЫВЕСКЕ

Особенно сочен контраст с прежним сообщением. В том же [README.md](https://github.com/ton-blockchain/tolk-vscode/blob/6e1eeb2ab5b098026923024ebc3d66a2ec7ada66/README.md) строки [13–17](https://github.com/ton-blockchain/tolk-vscode/blob/6e1eeb2ab5b098026923024ebc3d66a2ec7ada66/README.md#L13-L17) сообщали:

```text
**Tolk** is a new language for writing smart contracts in TON. Think of Tolk as the "next‑generation FunC".
Tolk compiler is literally a fork of FunC compiler, introducing familiar syntax similar to TypeScript,
but leaving all low-level optimizations untouched.

This extension fully covers Tolk grammar and will be kept up to date with Tolk evolution.
```

На строке [17](https://github.com/ton-blockchain/tolk-vscode/blob/6e1eeb2ab5b098026923024ebc3d66a2ec7ada66/README.md#L17) стоит обещание полного покрытия и постоянного ухода. В шапке того же файла, строка [4](https://github.com/ton-blockchain/tolk-vscode/blob/6e1eeb2ab5b098026923024ebc3d66a2ec7ada66/README.md#L4), уже стоит приговор «no longer maintained». Слово «fully» пережило само расширение, как золоченая вывеска над пустой конторой.

## ПОВТОРНАЯ ПЕЧАТЬ В ТУМАНЕ

Внутри документа сыщик нашел и более мелкую, но выразительную улику. В [строках 47–51](https://github.com/ton-blockchain/tolk-vscode/blob/6e1eeb2ab5b098026923024ebc3d66a2ec7ada66/README.md#L47-L51) контора описывает автоматическое обнаружение стандартных функций:

```text
They are listed in `common.tolk` and related files, automatically discovered by Tolk compiler.

This extension automatically also discovers them automatically (to provide auto-completion and so on).
**Most likely, you don't have to do anything — it just works**. Below is an explanation.
```

Слово `automatically` появляется в одной строке дважды, а между ними еще прячется `also`. Это не преступление против цепи, но прекрасный отпечаток пальца: документация торопилась доказать, что все работает само, и сама же застряла в автоповторе. Строка [50](https://github.com/ton-blockchain/tolk-vscode/blob/6e1eeb2ab5b098026923024ebc3d66a2ec7ada66/README.md#L50) теперь звучит как инструкция из закрытого магазина.

## СВИДЕТЕЛЬ С ГАЗОВЫМ РОЖКОМ

Коммит [6e1eeb2](https://github.com/ton-blockchain/tolk-vscode/commit/6e1eeb2ab5b098026923024ebc3d66a2ec7ada66) не прячет следы: он добавляет в начало файла сообщение о конце проекта, но оставляет ниже старую рекламу. Читателю предлагают одновременно поверить, что расширение «полностью покрывает» язык, и перейти в [ton-language-server](https://github.com/ton-blockchain/ton-language-server), потому что прежнее расширение больше не поддерживается.

Сыщик не обвиняет авторов в обмане: честная табличка о закрытии лучше молчаливой гробницы. Кринж здесь другого сорта — рекламный текст продолжает сиять сразу под некрологом, а фраза про автоматическое обнаружение сама дважды обнаруживает одно и то же слово.

## ПРИГОВОР СЫЩИКА

[ton-blockchain/tolk-vscode](https://github.com/ton-blockchain/tolk-vscode) в [коммите 6e1eeb2](https://github.com/ton-blockchain/tolk-vscode/commit/6e1eeb2ab5b098026923024ebc3d66a2ec7ada66) пойман на чудесном викторианском противоречии. [README.md](https://github.com/ton-blockchain/tolk-vscode/blob/6e1eeb2ab5b098026923024ebc3d66a2ec7ada66/README.md) обещает полную поддержку в строке [17](https://github.com/ton-blockchain/tolk-vscode/blob/6e1eeb2ab5b098026923024ebc3d66a2ec7ada66/README.md#L17), а в строках [1–8](https://github.com/ton-blockchain/tolk-vscode/blob/6e1eeb2ab5b098026923024ebc3d66a2ec7ada66/README.md#L1-L8) сообщает, что лавочка закрыта. В строке [50](https://github.com/ton-blockchain/tolk-vscode/blob/6e1eeb2ab5b098026923024ebc3d66a2ec7ada66/README.md#L50) документ еще и дважды запускает одно автоматическое действие.

Туман рассеялся. Расширение отправили в историю, рекламу оставили на месте, а газовый рожок выдал короткий сигнал: **если продукт уже стал экспонатом, не заставляйте его README изображать живого дворецкого**.

🐀