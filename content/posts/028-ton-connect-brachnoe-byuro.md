+++
title = "№ 28 — Брачное бюро: записная книжка на восемьсот анкет, гадание по морганию окна и письмо, из которого вынули само письмо"
date = 2026-08-28T11:06:00+03:00
description = "Двадцать восьмой выпуск «Вечернего Валидатора»: сыщик ступает на порог брачного бюро ton-connect/sdk — конторы, что сводит заявки с кошельками. В сейфе бюро лежит записная книжка на восемьсот строк зашитых анкет на случай, если господин с config.ton.org не подаст знака; жених по имени Gram Wallet прописан на чужом мосту; есть ли кошелёк в доме, контора узнаёт гаданием: мигнёт ли окно за двести миллисекунд; а если послание к невесте длиннее тысячи двадцати четырёх букв — писарь вынимает из конверта само послание и отправляет конверт пустым."
tags = ["ton-connect"]
+++

# 📰 Вечерний Валидатор

**№ 28.** *Лондон. Туман нынче розоватый — в городе открыли новое брачное бюро, и туман подле него пропах лавандовой водой и чужими секретами. Наш корреспондент, обойдя зал прощания (выпуск № 27), двигательную палату (выпуск № 26) и телеграфную контору (выпуск № 25), постучал в заведение с вывеской [ton-connect/sdk](https://github.com/ton-connect/sdk) — контору, что сводит заявки с кошельками. Ремесло старое как мир: вот невеста — страница в браузере, вот жених — кошелёк, где-то за семью туманами, и надобно их познакомить так, чтобы никто не узнал, что они знакомы. Сыщик поднял воротник, вошёл — и первое, что бросилось ему в глаза, была не стойка приёмной, а сейф. Сейф с записной книжкой.*

*Место происшествия: репозиторий [ton-connect/sdk](https://github.com/ton-connect/sdk), коммит [`273bc3a`](https://github.com/ton-connect/sdk/commit/273bc3a6050e6024886ca50c12677dc42ae142a9) (master от 06.08.2026). Файлы: [`packages/sdk/src/wallets-list-manager.ts`](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/sdk/src/wallets-list-manager.ts), [`packages/sdk/src/resources/fallback-wallets-list.ts`](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/sdk/src/resources/fallback-wallets-list.ts), [`packages/ui/src/app/utils/web-api.ts`](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/ui/src/app/utils/web-api.ts), [`packages/ui/src/app/utils/url-strategy-helpers.ts`](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/ui/src/app/utils/url-strategy-helpers.ts), [`packages/sdk/src/ton-connect.ts`](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/sdk/src/ton-connect.ts), [`packages/sdk/src/provider/injected/injected-provider.ts`](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/sdk/src/provider/injected/injected-provider.ts). Все цитаты дословные.*

---

## СЕНСАЦИЯ ПЕРВАЯ: ЗАПИСНАЯ КНИЖКА НА ВОСЕМЬСОТ АНКЕТ

По уставу конторы список невест и женихов положено получать по почте — из городской конфигурационной палаты. Смотрим в штатное расписание, [`packages/sdk/src/wallets-list-manager.ts`](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/sdk/src/wallets-list-manager.ts), строки [41–47](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/sdk/src/wallets-list-manager.ts#L41-L47):

```ts
        if (isQaModeEnabled()) {
            this.walletsListSource =
                'https://raw.githubusercontent.com/ton-connect/wallets-list-staging/refs/heads/main/wallets-v2.json';
        } else {
            this.walletsListSource =
                options?.walletsListSource ?? 'https://config.ton.org/wallets-v2.json';
        }
```

Честным клиентам — анкеты с `config.ton.org`, а своим, в режиме контрольной закупки, — сырые листы прямиком из чернового склада на `raw.githubusercontent.com`, из репозитория с красноречивым именем `wallets-list-staging`. Два класса читателей у одного издания. Но подождите хмыкать, читатель, — главное дальше. Что делает клерк, если почта не пришла вовсе? Строки [143–146](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/sdk/src/wallets-list-manager.ts#L143-L146):

```ts
        } catch (e) {
            logError(e);
            walletsList = FALLBACK_WALLETS_LIST;
```

Молча, не моргнув глазом, клерк отпирает сейф и вынимает записную книжку — [`FALLBACK_WALLETS_LIST`](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/sdk/src/resources/fallback-wallets-list.ts), строка [3](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/sdk/src/resources/fallback-wallets-list.ts#L3), — восемьсот одна строка зашитых прямо в конторский устав анкет. Сеть лежит, конфигурационная палата горит — бюро всё равно сведёт вас с тем, кого сочтёт нужным. И уж коль скоро сыщик раскрыл книжку, он не удержался и полистал. Анкета на строках [30–46](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/sdk/src/resources/fallback-wallets-list.ts#L30-L46) достойна рамки:

```ts
        app_name: 'gramwallet',
        name: 'Gram Wallet',
        image: 'https://config.ton.org/assets/gramwallet.png',
        about_url: 'https://gramwallet.io',
        universal_url: 'https://connect.gramwallet.io',
        deepLink: 'gramwallet-tc://',
        bridge: [
            {
                type: 'js',
                key: 'gramwallet'
            },
            {
                type: 'sse',
                url: 'https://tonconnectbridge.mytonwallet.org/bridge/'
            }
        ],
```

Жених по имени Gram Wallet: портрет на городском конфигурационном стенде, визитная карточка своя, диплинк фамильный — а мост для тайных свиданий, строка [43](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/sdk/src/resources/fallback-wallets-list.ts#L43), ведёт на чужой фамильный домен: `tonconnectbridge.mytonwallet.org`. Сыщик переписал адрес в блокнот и поставил три вопросительных знака: то ли родня, то ли квартира съёмная, то ли невесту приводят знакомиться в дом соседа. Свидетельство о родстве в книжке отсутствует.

## СЕНСАЦИЯ ВТОРАЯ: ГАДАНИЕ ПО МОРГАНИЮ ОКНА

Как бюро узнаёт, состоялось ли свидание? Действительно ли невеста, кликнув по диплинку, очутилась в кошельке — или он вовсе не установлен и надобно послать запасную карету? Всякая порядочная контора спросила бы у операционной системы. Эта — гадает. [`packages/ui/src/app/utils/web-api.ts`](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/ui/src/app/utils/web-api.ts), строки [43–62](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/ui/src/app/utils/web-api.ts#L43-L62):

```ts
/**
 * Open a deeplink in the same tab and fallback to a direct link after 200 ms.
 * The fallback will not work on Safari, on Windows, or on Firefox for Android.
 * @param href
 * @param fallback
 */
export function openDeeplinkWithFallback(href: string, fallback: () => void): void {
    const doFallback = (): void => {
        if (isBrowser('safari') || (isOS('android') && isBrowser('firefox')) || isOS('windows')) {
            // Safari does not support fallback to direct link.
            // Windows does not support fallback to direct link.
            // Firefox on Android does not support fallback to direct link.
            return;
        }

        fallback();
    };
    const fallbackTimeout = setTimeout(() => doFallback(), 200);
    window.addEventListener('blur', () => clearTimeout(fallbackTimeout), { once: true });

    openLink(href, '_self');
```

Вникнем в ритуал, строки [60–62](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/ui/src/app/utils/web-api.ts#L60-L62). Заводится песочные часы на двести миллисекунд. На окно вешается соглядатай: если окно «моргнуло» — случился `blur`, — значит, под окном раскрылся кошелёк, и запасную карету гасим. Не моргнуло за двести тысячных — выкатываем карету. Наука, читатель, точная как гадание на кофейной гуще: моргание окна могло случиться от прихода телеграммы, от щелчка по соседней вкладке, от склеротического порыва ветра. А над Safari, Windows и Firefox на Android, строки [51–55](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/ui/src/app/utils/web-api.ts#L51-L55), висит и вовсе чёрный флаг: «запасной вариант здесь не работает» — и функция, дойдя до места, чинно делает `return`. Не сообщает невесте, что кареты не будет. Просто уходит в туман. Комментарий над ритуалом, строки [44–45](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/ui/src/app/utils/web-api.ts#L44-L45), написан с честностью надгробия: *«The fallback will not work on Safari, on Windows, or on Firefox for Android»*. Даты рождения и смерти не проставлены, но и без них ясно.

## СЕНСАЦИЯ ТРЕТЬЯ: ПИСЬМО, ИЗ КОТОРОГО ВЫНУЛИ ПИСЬМО

Послания между невестой и женихом ходят по воздуху — в адресной строке, вложенные прямо в ссылку, в виде зашифрованного пакета с кратким именем `e`. И вот в приёмной бюро сыщик разглядел регламент, от которого у него отвисла челюсть, [`packages/ui/src/app/utils/web-api.ts`](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/ui/src/app/utils/web-api.ts), строки [25–32](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/ui/src/app/utils/web-api.ts#L25-L32):

```ts
export function openLink(href: string, target = '_self'): void {
    if (href.length > MAX_LINK_LENGTH) {
        href = removeEmbeddedRequestFromUniversalLink(href);
    }
    // TODO: should be extracted to upper layer
    setLastOpenedLink({ link: href });
    logDebug('openLink', href, target);
    window.open(href, target, 'noopener noreferrer');
}
```

Читайте строки [26–28](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/ui/src/app/utils/web-api.ts#L26-L28) как приговор: если письмо длиннее тысячи двадцати четырёх букв — писарь вызывает [`removeEmbeddedRequestFromUniversalLink`](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/ui/src/app/utils/url-strategy-helpers.ts#L634), которая, по её же определению на строке [634](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/ui/src/app/utils/url-strategy-helpers.ts#L634), делает ровно одно: `removeParamsFromUniversalLink(universalLink, ['e'])` — вынимает из конверта параметр `e`. То есть само письмо. После чего конверт — уже пустой, со штемпелем и без содержимого — торжественно вскрывается перед адресатом, строка [32](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/ui/src/app/utils/web-api.ts#L32). Зачем жениху пустой конверт — в регламенте не сказано; видимо, чтобы полюбовался почерком. Сыщик записал: лимит длины — дело операционной системы и браузера, но ответственность за худобу письма бюро взяло на себя и исполняет её единственным доступным способом — лишением письма смысла.

А над всей этой линейкой висит служебная записка редкой красоты, строки [12–17](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/ui/src/app/utils/web-api.ts#L12-L17):

```ts
// NOTE: this duplicates `maxUrlLength` in
// packages/sdk/src/provider/bridge/bridge-provider.ts. The duplication is a
// consequence of an architectural shortcoming — the SDK owns universal-link
// generation (and enforces its own length cap there), while the UI layer
// independently re-checks the produced link before opening it. Until the
// length policy is centralised, both constants must be kept in sync.
```

«Здесь две одинаковые меры длины: одна на фабрике, другая на воротах. Это следствие архитектурного недостатка. Пока недостаток не устранён, обе меры просим держать в синхронизации». Сыщик перечитал дважды. Контора знает о бреши, описала брешь в пяти строках, виновника назвала по имени — «архитектурный недостаток», лицо абстрактное, неуловимое, — и повесила над брешью не доску, а записку. Лондонские дожди записку размокнут обязательно; вопрос лишь, кто первым придёт её переписывать.

## СЕНСАЦИЯ ЧЕТВЁРТАЯ: ПОРТНОЙ С ДЕФИСОМ

В подвальчике бюро сидит портной, штопающий телеграм-адреса. У телеграмных ссылок своя мода: последний параметр к хвосту `t.me` принято пришивать не амперсандом, как у всех приличных адресов, а дефисом. И вот портной кроит, [`packages/ui/src/app/utils/url-strategy-helpers.ts`](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/ui/src/app/utils/url-strategy-helpers.ts), строки [44–51](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/ui/src/app/utils/url-strategy-helpers.ts#L44-L51):

```ts
    const newUrl = addQueryParameter(url, 'ret', returnStrategy);

    if (!isTelegramUrl(url)) {
        return newUrl;
    }

    const lastParam = newUrl.slice(newUrl.lastIndexOf('&') + 1);
    return newUrl.slice(0, newUrl.lastIndexOf('&')) + '-' + encodeTelegramUrlParameters(lastParam);
```

Весь город получает параметр через `&`, а телеграмный адрес — через хирургию: отрезать строку по последнему амперсанду, прилепить дефис, зашифровать отрезанное. Работает портной, надо отдать должное, аккуратно — `lastIndexOf` бьёт точно по последнему шву. Но сама фигура умолчания восхитительна: сначала параметр честно приклеивается амперсандом, строка [44](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/ui/src/app/utils/url-strategy-helpers.ts#L44), чтобы тут же, строками [50–51](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/ui/src/app/utils/url-strategy-helpers.ts#L50-L51), быть оторванным и пришитым заново — иной ниткой. Над верстаком висит табличка, строки [26–28](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/ui/src/app/utils/url-strategy-helpers.ts#L26-L28): «TODO: refactor this method». Портной табличку читает каждое утро и качает головой: перекроить-то перекроим, да кто ж нам даст второй такой дефис.

---

## МЕЛКИМ ПОЧЕРКОМ

*Наш сыщик не смог уйти без приложений к протоколу.*

**Процедурный кабинет.** При восстановлении прошлых знакомств — то есть сеанса из хранилища — в протоколе процедуры, [`packages/sdk/src/ton-connect.ts`](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/sdk/src/ton-connect.ts), строка [471](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/sdk/src/ton-connect.ts#L471), стоит пометка: `// TODO: potentially race condition here`. «Возможно, здесь гонка». Клерк знает, что два запроса, строки [472–475](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/sdk/src/ton-connect.ts#L472-L475), мчатся в `Promise.all` впритык, честно фиксирует это в деле — и подписывает протокол. Сыщик одобряет честность и тревожится за клиентов.

**Фокусник для Сафари.** В том же [`web-api.ts`](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/ui/src/app/utils/web-api.ts), строки [101–105](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/ui/src/app/utils/web-api.ts#L101-L105), лечится мобильный Сафари национальным средством: если у тела страницы нет атрибута `ontouchstart` — ему приписывают пустой `ontouchstart=""`. Атрибут-пустышка, талисман, шрам от прошлых боёв с `:active`. Работает — не трожь.

**Самопровозглашение.** Спросить у жениха, он ли настоящий встроенный кошелёк, нельзя — спрашивают у него самого: [`injected-provider.ts`](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/sdk/src/provider/injected/injected-provider.ts), строки [47–49](https://github.com/ton-connect/sdk/blob/273bc3a6050e6024886ca50c12677dc42ae142a9/packages/sdk/src/provider/injected/injected-provider.ts#L47-L49), `return this.window[injectedWalletKey]!.tonconnect.isWalletBrowser;`. В мире, где любой скрипт на странице может надеть табличку `tonconnect` и прилепить к ней флажок `isWalletBrowser`, такая справка выдаётся по предъявлении таблички. Брачное бюро верит женихам на слово; впрочем, в этом городе верят все.

---

*Сыщик вышел из бюро в розоватый туман и долго смотрел, как невесты с пустыми конвертами спешат к мостам на чужих доменах, а над Сафари и Windows висит чёрный флаг. «Восемьсот анкет в сейфе, — записал он напоследок. — Почта может не дойти. Книжка — дойдёт всегда». Газовый рожок на углу мигнул — ровно на двести миллисекунд, как по уставу. Свидание состоялось. Запасная карета отменена. Кто сел в карету — бюро не знает, и знать не желает: для этого у него есть TODO.*
