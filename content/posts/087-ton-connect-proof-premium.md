+++
title = "№ 87 — Пропуск, который пропускал лишнее"
date = 2026-09-27T17:00:00+03:00
description = "Восемьдесят седьмой выпуск «Вечернего Валидатора»: ton-connect/sdk закрыл лишние поля в ton_proof и перестал принимать обычных Telegram-посетителей за призраков без Premium."
tags = ["sdk"]
+++

# ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

**№ 87.** *Лондон. Вечерний туман сполз с крыш, газовый рожок сипит у редакции, а сыщик получил из [ton-connect/sdk](https://github.com/ton-connect/sdk) две записки одним коммитом. В первой охрана `ton_proof` наконец заметила лишние ключи внутри главного доказательства. Во второй Telegram-посетитель без Premium перестал считаться невидимкой. Входная дверь была строгой снаружи, но внутри оставляла один карман без досмотра, а у обычного гостя требовала пропуск, который ему никогда не выдавали.*

## МЕСТО ПРОИСШЕСТВИЯ

Улика найдена в репозитории [ton-connect/sdk](https://github.com/ton-connect/sdk), в коммите [`3760b0c`](https://github.com/ton-connect/sdk/commit/3760b0c8d260c33d69bcf618e9787a8813a744da), названном `fix: report non-premium Telegram users and close ton_proof validation gap (#583)`. Цитаты сняты с этого снимка. Главные комнаты дела — файл [`packages/sdk/src/validation/schemas.ts`](https://github.com/ton-connect/sdk/blob/3760b0c8d260c33d69bcf618e9787a8813a744da/packages/sdk/src/validation/schemas.ts) и файл [`packages/ui/src/app/utils/tma-api.ts`](https://github.com/ton-connect/sdk/blob/3760b0c8d260c33d69bcf618e9787a8813a744da/packages/ui/src/app/utils/tma-api.ts).

## ПЕРВЫЙ СЛЕД: ДОКАЗАТЕЛЬСТВО С ЧЕРНЫМ ХОДОМ

Функция `validateTonProofItemReply` уже проверяет сам конверт `ton_proof`: лишние ключи отсекаются на строках [600–601](https://github.com/ton-connect/sdk/blob/3760b0c8d260c33d69bcf618e9787a8813a744da/packages/sdk/src/validation/schemas.ts#L600-L601), а дальше выбирается ровно один путь — `proof` или `error` — на строках [604–613](https://github.com/ton-connect/sdk/blob/3760b0c8d260c33d69bcf618e9787a8813a744da/packages/sdk/src/validation/schemas.ts#L604-L613). С фасада все выглядит как приличная лондонская контора: список посетителей сверяют, два взаимоисключающих письма вместе не пропускают.

Но за дверью `proof` оставался отдельный карман. В коммите [`3760b0c`](https://github.com/ton-connect/sdk/commit/3760b0c8d260c33d69bcf618e9787a8813a744da) контора поставила там новый досмотр на строках [623–625](https://github.com/ton-connect/sdk/blob/3760b0c8d260c33d69bcf618e9787a8813a744da/packages/sdk/src/validation/schemas.ts#L623-L625):

```ts
        const allowedProofKeys = ['timestamp', 'domain', 'payload', 'signature'];
        if (hasExtraProperties(proof, allowedProofKeys)) {
            return 'ton_proof item contains extra properties';
        }
```

Список короток и вполне викторианский: время, домен, нагрузка и подпись. Все, что пришло рядом и не названо в списке, теперь получает отказ. До этого тот же валидатор уже допрашивал верхний конверт, но вложенное `proof` проходило в комнату с чужими бумагами. Коммит прямо называет это `ton_proof validation gap`: не пролом в стене, а щель в самом неудобном месте — там, где наружная печать уже обещает порядок.

## ВТОРОЙ СЛЕД: ЛИШНИЙ КЛЮЧ В ПОДПИСИ

На строках [615–621](https://github.com/ton-connect/sdk/blob/3760b0c8d260c33d69bcf618e9787a8813a744da/packages/sdk/src/validation/schemas.ts#L615-L621) сыщик видит старый порядок: если поле `proof` есть, его сначала проверяют как запись. Затем на строках [628–640](https://github.com/ton-connect/sdk/blob/3760b0c8d260c33d69bcf6187a8813a744da/packages/sdk/src/validation/schemas.ts#L628-L640) проверяются время и домен. Но теперь между этими двумя этажами стоит отдельный контроль ключей.

Сцена комична своей симметрией. Наружная коробка уже знала фразу `ton_proof item contains extra properties`, а внутренний конверт до коммита [`3760b0c`](https://github.com/ton-connect/sdk/commit/3760b0c8d260c33d69bcf618e9787a8813a744da) жил по принципу: «если поля похожи на правду, не мешайте джентльмену». Одна лишняя записка могла сидеть внутри доказательства, пока проверка шла дальше к `timestamp`, `domain`, `payload` и `signature`.

Редакция не утверждает, что любая лишняя строка автоматически превращалась в взлом. Факт уже и так достаточно сочен: схема заявляет закрытый набор допустимых полей, а исправление добавляет именно проверку этого набора. Маленький список становится шлагбаумом, который раньше стоял только на соседней улице.

## ТРЕТИЙ СЛЕД: ПОСЕТИТЕЛЬ БЕЗ PREMIUM БЫЛ ПРИЗРАКОМ

Вторая половина коммита [`3760b0c`](https://github.com/ton-connect/sdk/commit/3760b0c8d260c33d69bcf618e9787a8813a744da) ведет в Telegram Mini App. Тип гостя на строках [125–128](https://github.com/ton-connect/sdk/blob/3760b0c8d260c33d69bcf618e9787a8813a744da/packages/ui/src/app/utils/tma-api.ts#L125-L128) требует числовой ID и булево значение `isPremium`. Ничего подозрительного: у каждого гостя есть имя и отметка о статусе.

Подозрительность появляется в старом условии приема. Telegram не обязан присылать `is_premium` для пользователя без Premium. Поэтому прежняя проверка с двумя требованиями могла выкинуть целого посетителя только за отсутствие необязательной вывески. Коммит оставил признание в комментарии на строках [138–140](https://github.com/ton-connect/sdk/blob/3760b0c8d260c33d69bcf618e9787a8813a744da/packages/ui/src/app/utils/tma-api.ts#L138-L140):

```ts
            // Telegram omits is_premium entirely for non-premium users, so it cannot
            // be required here without dropping most of the audience.
            if (typeof user.id === 'number') {
```

Вот настоящий газетный заголовок: **«Большинство публики не прошло, потому что у нее не было необязательной печати»**. В старом варианте число и статус должны были прибыть парой. В новом варианте строка [140](https://github.com/ton-connect/sdk/blob/3760b0c8d260c33d69bcf618e9787a8813a744da/packages/ui/src/app/utils/tma-api.ts#L140) требует только числовой ID, а строки [141–144](https://github.com/ton-connect/sdk/blob/3760b0c8d260c33d69bcf618e9787a8813a744da/packages/ui/src/app/utils/tma-api.ts#L141-L144) сами превращают отсутствие Premium в честное `false`:

```ts
                telegramUser = {
                    id: user.id,
                    isPremium: user.is_premium === true
                };
```

Это не роскошь для избранных, а нормальная логика для обычного горожанина: нет флага — значит, не Premium; есть `true` — значит, Premium. Остальные не исчезают в тумане.

## ЧЕТВЕРТЫЙ СЛЕД: ДВЕ ЛУЖИ ОДНОГО КОММИТА

Коммит [`3760b0c`](https://github.com/ton-connect/sdk/commit/3760b0c8d260c33d69bcf618e9787a8813a744da) положил рядом два сюжета, которые на первый взгляд не связаны. В SDK закрыли внутреннюю щель в проверке доказательства, а в UI перестали путать отсутствие платной отметки с отсутствием пользователя.

Но почерк один. В первом деле код слишком доверял тому, что проверил внешний слой: вложенная запись оставалась без собственного списка разрешенных ключей. Во втором код слишком доверял наличию поля, которое внешний сервис присылает не всем: отсутствие `is_premium` трактовалось как отсутствие самого гостя. В обеих комнатах исправление возвращает смысл на место — проверять ровно то, что действительно обязательно.

У коммита есть и отдельная контрольная улика. В файле [`packages/sdk/tests/provider/bridge/universal-link.test.ts`](https://github.com/ton-connect/sdk/blob/3760b0c8d260c33d69bcf618e9787a8813a744da/packages/sdk/tests/provider/bridge/universal-link.test.ts) строки [24–43](https://github.com/ton-connect/sdk/blob/3760b0c8d260c33d69bcf618e9787a8813a744da/packages/sdk/tests/provider/bridge/universal-link.test.ts#L24-L43) различают длинную ссылку с вложенным запросом и короткую ссылку только для подключения. Это не главная улика выпуска, но полезная подпись под делом: тесты теперь признают, что большой конверт может не влезть в почтовую щель и тогда должен уйти без вложения.

## ХРОНИКА МЕЛКОГО КРИНЖА

*Отдел названий.* Изменение пришло сразу с двумя записками changeset: одна обещает исправить Telegram-пользователей без Premium, другая — закрыть лишние поля в `ton_proof`. В коммите [`3760b0c`](https://github.com/ton-connect/sdk/commit/3760b0c8d260c33d69bcf618e9787a8813a744da) даже канцелярия не стала делать вид, что это одна большая философская реформа.

*Отдел строгих списков.* Разрешенные поля `proof` записаны на строке [623](https://github.com/ton-connect/sdk/blob/3760b0c8d260c33d69bcf618e9787a8813a744da/packages/sdk/src/validation/schemas.ts#L623) обычным массивом. Никакого тумана типов, только четыре имени и одна дверь.

*Отдел невидимых гостей.* После разбора JSON на строках [132–137](https://github.com/ton-connect/sdk/blob/3760b0c8d260c33d69bcf618e9787a8813a744da/packages/ui/src/app/utils/tma-api.ts#L132-L137) код достает `user`, а затем больше не требует от него платный жетон. Лондонская приемная впервые узнала, что посетитель без золотой булавки все еще посетитель.

## ВЕРДИКТ СЫЩИКА

Репозиторий [ton-connect/sdk](https://github.com/ton-connect/sdk) пойман в коммите [`3760b0c`](https://github.com/ton-connect/sdk/commit/3760b0c8d260c33d69bcf618e9787a8813a744da) на двойной сцене. В файле [`packages/sdk/src/validation/schemas.ts`](https://github.com/ton-connect/sdk/blob/3760b0c8d260c33d69bcf618e9787a8813a744da/packages/sdk/src/validation/schemas.ts) вложенный `proof` наконец получает досмотр лишних ключей на строках [623–625](https://github.com/ton-connect/sdk/blob/3760b0c8d260c33d69bcf618e9787a8813a744da/packages/sdk/src/validation/schemas.ts#L623-L625). В файле [`packages/ui/src/app/utils/tma-api.ts`](https://github.com/ton-connect/sdk/blob/3760b0c8d260c33d69bcf618e9787a8813a744da/packages/ui/src/app/utils/tma-api.ts) обычный Telegram-посетитель проходит по ID на строке [140](https://github.com/ton-connect/sdk/blob/3760b0c8d260c33d69bcf618e9787a8813a744da/packages/ui/src/app/utils/tma-api.ts#L140), а отсутствие Premium спокойно становится `false` на строке [143](https://github.com/ton-connect/sdk/blob/3760b0c8d260c33d69bcf618e9787a8813a744da/packages/ui/src/app/utils/tma-api.ts#L143).

Приговор мягкий, но с печатью: не называй защитой наружную дверь, если внутренний конверт можно набить лишними бумагами; не называй человека призраком, если у него просто нет платной галочки. В Лондоне туман скрывает многое, но хороший валидатор обязан отличать пустой карман от пустого человека.

Газовый рожок дернулся. Сыщик закрыл папку, оставил на ней четыре разрешенных ключа и растворился в тумане.

🐀
