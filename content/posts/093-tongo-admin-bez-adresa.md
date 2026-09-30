+++
title = "№ 93 — Админ джеттона вышел без адреса"
date = 2026-09-30T17:00:00+03:00
description = "Девяносто третий выпуск «Вечернего Валидатора»: tongo разрешил get_jetton_data честно признать, что у джеттона нет администратора."
tags = ["tongo"]
+++

# ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

**№ 93.** *Лондон. Туман полз по мостовой, газовый рожок сипел в переулке, а сыщик получил депешу из [tonkeeper/tongo](https://github.com/tonkeeper/tongo). В ней джеттон явился на допрос без хозяина. Не потерял адрес, не забыл его в карете — просто признал, что адрес может быть пустым.*

## МЕСТО ПРОИСШЕСТВИЯ

Улика найдена в репозитории [tonkeeper/tongo](https://github.com/tonkeeper/tongo), в коммите [`1bbbb32`](https://github.com/tonkeeper/tongo/commit/1bbbb32609ce87319e3abd9317eb872c2da492b6), озаглавленном `allow admin in get_jetton_data to be null`. Главные комнаты дела — файл [`abi/get_methods.go`](https://github.com/tonkeeper/tongo/blob/1bbbb32609ce87319e3abd9317eb872c2da492b6/abi/get_methods.go) и схема [`abi/schemas/jettons.xml`](https://github.com/tonkeeper/tongo/blob/1bbbb32609ce87319e3abd9317eb872c2da492b6/abi/schemas/jettons.xml). Цитаты сверены с этим коммитом.

## ПЕРВЫЙ СЛЕД: ВЛАДЕЛЕЦ, КОТОРЫЙ МОЖЕТ БЫТЬ НИКЕМ

Дело касается метода `get_jetton_data`. Он возвращает пять сведений: запас, признак выпуска, адрес администратора, содержимое и код кошелька. Схема теперь без дипломатии сообщает: третья графа допускает пустоту. Дословная запись стоит в файле [`abi/schemas/jettons.xml`](https://github.com/tonkeeper/tongo/blob/1bbbb32609ce87319e3abd9317eb872c2da492b6/abi/schemas/jettons.xml), на строках [55–61](https://github.com/tonkeeper/tongo/blob/1bbbb32609ce87319e3abd9317eb872c2da492b6/abi/schemas/jettons.xml#L55-L61):

```xml
    <get_method name="get_jetton_data">
        <output>
            <int name="total_supply">int257</int>
            <int name="mintable">bool</int>
            <slice name="admin_address" nullable="true">msgaddress</slice>
            <cell name="jetton_content">any</cell>
            <cell name="jetton_wallet_code">any</cell>
        </output>
```

Сыщик задержал дыхание на слове `nullable`. Администратор здесь не обязательно скрывается за адресом. Иногда у джеттона нет управляющего лица, и схема наконец перестала делать вид, будто каждый сейф обязан иметь владельца.

## ВТОРОЙ СЛЕД: УКАЗАТЕЛЬ ВМЕСТО КАМЕННОГО ЛИЦА

Одной таблички в XML было бы мало. Генератор ответов тоже должен принять пустой результат, иначе схема обещает свободу, а приемная выдает отказ. В [`abi/get_methods.go`](https://github.com/tonkeeper/tongo/blob/1bbbb32609ce87319e3abd9317eb872c2da492b6/abi/get_methods.go) структура результата на строках [3480–3486](https://github.com/tonkeeper/tongo/blob/1bbbb32609ce87319e3abd9317eb872c2da492b6/abi/get_methods.go#L3480-L3486) теперь выглядит так:

```go
type GetJettonDataResult struct {
    TotalSupply      tlb.Int257
    Mintable         bool
    AdminAddress     *tlb.MsgAddress
    JettonContent    tlb.Any
    JettonWalletCode tlb.Any
}
```

Звездочка перед `tlb.MsgAddress` — маленький черный зонтик над всей сценой. Там может быть настоящий адрес, а может быть `nil`. Раньше тип требовал от каждого джеттона показать администратора, даже если контракт по замыслу никому не подчиняется.

## ТРЕТИЙ СЛЕД: СТРАЖА У СТЕКА НАУЧИЛАСЬ ГОВОРИТЬ «НЕТ»

Самая сочная улика сидит в проверке формы стека. Декодер не просто хранит указатель: он разрешает третьему значению быть либо срезом, либо пустым значением. Дословный фрагмент из [`abi/get_methods.go`](https://github.com/tonkeeper/tongo/blob/1bbbb32609ce87319e3abd9317eb872c2da492b6/abi/get_methods.go), строки [3508–3514](https://github.com/tonkeeper/tongo/blob/1bbbb32609ce87319e3abd9317eb872c2da492b6/abi/get_methods.go#L3508-L3514):

```go
func DecodeGetJettonDataResult(stack tlb.VmStack) (resultType string, resultAny any, err error) {
    if stack.Len() < 5 || (stack.Peek(stack.Len()-0-1).SumType != "VmStkTinyInt" && stack.Peek(stack.Len()-0-1).SumType != "VmStkInt") || (stack.Peek(stack.Len()-1-1).SumType != "VmStkTinyInt" && stack.Peek(stack.Len()-1-1).SumType != "VmStkInt") || (stack.Peek(stack.Len()-2-1).SumType != "VmStkSlice" && stack.Peek(stack.Len()-2-1).SumType != "VmStkNull") || (stack.Peek(stack.Len()-3-1).SumType != "VmStkCell") || (stack.Peek(stack.Len()-4-1).SumType != "VmStkCell") {
        return "", nil, fmt.Errorf("invalid stack format")
    }
    var result GetJettonDataResult
    err = stack.Unmarshal(&result)
    return "GetJettonDataResult", result, err
}
```

В середине этой длинной судебной цепочки стоит важная связка: `VmStkSlice` **или** `VmStkNull`. Пустой администратор перестал выглядеть как поврежденный документ. Стража у стека теперь различает два законных паспорта: адресную бумагу и отсутствие адресной бумаги.

## ЧЕТВЕРТЫЙ СЛЕД: КОМИТТ ПРИЗНАЛСЯ НА ДВЕРИ

Название коммита [`1bbbb32`](https://github.com/tonkeeper/tongo/commit/1bbbb32609ce87319e3abd9317eb872c2da492b6) — `allow admin in get_jetton_data to be null` — звучит как показание без адвоката. Не «улучшить совместимость», не «обновить модель», а разрешить администратору быть `null`.

Коммит [`1bbbb32`](https://github.com/tonkeeper/tongo/commit/1bbbb32609ce87319e3abd9317eb872c2da492b6) поправил сразу три слоя одного дела: схему в [`abi/schemas/jettons.xml`](https://github.com/tonkeeper/tongo/blob/1bbbb32609ce87319e3abd9317eb872c2da492b6/abi/schemas/jettons.xml) на строке [59](https://github.com/tonkeeper/tongo/blob/1bbbb32609ce87319e3abd9317eb872c2da492b6/abi/schemas/jettons.xml#L59), тип результата в [`abi/get_methods.go`](https://github.com/tonkeeper/tongo/blob/1bbbb32609ce87319e3abd9317eb872c2da492b6/abi/get_methods.go) на строке [3483](https://github.com/tonkeeper/tongo/blob/1bbbb32609ce87319e3abd9317eb872c2da492b6/abi/get_methods.go#L3483) и караул проверки на строке [3509](https://github.com/tonkeeper/tongo/blob/1bbbb32609ce87319e3abd9317eb872c2da492b6/abi/get_methods.go#L3509).

Это не доказательство кражи и не обвинение конкретного джеттона. Факт скромнее: прежняя модель не принимала пустого администратора, а новый коммит синхронизировал описание, Go-тип и разбор ответа. Но зрелище остается отменным: чтобы сказать «владельца нет», пришлось провести `null` через три ведомства.

## ХРОНИКА МЕЛКОЙ КАНЦЕЛЯРИИ

*Отдел запасов.* В структуре [`GetJettonDataResult`](https://github.com/tonkeeper/tongo/blob/1bbbb32609ce87319e3abd9317eb872c2da492b6/abi/get_methods.go#L3480-L3486) первые две графы остаются строгими: `TotalSupply` и `Mintable` не превращаются в туман. Пустым разрешен именно адрес администратора.

*Отдел стеков.* На строке [3509](https://github.com/tonkeeper/tongo/blob/1bbbb32609ce87319e3abd9317eb872c2da492b6/abi/get_methods.go#L3509) `VmStkNull` стоит рядом с `VmStkSlice`, а не заменяет все проверки подряд. Пустая графа не дает остальным четырем значениям выйти из строя.

*Отдел вывесок.* На строке [55](https://github.com/tonkeeper/tongo/blob/1bbbb32609ce87319e3abd9317eb872c2da492b6/abi/schemas/jettons.xml#L55) метод по-прежнему называется `get_jetton_data`. Сыщик проверил: пропал только обязательный хозяин, сам джеттон никуда не делся.

## ВЕРДИКТ СЫЩИКА

Репозиторий [tonkeeper/tongo](https://github.com/tonkeeper/tongo) пойман в коммите [`1bbbb32`](https://github.com/tonkeeper/tongo/commit/1bbbb32609ce87319e3abd9317eb872c2da492b6) на деле о необязательном хозяине. В схеме [`abi/schemas/jettons.xml`](https://github.com/tonkeeper/tongo/blob/1bbbb32609ce87319e3abd9317eb872c2da492b6/abi/schemas/jettons.xml) строка [59](https://github.com/tonkeeper/tongo/blob/1bbbb32609ce87319e3abd9317eb872c2da492b6/abi/schemas/jettons.xml#L59) ставит `nullable="true"`, структура [`GetJettonDataResult`](https://github.com/tonkeeper/tongo/blob/1bbbb32609ce87319e3abd9317eb872c2da492b6/abi/get_methods.go) хранит `*tlb.MsgAddress` на строке [3483](https://github.com/tonkeeper/tongo/blob/1bbbb32609ce87319e3abd9317eb872c2da492b6/abi/get_methods.go#L3483), а декодер принимает `VmStkNull` на строке [3509](https://github.com/tonkeeper/tongo/blob/1bbbb32609ce87319e3abd9317eb872c2da492b6/abi/get_methods.go#L3509).

Приговор мягкий, но выразительный: если контракт может жить без администратора, библиотека обязана уметь это произнести вслух. Иначе `null` стучит в стек, а ему отвечают: «такого гражданина нет в списках». Газовый рожок кашлянул, сыщик закрыл папку, и безадресный хозяин растворился в тумане.

🐀
