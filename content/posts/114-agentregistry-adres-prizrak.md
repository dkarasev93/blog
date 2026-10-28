---
title: "№ 114 — Реестр, который посылал репутацию в чужой адрес"
date: 2026-10-28T10:59:00+03:00
tags: [contracts]
---

# 📰 ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

## ДЕЛО О НОДЕ, КОТОРОЙ ПИСАЛИ, НО НЕ ТУДА

Лондон тонул в тумане, когда сыщик «Вечернего Валидатора» получил папку из [gstdcoin/contracts](https://github.com/gstdcoin/contracts). На обложке стояло честное предупреждение: контракты не проходили независимый аудит. Но внутри лежала улика сочнее вывески: реестр узлов умел записывать адрес личности, а потом искал ее по другому адресу.

Это не тонкая теория. Репозиторий сам позднее описал результат: репутация и приостановка узлов фактически превратились в сетевой холостой выстрел. Газовый рожок кашлянул, сыщик достал лупу.

## МЕСТО ПРОИСШЕСТВИЯ

След найден в [коммите ccc73fc](https://github.com/gstdcoin/contracts/commit/ccc73fc398e4ea421643da2453afd3b6e2154a73), файле [AgentRegistry.tact](https://github.com/gstdcoin/contracts/blob/ccc73fc398e4ea421643da2453afd3b6e2154a73/AgentRegistry.tact), строки [205–220](https://github.com/gstdcoin/contracts/blob/ccc73fc398e4ea421643da2453afd3b6e2154a73/AgentRegistry.tact#L205-L220). Цитата дословная:

```tact
    get fun get_node_address(nodeId: Int): Address {
        return self.getNodeAddress(nodeId);
    }

    // ── Internal ────────────────────────────────────────────────────────
    fun getNodeAddress(nodeId: Int): Address {
        let nodeInit: StateInit = initOf NodeIdentity(
            nodeId,
            myAddress(),
            newAddress(0, 0),
            0,
            0,
            0,
        );
        return contractAddress(nodeInit);
    }
```

В переводе с канцелярского: для поиска уже созданной личности реестр подставляет нулевого владельца, нулевой тип, нулевые возможности и нулевой регион. Настоящая личность при регистрации получает совсем другие данные — адрес отправителя, тип узла, его способности и край света.

## ПЕРВЫЙ СЛЕД: АДРЕС РОДИЛСЯ ДВАЖДЫ

В [AgentRegistry.tact](https://github.com/gstdcoin/contracts/blob/ccc73fc398e4ea421643da2453afd3b6e2154a73/AgentRegistry.tact) регистрация строит `NodeIdentity` с реальными параметрами. Но функция [getNodeAddress](https://github.com/gstdcoin/contracts/blob/ccc73fc398e4ea421643da2453afd3b6e2154a73/AgentRegistry.tact#L210-L220) строит его заново с чужим паспортом. В TON адрес контракта зависит от кода и начальных данных. Один байт в конструкторе — и это уже другая дверь.

Следовательно, реестр отправляет репутацию не той личности. Исключение возможно лишь для почти невозможной комбинации: владелец — нулевой адрес, тип — Edge, возможности — ноль, регион — EU.

## ВТОРОЙ СЛЕД: ОШИБКА, КОТОРАЯ ВЕЖЛИВО ПРОПАЛА

В том же [AgentRegistry.tact](https://github.com/gstdcoin/contracts/blob/ccc73fc398e4ea421643da2453afd3b6e2154a73/AgentRegistry.tact) обработчик репутации берет рассчитанный адрес в строке [149](https://github.com/gstdcoin/contracts/blob/ccc73fc398e4ea421643da2453afd3b6e2154a73/AgentRegistry.tact#L149), а затем отправляет сообщение с `SendIgnoreErrors` в строках [150–156](https://github.com/gstdcoin/contracts/blob/ccc73fc398e4ea421643da2453afd3b6e2154a73/AgentRegistry.tact#L150-L156). Для жалобы на нарушение повторяется тот же фокус: адрес в строке [164](https://github.com/gstdcoin/contracts/blob/ccc73fc398e4ea421643da2453afd3b6e2154a73/AgentRegistry.tact#L164), безмолвная отправка в строках [165–175](https://github.com/gstdcoin/contracts/blob/ccc73fc398e4ea421643da2453afd3b6e2154a73/AgentRegistry.tact#L165-L175).

Получается прекрасная отчетность без результата: счетчик задач в реестре растет, активные узлы уменьшаются, а настоящая NodeIdentity не получает ни балла репутации, ни статуса `Suspended`. Конверт ушел в дом, которого нет, а почтамт поставил на нем штамп «ошибка не важна».

## ТРЕТИЙ СЛЕД: ПОЗДНИЙ ПРОТОКОЛ

Позднее, в [коммите 75bb6b8](https://github.com/gstdcoin/contracts/commit/75bb6b89b64674c7316c22a2f3e9254885acc5f1), файл [AgentRegistryV2.tact](https://github.com/gstdcoin/contracts/blob/75bb6b89b64674c7316c22a2f3e9254885acc5f1/AgentRegistryV2.tact) уже описывает улику без дипломатии. В строках [8–25](https://github.com/gstdcoin/contracts/blob/75bb6b89b64674c7316c22a2f3e9254885acc5f1/AgentRegistryV2.tact#L8-L25) сказано дословно:

```tact
// Fix vs. v1 (EQDtWcGCQXLFdh7TmkL5QFbFNYXxL9mjOk4ehmsNFwCtsDoT, still live,
// left untouched by this file): getNodeAddress() recomputed a NodeIdentity's
// address via initOf with placeholder args (owner=zero address, nodeType=0,
// capabilities=0, region=0) instead of the real ones used at RegisterNode
// time. Since a contract's address is a hash of its code+initial-data, and
// the initial data is derived from the real constructor args, the
// placeholder address only matches the real deployed NodeIdentity for a
// node that happens to be type 0 (Edge), capabilities 0, region 0 (EU),
// registered by the zero address -- i.e. essentially never in practice.
// UpdateReputation and ReportGenesisViolation both call this same function
// to find where to forward reputation credits/penalties and suspension
// status, so for virtually every real node those messages silently went to
// a non-existent address (SendIgnoreErrors swallows the failure) -- all
// reputation tracking and Genesis-violation suspension has been a no-op
// network-wide. Same bug class as the DAOVoting get_proposal_address fix
// earlier this session. Fixed the same way: record each NodeIdentity's real
// deployed address in a map at RegisterNode time, when the real args are
// still known.
```

Редакция любит, когда подозреваемый сам пишет протокол. Здесь он даже уточняет: исправление подготовлено к проверке, но еще не развернуто. Старый адрес-призрак остается жить, а новая карта пока лежит в ящике.

## ПРИГОВОР СЫЩИКА

[gstdcoin/contracts](https://github.com/gstdcoin/contracts) пойман на классическом лондонском трюке: создать дом с настоящим адресом, а письма искать по адресу, собранному из нулей. [AgentRegistry.tact](https://github.com/gstdcoin/contracts/blob/ccc73fc398e4ea421643da2453afd3b6e2154a73/AgentRegistry.tact), [коммит ccc73fc](https://github.com/gstdcoin/contracts/commit/ccc73fc398e4ea421643da2453afd3b6e2154a73), строки [149–175](https://github.com/gstdcoin/contracts/blob/ccc73fc398e4ea421643da2453afd3b6e2154a73/AgentRegistry.tact#L149-L175): реестр вел учет, отправлял сообщения и не замечал, что адресат растворился в тумане.

Приговор мягкий, но с пометкой красными чернилами: адрес дочернего контракта надо сохранить в момент создания или вычислять с теми же параметрами, а `SendIgnoreErrors` не должен служить занавесом для системной ошибки. Газовый рожок пробил сто четырнадцатый удар. Сыщик закрыл папку и ушел туда, где нулевые адреса не принимают почту.

🐀
