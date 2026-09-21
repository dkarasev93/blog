+++
title = "№ 76 — Минтер, который сам выдал справку о неисправном жеттоне"
date = 2026-09-21T17:02:00+03:00
description = "Семьдесят шестой выпуск «Вечернего Валидатора»: веб-минтер распознает жеттоны, созданные его прошлой неисправной версией, и честно сообщает, что часть из них уже не спасти."
tags = ["minter"]
+++

# ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

**№ 76.** *Лондон. Туман лег на мостовую, газовый рожок у редакции сипит, а сыщик получил письмо из конторы, где чеканят жеттоны TON. На фасаде все чинно: форма, кошелек, кнопка выпуска. Но в задней комнате висит особая табличка: «Этот жеттон создан прошлой неисправной версией». Для одного хозяина есть кнопка ремонта. Для другого, если право администратора уже отозвано, остается только печальная рекомендация снести все и начать заново.*

## МЕСТО ПРОИСШЕСТВИЯ

Улика найдена в репозитории [ton-blockchain/minter](https://github.com/ton-blockchain/minter), в коммите [`6a68c32`](https://github.com/ton-blockchain/minter/commit/6a68c32d3672cb368e2776ca2034b79231eef043), в файлах [`src/pages/jetton/util.ts`](https://github.com/ton-blockchain/minter/blob/6a68c32d3672cb368e2776ca2034b79231eef043/src/pages/jetton/util.ts) и [`src/pages/jetton/FaultyDeploy.tsx`](https://github.com/ton-blockchain/minter/blob/6a68c32d3672cb368e2776ca2034b79231eef043/src/pages/jetton/FaultyDeploy.tsx). Цитаты сверены по этому коммиту.

Дословная записка из приемной, строки [9–14](https://github.com/ton-blockchain/minter/blob/6a68c32d3672cb368e2776ca2034b79231eef043/src/pages/jetton/util.ts#L9-L14):

```typescript
export const getFaultyMetadataWarning = (isAdminRevokedOwnership?: boolean) => {
  if (isAdminRevokedOwnership) {
    return "This token was created with a previous faulty version of the tool. The token is permanently unusable, please contact the admin to redeploy a new token";
  }
  return "This token was created with a previous faulty version of the tool. The token is now unusable but can be fixed, please contact the admin to fix it using this page";
};
```

## ПЕРВЫЙ СЛЕД: ЖЕТТОН С ПЕЧАТЬЮ «НЕИСПРАВЕН»

На строке [9](https://github.com/ton-blockchain/minter/blob/6a68c32d3672cb368e2776ca2034b79231eef043/src/pages/jetton/util.ts#L9) контора вводит отдельный глашатайский метод `getFaultyMetadataWarning`. Он не сообщает о временном сбое сети и не просит обновить страницу. Он формирует предупреждение о жеттоне, созданном «previous faulty version of the tool».

Ветвь на строках [10–12](https://github.com/ton-blockchain/minter/blob/6a68c32d3672cb368e2776ca2034b79231eef043/src/pages/jetton/util.ts#L10-L12) особенно выразительна. Если право администратора отозвано, интерфейс выносит приговор: `The token is permanently unusable`. Далее он предлагает связаться с администратором и развернуть новый жеттон. Газовый рожок кашлянул: жеттон может остаться жить в цепи, но его официальная судьба уже напоминает записку на двери закрытой лавки.

Вторая ветвь, строки [13–14](https://github.com/ton-blockchain/minter/blob/6a68c32d3672cb368e2776ca2034b79231eef043/src/pages/jetton/util.ts#L13-L14), оставляет шанс. Жеттон все еще назван `now unusable`, зато его можно починить через текущую страницу. Получается викторианская шкала порчи: пока администратор держит ключ, это сломанный жеттон; после отказа от владения это уже памятник.

## ВТОРОЙ СЛЕД: КАК КОНТОРА НАХОДИТ СТАРУЮ ОШИБКУ

В файле [`src/lib/jetton-minter.ts`](https://github.com/ton-blockchain/minter/blob/6a68c32d3672cb368e2776ca2034b79231eef043/src/lib/jetton-minter.ts) функция начинает разбор на строке [217](https://github.com/ton-blockchain/minter/blob/6a68c32d3672cb368e2776ca2034b79231eef043/src/lib/jetton-minter.ts#L217). Она возвращает не только метаданные, но и флаг неисправности, строки [217–220](https://github.com/ton-blockchain/minter/blob/6a68c32d3672cb368e2776ca2034b79231eef043/src/lib/jetton-minter.ts#L217-L220).

Дальше на строке [228](https://github.com/ton-blockchain/minter/blob/6a68c32d3672cb368e2776ca2034b79231eef043/src/lib/jetton-minter.ts#L228) флаг стартует как `false`. Но в строках [246–249](https://github.com/ton-blockchain/minter/blob/6a68c32d3672cb368e2776ca2034b79231eef043/src/lib/jetton-minter.ts#L246-L249) сыщик обнаруживает красный звонок:

```typescript
if (s.remainingRefs === 0) {
  isJettonDeployerFaultyOnChainData = true;
  return sliceToVal(s, buffer, true);
}
```

Ведомство считает отсутствие ссылок в ячейке признаком данных от старого, неисправного развертывателя. Иными словами, контора не просто читает жеттон: она носит с собой детектор собственного прошлого проступка. В обычной полицейской папке это называлось бы признанием, вложенным прямо в парсер.

Для исправного маршрута на строке [251](https://github.com/ton-blockchain/minter/blob/6a68c32d3672cb368e2776ca2034b79231eef043/src/lib/jetton-minter.ts#L251) чтение идет через `s.readRef()`. Контраст двух дверей прост: цепочка ссылок считается правильной, а плоская ячейка отправляет дело в архив неисправных.

## ТРЕТИЙ СЛЕД: ПОЧИНКА ТОЛЬКО ДЛЯ ОСТАВШЕГОСЯ АДМИНИСТРАТОРА

Компонент [`src/pages/jetton/FaultyDeploy.tsx`](https://github.com/ton-blockchain/minter/blob/6a68c32d3672cb368e2776ca2034b79231eef043/src/pages/jetton/FaultyDeploy.tsx) получает флаг на строке [19](https://github.com/ton-blockchain/minter/blob/6a68c32d3672cb368e2776ca2034b79231eef043/src/pages/jetton/FaultyDeploy.tsx#L19). Кнопка ремонта не выдается каждому свидетелю происшествия. На строке [81](https://github.com/ton-blockchain/minter/blob/6a68c32d3672cb368e2776ca2034b79231eef043/src/pages/jetton/FaultyDeploy.tsx#L81) окно открывается лишь при одновременном выполнении трех условий: данные признаны неисправными, текущий посетитель является администратором, а метаданные можно безопасно восстановить.

Дословная надпись из окна, строки [87–98](https://github.com/ton-blockchain/minter/blob/6a68c32d3672cb368e2776ca2034b79231eef043/src/pages/jetton/FaultyDeploy.tsx#L87-L98):

```tsx
            <Typography>
              This token was created with a previous faulty version of the deployer. Don’t worry,
              this can easily be fixed.
            </Typography>
            <br />
            <Typography>
              Click below to issue a fix transaction that will keep the token’s original data and
              fix the format.
            </Typography>
```

Фраза `Don’t worry` стоит рядом с признанием `previous faulty version`. Сыщик видел много лондонских объявлений, но редко встречал такое вежливое соседство: «ваш жеттон создан неисправным инструментом, не волнуйтесь, нажмите кнопку».

Сам вызов ремонта живет на строках [47–59](https://github.com/ton-blockchain/minter/blob/6a68c32d3672cb368e2776ca2034b79231eef043/src/pages/jetton/FaultyDeploy.tsx#L47-L59). Контора собирает исходные поля, передает адрес мастер-контракта и отправляет транзакцию через подключенный кошелек. Это не косметическая смена текста: приложение действительно готовит цепной ремонт.

## ЧЕТВЕРТЫЙ СЛЕД: КОГДА «НЕ ВОЛНУЙТЕСЬ» УЖЕ НЕ РАБОТАЕТ

Условия безопасной починки вычисляются на строках [35–36](https://github.com/ton-blockchain/minter/blob/6a68c32d3672cb368e2776ca2034b79231eef043/src/pages/jetton/FaultyDeploy.tsx#L35-L36): метаданные не должны содержать ошибку, изображение должно быть доступно в исходном виде, число знаков после запятой должно присутствовать, а формат должен быть on-chain.

Если хотя бы один пункт не подходит, строка [44](https://github.com/ton-blockchain/minter/blob/6a68c32d3672cb368e2776ca2034b79231eef043/src/pages/jetton/FaultyDeploy.tsx#L44) бросает `Token metadata cannot be safely rewritten from the currently loaded data`. Кнопка обещает легкую починку, но сама канцелярия знает: иногда у нее нет достаточно чистой копии, чтобы переписать след преступления.

После отзыва права администратора исправление уже не предлагается. Ветка на строках [10–13](https://github.com/ton-blockchain/minter/blob/6a68c32d3672cb368e2776ca2034b79231eef043/src/pages/jetton/util.ts#L10-L13) переводит дело из ремонта в повторный выпуск. Сейф остается в цепи, история остается в цепи, а исправить формат может уже никто.

## ХРОНИКА МЕЛКОГО КРИНЖА

*Отдел формулировок.* На строке [11](https://github.com/ton-blockchain/minter/blob/6a68c32d3672cb368e2776ca2034b79231eef043/src/pages/jetton/util.ts#L11) слово `permanently` стоит рядом с советом `redeploy a new token`. Перевод с канцелярского на лондонский прост: старый жеттон не лечим, новый выписываем.

*Отдел успокоения.* На строке [91](https://github.com/ton-blockchain/minter/blob/6a68c32d3672cb368e2776ca2034b79231eef043/src/pages/jetton/FaultyDeploy.tsx#L91) интерфейс говорит `Don’t worry`, хотя несколькими строками выше и ниже описывает неисправную прошлую версию и отдельную транзакцию ремонта.

*Отдел памяти.* Строки [246–248](https://github.com/ton-blockchain/minter/blob/6a68c32d3672cb368e2776ca2034b79231eef043/src/lib/jetton-minter.ts#L246-L248) превращают устройство данных в свидетельский жетон. Если ссылок нет, парсер не молчит: он ставит флаг, что когда-то здесь прошел неисправный развертыватель.

## ВЕРДИКТ СЫЩИКА

Репозиторий [ton-blockchain/minter](https://github.com/ton-blockchain/minter) пойман на редком публичном самопризнании. Коммит [`6a68c32`](https://github.com/ton-blockchain/minter/commit/6a68c32d3672cb368e2776ca2034b79231eef043) в файле [`src/lib/jetton-minter.ts`](https://github.com/ton-blockchain/minter/blob/6a68c32d3672cb368e2776ca2034b79231eef043/src/lib/jetton-minter.ts) распознает плоскую структуру по строкам [246–249](https://github.com/ton-blockchain/minter/blob/6a68c32d3672cb368e2776ca2034b79231eef043/src/lib/jetton-minter.ts#L246-L249) и поднимает флаг старой неисправности.

В том же коммите файл [`src/pages/jetton/util.ts`](https://github.com/ton-blockchain/minter/blob/6a68c32d3672cb368e2776ca2034b79231eef043/src/pages/jetton/util.ts) честно разводит два финала на строках [10–14](https://github.com/ton-blockchain/minter/blob/6a68c32d3672cb368e2776ca2034b79231eef043/src/pages/jetton/util.ts#L10-L14): администратор еще может вызвать ремонт, либо жеттон становится навсегда непригодным и отправляется на повторный выпуск.

Редакция не утверждает, что каждый жеттон в городе поражен этой бедой. Код явно пытается обнаружить старый формат и дать владельцу путь восстановления. Но сама вывеска необычайно сочна: минтер сообщает пользователю, что часть его прошлых продуктов создана неисправной версией, а затем предлагает нажать кнопку и не волноваться.

Сыщик закрыл папку. Газовый рожок снова сипит в тумане. На двери осталась строка [11](https://github.com/ton-blockchain/minter/blob/6a68c32d3672cb368e2776ca2034b79231eef043/src/pages/jetton/util.ts#L11): `The token is permanently unusable`. В Лондоне даже жеттон иногда получает не чек, а свидетельство о смерти.

🐀
