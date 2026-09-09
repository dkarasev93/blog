+++
title = "№ 53 — Дело о ставках и трех буквах WTF: elector acton-contracts ведет бухгалтерию в тумане"
date = 2026-09-09T11:01:00+03:00
description = "Пятьдесят третий выпуск «Вечернего Валидатора»: в elector из acton-contracts рядом с TODO осталась формула totalStakes и честное признание WTF, а ниже код уже размораживает ставки."
tags = ["acton-contracts"]
+++

# 📰 ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

**№ 53 · Среда, 9 сентября 2026 г. · Цена: 0.05 TON (ставки считаются по настроению)**

---

## ДЕЛО О БУХГАЛТЕРЕ, КОТОРЫЙ НАПИСАЛ WTF

Лондон тонул в тумане, мостовая блестела, а газовый рожок у редакции сипел, словно его заставили проводить ревизию в конторе, где чернила кончились прямо перед итоговой суммой. Сыщик «Вечернего Валидатора» получил папку из [ton-blockchain/acton-contracts](https://github.com/ton-blockchain/acton-contracts) — собрания контрактов TON, переписанных на Acton и Tolk.

Внутри лежала сцена, достойная отдельной витрины. Функция удаляет прошедшие выборы, достает их данные, встречает два комментария `TODO`, а затем прямо в середине финансового дела оставляет слово `WTF`. После этого программа не падает в обморок. Она продолжает размораживать средства и возвращает неиспользованные призы.

Место происшествия: репозиторий [ton-blockchain/acton-contracts](https://github.com/ton-blockchain/acton-contracts), коммит [`7af1cea`](https://github.com/ton-blockchain/acton-contracts/commit/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16), файл [`elector/contracts/credits.tolk`](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/elector/contracts/credits.tolk), строки [85–104](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/elector/contracts/credits.tolk#L85-L104). Цитата сверена через содержимое файла на этом коммите.

Протокол, файл [`elector/contracts/credits.tolk`](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/elector/contracts/credits.tolk), строки [85–104](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/elector/contracts/credits.tolk#L85-L104):

```tolk
    mutate self,
    pastElections: map<uint32, PastElection>,
    electionId: uint32,
): (map<uint32, PastElection>, int) {
    val res = pastElections.deleteAndGetDeleted(electionId);
    if (!res.isFound) {
        return (pastElections, 0);
    }
    val pastElection = res.loadValue();
    // TODO
    // totalStakes = fdict.stakes_sup() WTF
    val unusedPrizes = (pastElection.bonuses > 0)
        ? self.unfreezeWithBonuces(
            pastElection.frozenDict,
            pastElection.totalStake,
            pastElection.bonuses,
        )
        : self.unfreezeWithoutBonuces(pastElection.frozenDict, pastElection.totalStake);
    return (pastElections, unusedPrizes);
}
```

## ПЕРВЫЙ СЛЕД: ВЫБОРЫ УДАЛЕНЫ, ВОПРОСЫ ОСТАЛИСЬ

На строках [85–88](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/elector/contracts/credits.tolk#L85-L88) перед нами функция с мутацией состояния. Она принимает карту прошедших выборов и номер конкретных выборов, а затем обещает вернуть обновленную карту и число типа `int`. Это не записка на полях учебного примера. Тут бухгалтерия хранит историю выборов и их ставки.

Строка [89](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/elector/contracts/credits.tolk#L89) вызывает `deleteAndGetDeleted(electionId)`. Из карты удаляют запись, но одновременно сохраняют ее для дальнейшей работы. На строках [90–92](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/elector/contracts/credits.tolk#L90-L92) функция проводит простой допрос: если запись не найдена, она возвращает прежнюю карту и ноль.

А на строке [93](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/elector/contracts/credits.tolk#L93) найденный груз уже вскрыт: `val pastElection = res.loadValue();`. Дело должно перейти к расчету. И тут на сцену входят строки [94–95](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/elector/contracts/credits.tolk#L94-L95), где рядом стоят `TODO` и формула, закончившаяся словом `WTF`.

Газовый рожок кашлянул. Даже самый мрачный лондонский счетовод обычно пишет «уточнить сумму» или хотя бы ставит чернильную кляксу. Здесь же черновая мысль сохранила голос автора: `totalStakes = fdict.stakes_sup() WTF`. Ставки, карта, сумма и три буквы, будто из соседней комнаты кто-то увидел итог и не смог его принять.

## ВТОРОЙ СЛЕД: КОММЕНТАРИЙ ЗНАЕТ О ПРОПАВШЕЙ СУММЕ

Строка [95](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/elector/contracts/credits.tolk#L95) не просто содержит грубое междометие. Она называет конкретную недостающую операцию: `fdict.stakes_sup()`. По смыслу заметки, автор ожидал получить сумму ставок из некоторой структуры, но оставил эту мысль в комментарии, не превратив ее в исполняемый код.

Редакция не станет утверждать, что из этого фрагмента доказана потеря средств. Для такого вывода нужно знать весь путь формирования `PastElection`, правила выборов и смысл возвращаемого `unusedPrizes`. Но улика все равно тяжела: в функции, которая работает с замороженными ставками и бонусами, рядом с кодом стоит незакрытый вопрос о сумме.

Особенно выразительна строка [96](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/elector/contracts/credits.tolk#L96). Она сразу после `WTF` вводит `unusedPrizes`. То есть бухгалтер не получил ответ на записанный вопрос, но папка не закрылась. Машина пошла дальше с тем, что уже есть в `pastElection`.

На строках [97–101](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/elector/contracts/credits.tolk#L97-L101) при положительном числе бонусов вызывается `unfreezeWithBonuces`. Само имя метода несет отдельную маленькую улику: вместо ожидаемого `Bonuses` в нем написано `Bonuces`. Это не меняет смысл вызова для компилятора, но делает финансовую сцену еще более похожей на рукопись, которую переписывали в карете на ухабах.

## ТРЕТИЙ СЛЕД: РАЗМОРАЖИВАНИЕ РАБОТАЕТ ПОСЛЕ КРИКА

Ветвь на строках [96–102](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/elector/contracts/credits.tolk#L96-L102) выбирает один из двух путей. Если бонусы больше нуля, используются `pastElection.frozenDict`, `pastElection.totalStake` и `pastElection.bonuses`. Иначе строка [102](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/elector/contracts/credits.tolk#L102) отправляет те же замороженные данные и общую ставку в `unfreezeWithoutBonuces`.

Слово `totalStakes` в строке [95](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/elector/contracts/credits.tolk#L95) при этом не участвует в вычислении. Оно живет только в комментарии. Исполняемый код берет `pastElection.totalStake` на строках [99](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/elector/contracts/credits.tolk#L99) и [102](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/elector/contracts/credits.tolk#L102). Перед нами не доказанный баг, а контраст между незавершенным расчетом на бумаге и уверенным движением средств в коде.

На строке [103](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/elector/contracts/credits.tolk#L103) функция возвращает карту прошедших выборов и `unusedPrizes`. Запись уже удалена, средства обработаны, а вопрос из строки [95](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/elector/contracts/credits.tolk#L95) так и не получил ответа.

Коммит [`7af1cea`](https://github.com/ton-blockchain/acton-contracts/commit/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16) носит спокойное название `fix(dns): use sha256 for category keys (TEP-81) (#96)`. Вот где дело становится почти неприличным. Летопись коммита посвящена DNS и SHA-256, а в том же снимке файла [`elector/contracts/credits.tolk`](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/elector/contracts/credits.tolk) лежит незакрытый комментарий про ставки и `WTF`. Сыщик не обвиняет коммит в причинной связи. Он лишь фиксирует: криптографическая вывеска сияет, а в электоральной кладовой горит лампа над такой записью.

## ПРИГОВОР ГАЗОВОГО РОЖКА

[ton-blockchain/acton-contracts](https://github.com/ton-blockchain/acton-contracts) не пойман на доказанном хищении и не обязан пояснять каждое эмоциональное слово в старом комментарии. Но коммит [`7af1cea`](https://github.com/ton-blockchain/acton-contracts/commit/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16) оставил в файле [`elector/contracts/credits.tolk`](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/elector/contracts/credits.tolk) строки [94–95](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/elector/contracts/credits.tolk#L94-L95), где расчет суммы ставок соседствует с `TODO` и `WTF`.

Приговор мягок, но настойчив: либо удалить мертвую заметку, либо превратить ее в ясную проверку, тест и комментарий без крика из тумана. Если `pastElection.totalStake` уже является верным итогом, это стоит сказать прямо. Если сумму надо получать из `fdict.stakes_sup()`, незавершенный расчет не должен оставаться музейной табличкой в финансовой функции.

*Сыщик закрыл папку, еще раз перечитал строку [95](https://github.com/ton-blockchain/acton-contracts/blob/7af1cea3cd0b990ae7b53a67b858c8cbd9da1e16/elector/contracts/credits.tolk#L95) и погас газовую лампу. В Лондоне можно пережить туман, старую вывеску и даже опечатку в слове `Bonuces`. Но когда в бухгалтерской книге рядом с суммой появляется `WTF`, сыщик обязан проверить, кто оставил чернила открытыми.*

🐀
