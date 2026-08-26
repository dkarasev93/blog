+++
title = "№ 25 — Телеграфная контора: метрика на сто лет, изгнание на пять секунд и бюро повторных попыток, в котором повторов не положено ни одного"
date = 2026-08-26T11:10:00+03:00
description = "Двадцать пятый выпуск «Вечернего Валидатора»: сыщик поступает в телеграфную контору ядра ton — ADNL, оверлеи и DHT. Постоянным членам клуба выдают сертификат сроком на сто лет, а просроченного знакомца выметают через час; за фальшивую подпись положено изгнание на пять секунд, исполняемое уснувшим приставом; в бюро повторных попыток лимит попыток равен единице при счётчике, начинающемся с нуля, — повтор не наступает никогда; дворник адресного стола признаётся письменно, что пустые досье не выносит никогда, а вся такса конторы назначается броском двадцатигранной кости."
tags = ["ton"]
+++

# 📰 Вечерний Валидатор

**№ 25.** *Лондон. Туман нынче подан телеграфными проводами — говорят, он теперь не стелется, а передаётся от столба к столбу, со скидкой за соседство. Наш корреспондент, насмотревшись на городские канцелярии, где выдают справки (выпуск № 24) и отливают печати (выпуск № 23), отправился туда, куда все эти бумаги в конце концов уходят проводом: в телеграфную контору ядра [ton-blockchain/ton](https://github.com/ton-blockchain/ton) — в те самые этажи, где живут [`adnl/`](https://github.com/ton-blockchain/ton/tree/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/adnl), [`overlay/`](https://github.com/ton-blockchain/ton/tree/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/overlay) и [`dht/`](https://github.com/ton-blockchain/ton/tree/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/dht). Газета мельком осматривала сей телеграф в похоронном выпуске № 20 — тогда мы удивились депеше «рассылка не найдена», признанной самой странной из депеш. Нынче сыщик явился с ордером на полный обыск — и обыск, скажем прямо, удался на всю неделю.*

*Место происшествия: репозиторий [ton-blockchain/ton](https://github.com/ton-blockchain/ton), коммит [`3d478cb`](https://github.com/ton-blockchain/ton/commit/3d478cbde854be03a18ab2a59f8fc3c565cf7d14) (master от 17.08.2026). Файлы: [`overlay/overlay-peers.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/overlay/overlay-peers.cpp), [`overlay/overlay.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/overlay/overlay.cpp), [`overlay/overlay.hpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/overlay/overlay.hpp), [`dht/dht-query.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/dht/dht-query.cpp), [`dht/dht-query.hpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/dht/dht-query.hpp), [`adnl/adnl-peer-table.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/adnl/adnl-peer-table.cpp), [`adnl/adnl-peer-table.hpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/adnl/adnl-peer-table.hpp), [`adnl/adnl-local-id.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/adnl/adnl-local-id.cpp). Все цитаты дословные.*

---

## СЕНСАЦИЯ ПЕРВАЯ: МЕТРИКА НА СТО ЛЕТ

Телеграфная контора, как всякий порядочный клуб, делит знакомцев на две касты: членов постоянных — `is_persistent_node` — и публику приходящую. Для вступления в закрытые кружки — те самые оверлеи с `OverlayType::CertificatedMembers` — требуется сертификат членства, и контора сама определяет, до какого срока ваш сертификат годен. Вот канцелярия, выписывающая сроки, [`overlay/overlay-peers.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/overlay/overlay-peers.cpp), строки [898–911](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/overlay/overlay-peers.cpp#L898-L911):

```cpp
void OverlayImpl::update_member_certificate(OverlayMemberCertificate cert) {
  peer_list_.cert_ = std::move(cert);

  if (is_persistent_node(local_id_)) {
    peer_list_.local_cert_is_valid_until_ = td::Timestamp::in(86400.0 * 365 * 100); /* 100 years */
  } else {
    auto R = validate_peer_certificate(local_id_, &peer_list_.cert_);
    if (R.is_ok()) {
      peer_list_.local_cert_is_valid_until_ = td::Timestamp::at_unix(cert.expire_at());
    } else {
      peer_list_.local_cert_is_valid_until_ = td::Timestamp::never();
    }
  }
}
```

Читайте иерархию, читатель. Строка [901](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/overlay/overlay-peers.cpp#L901): если вы — постоянный член, вам даже не смотрят в сертификат. Строка [902](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/overlay/overlay-peers.cpp#L902): вам выписывают метрику `td::Timestamp::in(86400.0 * 365 * 100)` — сутки на дни года на сто — и снабжают скромной сноской `/* 100 years */`. Сто лет, читатель. Срок, при котором вопрос `has_valid_membership_certificate()` — «годен ли ваш сертификат», строки [913–922](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/overlay/overlay-peers.cpp#L913-L922), — для постоянного члена превращается в риторический: `.is_in_past()` на столетней метрике не случится при жизни ни читателя, ни писателя сих строк, ни, положим, самого города. Сыщик не нашёл здесь ни ключа, ни подписи, ни срока — только умножение трёх чисел и признание в скобках. Постоянство в этом клубе устанавливается не бумагой, а арифметикой.

Прочей же публике везёт меньше. Если сертификат не прошёл смотр — `td::Timestamp::never()`, строка [908](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/overlay/overlay-peers.cpp#L908): «никогда», срок годности, назначенный на дату, которая не наступит. Не «просрочен вчера», не «отозван» — никогда. А если прошёл — извольте жить по сроку из бумаги, строка [906](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/overlay/overlay-peers.cpp#L906), и не мешкая: в соседней комнате, строки [611–612](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/overlay/overlay-peers.cpp#L611-L612), уже стоит дворник с правилом `P->get_version() + 3600 < td::Clocks::system()` — приходящий знакомец, не подавший голоса час, выметается вон; повторно то же правило выписано на строке [655](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/overlay/overlay-peers.cpp#L655). Час молчания — и за дверь. Век членства — и ваши навсегда. Сыщик сверил единицы дважды: 3600 секунд для публики, 86400 · 365 · 100 секунд для своих. Контора меряет время двумя мерами, и обе выгравированы в одном файле, в трёхстах строках друг от друга.

## СЕНСАЦИЯ ВТОРАЯ: ИЗГНАНИЕ НА ПЯТЬ СЕКУНД

У конторы есть и полицейская функция: проверка подписей на депешах, циркулирующих в клубе. За фальшивую подпись положено изгнание. Вот камера приговоров, [`overlay/overlay.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/overlay/overlay.cpp), строки [1078–1093](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/overlay/overlay.cpp#L1078-L1093):

```cpp
td::Status OverlayImpl::check_signature_from_peer(PublicKey key, td::Slice message, td::Slice signature,
                                                  adnl::AdnlNodeIdShort message_from) {
  if (reject_signatures_from_.contains(message_from)) {
    return td::Status::Error("peer is temporary banned");
  }
  TRY_RESULT(enc, get_encryptor(std::move(key)));
  auto S = enc->check_signature(message, signature);
  if (S.is_error() && !message_from.is_zero()) {
    reject_signatures_from_.insert(message_from);
    LOG(WARNING) << "ban signatures from peer " << message_from << " for " << REJECT_SIGNATURES_DURATION << " s";
    auto task = [](OverlayImpl *self, adnl::AdnlNodeIdShort peer) -> td::actor::Task<> {
      co_await td::actor::coro_sleep(td::Timestamp::in(REJECT_SIGNATURES_DURATION));
      self->reject_signatures_from_.erase(peer);
      co_return {};
    };
    task(this, message_from).start().detach_silent();
  }
  return S;
}
```

Протокол наказания расписан с военной точностью. Уличённый в фальшивой подписи вносится в реестр изгнанных, строка [1086](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/overlay/overlay.cpp#L1086); в журнале появляется строгая запись «ban signatures from peer», строка [1087](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/overlay/overlay.cpp#L1087); и тут же нанимается пристав — корутина, которая ложится спать, строка [1089](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/overlay/overlay.cpp#L1089), просыпается, вычёркивает изгоя из реестра, строка [1090](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/overlay/overlay.cpp#L1090), — и растворяется, `detach_silent()`, строка [1092](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/overlay/overlay.cpp#L1092). Срок изгнания высечен в уставе, [`overlay/overlay.hpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/overlay/overlay.hpp), строка [606](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/overlay/overlay.hpp#L606): `static constexpr double REJECT_SIGNATURES_DURATION = 5.0;`.

Пять секунд, читатель. Подделка подписи — проступок, за который в этом городе в иных палатах рвут соединения, пишут протоколы и вспоминают о суде, — на телеграфе карается изгнанием длительностью в один глубокий вдох. Контора и сама сознаёт комизм: ответ психу с фальшивой печатью звучит извиняюще — `"peer is temporary banned"`, строка [1081](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/overlay/overlay.cpp#L1081), «пэр временно изгнан». Временно — это пять секунд; за них изгнанник успевает пропустить, пожалуй, полторы депеши. Сыщик уважает снисходительность, но занёс в досье: единственный пристав конторы нанимается по контракту на пять секунд и весь контракт спит.

## СЕНСАЦИЯ ТРЕТЬЯ: БЮРО ПОВТОРНЫХ ПОПЫТОК БЕЗ ПОВТОРОВ

Глубже, в справочном отделе DHT — там, где телеграф ищет телеграфистов, — сыщик открыл учреждение тонкое и трогательное: бюро повторных попыток. Устав его в двух местах. Сначала карточка посетителя, [`dht/dht-query.hpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/dht/dht-query.hpp), строки [85–88](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/dht/dht-query.hpp#L85-L88):

```cpp
  struct NodeInfo {
    DhtNode node;
    int failed_attempts = 0;
  };
```

И лимит, выбитый строкой [99](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/dht/dht-query.hpp#L99): `static const int MAX_ATTEMPTS = 1;`. Теперь сама процедура повтора, [`dht/dht-query.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/dht/dht-query.cpp), строки [92–97](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/dht/dht-query.cpp#L92-L97):

```cpp
    }
  } else {
    NodeInfo &info = nodes_[id_xor];
    if (++info.failed_attempts < MAX_ATTEMPTS) {
      pending_queries_.insert(id_xor);
    }
  }
```

Читатель, приложите пальцы к счёту вместе с сыщиком. Карточка рождается с нулём неудач — строка [87](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/dht/dht-query.hpp#L87). Первая же неудача прибавляет единицу — `++info.failed_attempts`, строка [95](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/dht/dht-query.cpp#L95), — и тут же спрашивает: меньше ли стало, чем `MAX_ATTEMPTS`, то есть меньше ли единицы единица? Не меньше. Вопрос не может получить ответа «да» ни при какой погоде: счётчик не бывает меньше нуля, а лимит не бывает больше единицы. Очередь повторных запросов — `pending_queries_`, честно пополняемая на входе, строка [80](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/dht/dht-query.cpp#L80), — после первого отказа не пополняется никогда. Бюро повторных попыток обставлено по всем правилам: есть счётчик, есть лимит, есть ветка кода, ведущая обратно в очередь. Не хватает одного — случая, когда вся эта мебель хоть раз пришла в движение. Чтобы повтор состоялся, лимиту следовало бы быть двойкой; чтобы двойка оправдывала название «максимум попыток», ей следовало бы стыдиться. Контора избрала средний путь: мебель оставить, повторы отменить.

## СЕНСАЦИЯ ЧЕТВЁРТАЯ: ДВОРНИК, ПРИЗНАЮЩИЙСЯ В ВЕЧНОМ МУСОРЕ

Адресный стол конторы — [`adnl/adnl-peer-table.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/adnl/adnl-peer-table.cpp) — ведёт досье на каждого корреспондента. Досье, по которым нет живых связей, положено выносить. Вот дворник, [`adnl/adnl-peer-table.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/adnl/adnl-peer-table.cpp), строки [652–663](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/adnl/adnl-peer-table.cpp#L652-L663):

```cpp
void AdnlPeerTableImpl::gc_peer_pairs(AdnlNodeIdShort local_id, LocalIdInfo &local_id_info) {
  while (local_id_info.peers_gc_order.size() > MAX_IDLE_PEER_PAIRS) {
    auto it = local_id_info.peers_gc_order.begin();
    AdnlNodeIdShort gc_peer_id = it->second;
    VLOG(adnl, INFO) << "Removing idle peer pair l_id=" << local_id << " p_id=" << gc_peer_id;
    peers_[gc_peer_id].peers.erase(local_id);
    if (peers_[gc_peer_id].peers.empty()) {
      // FIXME: if PeerInfo is empty from the start, it won't be erased ever
      peers_.erase(gc_peer_id);
    }
    local_id_info.peers_gc_order.erase(it);
  }
}
```

Обряд безупречен: лишние связи выносятся сверх нормы — норма выбита в шапке, [`adnl/adnl-peer-table.hpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/adnl/adnl-peer-table.hpp), строка [222](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/adnl/adnl-peer-table.hpp#L222), `MAX_IDLE_PEER_PAIRS = 2048`; пустые досье сжигаются, строка [660](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/adnl/adnl-peer-table.cpp#L660). Но над самой печкой, строка [659](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/adnl/adnl-peer-table.cpp#L659), висит признание: `// FIXME: if PeerInfo is empty from the start, it won't be erased ever` — «если досье пусто с самого начала, оно не будет стёрто никогда». Разберите механику пропажи, читатель: дворник сжигает только те пустые папки, которые *стали* пустыми при нём, пройдя через его очередь `peers_gc_order`. Папка, родившаяся пустой, в очередь не вставала, дворнику не видна — и остаётся в реестре навечно. Мусор, который никогда не был вещью, не подлежит выносу по инструкции. Сыщик не станет преувеличивать беду: пустое досье лёгкое. Но отметит канцелярскую красоту жанра: признание в вечном хранении подано формой FIXME — документом, который сам, по опыту нашей газеты, хранится вечно.

---

**От редакции.** Напоследок — такса. Вся депешная служба DHT назначает клиенту срок ожидания ответа, и вот тариф, одинаковый для всех шести окон — «найти узел», строка [116](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/dht/dht-query.cpp#L116), «найти ценность», строки [154](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/dht/dht-query.cpp#L154) и [171](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/dht/dht-query.cpp#L171), «сохранить ценность», строка [330](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/dht/dht-query.cpp#L330), «записать обратный провод», строка [413](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/dht/dht-query.cpp#L413), «попросить обратный звон», строка [453](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/dht/dht-query.cpp#L453): `td::Timestamp::in(2.0 + td::Random::fast(0, 20) * 0.1)`. Две секунды гарантированно, далее — бросок двадцатигранной кости, десятыми долями: от двух до четырёх секунд, как ляжет. Контора, где членство вычисляется на сто лет вперёд умножением, срок ожидания вычисляет игрой в кости. Адресная же книга — та самая, что публикуется в DHT, — живёт час: `ttl = td::Clocks::system() + 3600`, [`adnl/adnl-local-id.cpp`](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/adnl/adnl-local-id.cpp), строка [198](https://github.com/ton-blockchain/ton/blob/3d478cbde854be03a18ab2a59f8fc3c565cf7d14/adnl/adnl-local-id.cpp#L198). Сведения о вашем адресе устаревают за час, сертификат вашего членства — за сто лет, изгнание ваше — за пять секунд, а повтор попытки не наступит никогда. Сыщик погасил рожок и вышел в туман, размышляя, что время в телеграфной конторе измеряется не часами, а должностью: чем выше чин, тем длиннее секунда.

*Газета «Вечерний Валидатор» следит за конторами, где сроки назначаются умножением, а наказания — сном пристава. Следующий выпуск — когда бюро повторных попыток совершит первую. Мы подождём; у нас, в отличие от конторы, времени нет лимита.*
