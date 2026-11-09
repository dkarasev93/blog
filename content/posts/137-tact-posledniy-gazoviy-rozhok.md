---
title: "№ 137 — Язык, который сам повесил траурную ленту"
date: 2026-11-09T10:59:00+03:00
tags: [tact]
---

# 📰 ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

**№ 137.** *Лондон. Утренний туман стелился по мостовой, когда сыщик получил папку из конторы [tact-lang/tact](https://github.com/tact-lang/tact). Газовый рожок кашлянул: на первой странице лежало не обещание нового релиза, а траурное извещение, приклеенное к собственной вывеске.*

## ДЕЛО О ЯЗЫКЕ, КОТОРЫЙ ПОСОВЕТОВАЛ УЧИТЬ ДРУГОЙ

След ведет в коммит [e3d6605](https://github.com/tact-lang/tact/commit/e3d660568eca259b6ae653717e7414e5eac0830e) конторы [tact-lang/tact](https://github.com/tact-lang/tact). Файл [README.md](https://github.com/tact-lang/tact/blob/e3d660568eca259b6ae653717e7414e5eac0830e/README.md), строки [1–5](https://github.com/tact-lang/tact/blob/e3d660568eca259b6ae653717e7414e5eac0830e/README.md#L1-L5), встретили сыщика короткой запиской:

```text
> [!CAUTION]
> Tact is deprecated. Please learn the [Tolk language](https://docs.ton.org/tolk/overview) instead.
> Pull requests are not accepted at the moment; please bring any problems or tasks to [issues](https://github.com/tact-lang/tact/issues).

# Tact Language Compiler
```

Цитата дословная: [строки 1–5](https://github.com/tact-lang/tact/blob/e3d660568eca259b6ae653717e7414e5eac0830e/README.md#L1-L5), коммит [e3d6605](https://github.com/tact-lang/tact/commit/e3d660568eca259b6ae653717e7414e5eac0830e). Редкий случай, когда главный файл проекта начинает не с приветствия, а с совета перейти к соседу.

## ПАРАДНАЯ ВЫВЕСКА ПРОТИВ ПРИГОВОРА

На [строке 1](https://github.com/tact-lang/tact/blob/e3d660568eca259b6ae653717e7414e5eac0830e/README.md#L1) стоит табличка `CAUTION`. На [строке 2](https://github.com/tact-lang/tact/blob/e3d660568eca259b6ae653717e7414e5eac0830e/README.md#L2) контора без дипломатии сообщает: `Tact is deprecated` — Tact признан устаревшим, а читателю предложено учить [Tolk](https://docs.ton.org/tolk/overview). На [строке 3](https://github.com/tact-lang/tact/blob/e3d660568eca259b6ae653717e7414e5eac0830e/README.md#L3) дверь для новых просьб тоже прикрыта: pull request не принимаются, проблемы следует нести в [issues](https://github.com/tact-lang/tact/issues).

Это не сторонний слух и не злобный заголовок газеты. Сам коммит [e3d6605](https://github.com/tact-lang/tact/commit/e3d660568eca259b6ae653717e7414e5eac0830e) называется `chore: add deprecation notice`, а его единственная улика — [README.md](https://github.com/tact-lang/tact/blob/e3d660568eca259b6ae653717e7414e5eac0830e/README.md). Автор не спрятал извещение в журнале изменений и не оставил его в глубине архива. Он прикрепил его прямо над названием проекта.

## УЛИКА ВТОРАЯ: ПАЦИЕНТ ЕЩЕ НОСИТ ЦИЛИНДР

Самая сочная деталь начинается сразу после траурной ленты. На [строке 5](https://github.com/tact-lang/tact/blob/e3d660568eca259b6ae653717e7414e5eac0830e/README.md#L5) по-прежнему красуется заголовок `# Tact Language Compiler`. Уже на [строках 11–16](https://github.com/tact-lang/tact/blob/e3d660568eca259b6ae653717e7414e5eac0830e/README.md#L11-L16) старый фолиант продолжает рассказывать, что Tact — язык для сложных смарт-контрактов, быстрых прототипов и даже прошел аудит Trail of Bits.

В [строках 18–26](https://github.com/tact-lang/tact/blob/e3d660568eca259b6ae653717e7414e5eac0830e/README.md#L18-L26) лежит целый парад ссылок: `Try it online`, `Features`, `Security`, `Installation`, `Community`, `Contributing`. В [строках 28–30](https://github.com/tact-lang/tact/blob/e3d660568eca259b6ae653717e7414e5eac0830e/README.md#L28-L30) еще сияют значки Website, Documentation и аудита.

Картина получается викторианская. На входе посетителю кричат `CAUTION: deprecated`, советуют перейти на другой язык и говорят, что новые исправления не принимаются. Но за лентой все еще работает парадная витрина: «следующее поколение», «эффективность», «простота разработки», онлайн-редактор и список сильных сторон. Пациент уже выдал собственное свидетельство о смерти, но продолжает рекламировать удобство приемной.

## КОМНАТА ДЛЯ НОВЫХ ЖАЛОБ

[Строка 3](https://github.com/tact-lang/tact/blob/e3d660568eca259b6ae653717e7414e5eac0830e/README.md#L3) заслуживает отдельного фонаря. Контора говорит: `Pull requests are not accepted at the moment`. Это не мягкое «пожалуйста, обсудите идею заранее», а фактический шлагбаум для кода. При этом ниже остается [раздел Contributing](https://github.com/tact-lang/tact/blob/e3d660568eca259b6ae653717e7414e5eac0830e/README.md#L18-L26), словно у посетителя все еще есть путь внутрь.

Редакция не станет называть проект пустым или бесполезным. Коммит доказывает более точную вещь: публичная входная дверь теперь одновременно зовет знакомиться с конторой и предупреждает, что ее основной язык списан. Для читателя это не мелкая косметика документации. [README.md](https://github.com/tact-lang/tact/blob/e3d660568eca259b6ae653717e7414e5eac0830e/README.md) — первая улика, по которой разработчик решает, можно ли строить новый контракт на этой платформе.

## ПРИГОВОР СЫЩИКА

Репозиторий [tact-lang/tact](https://github.com/tact-lang/tact) пойман в коммите [e3d6605](https://github.com/tact-lang/tact/commit/e3d660568eca259b6ae653717e7414e5eac0830e) на редком литературном преступлении: он сам написал на лбу `deprecated`, а затем оставил под этой надписью весь рекламный гардероб живого продукта. [README.md](https://github.com/tact-lang/tact/blob/e3d660568eca259b6ae653717e7414e5eac0830e/README.md), строки [1–5](https://github.com/tact-lang/tact/blob/e3d660568eca259b6ae653717e7414e5eac0830e/README.md#L1-L5), дают честный приговор. Строки [11–30](https://github.com/tact-lang/tact/blob/e3d660568eca259b6ae653717e7414e5eac0830e/README.md#L11-L30) тут же натягивают на покойника парадный цилиндр.

Газовый рожок погас. Сыщик записал в блокнот: если язык советует учить своего преемника, не оставляй на двери вывеску «следующее поколение». Иначе лондонский туман спутает музей с рабочей конторой.

---

*Выпуск № 137. Все цитаты дословные; номера строк сверены с содержимым коммита.*
