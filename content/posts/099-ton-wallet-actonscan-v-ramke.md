+++
title = "№ 99 — ActonScan, впущенный в рамку Gram Wallet"
date = 2026-10-20T16:59:00+03:00
description = "Девяносто девятый выпуск «Вечернего Валидатора»: Gram Wallet официально пустил ActonScan в рамку Chrome-расширения."
tags = ["ton-wallet"]
+++

# ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

**№ 99.** *Лондон. Над Темзой висел туман, когда сыщик получил свежий циркуляр из [ton-blockchain/ton-wallet](https://github.com/ton-blockchain/ton-wallet). Gram Wallet, недавно сменивший старую вывеску, расширил рамку своего Chrome-дома. В новый список гостей вписан [ActonScan](https://actonscan.com). Газовый рожок кашлянул: в криптокошелек теперь впускают не только платежные конторы, но и целую комнату для осмотра Acton.*

## МЕСТО ПРОИСШЕСТВИЯ

Улика найдена в коммите [70fb938](https://github.com/ton-blockchain/ton-wallet/commit/70fb93872a2f811e8500b14da54b9584a44d4094), озаглавленном `[CI] Gram: Update the approved Chrome extension frame-src (#4)`. Главная дверь — файл [scripts/validate-endpoints.mjs](https://github.com/ton-blockchain/ton-wallet/blob/70fb93872a2f811e8500b14da54b9584a44d4094/scripts/validate-endpoints.mjs), строка [94](https://github.com/ton-blockchain/ton-wallet/blob/70fb93872a2f811e8500b14da54b9584a44d4094/scripts/validate-endpoints.mjs#L94). Цитата снята дословно с этого коммита:

```js
      'https://tonviewer.com https://testnet.tonviewer.com https://*.mywallet.io https://actonscan.com',
```

Вторая печать стоит в тесте [tests/chrome-release.test.mjs](https://github.com/ton-blockchain/ton-wallet/blob/70fb93872a2f811e8500b14da54b9584a44d4094/tests/chrome-release.test.mjs), строка [118](https://github.com/ton-blockchain/ton-wallet/blob/70fb93872a2f811e8500b14da54b9584a44d4094/tests/chrome-release.test.mjs#L118):

```js
      "frame-src 'self' https: https://buy-sandbox.moonpay.com/ https://buy.moonpay.com/ https://sell.moonpay.com/ https://sell-sandbox.moonpay.com/ https://*.onetrust.com/ https://dreamwalkers.io/ https://avanchange.com/ https://pay.walletconnect.com/ http://localhost:* https://tonscan.org https://testnet.tonscan.org https://tonviewer.com https://testnet.tonviewer.com https://*.mywallet.io https://actonscan.com;",
```

## ПЕРВЫЙ СЛЕД: ГОСТЬ ПРОШЕЛ В САМУ РАМКУ

В Chrome-расширении `frame-src` решает, какие страницы можно показать во встроенной рамке. Контора не добавила ActonScan в случайный список ссылок для справки. Она внесла адрес в эталонную политику, которую проверяет скрипт. На строках [89–95](https://github.com/ton-blockchain/ton-wallet/blob/70fb93872a2f811e8500b14da54b9584a44d4094/scripts/validate-endpoints.mjs#L89-L95) собран полный перечень для `frame-src`, а строка [94](https://github.com/ton-blockchain/ton-wallet/blob/70fb93872a2f811e8500b14da54b9584a44d4094/scripts/validate-endpoints.mjs#L94) отдает нам новую фамилию гостя.

Картина, признаться, роскошная. В одной комнате сидят MoonPay, OneTrust, Dreamwalkers, Avanchange, WalletConnect, TonScan, Tonviewer и теперь ActonScan. Рядом дежурит `http://localhost:*` — лаз для местной прислуги, которому в строгой политике почему-то тоже выдали пропуск. Граница между кошельком и викторианским вокзалом внешних окон стала длиннее на одно имя.

Редакция не станет кричать про дыру: сам по себе новый адрес не доказывает преступного поведения. Но факт сочен и точен — разрешенный круг встроенных страниц стал шире, а проверка обязана принять именно его.

## ВТОРОЙ СЛЕД: ТЕСТ СТАЛ СВИДЕТЕЛЕМ

Писарь не ограничился изменением списка. В тесте [tests/chrome-release.test.mjs](https://github.com/ton-blockchain/ton-wallet/blob/70fb93872a2f811e8500b14da54b9584a44d4094/tests/chrome-release.test.mjs) строка [118](https://github.com/ton-blockchain/ton-wallet/blob/70fb93872a2f811e8500b14da54b9584a44d4094/tests/chrome-release.test.mjs#L118) повторяет весь `frame-src` уже внутри утвержденного манифеста. Если забыть ActonScan в тестовом фолианте, CI не подпустит новый вариант политики к парадной двери.

На строках [121–123](https://github.com/ton-blockchain/ton-wallet/blob/70fb93872a2f811e8500b14da54b9584a44d4094/tests/chrome-release.test.mjs#L121-L123) начинается следующий раздел теста, а сам коммит меняет ровно по одной строке в основном валидаторе и в проверочном манифесте. Получается аккуратный двойной след: список разрешений и его зеркало идут рядом, чтобы ActonScan не оказался гостем, которого признали только на словах.

## ТРЕТИЙ СЛЕД: СТРАЖА НЕ ВПУСКАЕТ ЛИШНЕЕ

Интерес к делу подогревает соседняя стража. Скрипт разбирает политику, собирает директивы и затем сравнивает разрешенные источники. На строках [97–99](https://github.com/ton-blockchain/ton-wallet/blob/70fb93872a2f811e8500b14da54b9584a44d4094/scripts/validate-endpoints.mjs#L97-L99) каждая директива раскладывается на имя и источники. На строках [112–115](https://github.com/ton-blockchain/ton-wallet/blob/70fb93872a2f811e8500b14da54b9584a44d4094/scripts/validate-endpoints.mjs#L112-L115) отдельная проверка сравнивает `connect-src` с полным производственным списком.

Это важная деталь: новый адрес появился в `frame-src`, а не в `connect-src`. ActonScan разрешено показывать в рамке, но это не означает автоматического права для фоновых запросов расширения к любому новому узлу. Сыщик проверил формулировку дважды и не нашел оснований приписывать этому коммиту лишние полномочия.

## ЧЕТВЕРТЫЙ СЛЕД: КОМИТТ САМ ПРИЗНАЛСЯ

Название коммита [70fb938](https://github.com/ton-blockchain/ton-wallet/commit/70fb93872a2f811e8500b14da54b9584a44d4094) сообщает: речь идет об обновлении утвержденной рамки Chrome-расширения. Не «мелкая уборка», не «прочие исправления», а официальный прием нового адреса в список разрешенных.

Сцена смешна не потому, что ActonScan плох. Напротив, адрес получил пропуск как нормальный инструмент экосистемы TON. Смешно другое: кошелек, который должен быть строгой шкатулкой, ведет отдельный придворный реестр сайтов, куда можно смотреть через окно. В реестре уже есть `http://localhost:*`, тестовая сеть, песочница MoonPay и теперь сыщик ActonScan. Еще немного — и рамка начнет просить визитную карточку у каждого переулка Лондона.

## ХРОНИКА МЕЛКОЙ КАНЦЕЛЯРИИ

*Отдел зеркал.* В файле [tests/chrome-release.test.mjs](https://github.com/ton-blockchain/ton-wallet/blob/70fb93872a2f811e8500b14da54b9584a44d4094/tests/chrome-release.test.mjs) строка [118](https://github.com/ton-blockchain/ton-wallet/blob/70fb93872a2f811e8500b14da54b9584a44d4094/tests/chrome-release.test.mjs#L118) хранит длинную строку политики одной строкой, без милосердия к читателю.

*Отдел порядка.* В [scripts/validate-endpoints.mjs](https://github.com/ton-blockchain/ton-wallet/blob/70fb93872a2f811e8500b14da54b9584a44d4094/scripts/validate-endpoints.mjs) строки [97–111](https://github.com/ton-blockchain/ton-wallet/blob/70fb93872a2f811e8500b14da54b9584a44d4094/scripts/validate-endpoints.mjs#L97-L111) политика раскладывается, а затем запускается проверка соединений. Даже у газового рожка есть более короткая инструкция.

*Отдел новых имен.* В [scripts/validate-endpoints.mjs](https://github.com/ton-blockchain/ton-wallet/blob/70fb93872a2f811e8500b14da54b9584a44d4094/scripts/validate-endpoints.mjs) строка [94](https://github.com/ton-blockchain/ton-wallet/blob/70fb93872a2f811e8500b14da54b9584a44d4094/scripts/validate-endpoints.mjs#L94) теперь связывает ActonScan с `frame-src`, а не с общей дверью `connect-src`. Сыщик не смешивает ключи от разных комнат.

## ВЕРДИКТ СЫЩИКА

Репозиторий [ton-blockchain/ton-wallet](https://github.com/ton-blockchain/ton-wallet) пойман в коммите [70fb938](https://github.com/ton-blockchain/ton-wallet/commit/70fb93872a2f811e8500b14da54b9584a44d4094) на деле о новом госте в рамке. В файле [scripts/validate-endpoints.mjs](https://github.com/ton-blockchain/ton-wallet/blob/70fb93872a2f811e8500b14da54b9584a44d4094/scripts/validate-endpoints.mjs) строка [94](https://github.com/ton-blockchain/ton-wallet/blob/70fb93872a2f811e8500b14da54b9584a44d4094/scripts/validate-endpoints.mjs#L94) вписывает `https://actonscan.com` в `frame-src`. Тест [tests/chrome-release.test.mjs](https://github.com/ton-blockchain/ton-wallet/blob/70fb93872a2f811e8500b14da54b9584a44d4094/tests/chrome-release.test.mjs) строка [118](https://github.com/ton-blockchain/ton-wallet/blob/70fb93872a2f811e8500b14da54b9584a44d4094/tests/chrome-release.test.mjs#L118) ставит на приглашении вторую печать.

Приговор без драматической музыки: ActonScan получил место у окна, но не ключ от всей конторы. Gram Wallet расширил рамку, CI записал гостя в список, а сыщик записал в блокнот: в строгих кошельках самые сочные новости порой выглядят как одна длинная строка URL. Газовый рожок пробил один удар, туман сомкнулся, и новый гость скрылся за стеклом.

🐀