+++
title = "№ 73 — TON Unlocker и архив, выставленный на витрину"
date = 2026-09-20T11:01:00+03:00
description = "Семьдесят третий выпуск «Вечернего Валидатора»: сыщик входит в TON Unlocker и находит публичный архив вкладчиков, ручной вход по адресу и награду, посчитанную по общей кассе."
tags = ["ton-unlocker"]
+++

# ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

**№ 73.** *Лондон. Дневной туман еще не рассеялся, газовый рожок у редакции сипит, а сыщик получил сверток с вывеской TON Unlocker. Контора обещает вернуть людям их залоченные TON. Но при входе не спрашивает ни подпись, ни ключ, ни даже кошелек: достаточно назвать адрес. В подвале тем временем лежит сжатый список счетов всех вкладчиков.*

Репозиторий [ton-blockchain/ton-unlocker](https://github.com/ton-blockchain/ton-unlocker), коммит [`0e409aa`](https://github.com/ton-blockchain/ton-unlocker/commit/0e409aa2aff50ceeae584a5f4ad61fc0d0581f85), файл [`src/services/billsService.ts`](https://github.com/ton-blockchain/ton-unlocker/blob/0e409aa2aff50ceeae584a5f4ad61fc0d0581f85/src/services/billsService.ts), строки [23–61](https://github.com/ton-blockchain/ton-unlocker/blob/0e409aa2aff50ceeae584a5f4ad61fc0d0581f85/src/services/billsService.ts#L23-L61) и [66–93](https://github.com/ton-blockchain/ton-unlocker/blob/0e409aa2aff50ceeae584a5f4ad61fc0d0581f85/src/services/billsService.ts#L66-L93). Цитаты сверены по содержимому файла на этом коммите.

## АРХИВ ВКЛАДЧИКОВ НА УЛИЧНОЙ ВИТРИНЕ

Сначала сыщик подошел к окну загрузки. Контора сама сообщает каждому посетителю адрес архива, строки [30–31](https://github.com/ton-blockchain/ton-unlocker/blob/0e409aa2aff50ceeae584a5f4ad61fc0d0581f85/src/services/billsService.ts#L30-L31):

```ts
const billsPath = "https://locker.ton.org/bills.gz";
console.log(billsPath);
```

Далее приложение скачивает файл без всякой авторизации, разжимает его и превращает в JSON, строки [33–45](https://github.com/ton-blockchain/ton-unlocker/blob/0e409aa2aff50ceeae584a5f4ad61fc0d0581f85/src/services/billsService.ts#L33-L45):

```ts
this.loadingPromise = fetch(billsPath)
  .then((res) => {
    if (!res.ok) {
      throw new Error(`HTTP error! status: ${res.status}`);
    }
    return res.arrayBuffer();
  })
  .then((compressedData) => {
    const decompressedData = inflate(new Uint8Array(compressedData), {
      to: "string",
    });
    return JSON.parse(decompressedData);
  })
```

Внутри интерфейсной ведомости прямо перечислены адрес счета, адрес пользователя, общий вклад и время последнего вывода, строки [4–9](https://github.com/ton-blockchain/ton-unlocker/blob/0e409aa2aff50ceeae584a5f4ad61fc0d0581f85/src/services/billsService.ts#L4-L9):

```ts
interface BillData {
  billAddress: string;
  userAddress: string;
  totalDeposit: string;
  lastWithdrawTime: number;
}
```

Это не случайный лог и не временный дамп в консоли. После загрузки каждая запись попадает в карту по адресу пользователя, строки [46–56](https://github.com/ton-blockchain/ton-unlocker/blob/0e409aa2aff50ceeae584a5f4ad61fc0d0581f85/src/services/billsService.ts#L46-L56):

```ts
this.billsData = data;

// Create a map for quick lookup by user address
data.bills.forEach((bill) => {
  // Normalize address to raw format for consistent lookup
  const normalizedAddress = Address.parse(
    bill.userAddress
  ).toRawString();
  this.userToBillMap.set(normalizedAddress, bill);
});
```

Сыщик поправляет шляпу. Публичный клиент не просто знает, где лежит архив. Он превращает весь архив в адресную телефонную книгу, где любой посетитель может проверить, числится ли конкретный адрес среди вкладчиков.

## ПРИЕМНАЯ: «НАЗОВИТЕ АДРЕС»

В конторе есть и запасная дверь. Файл [`src/components/WalletConnector.tsx`](https://github.com/ton-blockchain/ton-unlocker/blob/0e409aa2aff50ceeae584a5f4ad61fc0d0581f85/src/components/WalletConnector.tsx) на строках [98–103](https://github.com/ton-blockchain/ton-unlocker/blob/0e409aa2aff50ceeae584a5f4ad61fc0d0581f85/src/components/WalletConnector.tsx#L98-L103) торжественно предлагает ее посетителю:

```tsx
<button
  onClick={() => setShowManualInput(true)}
  className="w-full border border-gray-300 hover:border-gray-400 text-gray-700 font-semibold py-3 px-6 rounded-lg transition-colors"
>
  Enter Address Manually
</button>
```

Форма принимает адрес, разбирает его и сверяет с тем самым публичным архивом, строки [31–48](https://github.com/ton-blockchain/ton-unlocker/blob/0e409aa2aff50ceeae584a5f4ad61fc0d0581f85/src/components/WalletConnector.tsx#L31-L48):

```ts
const parsed = Address.parse(address);
const nonBounceable = parsed.toString({ bounceable: false, testOnly: isTestnet });

// Check if address exists in bills
const exists = await billsService.validateUserAddress(address);
if (!exists) {
  setAddressError('This address has no deposits in the locker');
  return null;
}
```

Проверка в сервисе столь же скромна, строки [66–74](https://github.com/ton-blockchain/ton-unlocker/blob/0e409aa2aff50ceeae584a5f4ad61fc0d0581f85/src/services/billsService.ts#L66-L74):

```ts
async getBillByUserAddress(userAddress: string): Promise<BillData | null> {
  await this.loadBills();

  try {
    const normalizedAddress = Address.parse(userAddress).toRawString();
    return this.userToBillMap.get(normalizedAddress) || null;
  } catch {
    return null;
  }
}
```

Затем отдельная функция выносит вердикт, строки [90–93](https://github.com/ton-blockchain/ton-unlocker/blob/0e409aa2aff50ceeae584a5f4ad61fc0d0581f85/src/services/billsService.ts#L90-L93):

```ts
async validateUserAddress(userAddress: string): Promise<boolean> {
  const bill = await this.getBillByUserAddress(userAddress);
  return bill !== null;
}
```

Итак, охрана у двери проверяет лишь наличие адреса в заранее скачанном списке. Кошелек нужен для настоящего вывода, но для разведки достаточно строки, похожей на адрес. Газовый рожок кашлянул: база вкладчиков превращена в публичную приемную с поиском.

## НАГРАДА ИЗ ОБЩЕЙ КАССЫ

Досье имеет еще одну выразительную страницу — [`src/api/tonApiClient.ts`](https://github.com/ton-blockchain/ton-unlocker/blob/0e409aa2aff50ceeae584a5f4ad61fc0d0581f85/src/api/tonApiClient.ts). Там приложение сперва получает запись из локального архива, строки [47–58](https://github.com/ton-blockchain/ton-unlocker/blob/0e409aa2aff50ceeae584a5f4ad61fc0d0581f85/src/api/tonApiClient.ts#L47-L58), а затем берет общие показатели сейфа, строки [61–76](https://github.com/ton-blockchain/ton-unlocker/blob/0e409aa2aff50ceeae584a5f4ad61fc0d0581f85/src/api/tonApiClient.ts#L61-L76):

```ts
const lockerData = await billsService.getLockerData();

// Get last_withdraw_time from bill
const lastWithdrawTime = await this.getLastWithdrawTime(billData.billAddress);

// Get fixed values from bills.json
const totalUserDeposit = BigInt(billData.totalDeposit);

// Calculate total with reward using local data
const totalUserDepositAndReward = lockerData.totalCoinsLocked > 0n
  ? totalUserDeposit + (totalUserDeposit * lockerData.totalReward / lockerData.totalCoinsLocked)
  : 0n;
```

Награда отдельного вкладчика здесь вычисляется через долю в общей сумме `totalCoinsLocked`. Это не запрос к бухгалтеру контракта, а арифметика на данных, пришедших из публичного архива. В соседнем кабинете приложение все же спрашивает контракт о доступной сумме, строки [78–100](https://github.com/ton-blockchain/ton-unlocker/blob/0e409aa2aff50ceeae584a5f4ad61fc0d0581f85/src/api/tonApiClient.ts#L78-L100) — но сама итоговая витрина продолжает собирать часть цифр из местного расчета.

## ДАТЫ НА ГВОЗДЕ

В файле [`src/services/vestingCalculator.ts`](https://github.com/ton-blockchain/ton-unlocker/blob/0e409aa2aff50ceeae584a5f4ad61fc0d0581f85/src/services/vestingCalculator.ts) даты не прячутся за конфигом. Они прибиты к исходнику, строки [1–5](https://github.com/ton-blockchain/ton-unlocker/blob/0e409aa2aff50ceeae584a5f4ad61fc0d0581f85/src/services/vestingCalculator.ts#L1-L5):

```ts
// Vesting parameters from the locker contract
const DEPOSITS_END_TIME = 1698019200; // 23 Oct 2023 00:00:00 GMT
const VESTING_START_TIME = 1760227200; // Oct 12 2025 00:00:00 GMT  
const VESTING_TOTAL_DURATION = 94608000; // 3 years in seconds
const UNLOCK_PERIOD = 2592000; // 30 days in seconds
```

Формула считает разблокированную долю по числу прошедших периодов, строки [18–35](https://github.com/ton-blockchain/ton-unlocker/blob/0e409aa2aff50ceeae584a5f4ad61fc0d0581f85/src/services/vestingCalculator.ts#L18-L35). Контора честно сообщает, что ее календарь живет в коде клиента. Стоит сменить параметры в контракте, а не в этом фолианте, и витрина начнет рассказывать свою версию времени.

## РЕЗЮМЕ СЫЩИКА

[ton-blockchain/ton-unlocker](https://github.com/ton-blockchain/ton-unlocker) пойман не на одной случайной опечатке, а на целом театре доверия. Коммит [`0e409aa`](https://github.com/ton-blockchain/ton-unlocker/commit/0e409aa2aff50ceeae584a5f4ad61fc0d0581f85) оставляет публичный путь к архиву в [`src/services/billsService.ts`](https://github.com/ton-blockchain/ton-unlocker/blob/0e409aa2aff50ceeae584a5f4ad61fc0d0581f85/src/services/billsService.ts) на строках [30–45](https://github.com/ton-blockchain/ton-unlocker/blob/0e409aa2aff50ceeae584a5f4ad61fc0d0581f85/src/services/billsService.ts#L30-L45), строит карту вкладчиков на строках [49–56](https://github.com/ton-blockchain/ton-unlocker/blob/0e409aa2aff50ceeae584a5f4ad61fc0d0581f85/src/services/billsService.ts#L49-L56), а на строках [90–93](https://github.com/ton-blockchain/ton-unlocker/blob/0e409aa2aff50ceeae584a5f4ad61fc0d0581f85/src/services/billsService.ts#L90-L93) сводит допуск к вопросу «есть ли адрес в списке».

Сыщик закрыл папку. На двери осталась надпись из строки [30](https://github.com/ton-blockchain/ton-unlocker/blob/0e409aa2aff50ceeae584a5f4ad61fc0d0581f85/src/services/billsService.ts#L30): `https://locker.ton.org/bills.gz`. В Лондоне архив принято держать в сейфе. Но если сейф раздают по ссылке, первым делом надо хотя бы спросить, кто выдал ключ.

🐀
