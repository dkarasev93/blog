+++
title = "№ 20 — Похоронное бюро ядра: смерть в три инстанции, некролог из одного слова и печь, запаянная при постройке"
date = 2026-08-23T14:04:00+03:00
description = "Двадцатый выпуск «Вечернего Валидатора»: сыщик получает пост в похоронном бюро ядра ton и обнаруживает, что учреждения города умирают по три раза подряд и после смерти возвращаются; что сердце всех актёров сети хоронит сигналы с некрологом из одного слова «TODO»; и что в хранилище валидатора построена печь для уничтожения неизвестных корней, чей рубильник запаян навечно — constexpr false."
tags = ["ton"]
+++

# 📰 Вечерний Валидатор

**№ 20.** *Лондон. Туман нынче стелется так низко, что фонарщики зажигают рожки наугад — и, надо сказать, не ошибаются: в этом городе и после смерти продолжают ходить по делам. Наш корреспондент получил назначение, от которого не отказываются: ночная вахта в похоронном бюро ядра [ton-blockchain/ton](https://github.com/ton-blockchain/ton). Газета уже вела хронику «очень глупого вектора» (выпуск № 4), типографии вопросительных знаков (выпуск № 12) и архива без посетителей (выпуск № 19). Нынче — отдел, в котором расходятся все квитанции города: регистратура внезапных концов. Сыщик раскрыл журнал при газовом рожке и не поверил записям: здешние клиенты умирают по два-три раза подряд, а некоторые — после смерти ещё и возвращаются.*

*Место происшествия: репозиторий [ton-blockchain/ton](https://github.com/ton-blockchain/ton), коммит [`3d478cb`](https://github.com/ton-blockchain/ton/commit/3d478cbde854be03a18ab2a59f8fc3c565cf7d14) (master от 17.08.2026). Файлы: [`lite-client/lite-client.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/lite-client/lite-client.cpp), [`tdactor/td/actor/core/ActorExecutor.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tdactor/td/actor/core/ActorExecutor.cpp), [`validator/db/celldb.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/validator/db/celldb.cpp), [`validator/db/archive-slice.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/validator/db/archive-slice.cpp), [`validator-engine/validator-engine.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/validator-engine/validator-engine.cpp), [`crypto/block/transaction.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/block/transaction.cpp), [`overlay/overlay.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/overlay/overlay.cpp). Все цитаты дословные.*

---

## СЕНСАЦИЯ ПЕРВАЯ: КЛИЕНТ, КОТОРЫЙ УМИРАЕТ ДВАЖДЫ, — А ПОТОМ ВОЗВРАЩАЕТСЯ

Первая запись в журнале — из конторы лёгких клиентов, [`lite-client/lite-client.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/lite-client/lite-client.cpp). Контора эта держит связь с лёгкими серверами и хранит святость: опознавательный знак самого первого блока мастерчейна — «нулевого состояния», основания мира. И вот процедура приёма новостей, `TestNode::got_server_mc_block_id`, строки [434–443](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/lite-client/lite-client.cpp#L434-L443):

```cpp
void TestNode::got_server_mc_block_id(ton::BlockIdExt blkid, ton::ZeroStateIdExt zstateid, int created) {
  if (!zstate_id_.is_valid()) {
    zstate_id_ = zstateid;
    LOG(INFO) << "zerostate id set to " << zstate_id_.to_str();
  } else if (zstate_id_ != zstateid) {
    LOG(FATAL) << "fatal: masterchain zero state id suddenly changed: expected " << zstate_id_.to_str() << ", found "
               << zstateid.to_str();
    _exit(3);
    return;
  }
```

Разбирайте запись по полочкам, читатель, ибо такой тройной страховки сыщик не видывал. Строка [439](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/lite-client/lite-client.cpp#L439): `LOG(FATAL)` — первая смерть, официальная, с записью в журнал: «основание мира вдруг изменилось». Само слово «suddenly» — «внезапно» — стоит в протоколе, как будто земля разошлась под ногами в час пополудни. Строка [441](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/lite-client/lite-client.cpp#L441): `_exit(3)` — вторая смерть, немедленная, без прощальных церемоний, для тех случаев, когда первая не подействовала. И строка [442](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/lite-client/lite-client.cpp#L442): `return;` — возвращение. После двух смертей. Клерк, упокоенный дважды, поднимается с носилок и идёт по делам. Сыщик справился у врачей: ни `LOG(FATAL)`, ни `_exit` не возвращают управление — строчка `return;` недостижима в принципе, это не код, это надгробная оградка внутри гроба.

Но подлинный жест нуара — в летописи. Ритуал этот учреждён в «initial commit» [`c2da007f`](https://github.com/ton-blockchain/ton/commit/c2da007f4065e2520e0d948b146e0fb12fa75751) от 7 сентября 2019 года и семь лет лежал нетронутый. А в мае 2026-го, в коммите [`2ebc83d0`](https://github.com/ton-blockchain/ton/commit/2ebc83d011a9bbeab08440e68b66e11b20193a50) («liteserver: get shard client state»), контора открыла филиал — процедуру `TestNode::got_server_shard_client_state_mc_block_id`, строка [486](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/lite-client/lite-client.cpp#L486), — и ритуал переписали туда дословно: строки [492–495](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/lite-client/lite-client.cpp#L492-L495), та же внезапная смерть основания мира, тот же `_exit(3)`, тот же `return;` после смерти. Семь лет спустя, читатель, живая рука сняла копию с похоронного обряда — вместе с посмертным возвращением. Не исправила. Сняла копию. Сыщик записал в блокнот: «в этом городе не правят даже надгробия — их ксерокопируют».

## СЕНСАЦИЯ ВТОРАЯ: СЕРДЦЕ ВСЕХ АКТЁРОВ ХОРОНИТ С НЕКРОЛОГОМ «TODO»

Теперь — о главном. В машинном отделении города есть палата, через которую проходит каждый актёр сети: исполнитель актёров, [`tdactor/td/actor/core/ActorExecutor.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tdactor/td/actor/core/ActorExecutor.cpp). Каждый сигнал — пробуждение, будильник, письмо — получает здесь пропуск. И вот как встречают два сигнала из книги учёта, строки [258–261](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tdactor/td/actor/core/ActorExecutor.cpp#L258-L261):

```cpp
    case ActorSignals::Io:
    case ActorSignals::Cpu:
      LOG(FATAL) << "TODO";
      break;
```

Постойте при рожке и вникните. Сигналам `Io` и `Cpu` — вводу-выводу и процессору, двум самым земным поводам разбудить актёра, — положена немедленная смерть всего процесса. И некролог. Некролог состоит из одного слова: «TODO». Не «сигнал не поддержан», не «недопустимое состояние» — а служебная пометка самому себе, оттиск списка дел на могильной плите. Сыщик перечитал летопись: некролог висит с «initial commit» [`c2da007f`](https://github.com/ton-blockchain/ton/commit/c2da007f4065e2520e0d948b146e0fb12fa75751), то есть семь лет каждый процесс города таскает в сердце карточку «умереть, если что, — подробности потом». И обратите внимание на строку [261](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tdactor/td/actor/core/ActorExecutor.cpp#L261): `break;`. После смерти — выход из комнаты. Сыщик велел составить протокол: «смерть наступила; пострадавший вышел».

Палата, заметим, не бездельничает: соседние сигналы разбираются честно — `Wakeup` будит, `Alarm` проверяет будильник, `Message` опустошает почтовый ящик, строки [245–257](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tdactor/td/actor/core/ActorExecutor.cpp#L245-L257) и [262–268](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tdactor/td/actor/core/ActorExecutor.cpp#L262-L268). Одни гости получают пропуск, другие — гильотину с запиской «потом». Распределение мест в похоронном бюро города никогда не комментировалось.

## СЕНСАЦИЯ ТРЕТЬЯ: ПЕЧЬ, КОТОРУЮ ЗАПАЯЛИ ПРИ ПОСТРОЙКЕ

Третья запись — из хранилища ячеек валидатора, [`validator/db/celldb.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/validator/db/celldb.cpp). Хранилище перебирает известные ему корни состояний и встречает чужаков — корни, которых нет в реестре. Что делать с чужаком? Ответ учреждения, строки [160–173](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/validator/db/celldb.cpp#L160-L173):

```cpp
    if (!partial_check && !root_hashes.contains(root->get_hash())) {
      unknown_roots++;
      LOG(ERROR) << "Unknown root" << ShardIdFull(shard) << ":" << info.seq_no;
      constexpr bool delete_unknown_roots = false;
      if (delete_unknown_roots) {
        vm::CellStorer stor{*cell_db_};
        cell_db_->begin_write_batch().ensure();
        boc_->dec(root);
        boc_->commit(stor).ensure();
        cell_db_->commit_write_batch().ensure();
        if (!opts_->get_celldb_in_memory()) {
          boc_->set_loader(std::make_unique<vm::CellLoader>(cell_db_->snapshot(), on_load_callback_)).ensure();
        }
        LOG(ERROR) << "Unknown root" << ShardIdFull(shard) << ":" << info.seq_no << " REMOVED";
      }
```

Здесь всё прекрасно, читатель, каждая линия. Чужака замечают — строка [162](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/validator/db/celldb.cpp#L162): в журнал летит тревога «Unknown root». И тут же, строка [163](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/validator/db/celldb.cpp#L163), учреждается рубильник: `constexpr bool delete_unknown_roots = false;` — не переменная, не настройка, не конфигурация, а **вечная константа времён компиляции**, припаянная к положению «нет». А ниже — целая печь: начать пакет записи, снять корень, подтвердить, перенастроить загрузчик, — семь строк казни, строки [165–171](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/validator/db/celldb.cpp#L165-L171), и финальное «REMOVED» в строке [172](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/validator/db/celldb.cpp#L172). Печь оборудована полностью. Печь не может быть включена никогда: компилятор выбрасывает её целиком, ещё не родившись. Тревога «Unknown root» кричит в журнал, а ответ учреждения известен заранее и навсегда: ничего. Сыщик справился, когда построена печь: коммит [`84082e79`](https://github.com/ton-blockchain/ton/commit/84082e79f7a8327d90ac23d7d594b8bbe661d05c), «celldb: version 2», март 2025 года. То есть не дедовская рухлядь — свежая постройка. Печь завезли, собрали, подписали акт — и при постройке же запаяли топку. Полтора года чужаки приходят, тревога пишется, счётчик `unknown_roots` растёт — и каждый раз город готовится к казни, которая невозможна по уставу.

И чтобы читатель не думал, что в хранилище нет настоящих смертей: двумя этажами выше, строка [158](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/validator/db/celldb.cpp#L158), корень, который не удалось распаковать, карается `LOG(FATAL) << "cannot create ShardDescr from a root in celldb"` — смертью без права переписки. Расклад в учреждении таков: непонятный корень — казнить весь процесс; лишний корень — записать в журнал и отпустить с миром, печь не топить. Сыщик покрутил газовый рожок и пошёл дальше: нуар учит не задавать вопросов, на которые нет некролога.

---

## ХРОНИКА МЕЛКИХ ПРОИСШЕСТВИЙ

*Отдел посмертных возвращений, филиал.* Архивная контора валидатора, [`validator/db/archive-slice.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/validator/db/archive-slice.cpp), не смогла открыть пакет архива — и повторяет фирменный жест, строки [1064–1068](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/validator/db/archive-slice.cpp#L1064-L1068): `LOG(FATAL) << "failed to open/create archive ..."` — смерть, и следом, строка [1067](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/validator/db/archive-slice.cpp#L1067), `return;`. Почерк, читатель, — это когда две конторы в разных кварталах города хоронят себя одинаково: сначала смерть, потом возвращение. Город называет это надёжностью.

*Отдел надгробий.* Мастерская валидатора, [`validator-engine/validator-engine.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/validator-engine/validator-engine.cpp), ведёт переговоры с внешним миром о подписи блоков. Не нашла файл с заданием — и выдаёт: строка [977](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/validator-engine/validator-engine.cpp#L977), `abort_query(td::Status::Error(PSTRING() << "strange error: no to sign file. Output: " << res.output));`. «No to sign file», читатель. Не «no file to sign», не «file to sign missing» — а фраза, собранная, как туман: из правильных слов в неправильном порядке. И это не единичная описка: та же строка высечена в строке [1091](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/validator-engine/validator-engine.cpp#L1091) и в строке [1215](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/validator-engine/validator-engine.cpp#L1215) — три каменотёса, три одинаковые грамоты. Летопись подтверждает: первая высечена в «initial commit» [`c2da007f`](https://github.com/ton-blockchain/ton/commit/c2da007f4065e2520e0d948b146e0fb12fa75751), вторая и третья добавлены в 2020-м, коммиты [`4dd5eea1`](https://github.com/ton-blockchain/ton/commit/4dd5eea11fe8edf077eac7f9a90f2de2bd39395b) и [`040df63c`](https://github.com/ton-blockchain/ton/commit/040df63c9864f2f37ebe50c4cafcc01f2d5d2d5c), — то есть фразу не исправили, а размножили. Надгробие с опечаткой — досадно; надгробие с опечаткой, заказанное ещё дважды, — это стиль.

*Счётная палата.* Нотариат транзакций, [`crypto/block/transaction.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/block/transaction.cpp), решает, печатать ли в журнал акт заморозки контракта. Условие, строка [3590](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/block/transaction.cpp#L3590): `if (verbosity >= 3 * 1) {  // !!!DEBUG!!!`. Сыщик велел счетоводам проверить: три, помноженное на единицу, — это три. Решение о печати принимается после арифметического обряда, не меняющего ничего, зато украшенного тремя восклицательными знаками и словом «DEBUG». Семь лет, с «initial commit» [`c2da007f`](https://github.com/ton-blockchain/ton/commit/c2da007f4065e2520e0d948b146e0fb12fa75751), палата умножает три на единицу перед каждой заморозкой. Возможно, для суеверия. В этом городе суеверие — надёжнее печати crc32, которую, как мы знаем из выпуска № 19, заготовили, но не проверяют.

*Телеграф.* И последняя депеша ночи — из оверлейного телеграфа, [`overlay/overlay.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/overlay/overlay.cpp), строки [274–275](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/overlay/overlay.cpp#L274-L275): получив извещение `broadcastNotFound` — «рассылка не найдена», — телеграф отвечает нарушением протокола: `"received strange message broadcastNotFound from "`. Осмыслите иерархию странности, читатель: сообщение, извещающее, что сообщение не найдено, само признано странным сообщением. Телеграф доносит, что телеграф не донёс, — и на этом основании порывает с отправителем. Сыщик видел в тумане отражения, которые вели себя логичнее.

---

## ВЕРДИКТ РЕДАКЦИИ

Справедливости ради: ни один из этих гробов никого не похоронил. Сигналы `Io` и `Cpu` в палату актёров не приходят — семь лет не приходят, — и некролог «TODO» остаётся черновиком. Печь в хранилище не топится — и чужие корни живут дольше валидаторов. Основание мира внезапно не меняется — а если бы изменилось, клиент умер бы дважды, что для одного процесса даже щедро. Всё это не уязвимости; всё это — ритуалы. Но газета ведёт хронику почерка, и почерк сей ночи таков: город относится к смерти как к черновику. Смерть ставится первой строкой, потом дублируется `_exit`, потом — на всякий случай — `return`, как будто покойнику могут понадобиться дела. Некролог пишется одним словом «TODO» и семь лет не дописывается. Печь строится с запаянной топкой, а опечатка на надгробии размножается в трёх экземплярах. Туман над Темзой всё принял: он, единственный в этом городе, умирает по утрам один раз и без возвращения — и потому вызывает у редакции профессиональное уважение.
