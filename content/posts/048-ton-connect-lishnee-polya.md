+++
title = "№ 48 — Дело о лишнем поле: TON Connect впустил контрабанду в ton_proof"
date = 2026-09-06T16:58:53+03:00
description = "Сорок восьмой выпуск «Вечернего Валидатора»: ton-connect/sdk исправляет дыру в проверке ton_proof, где внутренняя структура proof пропускала неизвестные поля, а Telegram-ветка отбрасывала почти всех пользователей без Premium."
tags = ["ton-connect"]
+++

# 📰 ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

**№ 48 · Воскресенье, 6 сентября 2026 г. · Цена: 0.05 TON (за каждое поле доплата)**

---

## ДЕЛО О КОНВЕРТЕ С ЛИШНЕЙ БУМАГОЙ

Лондон утонул в тумане, мостовая блестела, а газовый рожок у редакции сипел, словно его пригласили на проверку пропуска, но забыли спросить, кто его выдал. Сыщик «Вечернего Валидатора» получил свежую папку из конторы [ton-connect/sdk](https://github.com/ton-connect/sdk) — набора инструментов, через который приложения договариваются с кошельками TON.

Коммит [`3760b0c`](https://github.com/ton-connect/sdk/commit/3760b0c8d260c33d69bcf618e9787a8813a744da) пришел с бодрым заголовком: `fix: report non-premium Telegram users and close ton_proof validation gap (#583)`. Внутри лежит двойное признание. Одна дверь не проверяла лишние поля в доказательстве `ton_proof`. Другая требовала от Telegram-пользователя признак Premium, который обычный пользователь мог вообще не получить.

Место происшествия: репозиторий [ton-connect/sdk](https://github.com/ton-connect/sdk), коммит [`3760b0c`](https://github.com/ton-connect/sdk/commit/3760b0c8d260c33d69bcf618e9787a8813a744da), файл [`packages/sdk/src/validation/schemas.ts`](https://github.com/ton-connect/sdk/blob/3760b0c8d260c33d69bcf618e9787a8813a744da/packages/sdk/src/validation/schemas.ts), строки [615–630](https://github.com/ton-connect/sdk/blob/3760b0c8d260c33d69bcf618e9787a8813a744da/packages/sdk/src/validation/schemas.ts#L615-L630). Цитата сверена с содержимым файла на этом коммите.

Протокол, файл [`packages/sdk/src/validation/schemas.ts`](https://github.com/ton-connect/sdk/blob/3760b0c8d260c33d69bcf618e9787a8813a744da/packages/sdk/src/validation/schemas.ts), строки [615–629](https://github.com/ton-connect/sdk/blob/3760b0c8d260c33d69bcf618e9787a8813a744da/packages/sdk/src/validation/schemas.ts#L615-L629):

```typescript
    if (hasProof) {
        const proof = (data as Record<string, unknown>).proof as
            | Record<string, unknown>
            | undefined;
        if (!isValidObject(proof)) {
            return "Invalid 'proof' object";
        }

        const allowedProofKeys = ['timestamp', 'domain', 'payload', 'signature'];
        if (hasExtraProperties(proof, allowedProofKeys)) {
            return 'ton_proof item contains extra properties';
        }

        if (!isValidNumber(proof.timestamp)) {
            return "Invalid 'proof.timestamp'";
        }
```

## СТАРАЯ ОХРАНА УЖЕ ПРОВЕРИЛА КОНВЕРТ

На строке [615](https://github.com/ton-connect/sdk/blob/3760b0c8d260c33d69bcf618e9787a8813a744da/packages/sdk/src/validation/schemas.ts#L615) охрана заходит в ветку `hasProof`. Строки [616–618](https://github.com/ton-connect/sdk/blob/3760b0c8d260c33d69bcf618e9787a8813a744da/packages/sdk/src/validation/schemas.ts#L616-L618) достают поле `proof` и считают его картой значений. На строках [619–621](https://github.com/ton-connect/sdk/blob/3760b0c8d260c33d69bcf618e9787a8813a744da/packages/sdk/src/validation/schemas.ts#L619-L621) проверяется, что перед нами вообще словарь.

Все выглядит чинно, пока сыщик не заглянул внутрь конверта. До коммита [`3760b0c`](https://github.com/ton-connect/sdk/commit/3760b0c8d260c33d69bcf618e9787a8813a744da) там не стоял отдельный список допустимых ключей. Проверки существовали для самой посылки и для соседней ошибки, но `proof` оказался особым джентльменом: его форму осмотрели, а содержимое карманов — нет.

Именно поэтому строки [623–625](https://github.com/ton-connect/sdk/blob/3760b0c8d260c33d69bcf618e9787a8813a744da/packages/sdk/src/validation/schemas.ts#L623-L625) выглядят как свежая решетка на окне. В список вписаны только `timestamp`, `domain`, `payload` и `signature`. Все остальное получает вердикт `ton_proof item contains extra properties`.

Газовый рожок кашлянул. Четыре законных предмета пропускают, пятый — неизвестный — отправляют обратно в туман. Но сам факт того, что коммит называет это `validation gap`, оставляет в досье прелестную сцену: проверка существует, проверка работает, а одна вложенная комната до последнего времени жила по правилу «не трогайте, там ничего нет».

## ПОЧЕМУ ЛИШНИЙ КЛЮЧ НЕ ПРОСТО УКРАШЕНИЕ

Редакция не станет считать всякое неизвестное поле немедленной катастрофой. Иногда форматы расширяют, а терпимый парсер помогает пережить обновление. Но `ton_proof` — это доказательство, которое проходит через границу между приложением и кошельком. Если схема обещает точный набор полей, лишняя записка внутри должна либо иметь смысл, либо быть отвергнута.

На строке [628](https://github.com/ton-connect/sdk/blob/3760b0c8d260c33d69bcf618e9787a8813a744da/packages/sdk/src/validation/schemas.ts#L628) после новой решетки начинается проверка `proof.timestamp`, а далее расследуются `domain`, `payload` и `signature`. То есть неизвестный ключ раньше мог пройти в комнату и остаться рядом с настоящими уликами, прежде чем охрана занялась обязательными полями. Новая проверка ставит общий порядок в начало коридора.

Сыщик ставит перо аккуратно: перед нами не доказательство того, что любой лишний ключ позволял подделать подпись. Из приведенных строк следует более узкий и надежный вывод: валидатор не отвергал неизвестные свойства внутри `proof`, хотя такая проверка уже была нужна и теперь добавлена. В нуаре даже маленькая щель заслуживает замера, особенно если на ней висит табличка «доказательство».

## ВТОРОЕ ДЕЛО: ПРЕМИУМ, КОТОРОГО НЕТ

В той же папке обнаружилась другая сцена. Файл [`packages/ui/src/app/utils/tma-api.ts`](https://github.com/ton-connect/sdk/blob/3760b0c8d260c33d69bcf618e9787a8813a744da/packages/ui/src/app/utils/tma-api.ts) на строках [132–148](https://github.com/ton-connect/sdk/blob/3760b0c8d260c33d69bcf618e9787a8813a744da/packages/ui/src/app/utils/tma-api.ts#L132-L148) разбирает данные Telegram. Старый привратник хотел увидеть и числовой идентификатор, и булево поле `is_premium`.

Коммит [`3760b0c`](https://github.com/ton-connect/sdk/commit/3760b0c8d260c33d69bcf618e9787a8813a744da) оставил в файле честный комментарий. Цитата из [`packages/ui/src/app/utils/tma-api.ts`](https://github.com/ton-connect/sdk/blob/3760b0c8d260c33d69bcf618e9787a8813a744da/packages/ui/src/app/utils/tma-api.ts), строки [137–144](https://github.com/ton-connect/sdk/blob/3760b0c8d260c33d69bcf618e9787a8813a744da/packages/ui/src/app/utils/tma-api.ts#L137-L144):

```typescript
            let user = JSON.parse(userRaw);
            // Telegram omits is_premium entirely for non-premium users, so it cannot
            // be required here without dropping most of the audience.
            if (typeof user.id === 'number') {
                telegramUser = {
                    id: user.id,
                    isPremium: user.is_premium === true
```

Строки [138–140](https://github.com/ton-connect/sdk/blob/3760b0c8d260c33d69bcf618e9787a8813a744da/packages/ui/src/app/utils/tma-api.ts#L138-L140) описывают уловку без дымовой завесы: Telegram не присылает `is_premium` для пользователей без Premium. Если требовать его наличие, дверь захлопнется перед большей частью публики.

Поэтому строка [140](https://github.com/ton-connect/sdk/blob/3760b0c8d260c33d69bcf618e9787a8813a744da/packages/ui/src/app/utils/tma-api.ts#L140) теперь проверяет только числовой `user.id`. А строка [143](https://github.com/ton-connect/sdk/blob/3760b0c8d260c33d69bcf618e9787a8813a744da/packages/ui/src/app/utils/tma-api.ts#L143) выставляет `isPremium` в `true` только при явном `user.is_premium === true`. Нет поля — значит, не Premium, а не исчезновение человека из реестра.

Викторианская картина готова. На одной двери неизвестный ключ проходил в доказательство, на другой обычный посетитель не проходил без золотой карточки, которую ему никто не выдавал. Одна ошибка слишком доверяла содержимому конверта, другая слишком верила отсутствующей строке. Обе исправлены тем же коммитом [`3760b0c`](https://github.com/ton-connect/sdk/commit/3760b0c8d260c33d69bcf618e9787a8813a744da).

## ВЕРДИКТ СЫЩИКА

[ton-connect/sdk](https://github.com/ton-connect/sdk) — полезная контора, а не разбойничий притон. Но коммит [`3760b0c`](https://github.com/ton-connect/sdk/commit/3760b0c8d260c33d69bcf618e9787a8813a744da) принес сочную двойную улику: в файле [`packages/sdk/src/validation/schemas.ts`](https://github.com/ton-connect/sdk/blob/3760b0c8d260c33d69bcf618e9787a8813a744da/packages/sdk/src/validation/schemas.ts) строки [623–625](https://github.com/ton-connect/sdk/blob/3760b0c8d260c33d69bcf618e9787a8813a744da/packages/sdk/src/validation/schemas.ts#L623-L625) добавляют проверку лишних свойств в `proof`, а в файле [`packages/ui/src/app/utils/tma-api.ts`](https://github.com/ton-connect/sdk/blob/3760b0c8d260c33d69bcf618e9787a8813a744da/packages/ui/src/app/utils/tma-api.ts) строки [138–143](https://github.com/ton-connect/sdk/blob/3760b0c8d260c33d69bcf618e9787a8813a744da/packages/ui/src/app/utils/tma-api.ts#L138-L143) перестают путать отсутствие Premium с отсутствием пользователя.

Приговор редакции таков: схема должна решать, какие поля живут в доказательстве, а интеграция с Telegram — отличать «поле не прислали» от «человек не существует». Неизвестные поля не надо пускать на бал без регистрации. Но и отсутствие парадного знака не повод вычеркивать посетителя из списка.

*Сыщик закрыл папку, наклеил на конверт четыре разрешенных слова и погас газовую лампу. В тумане Лондона контрабанда часто начинается с маленького поля. А иногда весь преступный замысел — это просто охранник, который ждет несуществующую карточку.*

🐀
