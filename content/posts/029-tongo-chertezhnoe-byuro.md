+++
title = "№ 29 — Чертёжное бюро: буква-двойник в регистрах несобранного двигателя, фраза, умершая на слове «simply», и машина из пустых ящиков"
date = 2026-08-28T17:10:00+03:00
description = "Двадцать девятый выпуск «Вечернего Валидатора»: сыщик спускается в запертую чертёжную палату tvm2 мастерских tonkeeper/tongo, где три года лежит проект второго двигателя. На чертеже четыре регистра подписаны кириллической буквой «с» — двойником, неотличимым от латинской; описание регистра c5 обрывается на слове «simply»; комментарии переписаны из фолианта TVM вместе с переносами и типографскими кавычками; а весь двигатель состоит из семидесяти шести строк подписей над пустыми ящиками — и ни одной детали."
tags = ["tongo"]
+++

# 📰 Вечерний Валидатор

**№ 29.** *Лондон. Туман нынче свинцовый — такой бывает над мастерскими, где работа давно остановилась, а дверь забыли запереть. Наш корреспондент, обойдя брачное бюро (выпуск № 28), зал прощания (выпуск № 27) и двигательную палату (выпуск № 26), ступил на территорию мастерских [tonkeeper/tongo](https://github.com/tonkeeper/tongo) — славного заведения, где куют инструменты для тех, кто пишет на языке Го. В главном цеху, [`tvm/`](https://github.com/tonkeeper/tongo/tree/1bbbb32609ce87319e3abd9317eb872c2da492b6/tvm), гудит исправный станок: честная обвязка вокруг готового чугунного двигателя. Но сыщика, как водится, потянуло не туда, где гудит, а туда, где тихо. В конце коридора нашлась комнатка с табличкой `tvm2` — вторая двигательная палата. Дверь была не заперта. Внутри — чертёжный стол, папка из одного листа и три года пыли.*

*Место происшествия: репозиторий [tonkeeper/tongo](https://github.com/tonkeeper/tongo), коммит [`1bbbb32`](https://github.com/tonkeeper/tongo/commit/1bbbb32609ce87319e3abd9317eb872c2da492b6) (master от 28.08.2026). Файлы: [`tvm2/tvm.go`](https://github.com/tonkeeper/tongo/blob/1bbbb32609ce87319e3abd9317eb872c2da492b6/tvm2/tvm.go), [`txemulator/trace.go`](https://github.com/tonkeeper/tongo/blob/1bbbb32609ce87319e3abd9317eb872c2da492b6/txemulator/trace.go), [`abi/generated_test.go`](https://github.com/tonkeeper/tongo/blob/1bbbb32609ce87319e3abd9317eb872c2da492b6/abi/generated_test.go), [`tl/parser/generator.go`](https://github.com/tonkeeper/tongo/blob/1bbbb32609ce87319e3abd9317eb872c2da492b6/tl/parser/generator.go). Все цитаты дословные.*

---

## СЕНСАЦИЯ ПЕРВАЯ: БУКВА С ФАЛЬШИВЫМ ПАСПОРТОМ

Чертёж, лежащий на столе, изображает сердце виртуальной машины — её регистры. Глянем на досье, [`tvm2/tvm.go`](https://github.com/tonkeeper/tongo/blob/1bbbb32609ce87319e3abd9317eb872c2da492b6/tvm2/tvm.go), строки [8–41](https://github.com/tonkeeper/tongo/blob/1bbbb32609ce87319e3abd9317eb872c2da492b6/tvm2/tvm.go#L8-L41):

```go
type TVM struct {
	// c0 — Contains the next continuation or return continuation (similar
	//to the subroutine return address in conventional designs). This value
	//must be a Continuation.
	c0 Continuation
	// c1 — Contains the alternative (return) continuation; this value must
	//be a Continuation. It is used in some (experimental) control flow
	//primitives, allowing TVM to define and call “subroutines with two exit
	//points”.
	с1 Continuation
	// c2 — Contains the exception handler. This value is a Continuation,
	//invoked whenever an exception is triggered.
	с2 Continuation
	// c3 — Contains the current dictionary, essentially a hashmap containing
	//the code of all functions used in the program. For reasons explained
	//later in 4.6, this value is also a Continuation, not a Cell as one might
	//expect.
	с3 Continuation
	// c4 — Contains the root of persistent data, or simply the data. This
	//value is a Cell. When the code of a smart contract is invoked, c4
	//points to the root cell of its persistent data kept in the blockchain
	//state. If the smart contract needs to modify this data, it changes c4
	//before returning.
	с4 boc.Cell
	// c5 — Contains the output actions. It is also a Cell initialized by a
	//reference to an empty cell, but its final value is considered one of the
	//smart contract outputs. For instance, the SENDMSG primitive, specific
	//for the TON Blockchain, simply
	c5 boc.Cell

	// c7 Contains the root of temporary data. It is a Tuple, initialized by
	//a reference to an empty Tuple before invoking the smart contract and
	//discarded after its termination.
	c7 Tuple
```

Читатель видит восемь регистров: c0, c1, c2, c3, c4, c5, c7 и далее по списку. Сыщик видит иное. Сыщик вооружился лупой судебной экспертизы и сверил каждую букву по байтам. Регистр на строке [12](https://github.com/tonkeeper/tongo/blob/1bbbb32609ce87319e3abd9317eb872c2da492b6/tvm2/tvm.go#L12) подписан честной латинской `c0`. А вот регистры на строках [17](https://github.com/tonkeeper/tongo/blob/1bbbb32609ce87319e3abd9317eb872c2da492b6/tvm2/tvm.go#L17), [20](https://github.com/tonkeeper/tongo/blob/1bbbb32609ce87319e3abd9317eb872c2da492b6/tvm2/tvm.go#L20), [25](https://github.com/tonkeeper/tongo/blob/1bbbb32609ce87319e3abd9317eb872c2da492b6/tvm2/tvm.go#L25) и [31](https://github.com/tonkeeper/tongo/blob/1bbbb32609ce87319e3abd9317eb872c2da492b6/tvm2/tvm.go#L31) подписаны другой рукой и другим алфавитом: в именах `с1`, `с2`, `с3`, `с4` стоит не латинская «c», а кириллическая «эс» — знак U+0441, в протоколе экспертизы байты `d1 81`. Регистры c5 и c7, строки [36](https://github.com/tonkeeper/tongo/blob/1bbbb32609ce87319e3abd9317eb872c2da492b6/tvm2/tvm.go#L36) и [41](https://github.com/tonkeeper/tongo/blob/1bbbb32609ce87319e3abd9317eb872c2da492b6/tvm2/tvm.go#L41), снова латинские. Итого в одной конторской книге восемь имён, четыре из которых — двойники с фальшивым паспортом.

Поймите, читатель, всю прелесть положения. Глазом эту подмену не увидеть ни в редакторе, ни на страницах Гитхаба, ни под каким фонарём: буквы совпадают до последнего волоска. Контора компилятора тоже не возражает — по уставу языка Го имя вольно состоять из букв любого алфавита мира, и служащий штампует бумаги, не глядя. Но горе тому клерку, кто придёт со стороны и попробует дотянуться до регистра латинской рукой: машина ответит, что такого служащего в штате нет, — и будет права. Сыщик записал в блокнот: идеальная минная закладка, из тех, что кладут не из злого умысла, а по чистой случайности раскладки клавиатуры, — оттого она и страшна.

Когда совершена подмена? Сыщик поднял метрики. Лист [`tvm2/tvm.go`](https://github.com/tonkeeper/tongo/blob/1bbbb32609ce87319e3abd9317eb872c2da492b6/tvm2/tvm.go) рождён в коммите [`10caa18`](https://github.com/tonkeeper/tongo/commit/10caa18dda449a00def384ad6f028070404b0409) от 06.06.2023 с лаконичной пометкой «rename methods» — и с того дня, по показаниям `git blame`, ни одна строка в нём не менялась. Более трёх лет, вплоть до нынешнего мастера [`1bbbb32`](https://github.com/tonkeeper/tongo/commit/1bbbb32609ce87319e3abd9317eb872c2da492b6), буква-двойник дежурит на чертеже, и никто в доме её не опознал. Хотя опознать её мог любой — стоило лишь попытаться написать имя регистра по памяти.

## СЕНСАЦИЯ ВТОРАЯ: ФРАЗА, УМЕРШАЯ НА СЛОВЕ «SIMPLY»

Перечитайте ещё раз описание регистра c5, строки [32–35](https://github.com/tonkeeper/tongo/blob/1bbbb32609ce87319e3abd9317eb872c2da492b6/tvm2/tvm.go#L32-L35):

```go
	// c5 — Contains the output actions. It is also a Cell initialized by a
	//reference to an empty cell, but its final value is considered one of the
	//smart contract outputs. For instance, the SENDMSG primitive, specific
	//for the TON Blockchain, simply
```

«For instance, the SENDMSG primitive, specific for the TON Blockchain, simply» — и всё. Дальше строка [36](https://github.com/tonkeeper/tongo/blob/1bbbb32609ce87319e3abd9317eb872c2da492b6/tvm2/tvm.go#L36), само поле, молчание. Фраза оборвана на полуслове, как записка человека, которого унесли из-за стола прямо во время письма. Что там делает SENDMSG — просто? Просто отправляет? Просто исчезает? Черновик не говорит. Сыщик обыскал стол: ни продолжения на полях, ни второго листа. Писарь сел дописывать описание, дошёл до самого интересного места — и не вернулся. Никогда. Три года, читатель, предложение ждёт своего глагола, и, судя по пыли на подоконнике, глагол этот ходит нынче по другому делу.

## СЕНСАЦИЯ ТРЕТЬЯ: ОТПЕЧАТКИ ПАЛЬЦЕВ ЧУЖОГО ФОЛИАНТА

Откуда вообще взялись описания на чертеже? Сыщик — старый графоман и узнаёт почерк переписчика. Смотрите на строки [47–48](https://github.com/tonkeeper/tongo/blob/1bbbb32609ce87319e3abd9317eb872c2da492b6/tvm2/tvm.go#L47-L48):

```go
	//completed). This component is similar to the instruction pointer reg-
	//ister (ip) in other architectures.
```

«reg-» на одной строке, «ister» на следующей. Это, читатель, перенос слова по слогам — след книжной вёрстки, шрам от колонки печатного листа. Описания регистров выписаны из самого фолианта TVM, из учёных глав про c0–c7, — переписаны старательно, но машинально, так, что вместе с текстом в комментарий переехали и типографские приметы оригинала: книжные кавычки-лапки вокруг “subroutines with two exit points” на строках [15–16](https://github.com/tonkeeper/tongo/blob/1bbbb32609ce87319e3abd9317eb872c2da492b6/tvm2/tvm.go#L15-L16) и учёные значки «0 ≤ gl ≤ gm, gc ≥ 0» на строке [58](https://github.com/tonkeeper/tongo/blob/1bbbb32609ce87319e3abd9317eb872c2da492b6/tvm2/tvm.go#L58). Сыщик не порицает: списывать с хорошей книги — старая лондонская традиция. Но когда с листа на лист переезжают даже переносы строк, это уже не цитирование — это переезд с вещами.

## СЕНСАЦИЯ ЧЕТВЁРТАЯ: ДВИГАТЕЛЬ ИЗ ПУСТЫХ ЯЩИКОВ

И вот главное, читатель. Перевернём чертёж и посмотрим, что за машину задумали в палате `tvm2`. Дно листа, строки [70–76](https://github.com/tonkeeper/tongo/blob/1bbbb32609ce87319e3abd9317eb872c2da492b6/tvm2/tvm.go#L70-L76):

```go
type Continuation struct {
}

type Tuple struct {
}
type StackValue struct {
}
```

Вот и весь двигатель. Три типа-таблички, за каждой — пустые фигурные скобки, как ящики без содержимого. Весь пакет `tvm2` — это один файл на семьдесят шесть строк, в котором начертана структура машины, тщательно, по учёному фолианту, расписаны её регистры (с одной буквой-двойником в комплекте), выписаны пределы газа, строки [62–67](https://github.com/tonkeeper/tongo/blob/1bbbb32609ce87319e3abd9317eb872c2da492b6/tvm2/tvm.go#L62-L67), — и не написано ни единой строчки, которая бы эту машину заводила. Ни метода, ни функции, ни исполнителя. Сыщик обыскал весь дом: пакет `tvm2` не вызывается никем и нигде — ни в одном цеху мастерских [tonkeeper/tongo](https://github.com/tonkeeper/tongo) на него нет даже ссылки в воротах `import`. Работает же заведение на соседнем станке [`tvm/`](https://github.com/tonkeeper/tongo/tree/1bbbb32609ce87319e3abd9317eb872c2da492b6/tvm) — честной обвязке вокруг готового эмулятора.

Картина вырисовывается классическая, знакомая любому лондонскому сыщику: летом 2023 года в мастерских решили построить второй двигатель — свой, родной, без чужого чугуна. Наняли чертёжника. Чертёжник пришёл, выписал из фолианта все регистры, развесил таблички, перенёс с буквами диковинную раскладку, дошёл до описания c5 — и слово «simply» стало последним, что слышали от него в этом городе. Чертёж остался лежать, где лежал. Три года мимо ходят люди, коммитят рядом, чинят соседние станки — и никто не заглядывает в палату номер два. А буква-двойник всё это время сидит в имени регистра, как фальшивый клерк в приёмной, — ждёт, пока кто-нибудь, наконец, возьмётся за второй двигатель и споткнётся о порог.

---

## МЕЛКИМ ПОЧЕРКОМ

*Сыщик не смог уйти из мастерских без приложений к протоколу.*

**Времянка на века.** В соседнем цеху, [`txemulator/trace.go`](https://github.com/tonkeeper/tongo/blob/1bbbb32609ce87319e3abd9317eb872c2da492b6/txemulator/trace.go), строки [293–296](https://github.com/tonkeeper/tongo/blob/1bbbb32609ce87319e3abd9317eb872c2da492b6/txemulator/trace.go#L293-L296), стоит постройка с табличкой `// todo: remove temporatry workaround after dynamic libs loading in emulator`. «Временное» написано с опечаткой — «temporatry», что для Лондона знак верный: времянка, названная криво, стоит дольше капитальной. Под табличкой — словарь из одной записи: хэш библиотеки и вся библиотека целиком, зашитая прямо в исходник одной строкой на две тысячи шестьсот сорок семь знаков, строка [295](https://github.com/tonkeeper/tongo/blob/1bbbb32609ce87319e3abd9317eb872c2da492b6/txemulator/trace.go#L295). Построена времянка в коммите [`b11f1e1`](https://github.com/tonkeeper/tongo/commit/b11f1e10f90b1f8b8bda8e1e0b20f0a79c7e4419) от 06.11.2025 под честным именем «add workaround for some contracts» и честно же используется на строках [329–330](https://github.com/tonkeeper/tongo/blob/1bbbb32609ce87319e3abd9317eb872c2da492b6/txemulator/trace.go#L329-L330). Сносить её, разумеется, будут — как только в эмуляторе научатся грузить библиотеки. Туман, читатель, тоже временный. Стоит с прошлого четверга.

**Допрос вещдока.** В испытательной палате, [`abi/generated_test.go`](https://github.com/tonkeeper/tongo/blob/1bbbb32609ce87319e3abd9317eb872c2da492b6/abi/generated_test.go), строка [3052](https://github.com/tonkeeper/tongo/blob/1bbbb32609ce87319e3abd9317eb872c2da492b6/abi/generated_test.go#L3052), сыщик нашёл вершину следственной мысли: над делом `WalletSignedV4ExtInMsgBody` висит записка `// TODO: what is it?`. Следователь спрашивает у вещдока, что он такое. Вещдок молчит — его показания тремя строками ниже заботливо закомментированы. Дело приостановлено за невыясненностью личности.

**Типография недоделок.** А вот в [`tl/parser/generator.go`](https://github.com/tonkeeper/tongo/blob/1bbbb32609ce87319e3abd9317eb872c2da492b6/tl/parser/generator.go), строки [357–358](https://github.com/tonkeeper/tongo/blob/1bbbb32609ce87319e3abd9317eb872c2da492b6/tl/parser/generator.go#L357-L358), работает машинка поиндустриальнее: генератор кода, не зная, как быть с типом `True`, не молчит, а печатает в каждый свежий сгенерированный файл честное признание — `builder.WriteString("// TODO: set optional field for True type \n")`. То есть долг тут не копят — долг тут тиражируют. Каждый прогон типографии разносит по свежим листам новую порцию долговых расписок. Сыщик залюбовался: вот же она, фабрика вечного ремонта, конвейерная лента.

---

*Сыщик вышел из мастерских в свинцовый туман и долго стоял у палаты номер два, глядя, как газовый рожок дрожит над табличкой `tvm2`. Где-то внутри, в семидесяти шести строках чертежа, четыре кириллические буквы притворяются латинскими, предложение про SENDMSG третий год ждёт глагола, а двигатель будущего стоит на месте — впрочем, куда ему деваться: он собран из одних подписей. «Город полон машин, — записал сыщик напоследок, закрывая блокнот. — Одни гудят, другие нарисованы. Опаснее всего третьи: нарисованные с буквой-двойником в регистре, — их починка начнётся с очной ставки алфавитов». Рожок мигнул. Буква «эс» в темноте была неотличима от «си». Именно этого она и добивалась.*
