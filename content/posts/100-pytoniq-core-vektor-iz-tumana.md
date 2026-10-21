+++
title = "№ 100 — Вектор, который собирался поглотить память"
date = 2026-10-21T10:58:00+03:00
description = "Сотый выпуск «Вечернего Валидатора»: десериализатор pytoniq-core принимал длину вектора за приказ повторить цикл до исчерпания памяти."
tags = ["pytoniq-core"]
+++

# ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

**№ 100.** *Лондон. Туман забил переулки, когда сыщик получил из [yungwine/pytoniq-core](https://github.com/yungwine/pytoniq-core) тонкую телеграмму. В ней значилось: «вектор длиннее данных». Формулировка казалась скучной, пока газовый рожок не осветил число `0x7fffffff`. Тогда стало ясно: маленький пакет мог явиться в контору с приказом повторить одну операцию два миллиарда сто сорок семь миллионов четыреста восемьдесят три тысячи шестьсот сорок семь раз.*

## МЕСТО ПРОИСШЕСТВИЯ

Улика найдена в коммите [`094d343`](https://github.com/yungwine/pytoniq-core/commit/094d34393221b298a1798646ee9d7d186e128924) репозитория [yungwine/pytoniq-core](https://github.com/yungwine/pytoniq-core), озаглавленном `reject tl vectors longer than the data`. Главный кабинет — файл [`pytoniq_core/tl/generator.py`](https://github.com/yungwine/pytoniq-core/blob/094d34393221b298a1798646ee9d7d186e128924/pytoniq_core/tl/generator.py), строки [305–310](https://github.com/yungwine/pytoniq-core/blob/094d34393221b298a1798646ee9d7d186e128924/pytoniq_core/tl/generator.py#L305-L310). Цитата снята дословно с этого коммита:

```python
                    if 'vector' in type_:
                        length = int.from_bytes(data[i:i + 4], 'little', signed=False)
                        i += 4
                        # every element takes at least a byte; it can already be past the end
                        if length > max(len(data) - i, 0):
                            raise TlError(f"wrong vector length {length}: {len(data) - i} bytes left")
```

## ПЕРВЫЙ СЛЕД: ЧЕТЫРЕ БАЙТА СТАЛИ ПРИГОВОРОМ

Схема TL сообщает длину вектора четырьмя байтами. Старый дежурный в [pytoniq_core/tl/generator.py](https://github.com/yungwine/pytoniq-core/blob/094d34393221b298a1798646ee9d7d186e128924/pytoniq_core/tl/generator.py) на строке [306](https://github.com/yungwine/pytoniq-core/blob/094d34393221b298a1798646ee9d7d186e128924/pytoniq_core/tl/generator.py#L306) честно читал это число, а на строке [312](https://github.com/yungwine/pytoniq-core/blob/094d34393221b298a1798646ee9d7d186e128924/pytoniq_core/tl/generator.py#L312) без предварительного допроса входил в `for _ in range(length)`.

Если в коротком сообщении лежал вектор с длиной `0x7fffffff`, цикл получал приказ явиться на два миллиарда с лишним заседаний. Каждый проход пытался разобрать данные, которых уже не было, а затем складывал результат в список. Ошибка формата превращалась в марш-бросок к исчерпанию памяти. Не взлом, не тайная лазейка, а четыре байта, которым канцелярия позволила назначить себе бесконечно длинную смену.

Сыщик отдельно отмечает границу доказанного. Коммит не утверждает, что любой удаленный пакет гарантированно уронит приложение: последствия зависят от того, кто подает данные и как библиотека используется. Но тестовый комментарий прямо описывает старое поведение, а исправление ставит перед циклом проверку вместимости. Этого достаточно для обвинения в опасном недосмотре, но не для театрального оглашения конца Лондона.

## ВТОРОЙ СЛЕД: ТЕСТ С ЧИСЛОМ ИЗ КАТАСТРОФЫ

Свидетель сидит в файле [`tests/test_tl.py`](https://github.com/yungwine/pytoniq-core/blob/094d34393221b298a1798646ee9d7d186e128924/tests/test_tl.py), строки [82–95](https://github.com/yungwine/pytoniq-core/blob/094d34393221b298a1798646ee9d7d186e128924/tests/test_tl.py#L82-L95). Сначала функция `overlay_nodes_with_length` берет исправный пустой вектор, а затем подменяет первые четыре байта длины:

```python
def overlay_nodes_with_length(schemas, length: int) -> bytes:
    raw = bytearray(schemas.serialize('overlay.nodes', {'nodes': []}))
    raw[4:8] = length.to_bytes(4, 'little')
    return bytes(raw)
```

После этого тест вызывает подозреваемого дважды. Первый раз ему подсовывают `0x7fffffff`, второй — скромную тройку. Обе длины больше оставшихся данных и обе обязаны закончиться `TlError`. В [строках 88–95](https://github.com/yungwine/pytoniq-core/blob/094d34393221b298a1798646ee9d7d186e128924/tests/test_tl.py#L88-L95) это сказано без тумана:

```python
def test_vector_longer_than_data_is_rejected():
    schemas = get_schemas()

    # used to loop 2**31 times appending empty elements until the process ran out of memory
    with pytest.raises(TlError):
        schemas.deserialize(overlay_nodes_with_length(schemas, 0x7fffffff))
    with pytest.raises(TlError):
        schemas.deserialize(overlay_nodes_with_length(schemas, 3))
```

Фраза `used to loop 2**31 times appending empty elements until the process ran out of memory` выглядит не как гипотеза, а как запись с места происшествия. Писарь сам оставил на полях признание: раньше контора крутила цикл два в степени тридцать один раз и добавляла пустые элементы, пока процесс не лишался памяти. Редакция не станет улучшать показания свидетеля — цитата говорит сама за себя.

## ТРЕТИЙ СЛЕД: СТРАЖА НЕ ПУТАЕТ ПУСТОЙ ПАКЕТ С БЕЗДОНОЙ

Новая проверка в [строках 308–310](https://github.com/yungwine/pytoniq-core/blob/094d34393221b298a1798646ee9d7d186e128924/pytoniq_core/tl/generator.py#L308-L310) сравнивает длину вектора с числом оставшихся байтов. Для пустого хвоста применяется `max(len(data) - i, 0)`, так что отрицательное количество оставшихся байтов не маскируется под большое число.

Приговор оформляется как `TlError`, а не как аварийный марш списка. Сообщение также оставляет след: `wrong vector length {length}: {len(data) - i} bytes left`. У читателя появляется хотя бы имя проступка и количество оставшихся байтов, вместо загадочного исчезновения процесса в лондонском тумане.

Но сыщик не закрывает дело раньше времени. Проверка исходит из простого допущения: каждый элемент занимает хотя бы один байт. Для TL-вектора это полезный быстрый заслон от невозможной длины, а не полная проверка корректности каждого элемента. Сам разбор элементов по-прежнему проходит в [строках 312–319](https://github.com/yungwine/pytoniq-core/blob/094d34393221b298a1798646ee9d7d186e128924/pytoniq_core/tl/generator.py#L312-L319). Газовый рожок уважает границы полномочий.

## ЧЕТВЕРТЫЙ СЛЕД: СЛУЧАЙНЫЕ БАЙТЫ ПРИКИНУЛИСЬ TL

Коммит не ограничился векторной дверью. В [файле `pytoniq_core/tl/generator.py`](https://github.com/yungwine/pytoniq-core/blob/094d34393221b298a1798646ee9d7d186e128924/pytoniq_core/tl/generator.py) строки [279–294](https://github.com/yungwine/pytoniq-core/blob/094d34393221b298a1798646ee9d7d186e128924/pytoniq_core/tl/generator.py#L279-L294) теперь ловят `TlError`, если поле с байтами случайно начинается с известного идентификатора конструктора, но целиком TL-сообщением не является. Тогда данные возвращаются как сырые байты.

Тест на [строках 112–118](https://github.com/yungwine/pytoniq-core/blob/094d34393221b298a1798646ee9d7d186e128924/tests/test_tl.py#L112-L118) проверяет именно этот маскарад. В него кладут тот же подозрительный вектор длиной `0x7fffffff` как содержимое `adnl.message.custom`, а затем требуют вернуть исходные байты без попытки разыграть их как TL. Вторая дверь закрыта: случайный герб на конверте больше не заставляет переписчика считать конверт целым романом.

## ПРИГОВОР СЫЩИКА

Репозиторий [yungwine/pytoniq-core](https://github.com/yungwine/pytoniq-core) пойман в коммите [`094d343`](https://github.com/yungwine/pytoniq-core/commit/094d34393221b298a1798646ee9d7d186e128924) на деле о длине, которая могла отправить процесс в двухмиллиардный цикл. В файле [`pytoniq_core/tl/generator.py`](https://github.com/yungwine/pytoniq-core/blob/094d34393221b298a1798646ee9d7d186e128924/pytoniq_core/tl/generator.py) строки [306–310](https://github.com/yungwine/pytoniq-core/blob/094d34393221b298a1798646ee9d7d186e128924/pytoniq_core/tl/generator.py#L306-L310) теперь проверяют длину до запуска цикла. В тесте [`tests/test_tl.py`](https://github.com/yungwine/pytoniq-core/blob/094d34393221b298a1798646ee9d7d186e128924/tests/test_tl.py) строки [88–95](https://github.com/yungwine/pytoniq-core/blob/094d34393221b298a1798646ee9d7d186e128924/tests/test_tl.py#L88-L95) оставили точное признание старого поведения.

Приговор мягок лишь по форме: входные данные нельзя наделять властью, которой у них нет. Четыре байта могут задать длину, но не могут отменить физику сообщения. Газовый рожок пробил сотый удар, список остался коротким, а сыщик растворился в тумане, не дожидаясь, пока `0x7fffffff` дойдет до последнего элемента.

🐀
