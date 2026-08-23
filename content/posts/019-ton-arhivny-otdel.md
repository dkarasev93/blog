+++
title = "№ 19 — Архивный отдел: печать без проверки, сброс без воды и контора, в которую никто не приходит"
date = 2026-08-23T11:03:00+03:00
description = "Девятнадцатый выпуск «Вечернего Валидатора»: сыщик спускается в архивный отдел tddb ядра ton и обнаруживает журнальную контору, у которой за семь лет не было ни одного посетителя, кроме собственного сторожа-теста; контрольную сумму crc32, которую заготовили, но так и не проверяют; службу сброса, чей кран не подключён к воде; и закрытие архива, которое всегда завершается успехом — независимо от того, что произошло внутри."
tags = ["ton"]
+++

# 📰 Вечерний Валидатор

**№ 19.** *Лондон. Туман нынче лёг на город ровно в 512 байт — меньше, и его бы поленились нагонять. Наш корреспондент вновь в машинном городе [ton-blockchain/ton](https://github.com/ton-blockchain/ton): газета уже писала про «очень глупый вектор» (выпуск № 4), типографию вопросительных знаков (выпуск № 12) и кладовую ключей (выпуск № 11). Нынче сыщик свернул в переулок, куда не заглядывает даже туман: архивный отдел [`tddb/td/db/binlog/`](https://github.com/ton-blockchain/ton/tree/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tddb/td/db/binlog) — журнальная контора, где принимают, хранят и перечитывают записи. Контора полностью укомплектована: читальный зал, писчее бюро, асинхронное писчее бюро, служба сброса, испытательный сторож. Сыщик просидел в приёмной ночь и не встретил ни одного посетителя. Тогда он проверил журнал посещений. Журнал оказался пуст.*

*Место происшествия: репозиторий [ton-blockchain/ton](https://github.com/ton-blockchain/ton), коммит [`3d478cb`](https://github.com/ton-blockchain/ton/commit/3d478cbde854be03a18ab2a59f8fc3c565cf7d14) (master от 17.08.2026). Файлы: [`tddb/td/db/binlog/Binlog.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tddb/td/db/binlog/Binlog.cpp), [`tddb/td/db/binlog/Binlog.h`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tddb/td/db/binlog/Binlog.h), [`tddb/td/db/binlog/BinlogReaderInterface.h`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tddb/td/db/binlog/BinlogReaderInterface.h), [`tddb/td/db/binlog/BinlogReaderHelper.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tddb/td/db/binlog/BinlogReaderHelper.cpp), [`tddb/test/binlog.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tddb/test/binlog.cpp). Все цитаты дословные.*

---

## СЕНСАЦИЯ ПЕРВАЯ: КОНТОРА, В КОТОРУЮ НИКТО НЕ ПРИХОДИТ

Начнём с главной улики ночи. Архивный отдел — заведение солидное: класс `Binlog` с синхронным и асинхронным перечитыванием журнала, строки [35–40](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tddb/td/db/binlog/Binlog.h#L35-L40), класс `BinlogWriter`, строка [49](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tddb/td/db/binlog/Binlog.h#L49), класс `BinlogWriterAsync`, строка [71](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tddb/td/db/binlog/Binlog.h#L71), — штат, буферы, актёры, обещания. Сыщик прочесал весь город с фонарём: кто включил в свои файлы заголовок `td/db/binlog/Binlog.h`? Кто вызывает `replay_sync`, `BinlogWriter`, `BinlogWriterAsync`? Ответ из летописи включений: **никто**. Единственная дверь, из которой выходит след, ведёт в [`tddb/test/binlog.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tddb/test/binlog.cpp#L22-L23) — собственный сторож конторы, обходящий её по ночам. Валидатор не пользуется архивом. Телеграф tonlib не пользуется. Ключница не пользуется. Вся нижняя библиотека баз данных города содержит полностью работающий журнальный механизм, к которому не подведено ни одной линии.

Теперь о сроках, читатель, ибо в них вся соль. Летопись коммитов файла [`Binlog.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tddb/td/db/binlog/Binlog.cpp): заложен в «initial commit» [`c2da007f`](https://github.com/ton-blockchain/ton/commit/c2da007f4065e2520e0d948b146e0fb12fa75751) от 7 сентября 2019 года, подлатан в «bugfixes» [`be9c34c6`](https://github.com/ton-blockchain/ton/commit/be9c34c62dbd5d46089a255db8bf52c600b08aa5) от 10 апреля 2020-го, а дальше — шесть лет тишины, нарушенной один-единственный раз: в ноябре 2025-го по залу прошёл парикмахер [`f262baea`](https://github.com/ton-blockchain/ton/commit/f262baeaba892f17b8f612f97b919a2b25e83bf0), «Reformat entire codebase with clang-format 21», — причесал перья, переставил отступы и удалился. Шесть лет, читатель. Архив, в который никто не ходит, чинят ровно один раз в шесть лет — и то гребёнкой. Сыщик записал в блокнот: «учреждение содержится в идеальной чистоте, ибо не бывает посетителей; посетителей не бывает, ибо не к кому ходить; а не к кому ходить, ибо учреждение чистое».

## СЕНСАЦИЯ ВТОРАЯ: ПЕЧАТЬ CRC32, КОТОРУЮ ЗАГОТОВИЛИ, НО НЕ ПРОВЕРЯЮТ

Каждый порядочный архив скрепляет записи контрольной суммой: перечитал журнал — сверь печать, не подменил ли кто листы. Наш архив печать заготовил. И только. Читальный зал, процедура `replay_sync`, [`Binlog.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tddb/td/db/binlog/Binlog.cpp), строки [153–157](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tddb/td/db/binlog/Binlog.cpp#L153-L157):

```cpp
  if (helper.unparsed_size() != 0) {
    return Status::Error(PSLICE() << "Got " << helper.unparsed_size() << " unparsed bytes in binlog");
  }

  //TODO: check crc32
  //TODO: allow binlog truncate
  return Status::OK();
```

Прочитан весь журнал, хвосты сверены, непрочитанных байтов нет — осталось сверить печать. И на этом месте, в строках [155–156](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tddb/td/db/binlog/Binlog.cpp#L155-L156), стоят две депеши самому себе: «check crc32» и «allow binlog truncate» — и тут же, в строке [157](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tddb/td/db/binlog/Binlog.cpp#L157), выдаётся квитанция `Status::OK()`. Печать не проверена, усечение журнала не разрешено, приём окончен, все свободны. Сыщик переспросил у вахтёра, давно ли так: вахтёр молча указал на год 2019-й. Депеши эти висят на гвозде семь лет — это не заметки, это роспись стены.

Но изящнее всего то, что под печать строили подмостки. Устав читального зала, [`BinlogReaderInterface.h`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tddb/td/db/binlog/BinlogReaderInterface.h), строки [35–39](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tddb/td/db/binlog/BinlogReaderInterface.h#L35-L39):

```cpp
  // called when all passed slices are invalidated
  // Till it is called reader may resue all slices given to it.
  // It makes possible to calculate crc32c in larger chunks
  // TODO: maybe we should just process all data that we can at once
  virtual void flush() {
  }
```

Видите, читатель: метод `flush()` введён в устав **специально**, чтобы читатель мог считать crc32c крупными кусками — строка [37](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tddb/td/db/binlog/BinlogReaderInterface.h#L37). Под контрольную сумму спроектирован протокол, написан комментарий, заготовлен вызов. Сама контрольная сумма в архиве не встречается нигде, кроме этой приписки, — сыщик обыскал весь отдел: слово «crc» обнаружено лишь в уставе и в депеше «TODO». Инфраструктура для проверки печати построена, проверка не назначена. Мост через Темзу с полосой досмотра, за которой нет будки досмотра. И венчает зал комментарий в строке [38](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tddb/td/db/binlog/BinlogReaderInterface.h#L38): «maybe we should just process all data that we can at once» — «может, стоит просто обрабатывать всё, что можем, за один раз». Семь лет, читатель. Семь лет контора размышляет, не обработать ли ей всё за один раз, и за семь лет не обработала ни одного посетителя — за неимением посетителей.

## СЕНСАЦИЯ ТРЕТЬЯ: ЗАКРЫТИЕ ВСЕГДА УСПЕШНО, А СБРОС — ПРИЗРАК

Переходим к писчему бюро. Порядок сдачи смены, [`Binlog.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tddb/td/db/binlog/Binlog.cpp), строки [224–232](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tddb/td/db/binlog/Binlog.cpp#L224-L232):

```cpp
Status BinlogWriter::sync() {
  flush();
  return fd_.sync();
}

Status BinlogWriter::close() {
  sync();
  fd_.close();
  return Status::OK();
}
```

Сыщик перечитал эти девять строк трижды — и всякий раз находил их прекраснее. Смотрите на каскад пренебрежения. Строка [225](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tddb/td/db/binlog/Binlog.cpp#L225): `flush();` — результат выброшен: если буфер не удалось выплеснуть на диск, бюро об этом не узнает. Строка [230](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tddb/td/db/binlog/Binlog.cpp#L230): `sync();` — результат выброшен снова: если диск отказал, дверь всё равно запирают. И строка [232](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tddb/td/db/binlog/Binlog.cpp#L232) — венец: `return Status::OK();`. Закрытие архива **не может закончиться неудачей**. Записи потеряны? OK. Диск молчит? OK. Писец утонул в тумане вместе с буфером? OK. Сыщик знает конторы, где отчёты приукрашивают; но чтобы акт закрытия учреждения был юридически неспособен быть неуспешным — такого не видел даже он.

Асинхронное писчее бюро этажом выше доводит искусство до совершенства. Там стоит служка `FlushHelperActor`, и вот его главная обязанность, строки [241–242](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tddb/td/db/binlog/Binlog.cpp#L241-L242):

```cpp
  void flush() {
    //TODO;
  }
```

Всё. Тело службы сброса — это точка с запятой и депеша «TODO;», написанная через точку с запятой же. При этом бюро торжественно принимает прошения о сбросе: `BinlogWriterAsync::flush()`, строки [339–341](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tddb/td/db/binlog/Binlog.cpp#L339-L341), — посылает служке запечатанное предписание `send_closure`, служка просыпается, вскрывает конверт, исполняет точку с запятой и возвращается ко сну. Полный церемониал: актёр, сигнал, очередь, — и в конце пустая комната. Сыщик низко поклонился: туман над Темзой работает по той же схеме — его вызывают по всей форме, а приходит пустота.

Для полноты картины — тарифная сетка бюро, строки [210–214](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tddb/td/db/binlog/Binlog.cpp#L210-L214):

```cpp
Status BinlogWriter::lazy_flush() {
  if (buf_reader_.reader_size() < 512) {
    return Status::OK();
  }
  return flush();
}
```

«Ленивый сброс»: пока в буфере меньше 512 байт — сброс не положен, и это, заметьте, единственная процедура во всём архиве, которая честно называет себя ленивой. Остальные ленятся инкогнито.

---

## ХРОНИКА МЕЛКИХ ПРОИСШЕСТВИЙ

*Отдел орфографии.* Протокол читального зала, [`BinlogReaderHelper.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tddb/td/db/binlog/BinlogReaderHelper.cpp), строка [82](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tddb/td/db/binlog/BinlogReaderHelper.cpp#L82): `return td::Status::Error("BinlogReader parseed nothing and asked for nothing");` — «распарсено», читатель, через две «e». И чтобы читатель не принял это за случайность, строка [85](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tddb/td/db/binlog/BinlogReaderHelper.cpp#L85) повторяет подпись: `"BinlogReader parseed more than was given"`. Дважды «parseed» — это уже не опечатка, это фамильный почерк. Клерк, семь лет не принимавший посетителей, имеет право на вывих глагола.

*Отдел нумерации.* Там же, строка [34](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tddb/td/db/binlog/BinlogReaderHelper.cpp#L34): `return td::Status::Error("BinlogReader decreased logevent size estimation (1)");`. Обратите внимание на «(1)». Сыщик обыскал весь отдел: случая «(2)» не существует. Нумерация открыта, серия не продолжена. Журнальная контора, в которой заготовили вторую страницу реестра — и не пришёл никто даже на первую.

*Отдел рукописей.* Единственный посетитель архива — сторож-тест [`tddb/test/binlog.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tddb/test/binlog.cpp), и у него есть реликвия, строка [633](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tddb/test/binlog.cpp#L633): `TEST(Binlog, Hands)` — «Руки». Испытание подаёт архиву зашифрованную рукопись, строки [634–638](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tddb/test/binlog.cpp#L634-L638): четыре строки base64, начинающиеся с `a0RCBAABKQCRMn1c2DaJhwrptxburpRt...`, — чей это журнал, откуда взят, что в нём записано, летопись не сообщает. Рукопись просто лежит в протоколе, как отпечаток ладони на стекле пустого зала. Руки есть. Читателя нет.

*Отдел уничтожения.* Процедура ликвидации архива, [`Binlog.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tddb/td/db/binlog/Binlog.cpp), строки [186–188](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/tddb/td/db/binlog/Binlog.cpp#L186-L188): `td::unlink(path).ignore();` — удалить файл и проигнорировать, удалился ли он. Контора последовательна до конца: создаётся без посетителей, живёт без проверки печати, закрывается всегда успешно, а уничтожается — не узнав, уничтожена ли.

*Соседний флигель: палата смарт-контрактов.* Пока сыщик был в архивном квартале, он заглянул в соседний флигель — [`crypto/vm/db/TonDb.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/vm/db/TonDb.cpp), — и обнаружил, что почерк заразителен. Открытие картотеки, строки [106–107](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/vm/db/TonDb.cpp#L106-L107): `// TODO: proper serialization` и тут же `td::unserialize(meta_, meta_serialized).ignore();` — «правильную сериализацию» назначат потом, а пока ошибку разбора метаданных проглатывают молча. Ниже, строки [115–116](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/vm/db/TonDb.cpp#L115-L116): `//FIXME: error handling` и `db_root_ = cell_db_->load_cell(root_hash).move_as_ok();` — корень базы изымается из хранилища с сопроводительной запиской «исправить: обработку ошибок» и немедленным признанием результата годным. А в строке [149](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/vm/db/TonDb.cpp#L149) при сдаче мешка ячеек стоит `boc.import_cells().ensure();  // FIXME` — третья записка о том же. Три депеши «потом» в одном флигеле.

*Отдел ошибок-депеш.* И финальный штрих ночи — из бюро мешков ячеек, [`crypto/vm/boc.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/vm/boc.cpp), строки [60–62](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/crypto/vm/boc.cpp#L60-L62): встретив в сериализации отсутствующие ячейки, бюро отвечает посетителю `return td::Status::Error("TODO: absent cells");`. Осмыслите, читатель: текст ошибки, который получит живой человек, — это депеша программиста самому себе. Не «отсутствуют ячейки», а «сделать: отсутствующие ячейки». Клиент стучится в окошко, а в ответ из окошка высовывается список дел конторы. Сыщик получал в этом городе отказы всякие — но чтобы отказ был оформлен как напоминалка на холодильнике — впервые.

---

## ВЕРДИКТ РЕДАКЦИИ

Скажем по справедливости: от пустого архива нет и пустого вреда. Журнальная контора никому не служит, а значит, некому пострадать от её несверенной печати; закрытие, которое всегда успешно, никого не обнадёжит, ибо некого обнадёживать; сброс-призрак ничьих данных не утопит, ибо данных нет. Это не катастрофа — это памятник: целый отдел, спроектированный на совесть в 2019 году — с протоколом под crc32c, с асинхронным писцом, со службой сброса, — и так и не заселённый. Газета ведёт хронику почерка, и почерк сей ночи таков: город умеет строить учреждения раньше, чем узнает, зачем они. Печать заготовлена, проверка не назначена. Кран сброса установлен, вода не подведена. Закрытие успешно по определению, удаление — по надежде. А в соседнем флигеле, уже с посетителями, ошибки разбора глотают молча, а в ответ на стук выдают список собственных дел. Туман над Темзой всё принял — он единственный, кто регулярно заполняет этот архив, и, надо отдать ему должное, никогда не требует сверки контрольной суммы.
