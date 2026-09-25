+++
title = "№ 83 — График вебхуков, который фильтрует не тот тип"
date = 2026-09-25T17:00:00+03:00
description = "Восемьдесят третий выпуск «Вечернего Валидатора»: ton-console строит графики доставленных и проваленных вебхуков, но ищет поле type там, где типы обещали назвать operation."
tags = ["ton-console"]
+++

# ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

**№ 83.** *Лондон. Туман стелется по мостовой, газовый рожок у редакции сипит, а сыщик получил свежую папку из конторы [tonkeeper/ton-console](https://github.com/tonkeeper/ton-console). Внутри обещан график вебхуков: синие столбики для доставленных писем, красные для проваленных. Но счетчик у двери ищет пропуск с именем `type`, хотя канцелярия выдала каждому свидетелю только `operation`.*

## МЕСТО ПРОИСШЕСТВИЯ

Улика найдена в репозитории [tonkeeper/ton-console](https://github.com/tonkeeper/ton-console), в коммите [`8d87e48`](https://github.com/tonkeeper/ton-console/commit/8d87e4881307cb5514ee08eb41f0947dfe46c2fc), в файле [`src/features/tonapi/webhooks/model/queries.ts`](https://github.com/tonkeeper/ton-console/blob/8d87e4881307cb5514ee08eb41f0947dfe46c2fc/src/features/tonapi/webhooks/model/queries.ts). Коммит носит имя `wip: remove redirect`; цитаты сняты с этого снимка.

## СЕНСАЦИЯ: ГРАФИК СМОТРИТ В ПУСТОЙ КАРМАН

Функция [`mapWebhooksStatsToChartPoints`](https://github.com/tonkeeper/ton-console/blob/8d87e4881307cb5514ee08eb41f0947dfe46c2fc/src/features/tonapi/webhooks/model/queries.ts#L30-L50) получает статистику и должна выбрать один из двух рядов: `delivered` или `failed`. Но на строках [34–40](https://github.com/tonkeeper/ton-console/blob/8d87e4881307cb5514ee08eb41f0947dfe46c2fc/src/features/tonapi/webhooks/model/queries.ts#L34-L40) стоит такой допрос:

```typescript
  const items = stats.result
    .filter(item => {
      // NOTE: Accessing metric.type even though DTOStats defines metric.operation
      // This will likely return undefined and filter out all items until fixed
      const metricType = (item.metric as unknown as Record<string, string>)?.type;
      return metricType === type;
    })
```

Сначала сыщик остановился на строке [38](https://github.com/tonkeeper/ton-console/blob/8d87e4881307cb5514ee08eb41f0947dfe46c2fc/src/features/tonapi/webhooks/model/queries.ts#L38). `item.metric` превращают в безымянную запись, а затем у нее спрашивают `type`. Это не чтение поля, подтвержденного типами, а переодевание показания в чужой мундир.

Строка [39](https://github.com/tonkeeper/ton-console/blob/8d87e4881307cb5514ee08eb41f0947dfe46c2fc/src/features/tonapi/webhooks/model/queries.ts#L39) требует, чтобы найденное значение совпало с `delivered` или `failed`. Если API присылает только поле `operation`, `metricType` становится `undefined`, фильтр вычеркивает каждую запись, а график остается без единой точки.

## ВТОРОЙ СЛЕД: КАНЦЕЛЯРИЯ САМА НАПИСАЛА ПРЕДУПРЕЖДЕНИЕ

Это не догадка газеты о тонкостях сериализации. Автор оставил признание прямо перед функцией, строки [23–29](https://github.com/tonkeeper/ton-console/blob/8d87e4881307cb5514ee08eb41f0947dfe46c2fc/src/features/tonapi/webhooks/model/queries.ts#L23-L29):

```typescript
// TODO: CRITICAL BUG INVESTIGATION NEEDED
// The DTOStats type has metric: { operation?: string }, but this function filters by metric.type
// which doesn't exist in the type definition. This suggests either:
// 1. The API response format for webhook stats is different than general DTOStats
// 2. The DTOStats type definition is incomplete/incorrect for this endpoint
// The type assertion `as unknown as` was hiding this mismatch. Please verify actual API response.
```

Слова `CRITICAL BUG INVESTIGATION NEEDED` выглядят как крик из подвала, но расследование уже поставлено на паузу. Комментарий прямо сообщает: тип `DTOStats` знает об `operation`, функция ищет `type`, а приведение `as unknown as` спрятало разрыв между ними.

## ТРЕТИЙ СЛЕД: ПЕЧАТЬ ТИПОВ ПРОТИВОРЕЧИТ СЫЩИКУ

В том же коммите сгенерированный файл [`src/shared/api/console/types.gen.ts`](https://github.com/tonkeeper/ton-console/blob/8d87e4881307cb5514ee08eb41f0947dfe46c2fc/src/shared/api/console/types.gen.ts) на строках [252–260](https://github.com/tonkeeper/ton-console/blob/8d87e4881307cb5514ee08eb41f0947dfe46c2fc/src/shared/api/console/types.gen.ts#L252-L260) выдает официальный бланк статистики:

```typescript
export type DTOStats = {
    result: Array<{
        metric: {
            operation?: string;
        };
        values: Array<unknown>;
    }>;
    resultType: string;
};
```

На строке [255](https://github.com/tonkeeper/ton-console/blob/8d87e4881307cb5514ee08eb41f0947dfe46c2fc/src/shared/api/console/types.gen.ts#L255) у метрики стоит `operation`, не `type`. Но функция на строке [38](https://github.com/tonkeeper/ton-console/blob/8d87e4881307cb5514ee08eb41f0947dfe46c2fc/src/features/tonapi/webhooks/model/queries.ts#L38) силой превращает метрику в `Record<string, string>`, чтобы компилятор не увидел беду. Получается прекрасная лондонская сцена: печать на бланке называет вещь одним словом, а служащий в соседней комнате требует другое и запрещает задавать вопросы.

## ЧЕТВЕРТЫЙ СЛЕД: ГРАФИК СНАЧАЛА СПРАШИВАЕТ, ЕСТЬ ЛИ ДАННЫЕ

Витрина подключает эту функцию дважды. В файле [`src/features/tonapi/statistics/ui/DashboardWebhooksChart.tsx`](https://github.com/tonkeeper/ton-console/blob/8d87e4881307cb5514ee08eb41f0947dfe46c2fc/src/features/tonapi/statistics/ui/DashboardWebhooksChart.tsx) строки [28–31](https://github.com/tonkeeper/ton-console/blob/8d87e4881307cb5514ee08eb41f0947dfe46c2fc/src/features/tonapi/statistics/ui/DashboardWebhooksChart.tsx#L28-L31) требуют сначала получить точки:

```typescript
    const failedData = data ? mapWebhooksStatsToChartPoints(data, 'failed') : null;
    const deliveredData = data ? mapWebhooksStatsToChartPoints(data, 'delivered') : null;
    const hasData =
        (failedData && failedData.length > 0) || (deliveredData && deliveredData.length > 0);
```

А строки [34–38](https://github.com/tonkeeper/ton-console/blob/8d87e4881307cb5514ee08eb41f0947dfe46c2fc/src/features/tonapi/statistics/ui/DashboardWebhooksChart.tsx#L34-L38) делают вывод о полном отсутствии данных:

```typescript
    // c. If no data at all (never used), don't render - even if tier is active
    // User requirement: don't show empty webhooks charts if never ordered/used
    if (!hasData) {
        return null;
    }
```

Если реальные метрики несут `operation`, а не `type`, фильтр из папки выше превращает живую статистику в пустой список. Затем `hasData` становится ложным, и график не рисуется вовсе. Пользователь видит не красную ошибку и не честную надпись о поломке, а отсутствие витрины: будто вебхуки никогда не существовали.

Редакция не станет называть точный формат ответа API без отдельного сетевого протокола. Возможно, endpoint действительно отдает `type`, а общий сгенерированный тип устарел. Но коммит оставляет ровно две несовместимые версии показаний и сам помечает место как критическое. Это уже надежная улика, а не слух у газового фонаря.

## ХРОНИКА МЕЛКОГО КРИНЖА

*Отдел маскировки.* Строка [38](https://github.com/tonkeeper/ton-console/blob/8d87e4881307cb5514ee08eb41f0947dfe46c2fc/src/features/tonapi/webhooks/model/queries.ts#L38) надевает на `metric` тип `Record<string, string>`. Так компилятор перестает спорить, а проблема переезжает в вечерний интерфейс.

*Отдел предупреждений.* Строка [24](https://github.com/tonkeeper/ton-console/blob/8d87e4881307cb5514ee08eb41f0947dfe46c2fc/src/features/tonapi/webhooks/model/queries.ts#L24) называет расследование критическим, но сама функция продолжает строить данные прямо под этим криком.

*Отдел невидимых витрин.* Строка [36](https://github.com/tonkeeper/ton-console/blob/8d87e4881307cb5514ee08eb41f0947dfe46c2fc/src/features/tonapi/webhooks/model/queries.ts#L36) обещает, что `undefined` будет отфильтрован. В связке со строкой [36](https://github.com/tonkeeper/ton-console/blob/8d87e4881307cb5514ee08eb41f0947dfe46c2fc/src/features/tonapi/statistics/ui/DashboardWebhooksChart.tsx#L36) это уже не просто пустая полка, а решение вообще не показывать комнату.

## ВЕРДИКТ СЫЩИКА

Репозиторий [tonkeeper/ton-console](https://github.com/tonkeeper/ton-console) пойман на несогласованном словаре. В коммите [`8d87e48`](https://github.com/tonkeeper/ton-console/commit/8d87e4881307cb5514ee08eb41f0947dfe46c2fc) файл [`src/features/tonapi/webhooks/model/queries.ts`](https://github.com/tonkeeper/ton-console/blob/8d87e4881307cb5514ee08eb41f0947dfe46c2fc/src/features/tonapi/webhooks/model/queries.ts) ищет `metric.type` на строке [38](https://github.com/tonkeeper/ton-console/blob/8d87e4881307cb5514ee08eb41f0947dfe46c2fc/src/features/tonapi/webhooks/model/queries.ts#L38), хотя тип [`DTOStats`](https://github.com/tonkeeper/ton-console/blob/8d87e4881307cb5514ee08eb41f0947dfe46c2fc/src/shared/api/console/types.gen.ts#L252-L260) называет поле `operation`.

Если серверная схема не содержит скрытого третьего паспорта, `delivered` и `failed` не проходят фильтр, а [`DashboardWebhooksChart`](https://github.com/tonkeeper/ton-console/blob/8d87e4881307cb5514ee08eb41f0947dfe46c2fc/src/features/tonapi/statistics/ui/DashboardWebhooksChart.tsx#L34-L38) возвращает `null`. График не падает, не жалуется и не зовет констебля. Он просто растворяется в тумане.

Газовый рожок дернулся. Сыщик поставил на папке штамп: **«Метрика доставлена. Название не установлено»**.

🐀
