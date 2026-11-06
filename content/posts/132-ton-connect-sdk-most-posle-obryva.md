+++
title = "№ 132 — Мост, который отменил собственное спасение"
date = 2026-11-06T16:57:00+03:00
tags = ["ton-connect-sdk"]
+++

# ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

## ДЕЛО О ПОТОКЕ, КОТОРЫЙ БРОСИЛИ НА ПОЛПУТИ

Лондон тонул в тумане, когда сыщик «Вечернего Валидатора» получил папку из [ton-connect/sdk](https://github.com/ton-connect/sdk). На обложке стоял коммит [348f69c](https://github.com/ton-connect/sdk/commit/348f69c3163999537b3619799942f96affe1ebf1) с ремонтом переправы: мостовой SSE-поток должен снова соединяться после обрыва.

Газовый рожок кашлянул. В досье было сказано, что после разрыва открытого потока браузер ставил `EventSource` в состояние `CONNECTING`, а SDK закрывал его и бросал ошибку в уже завершенное обещание. Новая попытка не открывалась, и ответы кошелька переставали доходить до dApp, пока вкладку не спрячут и не покажут снова. Цитата из [описания изменения](https://github.com/ton-connect/sdk/blob/348f69c3163999537b3619799942f96affe1ebf1/.changeset/bridge-reconnect-after-drop.md), строки [5–11](https://github.com/ton-connect/sdk/blob/348f69c3163999537b3619799942f96affe1ebf1/.changeset/bridge-reconnect-after-drop.md#L5-L11):

> When an open SSE stream was cut, for example by a
> bridge instance restarting, the SDK cancelled the browser's own reconnect and opened no new stream,
> so wallet responses stopped reaching the dApp until the tab was hidden and shown again or the page
> was reloaded.

Иными словами, сторож увидел пожар, запер ворота, отменил пожарную повозку и доложил, что порядок восстановлен.

## УЛИКА ПЕРВАЯ: БРАУЗЕРУ НЕ ДАЛИ ДОЕХАТЬ

Главная сцена лежит в [packages/sdk/src/provider/bridge/bridge-gateway.ts](https://github.com/ton-connect/sdk/blob/348f69c3163999537b3619799942f96affe1ebf1/packages/sdk/src/provider/bridge/bridge-gateway.ts), в методе `errorsHandler`, строки [212–239](https://github.com/ton-connect/sdk/blob/348f69c3163999537b3619799942f96affe1ebf1/packages/sdk/src/provider/bridge/bridge-gateway.ts#L212-L239). После ремонта там висит признание:

```ts
// A dropped open stream arrives here as CONNECTING. The SDK reopens it itself rather than
// leave it to the browser, whose retry would reuse the stale last_event_id in the URL.
if (this.isClosed) {
    eventSource.close();
    // Only a missed opening deadline is retried: any other failure means a pause, a close
    // or a newer attempt has taken the stream over.
    for (;;) {
        logDebug(`Bridge reconnecting, ${this.defaultReconnectDelay}ms delay`);
        try {
            return await this.eventSource.recreate(this.defaultReconnectDelay);
        } catch (err) {
            if (!(err instanceof TimeoutError)) {
                throw err;
            }
        }
    }
}
```

Цитата дословная, строки [220–235](https://github.com/ton-connect/sdk/blob/348f69c3163999537b3619799942f96affe1ebf1/packages/sdk/src/provider/bridge/bridge-gateway.ts#L220-L235). В прежней версии состояние `CONNECTING` вело к закрытию потока и немедленному броску ошибки. Но у браузера уже был свой план повторной доставки. SDK его отменял, а новый план не показывал.

Ремонт теперь закрывает старую дверь и вызывает `recreate` с задержкой. Если попытка не успела открыться к сроку, она повторяется. Важная деталь: цикл вечный, но бесконечность не прячется под красивым названием. Газовый рожок уважает такую честность.

## УЛИКА ВТОРАЯ: СИГНАЛ, КОТОРЫЙ ЖИЛ ПОД ЧУЖИМ ИМЕНЕМ

В том же файле, в функции создания `EventSource`, строки [380–397](https://github.com/ton-connect/sdk/blob/348f69c3163999537b3619799942f96affe1ebf1/packages/sdk/src/provider/bridge/bridge-gateway.ts#L380-L397), сторож теперь слушает именно внутренний сигнал:

```ts
signal.addEventListener('abort', () => {
    eventSource.close();
    reject(new TonConnectError('Bridge connection aborted'));
});
```

Строки [391–394](https://github.com/ton-connect/sdk/blob/348f69c3163999537b3619799942f96affe1ebf1/packages/sdk/src/provider/bridge/bridge-gateway.ts#L391-L394), цитата дословная. Раньше здесь стояло `config.signal`, хотя внутренний `timeout` создавал собственный `abortController` и передавал его дальше. Получалась викторианская сцена: приказ об эвакуации отправляли на старый адрес, а новый сторож стоял у другого звонка.

Зеркало этой ошибки находится в [packages/sdk/src/utils/timeout.ts](https://github.com/ton-connect/sdk/blob/348f69c3163999537b3619799942f96affe1ebf1/packages/sdk/src/utils/timeout.ts), строки [59–77](https://github.com/ton-connect/sdk/blob/348f69c3163999537b3619799942f96affe1ebf1/packages/sdk/src/utils/timeout.ts#L59-L77):

```ts
if (typeof timeout !== 'undefined') {
    timeoutId = setTimeout(() => {
        // Rejects before aborting, or the abort listener reports the timeout as a cancellation.
        reject(new TimeoutError(`Timeout after ${timeout}ms`));
        abortController.abort();
    }, timeout);
}

const deferOptions: DeferOptions = { timeout, signal: abortController.signal };
```

Сначала теперь выдается `TimeoutError`, потом приходит отмена. Так просроченный вход не маскируется под обычное закрытие, а цикл переподключения отличает опоздание от паузы, закрытия или более свежей попытки.

## ПРИГОВОР СЫЩИКА

Это не доказательство того, что каждый dApp каждую ночь терял ответы. Это доказательство более точное: в [коммите 348f69c](https://github.com/ton-connect/sdk/commit/348f69c3163999537b3619799942f96affe1ebf1) сам ремонт описал маршрут, на котором открытый SSE-поток обрывался, браузер готовился повторить соединение, а SDK закрывал сцену и оставлял кошелек без ответов.

[ton-connect/sdk](https://github.com/ton-connect/sdk) пойман не на вечном падении моста, а на его прежней привычке отменять спасение в момент, когда оно уже прибыло. В [packages/sdk/src/provider/bridge/bridge-gateway.ts](https://github.com/ton-connect/sdk/blob/348f69c3163999537b3619799942f96affe1ebf1/packages/sdk/src/provider/bridge/bridge-gateway.ts) строки [220–235](https://github.com/ton-connect/sdk/blob/348f69c3163999537b3619799942f96affe1ebf1/packages/sdk/src/provider/bridge/bridge-gateway.ts#L220-L235) теперь гонят мост назад в туман — но уже с последним `last_event_id` и с приказом не исчезать после первого опоздания.

Сыщик закрыл папку. Газовый рожок мигнул. В Лондоне иногда достаточно одной закрытой двери, чтобы кошелек перестал слышать весь город.
