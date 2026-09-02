+++
title = "№ 39 — Пароль, которого нет: pytoniq-core принимает защитную фразу за декоративную занавеску"
date = 2026-09-02T11:00:00+03:00
description = "Тридцать девятый выпуск «Вечернего Валидатора»: в pytoniq-core функции создания ключей торжественно принимают пароль мнемоники, передают его по цепочке и затем честно признаются, что пароль еще не реализован."
tags = ["pytoniq-core"]
+++

# 📰 ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

**№ 39 · Среда, 2 сентября 2026 г. · Цена: 0.05 TON (пароль принимается, но не учитывается)**

---

## ПАРОЛЬ, КОТОРОГО НЕТ

Лондон просыпался нехотя. Туман лежал на мостовой, как забытый seed-файл, а газовый рожок у редакции кашлял в сторону криптографических переулков. Сыщик «Вечернего Валидатора» получил наводку на [yungwine/pytoniq-core](https://github.com/yungwine/pytoniq-core) — библиотеку, которая умеет разбирать ячейки TON, считать хеши и выводить из мнемоники ключи. Вроде бы надежная контора. Вроде бы.

Место происшествия: [yungwine/pytoniq-core](https://github.com/yungwine/pytoniq-core), коммит [`0b7adf9`](https://github.com/yungwine/pytoniq-core/commit/0b7adf9895b85a76a5ce5356d5621e8fd2e980fe), файл [`pytoniq_core/crypto/keys.py`](https://github.com/yungwine/pytoniq-core/blob/0b7adf9895b85a76a5ce5356d5621e8fd2e980fe/pytoniq_core/crypto/keys.py), строки [51–55](https://github.com/yungwine/pytoniq-core/blob/0b7adf9895b85a76a5ce5356d5621e8fd2e980fe/pytoniq_core/crypto/keys.py#L51-L55). Цитата сверена с содержимым файла на этом коммите.

На двери нужного кабинета висела табличка: «mnemonic_to_entropy». Внутри сыщик нашел функцию с карманом для пароля. Карман есть. Пароля в нем нет.

Протокол, строки [51–55](https://github.com/yungwine/pytoniq-core/blob/0b7adf9895b85a76a5ce5356d5621e8fd2e980fe/pytoniq_core/crypto/keys.py#L51-L55):

```python
def mnemonic_to_entropy(mnemo_words: List[str], password: Optional[str] = None):
    # TODO: implement password
    sign = hmac.new((" ".join(mnemo_words)).encode(
        'utf-8'), bytes(0), hashlib.sha512).digest()
    return sign
```

Строка [52](https://github.com/yungwine/pytoniq-core/blob/0b7adf9895b85a76a5ce5356d5621e8fd2e980fe/pytoniq_core/crypto/keys.py#L52) не шепчет, а прямо пишет: `# TODO: implement password`. Это редкая честность в эпоху, когда многие механизмы предпочитают выглядеть готовыми до первого выстрела.

Но комизм сцены в другом. Аргумент `password` уже стоит в официальной форме функции. Он принимает значение, не падает, не ругается, не требует записки. Он просто проходит мимо гардероба. В HMAC попадает строка из слов мнемоники, ключом становится `bytes(0)`, а пароль остается в тумане.

## СЛЕД В ВЫСШИХ КАБИНЕТАХ

Сыщик поднялся по лестнице вызовов. На верхнем этаже сидела функция [`mnemonic_to_seed`](https://github.com/yungwine/pytoniq-core/blob/0b7adf9895b85a76a5ce5356d5621e8fd2e980fe/pytoniq_core/crypto/keys.py#L58-L60), строки [58–60](https://github.com/yungwine/pytoniq-core/blob/0b7adf9895b85a76a5ce5356d5621e8fd2e980fe/pytoniq_core/crypto/keys.py#L58-L60):

```python
def mnemonic_to_seed(mnemo_words: List[str], seed: bytes, password: Optional[str] = None):
    entropy = mnemonic_to_entropy(mnemo_words, password)
    return hashlib.pbkdf2_hmac("sha512", entropy, seed, PBKDF_ITERATIONS)
```

Вот он, почтенный конвейер: пароль принимается и передается дальше. Снаружи все выглядит солидно. Но внутри `mnemonic_to_entropy` этот пассажир теряет билет. В результате вызов с `password="сверхсекретно"` и вызов без пароля используют одну и ту же энтропию, если мнемоника и остальные параметры совпадают.

Читатель может проверить улику без лупы: строка [59](https://github.com/yungwine/pytoniq-core/blob/0b7adf9895b85a76a5ce5356d5621e8fd2e980fe/pytoniq_core/crypto/keys.py#L59) действительно передает пароль, но строки [52–55](https://github.com/yungwine/pytoniq-core/blob/0b7adf9895b85a76a5ce5356d5621e8fd2e980fe/pytoniq_core/crypto/keys.py#L52-L55) его не используют. Это не замок без ключа. Это замок, который вежливо принимает ключ и отпирается без него.

## КЛЮЧЕВОЙ СВИДЕТЕЛЬ

Дальше в коридоре обнаружилась [`mnemonic_to_private_key`](https://github.com/yungwine/pytoniq-core/blob/0b7adf9895b85a76a5ce5356d5621e8fd2e980fe/pytoniq_core/crypto/keys.py#L63-L69). Строки [63–69](https://github.com/yungwine/pytoniq-core/blob/0b7adf9895b85a76a5ce5356d5621e8fd2e980fe/pytoniq_core/crypto/keys.py#L63-L69):

```python
def mnemonic_to_private_key(mnemo_words: List[str], password: Optional[str] = None) -> Tuple[bytes, bytes]:
    """
    :rtype: (bytes(public_key), bytes(secret_key))
    """
    seed = mnemonic_to_seed(
        mnemo_words, 'TON default seed'.encode('utf-8'), password)
    return crypto_sign_seed_keypair(seed[:32])
```

Функция обещает вернуть публичный и секретный ключи. Пароль она тоже обещает учесть: строка [68](https://github.com/yungwine/pytoniq-core/blob/0b7adf9895b85a76a5ce5356d5621e8fd2e980fe/pytoniq_core/crypto/keys.py#L68) отправляет его в конвейер. Но конвейер ведет к тому же кабинету с табличкой `TODO`.

Так и рождается криптографический водевиль. Пользователь думает, что добавил второй замок. Библиотека кивает с серьезным видом и делает тот же ключ, что и без второго замка. Не злой умысел, не тайный взломщик, а самый опасный для доверчивого джентльмена вид призрака: параметр-фантом.

## ПРИЗРАК В МАСТЕРСКОЙ НОВЫХ МНЕМОНИК

Сыщик уже собирался уходить, когда из соседней комнаты донесся шелест новых фраз. На вывеске значилось [`mnemonic_new`](https://github.com/yungwine/pytoniq-core/blob/0b7adf9895b85a76a5ce5356d5621e8fd2e980fe/pytoniq_core/crypto/keys.py#L80-L93). Строки [80–88](https://github.com/yungwine/pytoniq-core/blob/0b7adf9895b85a76a5ce5356d5621e8fd2e980fe/pytoniq_core/crypto/keys.py#L80-L88):

```python
def mnemonic_new(words_count: int = 24, password: Optional[str] = None) -> List[str]:
    while True:
        mnemo_arr = []

        for _ in range(words_count):
            idx = get_secure_random_number(0, len(words))
            mnemo_arr.append(words[idx])

        if not is_basic_seed(mnemonic_to_entropy(mnemo_arr)):
            continue
```

Здесь пароль снова торжественно присутствует в подписи функции, строка [80](https://github.com/yungwine/pytoniq-core/blob/0b7adf9895b85a76a5ce5356d5621e8fd2e980fe/pytoniq_core/crypto/keys.py#L80), но проверка новой мнемоники на строке [88](https://github.com/yungwine/pytoniq-core/blob/0b7adf9895b85a76a5ce5356d5621e8fd2e980fe/pytoniq_core/crypto/keys.py#L88) вызывает `mnemonic_to_entropy` вообще без него. Пароль не просто забыт в комнате: его даже не пригласили на проверку.

Справедливости ради, отсутствие реализации честно помечено комментарием. Но интерфейс уже продает видимость готовности. В криптографии это тот случай, когда вывеска «второй сейф» висит над дверью пустого шкафа.

## ВЕРДИКТ СЫЩИКА

[pytoniq-core](https://github.com/yungwine/pytoniq-core) — полезная библиотека, а не злодей из подворотни. Однако коммит [`0b7adf9`](https://github.com/yungwine/pytoniq-core/commit/0b7adf9895b85a76a5ce5356d5621e8fd2e980fe) оставил нам сочную улику: API принимает пароль мнемоники на строке [51](https://github.com/yungwine/pytoniq-core/blob/0b7adf9895b85a76a5ce5356d5621e8fd2e980fe/pytoniq_core/crypto/keys.py#L51), передает его на строке [59](https://github.com/yungwine/pytoniq-core/blob/0b7adf9895b85a76a5ce5356d5621e8fd2e980fe/pytoniq_core/crypto/keys.py#L59), а затем теряет его на строке [52](https://github.com/yungwine/pytoniq-core/blob/0b7adf9895b85a76a5ce5356d5621e8fd2e980fe/pytoniq_core/crypto/keys.py#L52). Три двери, один сквозняк.

Редакционный совет выносит мягкий, но громкий приговор: если пароль еще не работает, интерфейс не должен делать вид, что он работает. Временная заглушка в криптографическом коридоре быстро становится постоянной мебелью. А человек, положившийся на декоративный замок, узнает правду только тогда, когда в тумане уже закрылась последняя дверь.

*Сыщик задул газовую лампу, перечеркнул слово «защищено» и оставил на столе записку: TODO — это не пароль.*

🐀
