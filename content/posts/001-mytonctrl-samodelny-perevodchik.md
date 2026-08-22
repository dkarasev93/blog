+++
title = "№ 1 — Самодельный переводчик в подвале mytonctrl"
date = 2026-08-14T12:00:00+03:00
description = "Первый выпуск «Вечернего Валидатора»: механизм интернационализации, достойный клейма и позора."
tags = ["mytonctrl"]
+++

# 📰 Вечерний Валидатор

**№ 1.** *Лондон. Туман. Газ в лампах мигает.*

## СЕНСАЦИЯ: В ПОДВАЛЕ MYTONCTRL ОТКОПАН САМОДЕЛЬНЫЙ ПЕРЕВОДЧИК

Ночью, при свете лупы, наш сыщик проник в [`mypylib/mypylib.py`](https://github.com/ton-blockchain/mytonctrl/blob/90046e0/mypylib/mypylib.py#L423) (строка 423, коммит [`90046e0`](https://github.com/ton-blockchain/mytonctrl/commit/90046e0) конторы [mytonctrl](https://github.com/ton-blockchain/mytonctrl)) и обнаружил там механизм интернационализации, каковой достоин клейма и позора:

```python
def translate(self, text: str) -> str:
    if self.translate_dict is None:
        return text
    lang = self.get_lang()
    text_list = text.split(' ')
    for item in text_list:
        sitem = self.translate_dict.get(item)
        if sitem is None:
            continue
        ritem = sitem.get(lang)
        if ritem is not None:
            text = text.replace(item, ritem)
    return text
```

Разбирательство показало: машинка разрывает текст на слова по пробелам и заменяет каждое слово **как подстроку по всему тексту**. Стоит в словарь попасть короткому слову вроде `on` — и всякий `button` в интерфейсе становится `buttвкл`. Пока контора спасается лишь тем, что все зовут её с одиночными ключами вроде `ton_status_head` — но это, с позволения сказать, не инженерия, это ходьба по краю крыши в тумане.

Словарь, между прочим, на 161 ключ и включает переводы на традиционный китайский (`zh_TW`). Валидаторы Формозы, вероятно, оценили.

**Вердикт:** работает случайно, ломается красиво. Холмс бы сказал: «Элементарно, Ватсон — это не переводчик, это приглашение к катастрофе».

*Ваш корреспондент из тумана.* 🕵️
