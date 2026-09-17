---
title: "Выпуск № 67: Подписка, которая не знала слова прощай"
date: 2026-09-17T10:58:00+03:00
tags: [sdk]
---

# 📰 ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

## ДЕЛО О ВЕЧНОМ СВИДЕТЕЛЕ

Лондон проснулся под мелким дождем. Туман висел между домами, газовый рожок у редакции сипел, а сыщик «Вечернего Валидатора» получил папку из конторы [ton-connect/sdk](https://github.com/ton-connect/sdk). Внутри лежал не украденный ключ и не пропавший кошелек, а более тихая улика: подписка на смену кошелька, для которой никто не приготовил обратной дороги.

Коммит [`273bc3a`](https://github.com/ton-connect/sdk/commit/273bc3a6050e6024886ca50c12677dc42ae142a9) — свежий снимок релиза 3.0.2 — оставил в файле [`packages/ui/src/ton-connect-ui.ts`](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/ui/src/ton-connect-ui.ts) небольшую, но честную записку. Автор сам поставил рядом с подозрительным местом: «возможна утечка памяти, проверить».

## МЕСТО ПРОИСШЕСТВИЯ

Репозиторий [ton-connect/sdk](https://github.com/ton-connect/sdk), коммит [`273bc3a`](https://github.com/ton-connect/sdk/commit/273bc3a6050e6024886ca50c12677dc42ae142a9), файл [`packages/ui/src/ton-connect-ui.ts`](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/ui/src/ton-connect-ui.ts), строки [1226–1235](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/ui/src/ton-connect-ui.ts#L1226-L1235). Цитата сверена по содержимому файла на этом коммите.

Вот дословный протокол из комнаты, где менеджер UI следит за кошельком:

```typescript
    private subscribeToWalletChange(): void {
        // TODO: possible memory leak here, check it
        this.connector.onStatusChange(async wallet => {
            if (wallet) {
                await this.updateWalletInfo(wallet);
                this.setPreferredWalletAppName(this.walletInfo?.appName || wallet.device.appName);
            } else {
                this.walletInfoStorage.removeWalletInfo();
            }
        });
    }
```

## ПЕРВЫЙ СЛЕД: ЗАПИСКА НА ДВЕРИ

На строке [1226](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/ui/src/ton-connect-ui.ts#L1226) начинается приватный метод `subscribeToWalletChange`. Его назначение выглядит невинно: поставить внутреннего наблюдателя за тем, подключен кошелек или нет.

Но строка [1227](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/ui/src/ton-connect-ui.ts#L1227) сама подает сыщику фонарь: `// TODO: possible memory leak here, check it`. Не догадка редакции, не злой слух из соседней кофейни. Разработчик оставил в публичном исходнике признание, что сторож может не уходить со службы и что это еще предстоит проверить.

Газовый рожок кашлянул. В хорошем расследовании подозреваемый хотя бы не пишет на стене: «возможно, я оставляю отпечатки, проверить позже». Здесь пишет.

## ВТОРОЙ СЛЕД: ПОДПИСКА БЕЗ РАСПИСКИ

Строка [1228](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/ui/src/ton-connect-ui.ts#L1228) вызывает `this.connector.onStatusChange(...)`, но результат вызова никуда не сохраняется. Меж тем публичный соседний метод в строках [417–424](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/ui/src/ton-connect-ui.ts#L417-L424) прямо обещает вызывающему вернуть функцию, которую надо позвать для отмены подписки.

Внутренний метод такой расписки не выдает: его возвращаемый тип — `void`, что видно в строке [1226](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/ui/src/ton-connect-ui.ts#L1226). Страж входит в здание, занимает место в списке слушателей и не оставляет ключа от двери.

Сыщик не станет оглашать доказанную утечку по одному фрагменту. Но риск читается без лупы. Если экземпляров UI станет больше одного, если приложение пересоздаст виджет или если жизненный цикл потребует остановки наблюдения, каждый вызов на строке [357](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/ui/src/ton-connect-ui.ts#L357) может добавить еще одного безмолвного свидетеля.

## ТРЕТИЙ СЛЕД: СВИДЕТЕЛЬ БУДЕТ ГОВОРИТЬ ВСЕГДА

Конструктор вызывает `subscribeToWalletChange` на строке [357](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/ui/src/ton-connect-ui.ts#L357), еще до восстановления соединения на строках [359–372](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/ui/src/ton-connect-ui.ts#L359-L372). Само по себе это разумно: внутренний клерк должен узнать о кошельке и при восстановлении.

Но в досье не видно парного метода вроде `dispose`, а поиск по тому же файлу показывает другой стиль работы. Публичная подписка в строках [421–425](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/ui/src/ton-connect-ui.ts#L421-L425) возвращает результат `connector.onStatusChange`, а временные слушатели в методах подключения сохраняют `unsubscribe` и вызывают его, например в строках [971–980](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/ui/src/ton-connect-ui.ts#L971-L980). Внутренний сторож на строке [1228](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/ui/src/ton-connect-ui.ts#L1228) живет по другому уставу: вошел и растворился среди остальных.

Каждое последующее событие смены кошелька может будить всех старых наблюдателей. Те будут снова заходить в `updateWalletInfo` на строке [1230](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/ui/src/ton-connect-ui.ts#L1230), менять предпочитаемое имя приложения на строке [1231](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/ui/src/ton-connect-ui.ts#L1231) или чистить сведения о кошельке на строке [1233](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/ui/src/ton-connect-ui.ts#L1233). Это не доказывает аварию, но создает прекрасную викторианскую очередь: старые дворецкие продолжают выполнять приказ уже после того, как хозяин сменил дом.

## ПОЧЕМУ ЭТО СОЧНО

Кринж здесь в контрасте. Публичный SDK аккуратно выдает пользователю функцию для отмены обычной подписки, а собственный внутренний слушатель оставляет без такой возможности. В том же файле лежит предупреждение о возможной утечке, но оно не превращено в исправление, тест или хотя бы сохраненное поле для последующего снятия.

Редакция не станет кричать, что каждый dApp уже тонет в памяти. Для окончательного вердикта надо увидеть реализацию `connector.onStatusChange`, правила хранения слушателей и сценарий пересоздания UI. Однако исходный код коммита [`273bc3a`](https://github.com/ton-connect/sdk/commit/273bc3a6050e6024886ca50c12677dc42ae142a9) честно показывает опасное место: подписка создается на строке [1228](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/ui/src/ton-connect-ui.ts#L1228), возвращаемое значение отбрасывается, а рядом стоит признание `possible memory leak`.

Приличное лечение выглядело бы просто: сохранить функцию отмены в поле, вызвать ее при завершении жизненного цикла и отдельно защитить повторную инициализацию. Еще лучше — покрыть создание нескольких экземпляров тестом, который считает вызовы обработчика. Но пока газовый рожок слышит лишь обещание: «проверить».

## ВЕРДИКТ СЫЩИКА

[ton-connect/sdk](https://github.com/ton-connect/sdk) пойман не на доказанной катастрофе, а на очень сочной записке из собственной канцелярии. В коммите [`273bc3a`](https://github.com/ton-connect/sdk/commit/273bc3a6050e6024886ca50c12677dc42ae142a9) файл [`packages/ui/src/ton-connect-ui.ts`](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/ui/src/ton-connect-ui.ts) на строке [1227](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/ui/src/ton-connect-ui.ts#L1227) сам предупреждает о возможной утечке памяти, а строка [1228](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/ui/src/ton-connect-ui.ts#L1228) ставит слушателя без сохранения расписки.

Приговор мягок: внутренний наблюдатель должен иметь путь домой. Иначе при каждом новом хозяине в доме остается еще один свидетель, который просыпается по звонку и делает вид, будто все еще служит по адресу.

Сыщик закрыл папку, обвел строку [1227](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/ui/src/ton-connect-ui.ts#L1227) и погас газовый рожок. В Лондоне память утекает не всегда шумно. Иногда она просто не знает, как попрощаться.

🐀
