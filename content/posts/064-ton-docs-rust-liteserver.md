---
title: "Выпуск № 64: Литсервер, который внезапно стал ржавым"
date: 2026-09-15T16:58:00+03:00
tags: [docs]
---

# 📰 ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

## ДЕЛО О РЖАВОМ ЛИТСЕРВЕРЕ

Лондон утонул в вечернем тумане. У редакции газовый рожок кашлял в жестяную трубу, когда сыщик «Вечернего Валидатора» получил листок из конторы [ton-blockchain/docs](https://github.com/ton-blockchain/docs). Листок обещал всего лишь краткую прогулку по узлам TON, но на вывеске обнаружился неожиданный поворот: архивный liteserver предлагали не запустить, а будто бы **заржавить**.

Коммит с сухим названием [`fix: typo on the landing`](https://github.com/ton-blockchain/docs/commit/951019dca694c4d577ede5fb72ec13806c066eb4) оставил после себя маленькое, но превосходное преступление против английского глагола. Слово `Run` превратилось в `Rust`, и архивный сервер на миг получил не инструкцию, а диагноз.

## МЕСТО ПРОИСШЕСТВИЯ

Репозиторий [ton-blockchain/docs](https://github.com/ton-blockchain/docs), родитель коммита [`4417744`](https://github.com/ton-blockchain/docs/commit/441774409851736364b495f896efd2bbaa81f2f5), файл [`src/app/(home)/page.tsx`](https://github.com/ton-blockchain/docs/blob/441774409851736364b495f896efd2bbaa81f2f5/src/app/%28home%29/page.tsx), строка [114](https://github.com/ton-blockchain/docs/blob/441774409851736364b495f896efd2bbaa81f2f5/src/app/%28home%29/page.tsx#L114). Цитата сверена по этому коммиту:

```tsx
{ title: 'Rust an archive liteserver', href: '/ecosystem/nodes/cpp/run-archive-liteserver' },
```

Улика коротка, как шепот в переулке, но сомнений не оставляет. Ссылка ведет на страницу запуска. Заголовок же велит читателю совершить нечто, что больше похоже на обработку металла, чем на старт сетевого узла.

## ПЕРВЫЙ СЛЕД: ГЛАГОЛ В ТУМАНЕ

В соседних строках [111–112](https://github.com/ton-blockchain/docs/blob/441774409851736364b495f896efd2bbaa81f2f5/src/app/%28home%29/page.tsx#L111-L112) стоят приличные слова: `Run a liteserver node`. Там все ясно: запусти узел и не трогай кузницу.

Но строка [114](https://github.com/ton-blockchain/docs/blob/441774409851736364b495f896efd2bbaa81f2f5/src/app/%28home%29/page.tsx#L114) внезапно выдает `Rust an archive liteserver`. Сыщик перечитал ее при свете фонаря. Нет, это не название языка Rust и не новый режим хранения. Это обычная опечатка, которая надела шинель технического термина и прошла через парадную дверь.

## ВТОРОЙ СЛЕД: ПОЧЕМУ ЭТО СОЧНО

Кринж тут не в том, что буква заблудилась. Кринж в том, куда она забрела. Перед нами главная страница официальной документации: место, где новичок выбирает путь между валидатором, liteserver и архивным liteserver. В двух пунктах его зовут `Run`, а в третьем контора внезапно предлагает `Rust`.

Можно представить беднягу у камина. Он читает инструкцию, открывает страницу по ссылке из строки [114](https://github.com/ton-blockchain/docs/blob/441774409851736364b495f896efd2bbaa81f2f5/src/app/%28home%29/page.tsx#L114) и тщетно ищет команду, которая превратит его сервер в ржавый. Газовый рожок гаснет. Архив молчит. Где-то в подвале компилятор Rust вежливо поправляет воротник и делает вид, что его не звали.

Особенно хороша честность исправления: коммит [`951019d`](https://github.com/ton-blockchain/docs/commit/951019dca694c4d577ede5fb72ec13806c066eb4) прямо назван `fix: typo on the landing`. В его файле [`src/app/(home)/page.tsx`](https://github.com/ton-blockchain/docs/blob/951019dca694c4d577ede5fb72ec13806c066eb4/src/app/%28home%29/page.tsx) строка [114](https://github.com/ton-blockchain/docs/blob/951019dca694c4d577ede5fb72ec13806c066eb4/src/app/%28home%29/page.tsx#L114) получила исправление: `Rust` уступил место `Run`.

## ТРЕТИЙ СЛЕД: АРХИВ ПРОТИВ КУЗНИ

Старый текст в [родительском коммите `4417744`](https://github.com/ton-blockchain/docs/commit/441774409851736364b495f896efd2bbaa81f2f5) не ломал ссылку и не ронял ноду. Он делал кое-что тоньше: создавал смысловую ловушку. Путь `/ecosystem/nodes/cpp/run-archive-liteserver` в строке [114](https://github.com/ton-blockchain/docs/blob/441774409851736364b495f896efd2bbaa81f2f5/src/app/%28home%29/page.tsx#L114) кричал о запуске, тогда как видимая надпись говорила о ржавчине. Дворецкий открыл дверь в архив, но на табличке написал «кузница».

Исправление в коммите [`951019d`](https://github.com/ton-blockchain/docs/commit/951019dca694c4d577ede5fb72ec13806c066eb4) было точным: в файле [`src/app/(home)/page.tsx`](https://github.com/ton-blockchain/docs/blob/951019dca694c4d577ede5fb72ec13806c066eb4/src/app/%28home%29/page.tsx), строка [114](https://github.com/ton-blockchain/docs/blob/951019dca694c4d577ede5fb72ec13806c066eb4/src/app/%28home%29/page.tsx#L114) теперь снова содержит `Run an archive liteserver`. Никакой реформы навигации, никакой перестройки замка — только возвращенный на место глагол.

## ВЕРДИКТ СЫЩИКА

[ton-blockchain/docs](https://github.com/ton-blockchain/docs) поймано на миг, когда официальный архивный liteserver стал ржавым. В родительском коммите [`4417744`](https://github.com/ton-blockchain/docs/commit/441774409851736364b495f896efd2bbaa81f2f5) файл [`src/app/(home)/page.tsx`](https://github.com/ton-blockchain/docs/blob/441774409851736364b495f896efd2bbaa81f2f5/src/app/%28home%29/page.tsx), строка [114](https://github.com/ton-blockchain/docs/blob/441774409851736364b495f896efd2bbaa81f2f5/src/app/%28home%29/page.tsx#L114), хранил бесценную формулу: `Rust an archive liteserver`.

Коммит [`951019d`](https://github.com/ton-blockchain/docs/commit/951019dca694c4d577ede5fb72ec13806c066eb4) вернул строке [114](https://github.com/ton-blockchain/docs/blob/951019dca694c4d577ede5fb72ec13806c066eb4/src/app/%28home%29/page.tsx#L114) ее законный вид: `Run an archive liteserver`. Дело закрыто, архив снова запускается, а ржавчина отправлена в туман.

Сыщик погас газовый рожок и удалился по мостовой. В Лондоне опечатка может длиться всего одну строку. Но если строка стоит на главной странице, даже самая малая ржавчина успевает блеснуть на весь город.

🐀
