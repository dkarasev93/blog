+++
title = "№ 11 — Соль, известная всему Лондону: «dummy secret» в кладовой ключей tonlib"
date = 2026-08-19T11:10:00+03:00
description = "Одиннадцатый выпуск «Вечернего Валидатора»: сыщик спускается в кладовую ключей телеграфной конторы tonlib и находит там секрет, вышитый прямо на стене, ключ-призрак из тридцати двух нулей и ведомость ошибок, состоящую из одного слова «TODO»."
tags = ["ton", "tonlib"]
+++

# 📰 Вечерний Валидатор

**№ 11.** *Лондон. Туман нынче сырой, как губка клерка, и газовые рожки шипят на прохожих с укоризной. Наш корреспондент возвращается в машинное отделение [ton-blockchain/ton](https://github.com/ton-blockchain/ton) — но не в котельную, где валяется «очень глупый вектор» (выпуск № 4), а этажом выше, в телеграфную контору tonlib: именно она принимает депеши от кошельков, подписывает их и хранит ключи клиентов в собственной кладовой. В кладовой, читатель, есть сейф. У сейфа — секрет. У секрета — табличка.*

*Место происшествия: репозиторий [ton-blockchain/ton](https://github.com/ton-blockchain/ton), коммит [`3d478cb`](https://github.com/ton-blockchain/ton/commit/3d478cbde854be03a18ab2a59f8fc3c565cf7d14) (master от 17.08.2026), кладовая [`tonlib/tonlib/KeyStorage.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tonlib/tonlib/KeyStorage.cpp). Все цитаты дословные.*

---

## СЕНСАЦИЯ ПЕРВАЯ: СЕКРЕТ, ВЫШИТЫЙ НА СТЕНЕ КЛАДОВОЙ

Процедура проста и по-лондонски честна. Клиент просит выдать его ключ в зашифрованном виде — чтобы перенести на другую машину. Кладовщик соглашается и идёт к сейфу. Файл [`tonlib/tonlib/KeyStorage.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tonlib/tonlib/KeyStorage.cpp), строки [175–183](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tonlib/tonlib/KeyStorage.cpp#L175-L183):

```cpp
td::SecureString get_dummy_secret() {
  return td::SecureString("dummy secret of 32 bytes length!");
}
td::Result<KeyStorage::ExportedEncryptedKey> KeyStorage::export_encrypted_key(InputKey input_key,
                                                                              td::Slice key_password) {
  TRY_RESULT(decrypted_key, export_decrypted_key(std::move(input_key)));
  TRY_RESULT(res, decrypted_key.encrypt(key_password, get_dummy_secret()));
  return ExportedEncryptedKey{std::move(res.encrypted_data)};
}
```

Остановимся на строке [176](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tonlib/tonlib/KeyStorage.cpp#L176). Секрет, которым солят шифрование экспортируемого ключа, — не случаен. Он написан в исходниках, по слогам, с восклицательным знаком: `dummy secret of 32 bytes length!`. Сыщик, от скуки полицейского ремесла, пересчитал буквы: ровно тридцать два, включая восклицательный знак. Табличка не врёт. Табличка просто висит на стене публичного репозитория.

Чтобы оценить тонкость происшествия, сыщик поднял чертёж самого замка — [`tonlib/tonlib/keys/DecryptedKey.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tonlib/tonlib/keys/DecryptedKey.cpp), строки [37–43](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tonlib/tonlib/keys/DecryptedKey.cpp#L37-L43):

```cpp
td::Result<EncryptedKey> DecryptedKey::encrypt(td::Slice local_password, td::Slice old_secret) const {
  td::SecureString secret(32);
  if (old_secret.size() == td::as_slice(secret).size()) {
    secret.as_mutable_slice().copy_from(old_secret);
  } else {
    td::Random::secure_bytes(secret.as_mutable_slice());
  }
```

Устройство замка безупречно: подай пустую соль — будет честная случайная из `td::Random::secure_bytes` (строка [42](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tonlib/tonlib/keys/DecryptedKey.cpp#L42)). Подай старую в тридцать два байта — замок покорно возьмёт её (строки [39–40](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tonlib/tonlib/keys/DecryptedKey.cpp#L39-L40)). И вот тут канцелярия принимает решение, достойное особой строки в хронике: вместо случайной соли всякий раз подаётся одна и та же, известная каждому читателю GitHub. «Dummy secret» годится ровно под одно условие замка — что он в тридцать два байта, — и потому проходит везде, куда прошла бы настоящая случайность. Приёмка же, строка [27](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tonlib/tonlib/keys/EncryptedKey.cpp#L27) файла [`EncryptedKey.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tonlib/tonlib/keys/EncryptedKey.cpp), проверяет лишь длину: не подошёл размер — «invalid secret size», подошёл — милости просим, хоть вышито на заборе.

Далее соль вместе с паролем уходит в PBKDF на сто тысяч итераций — [`EncryptedKey.h`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tonlib/tonlib/keys/EncryptedKey.h), строка [29](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tonlib/tonlib/keys/EncryptedKey.h#L29), `PBKDF_ITERATIONS = 100000`. Сто тысяч итераций против перебора — и одна соль на весь Лондон, чтобы перебиральщику не пришлось лишний раз вставать с кресла: заготовку под словарную атаку можно строить одну на всех, соль-то известна заранее. Справедливости ради: без пароля клиента сейф не откроется и с этой солью; взломщику всё ещё нужен пароль. Но соль в криптографии придумана именно затем, чтобы у каждого сейфа она была своя. В кладовой tonlib она одна, и вывешена у входа.

Обратная процедура тоже на месте: при ввозе ключа обратно кладовая заново собирает замок с той же табличкой — строки [190–195](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tonlib/tonlib/KeyStorage.cpp#L190-L195) подставляют `get_dummy_secret()` и отключают проверку публичного ключа (`decrypt(key_password, false)`), ибо публичный ключ при экспорте вообще не вывозится: в строке [192](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tonlib/tonlib/KeyStorage.cpp#L192) в замок вкладывается пустышка `td::Ed25519::PublicKey(td::SecureString())`. Замок без имени владельца, соль с забора — а дальше «TON local key» и сто тысяч итераций. Благородство механизма, кустарность ключа от него.

## СЕНСАЦИЯ ВТОРАЯ: КЛЮЧ-ПРИЗРАК ИЗ ТРИДЦАТИ ДВУХ НУЛЕЙ

Пока сыщик стоял у сейфа, из тумана проступила вторая фигура. В той же кладовой, строки [206–212](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tonlib/tonlib/KeyStorage.cpp#L206-L212):

```cpp
KeyStorage::PrivateKey KeyStorage::fake_private_key() {
  return PrivateKey{td::SecureString(32, 0)};
}

KeyStorage::InputKey KeyStorage::fake_input_key() {
  return InputKey{{td::SecureString(32, 0), td::SecureString(32, 0)}, {}};
}
```

В конторе заведён фиктивный ключ. Приватная его часть — тридцать два нулевых байта; пароль — пуст; публичная часть — тоже нули. Опознание призрака ведётся по строгой форме, строки [214–228](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tonlib/tonlib/KeyStorage.cpp#L214-L228): лямбда `is_zero` добросовестно пересчитывает каждый байт, и лишь когда пуст пароль (ноль байтов), нулев секрет (тридцать два) и нулев публичный ключ (тридцать два), — кладовщик всплёскивает руками и выдаёт фиктивный приватный ключ, не заглядывая в сейф вовсе: строки [111–114](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tonlib/tonlib/KeyStorage.cpp#L111-L114).

Для чего призрак? Дело честное: в телеграммах конторы, схема [`tl/generate/scheme/tonlib_api.tl`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tl/generate/scheme/tonlib_api.tl), строка [34](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tl/generate/scheme/tonlib_api.tl#L34), прямо значится `inputKeyFake = InputKey;` — клиент вправе спросить состояние счёта, не являя ключа, и контора отвечает, подставляя нулевого двойника: [`TonlibClient.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tonlib/tonlib/TonlibClient.cpp), строки [3791–3795](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tonlib/tonlib/TonlibClient.cpp#L3791-L3795). Механика полезная, как штатный манекен у портного. И всё же газета не может не отметить: в кладовой, где лежат настоящие ключи клиентов, на гвозде рядом висит ключ из тридцати двух нулей — валидный, между прочим, приватный ключ Ed25519, — и троица `fake_private_key` / `fake_input_key` / `is_fake_input_key` гордо значится в публичной описи [`KeyStorage.h`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tonlib/tonlib/KeyStorage.h), строки [77–79](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tonlib/tonlib/KeyStorage.h#L77-L79). Не дай туман, чей-нибудь настоящий кошелёк окажется нулевым: кладовая даже не заметит подмены — она сама его подменит первой.

---

## ХРОНИКА МЕЛКИХ ПРОИСШЕСТВИЙ

*Отдел ведомостей ошибок.* В справочном окне конторы — [`tonlib/tonlib/tonlib-cli.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tonlib/tonlib/tonlib-cli.cpp) — сыщик обнаружил, как клиенту отвечают на просьбу закрыть платёжный канал. Строки [880–882](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tonlib/tonlib/tonlib-cli.cpp#L880-L882):

```cpp
  void pchan_delete(td::ConstParser& parser, td::Promise<td::Unit> promise) {
    promise.set_error(td::Status::Error("TODO"));
  }
```

Диагноз безапелляционный: ошибка есть, и зовут её «TODO». Не «не сделано», не «приходите завтра» — само слово «TODO» выдано клиенту как официальное заключение, строка [881](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tonlib/tonlib/tonlib-cli.cpp#L881). В Лондоне так отвечает сыскная контора, у которой план расследования подменил расследование.

*Отдел землемеров.* А вот и жемчужина дня, ради которой стоило поднять воротник. Контора пересчитывает нулевой блок шардчейна в свою внутреннюю форму, [`TonlibClient.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tonlib/tonlib/TonlibClient.cpp), строки [6037–6040](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tonlib/tonlib/TonlibClient.cpp#L6037-L6040):

```cpp
auto to_tonlib_api(const ton::lite_api::tonNode_zeroStateIdExt& zeroStateId)
    -> tonlib_api_ptr<tonlib_api::ton_blockIdExt> {
  return tonlib_api::make_object<tonlib_api::ton_blockIdExt>(  //TODO check wether shard indeed 0???
      zeroStateId.workchain_, 0, 0, zeroStateId.root_hash_.as_slice().str(), zeroStateId.file_hash_.as_slice().str());
```

Шард проставлен нулём, seqno проставлен нулём — и над ними, строка [6039](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tonlib/tonlib/TonlibClient.cpp#L6039), дрожит самая честная пометка года: «TODO check wether shard indeed 0???». Три вопросительных знака, слово «wether» — что по-английски, между прочим, «баран», — и полная неуверенность в том, нулевой ли шард у нулевого блока. Землемер воткнул вешку, отошёл на шаг и спросил у тумана, туда ли воткнул. Туман, по обычаю, промолчал — и вешка с пометкой стоит по сей день, при мастере от 17 августа 2026 года.

---

## ВЕРДИКТ РЕДАКЦИИ

Скажем честно, как учит протокол: кладовая tonlib в целом добросовестна — PBKDF со ста тысячами итераций, разделение паролей, фиктивный ключ для анонимных запросов, и всё это под капотом, которым пользуются кошельки половины города. Без пароля клиента «dummy secret» не откроет ничего. Но газета ведёт хронику почерка, а почерк таков: соль вышита на стене, фиктивный ключ сведён к тридцати двум нулям и висит на видном гвозде, ведомость ошибок состоит из слова «TODO», а над координатой нулевого шарда дрожат три вопросительных знака и один баран. Сыщик закрыл блокнот, погасил газовый рожок и вышел в туман. Туман над Темзой, по обычаю, всё принял — и, похоже, уже давно знает секрет в тридцать два байта.
