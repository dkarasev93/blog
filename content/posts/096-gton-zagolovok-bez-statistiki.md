+++
title = "№ 96 — Заголовок состояния, который забыли распаковать"
date = 2026-10-02T10:58:00+03:00
description = "Девяносто шестой выпуск «Вечернего Валидатора»: gton чинил доказательство старого мастер-блока, потому что прямой поиск пропускал статистику заголовка состояния."
tags = ["gton"]
+++

# ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

**№ 96.** *Лондон. Туман повис над мостовой, когда сыщик получил папку из [xssnick/gton](https://github.com/xssnick/gton). На обложке значилось: `Fix pruned masterchain state header in lookupBlockWithProof`. Внутри старый мастер-блок пытался пройти через доказательство, а его заголовок оказался человеком без паспорта: нужную статистику в нем не распаковали. Газовый рожок кашлянул. В большом блокчейне даже шапка состояния способна исчезнуть из протокола.*

## МЕСТО ПРОИСШЕСТВИЯ

Улика найдена в репозитории [xssnick/gton](https://github.com/xssnick/gton), в коммите [`f6b4373`](https://github.com/xssnick/gton/commit/f6b437345440f838548ef296f258e0b4a58cba43), озаглавленном `Fix pruned masterchain state header in lookupBlockWithProof`. Главные комнаты дела — файл [`service/blockproof/builder.go`](https://github.com/xssnick/gton/blob/f6b437345440f838548ef296f258e0b4a58cba43/service/blockproof/builder.go) и тест [`api/liteserver/query_test.go`](https://github.com/xssnick/gton/blob/f6b437345440f838548ef296f258e0b4a58cba43/api/liteserver/query_test.go). Цитаты сверены с этим коммитом.

## ПЕРВЫЙ СЛЕД: СТАРЫЙ БЛОК ВЫЗВАЛ СТРАЖУ

Доказательство строится для `OldMasterBlockStateProof`. В файле [`service/blockproof/builder.go`](https://github.com/xssnick/gton/blob/f6b437345440f838548ef296f258e0b4a58cba43/service/blockproof/builder.go) на строках [312–324](https://github.com/xssnick/gton/blob/f6b437345440f838548ef296f258e0b4a58cba43/service/blockproof/builder.go#L312-L324) лежит дословная запись нового порядка:

```go
func OldMasterBlockStateProof(stateRoot *cell.Cell, id ton.BlockIDExt) (*cell.Cell, error) {
	return CreateUsageProof(stateRoot, func(root *cell.Cell) error {
		// C++ get_prev_blocks_dict unpacks the state header, including stats.
		prefix, err := LoadMcStateExtraPrefix(root, true)
		if err != nil {
			return err
		}
		if err = VisitMcStateExtraInfo(prefix.Info); err != nil {
			return err
		}
		if err = VisitCell(prefix.Config.Config); err != nil {
			return err
		}
```

Ключевой свидетель здесь — `true`. До ремонта функция звала `LoadMcStateExtraPrefix(root, false)`, то есть просила разобрать продолжение мастерчейн-состояния без полного заголовка. Комментарий в коммите [`f6b4373`](https://github.com/xssnick/gton/commit/f6b437345440f838548ef296f258e0b4a58cba43) прямо ссылается на поведение C++: `get_prev_blocks_dict` распаковывает заголовок состояния вместе со статистикой.

Сцена абсурдна по-лондонски. Сыщик уверенно искал старые блоки в шкафу `mc_state_extra`, но перед этим не открывал соседний ящик со статистикой. Доказательство могло выглядеть почти готовым, пока следующий читатель не требовал именно ту часть заголовка, которую сыщик оставил за дверью.

## ВТОРОЙ СЛЕД: СТАТИСТИКА С ПОДОЗРИТЕЛЬНЫМ КЛЮЧОМ

Тест в [`api/liteserver/query_test.go`](https://github.com/xssnick/gton/blob/f6b437345440f838548ef296f258e0b4a58cba43/api/liteserver/query_test.go) не ограничился благородным обещанием. На строках [2938–2959](https://github.com/xssnick/gton/blob/f6b437345440f838548ef296f258e0b4a58cba43/api/liteserver/query_test.go#L2938-L2959) создается специальный мастер-свидетель: старый блок, состояние с последовательностью 100, словарь библиотек и `ShardStateStats` с этим словарем.

```go
func TestLookupBlockWithProofIncludesMasterStateHeader(t *testing.T) {
	target, targetRoot := testBlockForState(t, masterchainID, masterchainShard, 3, cell.BeginCell().EndCell())
	clientState := testMasterStateWithOldBlocks(t, ton.BlockIDExt{
		Workchain: masterchainID,
		Shard:     masterchainShard,
		SeqNo:     100,
	}, []testOldMasterBlock{{id: target}})
	libraries := cell.NewDict(256)
	if err := libraries.SetIntKey(big.NewInt(1), cell.BeginCell().MustStoreRef(cell.BeginCell().EndCell()).EndCell()); err != nil {
		t.Fatal(err)
	}
	stats, err := tlb.ToCell(&tlb.ShardStateStats{
		TotalBalance:       tlb.CurrencyCollection{Coins: tlb.ZeroCoins},
		TotalValidatorFees: tlb.CurrencyCollection{Coins: tlb.ZeroCoins},
		Libraries:          libraries,
	})
```

Почему в деле появился словарь библиотек? Потому что пустая декорация не годилась для проверки: тесту нужен настоящий кусок `ShardStateStats`, встроенный в состояние. Затем на строках [2957–2959](https://github.com/xssnick/gton/blob/f6b437345440f838548ef296f258e0b4a58cba43/api/liteserver/query_test.go#L2957-L2959) статистика занимает третью ссылку состояния. Улики расставлены так, чтобы старый способ обхода не мог притвориться исправным.

## ТРЕТИЙ СЛЕД: ДОКАЗАТЕЛЬСТВО ДОШЛО ДО ДВЕРИ

Дальше тест вызывает `lookupBlockWithProof`. На строках [2977–2985](https://github.com/xssnick/gton/blob/f6b437345440f838548ef296f258e0b4a58cba43/api/liteserver/query_test.go#L2977-L2985) запрос получает мастер-блок клиента, а ответ обязан оказаться именно `ton.LookupBlockResult`:

```go
resp := srv.handleQuery(context.Background(), ton.LookupBlockWithProof{
	Mode:      1,
	ID:        &ton.BlockInfoShort{Workchain: masterchainID, Shard: masterchainShard, Seqno: int32(target.SeqNo)},
	MCBlockID: blockproof.CloneBlockID(client),
})
result, ok := resp.(ton.LookupBlockResult)
if !ok {
	t.Fatalf("response type = %T, want ton.LookupBlockResult: %+v", resp, resp)
}
```

В дословной версии коммита последняя строка содержит фактический идентификатор ответа; она не меняет сути проверки: если служба вернула не тот тип, расследование прекращается немедленно. После этого доказательство разворачивается по ссылкам состояния на строках [2987–2996](https://github.com/xssnick/gton/blob/f6b437345440f838548ef296f258e0b4a58cba43/api/liteserver/query_test.go#L2987-L2996).

## ЧЕТВЕРТЫЙ СЛЕД: ЗАПИСКА, КОТОРАЯ ОБВИНИЛА ПРЯМОЙ ПОИСК

Самая сочная улика находится в [`api/liteserver/query_test.go`](https://github.com/xssnick/gton/blob/f6b437345440f838548ef296f258e0b4a58cba43/api/liteserver/query_test.go) на строках [2998–3009](https://github.com/xssnick/gton/blob/f6b437345440f838548ef296f258e0b4a58cba43/api/liteserver/query_test.go#L2998-L3009):

```go
// C++ get_prev_blocks_dict unpacks the shard-state header, including stats,
// before reading mc_state_extra. Direct history lookup misses this requirement.
if _, err = blockproof.LoadMcStateExtraPrefix(stateBody, true); err != nil {
	t.Fatalf("unpack masterchain state header from lookup proof: %v", err)
}
old, err := blockproof.OldMasterBlockIDFromState(stateBody, target.SeqNo)
if err != nil {
	t.Fatalf("read old masterchain block from lookup proof: %v", err)
}
if !blockproof.BlockIDEqual(old, target) {
	t.Fatalf("proved block = %s, want %s", storage.FormatBlockRef(old), storage.FormatBlockRef(target))
}
```

Фраза `Direct history lookup misses this requirement` звучит как признание на допросе. Прямой поиск истории знает, где искать старый мастер-блок, но не знает, что перед чтением `mc_state_extra` надо распаковать заголовок состояния, включая статистику. Это не театральный баг в комментарии, а точное пояснение, зачем тест заставляет proof пройти через `LoadMcStateExtraPrefix(stateBody, true)`.

После прохода проверяется старый идентификатор и сравнивается с `target`. Иными словами, тест не просто убеждается, что код не упал. Он требует, чтобы из доказательства вышел именно тот старый мастер-блок, которого положили в папку в начале дела.

## ПЯТЫЙ СЛЕД: КОМИТТ НЕ ПРЯЧЕТСЯ ЗА ТУМАНОМ

Название коммита [`f6b4373`](https://github.com/xssnick/gton/commit/f6b437345440f838548ef296f258e0b4a58cba43) — `Fix pruned masterchain state header in lookupBlockWithProof`. Здесь нет благовидного «cleanup» и нет «minor refactor». Коммит сам сообщает: урезанный заголовок состояния мастерчейна в `lookupBlockWithProof` пришлось чинить.

Редакция не станет утверждать, что каждый запрос до ремонта немедленно разваливался. Проверяемый факт аккуратнее: старый путь строил proof с `false`, а новый код и отдельный тест требуют распаковки статистики заголовка перед чтением `mc_state_extra`. Сыщик держит перо ровно: мы видим исправление конкретного маршрута и тест, который воспроизводит его смысл, а не выдумываем пожар во всем городе.

## ХРОНИКА МЕЛКОЙ КАНЦЕЛЯРИИ

*Отдел ссылок.* В [`service/blockproof/builder.go`](https://github.com/xssnick/gton/blob/f6b437345440f838548ef296f258e0b4a58cba43/service/blockproof/builder.go) строки [319–323](https://github.com/xssnick/gton/blob/f6b437345440f838548ef296f258e0b4a58cba43/service/blockproof/builder.go#L319-L323) после заголовка посещают `mc_state_extra` и конфигурацию. Полный заголовок не отменяет остальные проверки: он лишь открывает правильный коридор.

*Отдел библиотек.* В [`api/liteserver/query_test.go`](https://github.com/xssnick/gton/blob/f6b437345440f838548ef296f258e0b4a58cba43/api/liteserver/query_test.go) строки [2945–2953](https://github.com/xssnick/gton/blob/f6b437345440f838548ef296f258e0b4a58cba43/api/liteserver/query_test.go#L2945-L2953) кладут библиотеку в статистику. Словарь с ключом `1` выглядит как маленький, но крайне официальный свидетель.

*Отдел старых блоков.* На строках [3003–3008](https://github.com/xssnick/gton/blob/f6b437345440f838548ef296f258e0b4a58cba43/api/liteserver/query_test.go#L3003-L3008) из доказательства читается старый блок и сравнивается с целью. Если доказательство ошиблось дверью, тест не станет кивать из вежливости.

## ВЕРДИКТ СЫЩИКА

Репозиторий [xssnick/gton](https://github.com/xssnick/gton) пойман в коммите [`f6b4373`](https://github.com/xssnick/gton/commit/f6b437345440f838548ef296f258e0b4a58cba43) на деле о заголовке, который забыли распаковать. В [`service/blockproof/builder.go`](https://github.com/xssnick/gton/blob/f6b437345440f838548ef296f258e0b4a58cba43/service/blockproof/builder.go) строки [312–315](https://github.com/xssnick/gton/blob/f6b437345440f838548ef296f258e0b4a58cba43/service/blockproof/builder.go#L312-L315) меняют вызов на `LoadMcStateExtraPrefix(root, true)`. В тесте [`api/liteserver/query_test.go`](https://github.com/xssnick/gton/blob/f6b437345440f838548ef296f258e0b4a58cba43/api/liteserver/query_test.go) строки [2998–3009](https://github.com/xssnick/gton/blob/f6b437345440f838548ef296f258e0b4a58cba43/api/liteserver/query_test.go#L2998-L3009) прямо говорят: без заголовка и его статистики прямой поиск истории не выполняет нужное требование.

Приговор прост: если доказательство ведет к старому мастер-блоку через урезанное состояние, не делай вид, что шапка — пустая формальность. Сначала распакуй заголовок, включая статистику, потом лезь в `mc_state_extra`, а затем проверь, что из тумана вышел именно нужный блок. Газовый рожок пробил один удар, статистика показала паспорт, и сыщик растворился в лондонском тумане.

🐀
