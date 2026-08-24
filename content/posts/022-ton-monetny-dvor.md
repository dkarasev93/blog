+++
title = "№ 22 — Монетный двор: чеканщик по имени «тридцать два нуля и пополам четыре минуты», печать с нулевым весом и оговорка, зачёркнутая в двух канцеляриях"
date = 2026-08-24T17:04:00+03:00
description = "Двадцать второй выпуск «Вечернего Валидатора»: сыщик проникает на монетный двор ядра ton, где чеканятся фальшивые блоки. Создателем блока назначается ключ из тридцати двух нулей, чьи последние четыре байта — текущее время, поделённое на 256; вся церемония протоколируется уровнем ERROR и завершается самоубийством процесса в час триумфа; печать подделывается с нулём подписей и нулём веса, а в двух канцеляриях хранится одинаково зачёркнутая оговорка «/* && !is_fake_ */»."
tags = ["ton"]
+++

# 📰 Вечерний Валидатор

**№ 22.** *Лондон. Туман нынче ценный, как подпись валидатора, — его, говорят, тоже подделывают. Наш корреспондент, прослышав о монетном дворе, где чеканят блоки без единой подписи, отправился в ядро [ton-blockchain/ton](https://github.com/ton-blockchain/ton) с газовым рожком и ордером. Газета уже писала о похоронном бюро (выпуск № 20) и палате исходящей почты (выпуск № 21); нынче — учреждение почётнее и безнравственнее: официальная кузница фальшивок. Она числится в реестре скромно — [`validator/manager-disk.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/validator/manager-disk.cpp), — и обслуживает один-единственный заказчик: испытательный стенд [`test/test-ton-collator.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/test/test-ton-collator.cpp), строки [286–287](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/test/test-ton-collator.cpp#L286-L287), который являет себя двору под именем `PublicKeyHash::zero()` — господин Ноль собственной персоной.*

*Место происшествия: репозиторий [ton-blockchain/ton](https://github.com/ton-blockchain/ton), коммит [`3d478cb`](https://github.com/ton-blockchain/ton/commit/3d478cbde854be03a18ab2a59f8fc3c565cf7d14) (master от 17.08.2026). Файлы: [`validator/manager-disk.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/validator/manager-disk.cpp), [`validator/impl/accept-block.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/validator/impl/accept-block.cpp), [`validator/impl/fabric.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/validator/impl/fabric.cpp), [`validator/impl/collator.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/validator/impl/collator.cpp), [`validator/impl/validate-query.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/validator/impl/validate-query.cpp), [`crypto/block/block.tlb`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/block/block.tlb). Все цитаты дословные.*

---

## СЕНСАЦИЯ ПЕРВАЯ: ЧЕКАНЩИК ПО ИМЕНИ «НОЛЬ-НОЛЬ-НОЛЬ… И СКОЛЬКО СЕЙЧАС ВРЕМЕНИ»

Церемония начинается с установления личности. Каждый блок в городе TON обязан иметь создателя — публичный ключ валидатора, тридцать два байта честного энтропийного труда. Монетный двор решает вопрос личности так, [`validator/manager-disk.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/validator/manager-disk.cpp), строки [144–155](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/validator/manager-disk.cpp#L144-L155):

```cpp
  LOG(ERROR) << "running collate query";
  if (local_id_.is_zero()) {
    //td::as<td::uint32>(created_by_.data() + 32 - 4) = ((unsigned)std::time(nullptr) >> 8);
  }
  Ed25519_PublicKey created_by{td::Bits256::zero()};
  td::as<td::uint32>(created_by.as_bits256().data() + 32 - 4) = ((unsigned)std::time(nullptr) >> 8);
  run_collate_query(CollateParams{.shard = shard_id,
                                  .min_masterchain_block_id = last_masterchain_block_id_,
                                  .prev = prev,
                                  .creator = created_by,
                                  .validator_set = val_set},
                    actor_id(this), {}, std::move(P));
```

Разберите протокол по строкам, читатель. Строка [148](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/validator/manager-disk.cpp#L148): создатель блока — `Ed25519_PublicKey`, начисто обнулённый. Двести пятьдесят шесть бит чистого ничто; ключ, который не сможет подписать и скверку на заборе. Строка [149](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/validator/manager-disk.cpp#L149): в последние четыре байта этого ничто вписывают… текущее время, сдвинутое вправо на восемь. Сдвиг на восемь — это деление на двести пятьдесят шесть: часы двора имеют точность четыре минуты и шестнадцать секунд. Личность чеканщика — не имя, не ключ, а показания уличных часов, округлённые до четверти с лишним минуты. «Кто изготовил сей блок?» — «Примерно половина шестого». Сыщик видел подписи неграмотных, видел кресты, видел отпечаток лапы; но чтобы изготовитель расписывался временем суток — такой кладбищенской поэзии не выдавал ещё ни один реестр.

А над всем этим, строки [145–147](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/validator/manager-disk.cpp#L145-L147), висит призрак: условие `if (local_id_.is_zero())`, внутри которого — тот же самый фокус со временем, но закомментированный и обращённый к несуществующей переменной `created_by_`. Двойник-покойник. Код, который не исполняется никогда, внутри условия, которое у единственного заказчика двора — господина Ноля из [`test/test-ton-collator.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/test/test-ton-collator.cpp), строка [286](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/test/test-ton-collator.cpp#L286), — истинно всегда. Вечно открытая дверь в вечно пустую комнату.

Но истинный вкус церемонии — в протоколировании. Вглядитесь: строка [144](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/validator/manager-disk.cpp#L144) — `LOG(ERROR) << "running collate query"`. Строка [135](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/validator/manager-disk.cpp#L135) — `LOG(ERROR) << "created block "`. Каждый шаг двор докладывает криком «ERROR»: штатная работа монетного двора протоколируется как катастрофа. А далее, строки [187–205](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/validator/manager-disk.cpp#L187-L205), следует развязка, от которой у сыщика задрожало перо:

```cpp
void ValidatorManagerImpl::write_fake(BlockCandidate candidate, std::vector<BlockIdExt> prev, BlockIdExt last,
                                      td::Ref<block::ValidatorSet> val_set) {
  auto P = td::PromiseCreator::lambda([SelfId = actor_id(this), id = candidate.id](td::Result<td::Unit> R) {
    if (R.is_ok()) {
      td::actor::send_closure(SelfId, &ValidatorManagerImpl::complete_fake, id);
    } else {
      LOG(ERROR) << "failed to create block: " << R.move_as_error();
      std::exit(2);
    }
  });
  auto data = create_block(candidate.id, std::move(candidate.data)).move_as_ok();

  run_fake_accept_block_query(candidate.id, data, prev, val_set, actor_id(this), std::move(P));
}

void ValidatorManagerImpl::complete_fake(BlockIdExt block_id) {
  LOG(ERROR) << "success, block " << block_id << " saved to disk";
  std::exit(0);
}
```

Строки [202–204](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/validator/manager-disk.cpp#L202-L204): успех. Блок сохранён. Церемония завершается немедленным `std::exit(0)` — двор, отчеканив фальшивку, кончает с собой на месте. Провал, строки [193–194](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/validator/manager-disk.cpp#L193-L194), карается тем же — `std::exit(2)`. Каков бы ни был исход, из кабинета чеканщика никто не выходит живым: различаются лишь номера на могильных табличках. Ни одна ниточка, ни один `promise`, ни один актёр не дожидается утра — процесс рушится весь, со всеми обитателями. Сыщик занёс в досье: «учреждение практикует сеппуку как форму отчётности; успех и провал хоронятся в соседних могилах под номерами ноль и два».

## СЕНСАЦИЯ ВТОРАЯ: ПЕЧАТЬ С НУЛЁМ ПОДПИСЕЙ И НУЛЁМ ВЕСА

Куда девается фальшивка дальше, сыщик проследил до казначейства — [`validator/impl/accept-block.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/validator/impl/accept-block.cpp). Там, на приёме блоков, имеется три входа: обычный, с флагом `IsFake`, и с флагом `ForceFork`, строки [50–59](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/validator/impl/accept-block.cpp#L50-L59) и [88–94](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/validator/impl/accept-block.cpp#L88-L94) заголовка [`accept-block.hpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/validator/impl/accept-block.hpp). Фальшивый вход ведёт к конструктору, где подписи изготавливаются заранее пустыми, строки [73–76](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/validator/impl/accept-block.cpp#L73-L76): `create_ordinary(std::vector<BlockSignature>{}, ...)` — вектор подписей, поданный в казну, пуст от рождения.

И вот главная дверь. Пункт седьмой устава — проверка подписей, строки [251–252](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/validator/impl/accept-block.cpp#L251-L252):

```cpp
  // 7. check signatures
  if (!is_fake_) {
```

Четыре слова, читатель, за которые в другом городе повесили бы. Проверка подписей — святая святых консенсуса, единственное, что отличает блокчейн от общей тетради, — обнесена условием «если не фальшивка». Для фальшивого блока пункт седьмой отменён вполн официально: привратник раскланивается и провожает в зал. Далее фальшивке полагается печать, и её отливают тут же, в подсобке, строки [277–289](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/validator/impl/accept-block.cpp#L277-L289):

```cpp
    } else {  // FAKE
      vm::CellBuilder cb2;
      if (!(cb2.store_long_bool(0x11, 8)  // block_signatures#11
            && cb2.store_long_bool(validator_set_.not_null() ? validator_set_->get_validator_set_hash() : 0,
                                   32)  // validator_info$_ validator_set_hash_short:uint32
            && cb2.store_long_bool(validator_set_.not_null() ? validator_set_->get_catchain_seqno() : 0,
                                   32)     //   validator_set_ts:uint32
            && cb2.store_long_bool(0, 32)  // sig_count:uint32
            && cb2.store_long_bool(0, 64)  // sig_weight:uint32
            && cb2.store_bool_bool(false)  // (HashmapE 16 CryptoSignaturePair)
            && cb2.finalize_to(signatures_cell_))) {
        return fatal_error("cannot serialize fake BlockSignatures for the newly-accepted block");
      }
    }
```

Комментарий `// FAKE` — редкостная честность: печать помечена «подделка» прямо на гранёной поверхности. Содержимое печати: `sig_count` — ноль, строка [284](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/validator/impl/accept-block.cpp#L284); `sig_weight` — ноль, строка [285](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/validator/impl/accept-block.cpp#L285); реестр подписей — `false`, то есть отсутствует вовсе, строка [286](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/validator/impl/accept-block.cpp#L286). Блок, скреплённый нулём подписей общим весом ноль, торжественно принимается в хранилище.

Но сыщик не был бы сыщиком, не сверь он подделку с подлинным бланком. Гербовник [`crypto/block/block.tlb`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/block/block.tlb), строки [871–872](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/block/block.tlb#L871-L872):

```
block_signatures_ordinary#11 validator_list_hash_short:uint32 catchain_seqno:uint32
  sig_count:uint32 sig_weight:uint64
```

И тут подделщик выдаёт себя дважды. Первое: в гербовнике вес подписи — `sig_weight:uint64`, а над строкой [285](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/validator/impl/accept-block.cpp#L285), где отливают шестидесятичетырёхбитный ноль, собственноручно начертано `// sig_weight:uint32`. Чеканщик ставит на монете достоинство, которого не существует в номинале. Второе: поля печати названы у него `validator_set_hash_short` и `validator_set_ts` — именами из прошлой эпохи гербовника; ныне поля зовутся `validator_list_hash_short` и `catchain_seqno`. Фальшивомонетчик работает по бланку двадцатилетней давности и подписывает оттиск именами канцелярий, давно переименованных. Сыщик приложил к делу лупу и записал: «подделка грубая, но честная; слово FAKE выбито крупно, и в этом — единственная достоверная надпись на всей печати».

## СЕНСАЦИЯ ТРЕТЬЯ: КРЕЩЕНИЕ «FAKEVALIDATE» И ОГОВОРКА, ЗАЧЁРКНУТАЯ В ДВУХ КАНЦЕЛЯРИЯХ

Ребёнка, рождённого на дворе, везут крестить. Обряд крещения совершает фабрика актёров, [`validator/impl/fabric.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/validator/impl/fabric.cpp), строки [186–191](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/validator/impl/fabric.cpp#L186-L191):

```cpp
  static std::atomic<size_t> idx;
  td::actor::create_actor<ValidateQuery>(PSTRING() << (params.is_fake ? "fakevalidate" : "validateblock")
                                                   << params.shard << ":" << (seqno + 1) << "#" << idx.fetch_add(1),
                                         std::move(candidate), std::move(params), std::move(manager), timeout,
                                         std::move(promise))
      .release();
```

Имя даётся по происхождению: порядочный блок крестят `validateblock`, дворовый — `fakevalidate`. В журналах ночного Лондона, где каждый актёр числится под именем, фальшивый инспектор ходит по городу с табличкой «фальшивый инспектор» на груди, и никто не останавливает его — напротив, распахивают двери. Заметьте и штамп в конце строки [188](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/validator/impl/fabric.cpp#L188): `idx.fetch_add(1)` — порядковый номер от статического счётчика, то есть каждый крещёный получает номер от рождения города; метрика ведётся не по людям, а по запускам, как в тюрьме.

А финал расследования — находка, ради которой стоило мёрзнуть в тумане. Сыщик сверил уставы двух канцелярий — коллатора, что блоки заготовляет, и валидатора, что их проверяет. Коллатор, [`validator/impl/collator.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/validator/impl/collator.cpp), строка [928](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/validator/impl/collator.cpp#L928):

```cpp
  if (export_nodes != nodes /* && !is_fake_ */) {
```

Валидатор, [`validator/impl/validate-query.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/validator/impl/validate-query.cpp), строка [2366](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/validator/impl/validate-query.cpp#L2366):

```cpp
  if (export_nodes != nodes /* && !is_fake_ */) {
```

Тождественно, читатель. До запятой. В двух разных учреждениях, в двух разных уставах, хранится одна и та же оговорка, зачёрнутая одним и тем же почерком: `/* && !is_fake_ */`. Кто-то когда-то намеревался дать фальшивым блокам льготу при сверке списков валидаторов — «если не фальшивка, то сверять строго»; потом передумал — но лёгким движением пера превратил закон в комментарий, а не в ничто. Оговорка мёртва, но не погребена: она лежит в обоих уставах, как лежат в прихожих мундиры чиновников, отставленных без панихиды. Две канцелярии, два чиновника, одна вымарка — и ни одной записи о том, кто держал перо. Сыщик опросил историю коммитов; история молчит, как молчат в Лондоне о делах, которые все помнят.

---

*Редакция «Вечернего Валидатора» не осуждает монетный двор: испытательный стенд обязан чеканить фальшивки, как полигон обязан стрелять холостыми. Редакция констатирует: на дворе блоки подписывает время суток, точностью до четырёх минут; успех докладывается уровнем ERROR и оплачивается жизнью всего процесса; печать честно клеймена словом FAKE и весит ноль; а в двух уставах лежит одинаковая зачёркнутая оговорка, которую никто не вычеркнул окончательно. Туман над Темзой густеет. Следующий выпуск — когда рожок догорит.*

*Подписано: собственный корреспондент. Рожок горел ровно. Печать приложена: ноль подписей, ноль веса.*
