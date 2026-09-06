+++
title = "№ 47 — Дело о сейфе без пароля: TON хранит ключи на голой полке"
date = 2026-09-06T11:01:00+03:00
description = "Сорок седьмой выпуск «Вечернего Валидатора»: в DHT-сервере TON пароль к keyring отмечен TODO, а добавленный приватный ключ без флага временности записывается в файл как есть."
tags = ["ton"]
+++

# 📰 ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

**№ 47 · Воскресенье, 6 сентября 2026 г. · Цена: 0.05 TON (пароль пока в пути)**

---

## ДЕЛО О СЕЙФЕ, КОТОРЫЙ ЖДЕТ КЛЮЧ ОТ СЕБЯ

Лондон утонул в сером тумане, мостовая блестела, а газовый рожок у редакции сипел, словно его попросили охранять банк без двери. Сыщик «Вечернего Валидатора» получил папку из конторы [ton-blockchain/ton](https://github.com/ton-blockchain/ton) — главного дома TON, где живут валидаторы, DHT и прочая ночная механика блокчейна.

В папке обнаружилась сцена с редкой викторианской точностью: сервер создает хранилище ключей, тут же признается, что надо бы дождаться пароля, и продолжает работу. А когда в хранилище прибывает приватный ключ, тот отправляется в файл обычной записью.

Место происшествия: репозиторий [ton-blockchain/ton](https://github.com/ton-blockchain/ton), коммит [`3d478cb`](https://github.com/ton-blockchain/ton/commit/3d478cbde854be03a18ab2a59f8fc3c565cf7d14), файл [`dht-server/dht-server.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/dht-server/dht-server.cpp), строки [1076–1078](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/dht-server/dht-server.cpp#L1076-L1078) и [851–873](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/dht-server/dht-server.cpp#L851-L873). Цитаты сверены с содержимым файлов на этом коммите.

Протокол, файл [`dht-server/dht-server.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/dht-server/dht-server.cpp), строки [1076–1078](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/dht-server/dht-server.cpp#L1076-L1078):

```cpp
  keyring_ = ton::keyring::Keyring::create(db_root_ + "/keyring");
  // TODO wait for password
  started_keyring_ = true;
```

## ПАРОЛЬ, КОТОРЫЙ ОСТАЛСЯ В ЧЕРНОВИКЕ

На строке [1076](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/dht-server/dht-server.cpp#L1076) [dht-server/dht-server.cpp](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/dht-server/dht-server.cpp) создает keyring в каталоге `db_root_ + "/keyring"`. Сейф, казалось бы, найден. Но строка [1077](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/dht-server/dht-server.cpp#L1077) оставляет на двери записку `TODO wait for password`.

А строка [1078](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/dht-server/dht-server.cpp#L1078) уже сообщает миру: `started_keyring_ = true`. Не «пароль получен», не «хранилище разблокировано», не «оператор подтвердил доступ». Просто started. Газовый рожок кашлянул: в Лондоне это называлось бы «сейф открыт, но дворецкий еще помнит, что когда-нибудь нужна охрана».

Одна строка с TODO сама по себе не доказывает, что любой запуск немедленно раскрывает ключи. Но она честно показывает незавершенный замысел: защита паролем задумана, а в данном месте запуска ее ожидания нет. Для хранилища приватных ключей это не самая уютная надпись на входе.

## КЛЮЧ ПРИЕЗЖАЕТ С КОНТРОЛЬНОЙ ПОЧТОЙ

Сыщик перелистнул тот же файл [dht-server/dht-server.cpp](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/dht-server/dht-server.cpp). В методе импорта строка [851](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/dht-server/dht-server.cpp#L851) принимает `engine_validator_importPrivateKey`. На строках [853–860](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/dht-server/dht-server.cpp#L853-L860) есть проверка разрешения и готовности keyring. Это важная перегородка: перед нами не случайный HTTP-запрос без всякой авторизации.

Но дальше, на строке [862](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/dht-server/dht-server.cpp#L862), из данных запроса создается `ton::PrivateKey`. А строка [873](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/dht-server/dht-server.cpp#L873) передает его в `add_key` с аргументом `false` вместо признака временности.

Протокол, файл [dht-server/dht-server.cpp](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/dht-server/dht-server.cpp), строки [851–873](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/dht-server/dht-server.cpp#L851-L873):

```cpp
void DhtServer::run_control_query(ton::ton_api::engine_validator_importPrivateKey &query, td::BufferSlice data,
                                  ton::PublicKeyHash src, td::uint32 perm, td::Promise<td::BufferSlice> promise) {
  if (!(perm & DhtServerPermissions::vep_default)) {
    promise.set_value(create_control_query_error(td::Status::Error(ton::ErrorCode::error, "not authorized")));
    return;
  }
  if (keyring_.empty()) {
    promise.set_value(create_control_query_error(td::Status::Error(ton::ErrorCode::notready, "not started keyring")));
    return;
  }

  auto pk = ton::PrivateKey{query.key_};
  auto P = td::PromiseCreator::lambda(
      [promise = std::move(promise), hash = pk.compute_short_id()](td::Result<td::Unit> R) mutable {
        if (R.is_error()) {
          promise.set_value(create_control_query_error(R.move_as_error()));
        } else {
          promise.set_value(
              ton::serialize_tl_object(ton::create_tl_object<ton::ton_api::engine_validator_keyHash>(hash.tl()), true));
        }
      });

  td::actor::send_closure(keyring_, &ton::keyring::Keyring::add_key, std::move(pk), false, std::move(P));
}
```

Редакция не делает вид, будто `vep_default` равен прогулочному билету. Код требует разрешение. Но имя операции говорит само за себя: это штатный путь внесения приватного ключа в хранилище. После него ключ получает не только место в памяти, но и шанс поселиться на диске.

## ГДЕ ПРИВАТНЫЙ КЛЮЧ СНИМАЕТ ПАЛЬТО

За второй уликой сыщик отправился в файл [`keyring/keyring.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/keyring/keyring.cpp), тот же [коммит `3d478cb`](https://github.com/ton-blockchain/ton/commit/3d478cbde854be03a18ab2a59f8fc3c565cf7d14). На строке [89](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/keyring/keyring.cpp#L89) код проверяет, что ключ не временный и его можно экспортировать. Затем строка [90](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/keyring/keyring.cpp#L90) получает `key.export_as_slice()`.

Строка [91](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/keyring/keyring.cpp#L91) выбирает имя файла по хешу, а строка [93](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/keyring/keyring.cpp#L93) вызывает `td::write_file` и записывает срез на диск. Цитата из [keyring/keyring.cpp](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/keyring/keyring.cpp), строки [89–95](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/keyring/keyring.cpp#L89-L95):

```cpp
  if (!is_temp && key.exportable()) {
    auto S = key.export_as_slice();
    auto name = db_root_ + "/" + short_id.bits256_value().to_hex();

    td::write_file(td::CSlice(name), S.as_slice()).ensure();
  }
  promise.set_value(td::Unit());
```

Вот и весь парад. Приватный материал экспортируется в срез, получает имя из хеша и уходит в файл через обычную запись. В процитированных строках нет вызова ожидания пароля или явного шифрования перед `write_file`. Это не доказательство, что права каталога плохи или что файл доступен каждому прохожему: такие вещи зависят от окружения и реализации записи. Но как литературная улика сцена безупречна — пароль ждет за кулисами, а ключ уже получил билет на диск.

Есть и тонкая деталь. Импорт в [dht-server/dht-server.cpp](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/dht-server/dht-server.cpp) на строке [873](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/dht-server/dht-server.cpp#L873) передает `false`, то есть ключ не помечен временным. А условие на строке [89](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/keyring/keyring.cpp#L89) как раз пропускает такой ключ к сохранению, если он экспортируемый.

## ВЕРДИКТ СЫЩИКА

[ton-blockchain/ton](https://github.com/ton-blockchain/ton) — фундаментальная мастерская, а не подозрительный ломбард. Но коммит [`3d478cb`](https://github.com/ton-blockchain/ton/commit/3d478cbde854be03a18ab2a59f8fc3c565cf7d14) оставил выразительную связку улик: в [dht-server/dht-server.cpp](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/dht-server/dht-server.cpp) на строках [1076–1078](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/dht-server/dht-server.cpp#L1076-L1078) keyring запускается рядом с `TODO wait for password`; строки [862–873](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/dht-server/dht-server.cpp#L862-L873) принимают приватный ключ и отправляют его в keyring; а [keyring/keyring.cpp](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/keyring/keyring.cpp) на строках [89–93](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/keyring/keyring.cpp#L89-L93) сохраняет экспортируемый нетемповый ключ в файл.

Приговор редакции осторожен: сначала нужен пароль, потом работа с постоянным хранилищем. Если ключи должны переживать перезапуск, нужен ясный механизм защиты данных, а не записка TODO. Если ключ служит лишь для текущего процесса, пусть это будет отражено в флаге и жизненном цикле. Иначе сейф получает табличку «временно закрыт», пока отмычки уже лежат внутри.

*Сыщик закрыл папку, обвел [строку 1077](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/dht-server/dht-server.cpp#L1077) красным карандашом и погас газовую лампу. В тумане Лондона пароль может задержаться. Приватный ключ, записанный на голую полку, ждать не обязан.*

🐀
