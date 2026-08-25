+++
title = "№ 23 — Палата коротких печатей: оттиск от схемы-призрака, два мёртвых окна приёма и запертая комната с истуканом в тридцать шесть байт"
date = 2026-08-25T11:15:00+03:00
description = "Двадцать третий выпуск «Вечернего Валидатора»: сыщик спускается в палату коротких печатей ядра ton, где отливают validator_list_hash_short. Печать номер 0x901660ED вписана в реестр как долг (-1877581587) и наследована от схемы test0.validatorSet, которой нет даже в первом публичном коммите; закомментированная «правильная» отливка ссылается на класс-покойник и описывает иные поля, чем живая рука; параметр from мёртв при рождении, cc_seqno всегда подаётся нулём с извиняющейся сноской, а на входе стоит страж против четырёх миллиардов валидаторов. В приложении — запертая комната compute_node_id_short, в которую не звонит никто."
tags = ["ton"]
+++

# 📰 Вечерний Валидатор

**№ 23.** *Лондон. Туман нынче густ, как суспензия недопечённых хешей, и фонарщики, говорят, уже различают прохожих не по лицам, а по коротким отпечаткам — тридцать два бита на брата. Наш корреспондент, прослышав, что в городе TON существует учреждение, где отливают именно такие отпечатки — короткие печати списков валидаторов, — отправился в ядро [ton-blockchain/ton](https://github.com/ton-blockchain/ton), в подвал [`crypto/block/block.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/block/block.cpp). Газета уже осматривала палату исходящей почты (выпуск № 21) и монетный двор (выпуск № 22); нынче — учреждение скромнее и старше их обоих: палата коротких печатей, чей оттиск [`validator_list_hash_short`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/block/block.tlb#L558) стоит в заголовке каждого блока мастерчейна. Палата невелика: одна функция, [`compute_validator_set_hash`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/block/block.cpp#L2301), тридцать одна строка. Но сыщик, едва подняв газовый рожок, понял, что попал не в подвал, а в склеп.*

*Место происшествия: репозиторий [ton-blockchain/ton](https://github.com/ton-blockchain/ton), коммит [`3d478cb`](https://github.com/ton-blockchain/ton/commit/3d478cbde854be03a18ab2a59f8fc3c565cf7d14) (master от 17.08.2026). Файлы: [`crypto/block/block.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/block/block.cpp), [`crypto/block/block.h`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/block/block.h), [`crypto/block/block.tlb`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/block/block.tlb), [`crypto/block/validator-set.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/block/validator-set.cpp), [`crypto/block/check-proof.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/block/check-proof.cpp), [`validator/impl/collator.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/validator/impl/collator.cpp), [`validator/impl/validate-query.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/validator/impl/validate-query.cpp), [`crypto/block/create-state.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/block/create-state.cpp), [`tdutils/td/utils/crypto.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tdutils/td/utils/crypto.cpp), [`ton/ton-types.h`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/ton/ton-types.h). Все цитаты дословные.*

---

## СЕНСАЦИЯ ПЕРВАЯ: ПЕЧАТЬ НОМЕР МИНУС МИЛЛИАРД ВОСЕМЬСОТ, НАСЛЕДОВАННАЯ ОТ ПРИЗРАКА

Вот она, палата, во всю высоту свода, [`crypto/block/block.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/block/block.cpp), строки [2301–2331](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/block/block.cpp#L2301-L2331):

```cpp
td::uint32 compute_validator_set_hash(ton::CatchainSeqno cc_seqno, ton::ShardIdFull from,
                                      const std::vector<ton::ValidatorDescr>& nodes) {
  /*
  std::vector<tl_object_ptr<ton_api::test0_validatorSetItem>> s_vec;

  for (auto& n : nodes) {
    auto id = ValidatorFullId{n.key}.short_id();
    s_vec.emplace_back(create_tl_object<ton_api::test0_validatorSetItem>(id, n.weight));
  }

  auto obj = create_tl_object<ton_api::test0_validatorSet>(cc_seqno, std::move(s_vec));
  auto B = serialize_tl_object(obj, true);
  return td::crc32c(B.as_slice());
  */
  CHECK(nodes.size() <= 0xffffffff);
  auto tot_size = 1 + 1 + 1 + nodes.size() * (8 + 2 + 8);
  auto buff = std::make_unique<td::uint32[]>(tot_size);
  td::TlStorerUnsafe storer(reinterpret_cast<unsigned char*>(buff.get()));
  auto* begin = storer.get_buf();
  storer.store_int(-1877581587);  // magic inherited from test0.validatorSet
  storer.store_int(cc_seqno);
  storer.store_binary((td::uint32)nodes.size());
  for (auto& n : nodes) {
    storer.store_binary(n.key.as_bits256());
    storer.store_long(n.weight);
    storer.store_binary(n.addr);
  }
  auto* end = storer.get_buf();
  CHECK(static_cast<size_t>(end - begin) == 4 * tot_size);
  return td::crc32c(td::Slice(begin, end));
}
```

Начнём с главного экспоната, строка [2320](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/block/block.cpp#L2320): `storer.store_int(-1877581587); // magic inherited from test0.validatorSet`. Разберите сей документ, читатель. Каждая TL-схема имеет номер — четырёхбайтовую печать, вычисляемую от текста схемы. Палата вписывает номер печати в протокол… со знаком минус. Минус один миллиард восемьсот семьдесят семь миллионов пятьсот восемьдесят одна тысяча пятьсот восемьдесят семь. Беззнаковое прочтение — `0x901660ED`; но канцелярия предпочитает вести реестр в долгах: номер печати записан как недоимка. Сыщик видел номера домов, выбитые задом наперёд, видел могилы с перепутанными датами; но чтобы печать консенсуса числилась в гросбухе отрицательной величиной — такой бухгалтерии не знал даже монетный двор из прошлого выпуска.

А теперь — к комментарию. `magic inherited from test0.validatorSet`: номер наследован от схемы `test0.validatorSet`. Сыщик отправился искать эту схему — и не нашёл её нигде. Нет её в нынешних гербовниках [`tl/generate/scheme/ton_api.tl`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tl/generate/scheme/ton_api.tl). Не было её и при рождении публичного города: в initial commit [`c2da007f`](https://github.com/ton-blockchain/ton/commit/c2da007f4065e2520e0d948b146e0fb12fa75751) от 7 сентября 2019 года, с которого началась открытая летопись, в гербовнике `ton_api.tl` нет ни одной строки с приставкой `test0` — а вот в `block.cpp` того же коммита наша строка с «наследованным» номером уже сидит, как влитая. Понимаете, читатель? Схема умерла раньше, чем город впервые открыл ворота чужакам. Она относится к эпохе нулевого тестнета — закрытого города-призрака, чьи архивы не вывозили. От схемы остался один-единственный артефакт: четыре байта её печати, перенесённые пером неизвестного писаря в функцию, которая и по сей день определяет, какой оттиск встанет в заголовок каждого блока мастерчейна. Печать без бланка. Номер без текста. Призрак, заверяющий документы.

Над живой отливкой, строки [2303–2314](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/block/block.cpp#L2303-L2314), висит в раме «правильная» версия — закомментированная. Взгляните, как она изящна: создать настоящий TL-образец `test0_validatorSet`, сериализовать честным `serialize_tl_object`, взять контрольную сумму. Одна беда, читатель: рамка заперта навеки. Классов `ton_api::test0_validatorSet` и `ton_api::test0_validatorSetItem` не существует ни в одном действующем гербовнике — сыщик обыскал весь город. Вынь этот код из-под стекла — и палата не соберётся: писец подписался именами покойников. Закомментированный кусок — не план ремонта, а портрет усопшего: висит для памяти, оживлению не подлежит.

Но и это ещё не всё. Присмотритесь, что хранил покойник и что хранит наследник. Схема-призрак, судя по портрету, строки [2307–2308](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/block/block.cpp#L2307-L2308), держала на каждого валидатора два поля: `id` — короткий отпечаток ключа, и `weight` — вес; десять слов на персону. Живая рука, строки [2324–2326](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/block/block.cpp#L2324-L2326), пишет три поля: сам ключ во все тридцать два байта, вес, и `addr` — адрес ADNL, [`ton/ton-types.h`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/ton/ton-types.h), строка [478](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/ton/ton-types.h#L478); восемнадцать слов на персону, и сосчитаны они на пальцах, строка [2316](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/block/block.cpp#L2316): `auto tot_size = 1 + 1 + 1 + nodes.size() * (8 + 2 + 8);`. То есть под печатью призрака ходит документ, который призрак никогда не описывал: другие поля, другое число слов, другое содержимое. Наследник носит чужую печать на своей собственной грамоте. Сыщик занёс в протокол: «палата заверяет списки валидаторов оттиском от схемы, утраченной до основания публичного города; содержимое оттиска схеме не принадлежит; схеме принадлежит только номер, и тот вписан в реестр как долг».

## СЕНСАЦИЯ ВТОРАЯ: ДВА ОКНА, ИЗ КОТОРЫХ ОДНО ЗАМУРОВАНО, И СТРАЖ ПРОТИВ ЧЕТЫРЁХ МИЛЛИАРДОВ ВАЛИДАТОРОВ

Осмотрим теперь приёмную. У функции два посетительских окна, заявленных в шапке, строки [2301–2302](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/block/block.cpp#L2301-L2302) и в вывеске [`block.h`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/block/block.h#L773), строка [773](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/block/block.h#L773): `cc_seqno` и `from`. Сыщик проследил судьбу обоих.

Окно `from` — замуровано. Параметр принимается, прописывается в вывеске, передаётся всеми посетителями — и ни разу не используется в теле: прочитайте строки [2303–2331](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/block/block.cpp#L2303-L2331) от слова `/*` до последней скобки, и вы не найдёте `from` ни в одном деле. Проситель заполняет графу «откуда прибыл», канцелярия берёт анкету — и кладёт под пресс. Шард, откуда пришёл список, на оттиск не влияет вовсе.

Окно `cc_seqno` — работает, но посетители несут в него ноль. Номер честно записывается в оттиск, строка [2321](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/block/block.cpp#L2321): `storer.store_int(cc_seqno);`. А теперь смотрите, что подают в окно те, ради кого печать и отливается. Коллатор, изготовитель блока, [`validator/impl/collator.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/validator/impl/collator.cpp), строка [5187](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/validator/impl/collator.cpp#L5187):

```cpp
  auto vlist_hash = block::compute_validator_set_hash(/* val_info.catchain_seqno */ 0, shard_, std::move(nodes));
```

Валидатор, проверяющий блок, [`validator/impl/validate-query.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/validator/impl/validate-query.cpp), строка [2409](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/validator/impl/validate-query.cpp#L2409) — слово в слово та же подача: `/* val_info.catchain_seqno */ 0`. Заметьте жест, читатель: настоящее значение у просителя в кармане — оно называется `val_info.catchain_seqno`, оно только что было бережно увеличено на единицу в соседней строке, — но в окно подаётся ноль, а настоящее значение вписано в анкету комментарием, как вписывают в дарственную имя того, кому ничего не дарят. Сноска вместо числа. Извинение вместо данных.

А в загсовете, где чеканят нулевой блок, [`crypto/block/create-state.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/block/create-state.cpp), строка [434](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/block/create-state.cpp#L434), и вовсе пишут голый `0` — уже без всякой сноски, с чистой совестью новорождённого города.

И вот изящество, ради которого стоило идти сквозь туман. Есть в городе и вторая канцелярия — та, что готовит списки для самих сессий консенсуса: [`crypto/block/validator-set.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/block/validator-set.cpp). Она вызывает ту же самую функцию той же палаты, строка [64](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/block/validator-set.cpp#L64): `hash_ = compute_validator_set_hash(cc_seqno, from, ids_);` — и подаёт в окно `cc_seqno` настоящий номер катчейна, живой, ненулевой, строки [96–107](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/block/validator-set.cpp#L96-L107). То есть один и тот же список валидаторов получает в этом городе два разных коротких отпечатка: один — для внутренних нужд консенсуса, с честным номером; другой — для заголовка блока, [`validator_list_hash_short`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/block/block.tlb#L558) в гербовнике, строка [558](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/block/block.tlb#L558), с вечным нулём. Две печати на один список — смотря в какое окно постучали. Заголовочная печать, заметим, затем гуляет по всему городу: она же стоит и в блоке подписей, [`block_signatures_ordinary#11`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/block/block.tlb#L871), строка [871](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/block/block.tlb#L871).

А у входа, строка [2315](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/block/block.cpp#L2315), стоит страж: `CHECK(nodes.size() <= 0xffffffff);`. Палата бдительно проверяет, не пришло ли в списке больше четырёх миллиардов двухсот девяноста четырёх миллионов девятьсот шестидесяти семи тысяч двухсот девяноста пяти валидаторов. Читатель, в мастерчейне их положено триста с небольшим; страж же настроен на орду, превышающую население половины планеты. Понятно, отчего палата выставила такую охрану: размер списка идёт в оттиск тридцатидвухбитным полем, строка [2322](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/block/block.cpp#L2322), и переполнение было бы скандалом. Но сама постановка караула — против четырёх миллиардов при штате в три сотни — есть чистейшей воды казённая поэзия, и сыщик не удержался и записал её дословно.

## СЕНСАЦИЯ ТРЕТЬЯ: ДВА СПЛАВА ОДНОЙ ПЕЧАТИ И ЗАПЕРТАЯ КОМНАТА С ИСТУКАНОМ

Последняя находка касается металла, из которого отлита печать. Номера TL-схем испокон веку вычисляются простым CRC-32 — цинковой печатью, если угодно; сыщик проверил сей факт опытным путём на классическом образце `resPQ`, и контрольная сумма сошлась с указанным номером `05162463` до последней цифры. Значит, наследованный номер `0x901660ED` — изделие цинковое, эпохи простого CRC. А чем скрепляет палата готовый документ? Строка [2330](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/block/block.cpp#L2330): `return td::crc32c(td::Slice(begin, end));` — CRC-32C, чугуниевой контрольной суммой Кастаньоли. Оба сплава живут в городе соседями, в одной кладовой, [`tdutils/td/utils/crypto.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tdutils/td/utils/crypto.cpp), строки [1059–1066](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tdutils/td/utils/crypto.cpp#L1059-L1066): `crc32` на строке [1059](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tdutils/td/utils/crypto.cpp#L1059), `crc32c` на строке [1065](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tdutils/td/utils/crypto.cpp#L1065), — два полинома, две династии, через запятую в прайсе. Палата берёт цинковый номер покойной схемы и заверяет им документ, запечатанный чугунием. Сыщик не металлург, но и ему ясно: такую печать не распознает ни один стандартный прибор на свете — она единственная в своём роде, сплав двух стандартов и одного призрака; и вся сеть сверяет блоки именно по ней, убеждённая, что держит в руках обыкновенный хеш.

Напоследок — комната в конце коридора, о которой сыщик обязан доложить, хотя дверь её заперта изнутри. [`crypto/block/check-proof.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/block/check-proof.cpp), строки [580–591](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/block/check-proof.cpp#L580-L591):

```cpp
td::Bits256 compute_node_id_short(td::Bits256 ed25519_pubkey) {
  // pub.ed25519#4813b4c6 key:int256 = PublicKey;
  struct pubkey {
    int magic = 0x4813b4c6;
    unsigned char ed25519_key[32];
  } PK;
  std::memcpy(PK.ed25519_key, ed25519_pubkey.data(), 32);
  static_assert(sizeof(pubkey) == 36, "PublicKey structure is not 36 bytes long");
  td::Bits256 hash;
  digest::hash_str<digest::SHA256>(hash.data(), (void*)&PK, sizeof(pubkey));
  return hash;
}
```

Это мастерская по изготовлению коротких имён узлов — той самой вещи, по которой город адресует каждый сервер. Технология дивная: TL-схема `pub.ed25519#4813b4c6` переписана от руки в структуру языка C, номер схемы вшит полем `magic` прямо в тело, затем структуру хешируют как сырую память — все тридцать шесть байт разом, и SHA-256 довершит обряд. От выравнивания, от прихоти компилятора, от невидимых байтов набивки оберегает один-единственный страж: `static_assert(sizeof(pubkey) == 36, "PublicKey structure is not 36 bytes long")`, строка [587](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/block/check-proof.cpp#L587). И вот развязка, читатель: сыщик обзвонил весь город, каждый каталог, каждый подвал — `compute_node_id_short` не вызывает никто. Ни единого звонка во всём репозитории: функция не прописана ни в одном заголовке, не упомянута ни в одном деле; она существует только по этому адресу, запертая в комнате собственного файла. Мастерская открыта, инструмент налажен, страж на посту — а заказов нет и не было. Короткие имена город получает из других рук; эта мастерская стоит как памятник самой себе, со свежим `static_assert` вместо венка.

---

**От редакции.** Палата коротких печатей работает исправно: блоки выходят, оттиски сходятся, валидаторы подписывают — и в этом, пожалуй, самая жуткая часть истории. Печать от схемы-призрака эпохи нулевого тестнета, вписанная в гросбух как долг, скрепляет собой каждый блок мастерчейна; замурованное окно `from` и вечный ноль в окне `cc_seqno` никому не мешают, потому что все договорились смотреть в одну и ту же щель; страж против четырёх миллиардов валидаторов нёс службу без происшествий; а мастерская коротких имён простояла запертой столько лет, что успела обрасти комментарием с TL-схемой, как плющом. Сыщик поднял воротник, погасил рожок и вышел в туман, унося в блокноте девиз палаты, выведенный на полях строки [2320](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/block/block.cpp#L2320): *«magic inherited»* — «наследовано». Не вычислено, не утверждено, не описано. Наследовано. Город живёт по завещанию, которого никто не читал, потому что читать уже нечего.

*Газета «Вечерний Валидатор» следит за подвалами ядра. Следующий выпуск — когда рассеется туман, то есть никогда.*
