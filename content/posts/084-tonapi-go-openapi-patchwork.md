+++
title = "№ 84 — Спецификация, которую чинят скриптом с ножницами"
date = 2026-09-26T10:58:00+03:00
description = "Восемьдесят четвертый выпуск «Вечернего Валидатора»: tonapi-go получает OpenAPI из чужого источника, а перед генерацией клиента чинит его awk, sed и пять контрольных вопросов."
tags = ["tonapi-go"]
+++

# ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

**№ 84.** *Лондон. Утренний туман прижался к мостовой, газовый рожок у редакции сипит, а сыщик получил папку из конторы [tonkeeper/tonapi-go](https://github.com/tonkeeper/tonapi-go). На обложке обещан Go-клиент для TonAPI. Внутри лежит не одна спецификация, а семейная драма: файл приходит сверху с пропавшей охраной и неверным именем действия, а перед сборкой его чинят локальным bash-скриптом. Канцелярия называет это патчем. Сыщик называет это утренним кроем OpenAPI. *

## МЕСТО ПРОИСШЕСТВИЯ

Улика найдена в репозитории [tonkeeper/tonapi-go](https://github.com/tonkeeper/tonapi-go), в коммите [`e055d93`](https://github.com/tonkeeper/tonapi-go/commit/e055d93bd26a091ddcf901b1f93720af6bcebd5c), в файлах [`scripts/patch-openapi.sh`](https://github.com/tonkeeper/tonapi-go/blob/e055d93bd26a091ddcf901b1f93720af6bcebd5c/scripts/patch-openapi.sh) и [`spec_test.go`](https://github.com/tonkeeper/tonapi-go/blob/e055d93bd26a091ddcf901b1f93720af6bcebd5c/spec_test.go). Коммит носит имя `update swagger, add spec patch script and tests`; цитаты сняты с этого снимка.

## СЕНСАЦИЯ: ВНЕШНЯЯ СПЕЦИФИКАЦИЯ ПРИХОДИТ БЕЗ ОХРАНЫ

Скрипт прямо поясняет причину своего появления. В комментарии на строках [3–5](https://github.com/tonkeeper/tonapi-go/blob/e055d93bd26a091ddcf901b1f93720af6bcebd5c/scripts/patch-openapi.sh#L3-L5) сказано дословно:

```bash
# Re-applies the local patches the upstream TonAPI spec is missing: the bearer
# security scheme, without which ogen drops oas_security_gen.go and the package
# stops compiling, and the Action.type value that must match its payload field.
```

Картина маслом: генератор клиента смотрит на документ, не видит bearer-схему, выбрасывает файл `oas_security_gen.go`, и пакет перестает собираться. Но вместо разговора с источником контора заводит собственный отдел реставрации. В нем два экспоната: пропавший пропуск для bearer и перепутанное имя поля в `Action.type`.

Порядок действий зафиксирован на строках [21–36](https://github.com/tonkeeper/tonapi-go/blob/e055d93bd26a091ddcf901b1f93720af6bcebd5c/scripts/patch-openapi.sh#L21-L36): если в спецификации нет верхнего `security:`, awk вставляет его прямо перед `paths:`. Причем рядом появляются сразу два режима — bearer и анонимный вход:

```bash
# Top-level security: bearer token, or anonymous.
if grep -q '^security:' "$spec"; then
	note 'security: already present'
else
	grep -q '^paths:$' "$spec" || fail "no top-level 'paths:' key to anchor the security block to"
	awk '
		/^paths:$/ && !inserted {
			print "security:"
			print "  - bearerAuth: [ ]"
			print "  - { }"
```

Сыщик не спорит с идеей публичного режима. Он отмечает декорацию: официальный входной документ сначала приходится распороть по месту, чтобы генератор вновь увидел охрану.

## ВТОРОЙ СЛЕД: ПРОПУСК ВПИСЫВАЮТ ПОСЛЕ ТОГО, КАК ЕГО ПОТЕРЯЛИ

Следующая дверь — строки [39–55](https://github.com/tonkeeper/tonapi-go/blob/e055d93bd26a091ddcf901b1f93720af6bcebd5c/scripts/patch-openapi.sh#L39-L55). Скрипт проверяет, есть ли `securitySchemes`, а если нет, врезает его прямо под `components:`:

```bash
# The bearerAuth scheme the block above refers to.
if grep -q '^  securitySchemes:$' "$spec"; then
	note 'components.securitySchemes: already present'
else
	grep -q '^components:$' "$spec" || fail "no top-level 'components:' key to anchor securitySchemes to"
	awk '
		{ print }
		/^components:$/ && !inserted {
			print "  securitySchemes:"
			print "    bearerAuth:"
			print "      type: http"
			print "      scheme: bearer"
```

Викторианская сцена проста: на фасаде написано «спецификация API», но сторожевой пост существует лишь после ночной работы `awk`. Входная бумага и локальная бумага выглядят как близнецы, пока генератор не спросит, где именно лежит охрана.

## ТРЕТИЙ СЛЕД: ДЕЙСТВИЕ НАЗВАЛИ НЕ ТЕМ ИМЕНЕМ

Вторая поправка еще сочнее. На строках [57–67](https://github.com/tonkeeper/tonapi-go/blob/e055d93bd26a091ddcf901b1f93720af6bcebd5c/scripts/patch-openapi.sh#L57-L67) контора меняет enum и ключ свойства:

```bash
# Only the enum entry and the property key are renamed, the schema they point at
# really is called SetSignatureAllowedAction.
if grep -q '^            - SetSignatureAllowedAction$' "$spec" ||
	grep -q '^        SetSignatureAllowedAction:$' "$spec"; then
	sed -e 's/^            - SetSignatureAllowedAction$/            - SetSignatureAllowed/' \\
		-e 's/^        SetSignatureAllowedAction:$/        SetSignatureAllowed:/' \\
		"$spec" >"$tmp" && mv "$tmp" "$spec"
	note 'Action.SetSignatureAllowed: restored'
```

То есть поле `Action.type` должно выбрать свой payload, но получает имя, которое относится к самой схеме действия, а не к полю на конверте. Локальная портновская мастерская исправляет это двумя заменами `sed`, после чего делает вид, что все так и было.

Тестовая папка подтверждает диагноз. На строках [71–73](https://github.com/tonkeeper/tonapi-go/blob/e055d93bd26a091ddcf901b1f93720af6bcebd5c/spec_test.go#L71-L73) автор оставил честную формулировку:

```go
// TestSpecActionUnionIsConsistent tests that every Action.type value names the property holding
// that action's payload. The generated code compiles either way, so nothing else notices.
func TestSpecActionUnionIsConsistent(t *testing.T) {
```

Вот где газовый рожок дал трещину. **Generated code compiles either way, so nothing else notices** — код собирается при любом из двух имен, а смысл связки может уже смотреть в пустой карман. Компилятор пропускает подозреваемого, потому что паспорт формально читаем.

Сам допрос устроен на строках [95–109](https://github.com/tonkeeper/tonapi-go/blob/e055d93bd26a091ddcf901b1f93720af6bcebd5c/spec_test.go#L95-L109): тест берет enum `Action.type`, требует для каждого значения одноименное payload-свойство и ругается, если в доме остались комнаты без ключа. Это уже не просто косметика: тест ловит рассогласование, которое генератор спокойно проглатывает.

## ЧЕТВЕРТЫЙ СЛЕД: ПЯТЬ ВОПРОСОВ ПОСЛЕ РЕМОНТА

В конце скрипт устраивает сам себе очную ставку. На строках [69–80](https://github.com/tonkeeper/tonapi-go/blob/e055d93bd26a091ddcf901b1f93720af6bcebd5c/scripts/patch-openapi.sh#L69-L80) лежит финальная проверка:

```bash
# Every patch must be observable in the result, whichever branch produced it.
while IFS= read -r probe; do
	grep -q "$probe" "$spec" || fail "verification failed: /$probe/ not found in $spec after patching"
done <<'PROBES'
^security:$
^  - bearerAuth: \\[ \\]$
^    bearerAuth:$
^            - SetSignatureAllowed$
^        SetSignatureAllowed:$
PROBES

note "$spec is patched"
```

Пять пробников стоят у двери и спрашивают: есть ли верхний `security`, bearer, схема, верное enum-имя и верный ключ? Если все ответы положительны, контора печатает `spec is patched`. Не «источник исправлен», не «схема синхронизирована», а именно «этот экземпляр пропатчен».

Рядом тест [TestSpecDeclaresBearerAuth](https://github.com/tonkeeper/tonapi-go/blob/e055d93bd26a091ddcf901b1f93720af6bcebd5c/spec_test.go#L37-L69) проверяет ту же историю уже на YAML: bearer должен иметь `type: http`, `scheme: bearer`, а верхний список обязан предлагать и bearer, и анонимный вариант. Канцелярия не доверяет даже собственной заплатке, что, по меркам туманных улиц, вполне разумно.

## ХРОНИКА МЕЛКОГО КРИНЖА

*Отдел временной ткани.* Временный файл получает имя [`$spec.patch-openapi.tmp`](https://github.com/tonkeeper/tonapi-go/blob/e055d93bd26a091ddcf901b1f93720af6bcebd5c/scripts/patch-openapi.sh#L10-L14), а ловушка на выходе удаляет его. Даже у заплатки есть гардеробная.

*Отдел строгих предупреждений.* На строках [7–8](https://github.com/tonkeeper/tonapi-go/blob/e055d93bd26a091ddcf901b1f93720af6bcebd5c/scripts/patch-openapi.sh#L7-L8) сказано, что неизвестный сдвиг upstream ведет к ненулевому завершению. Скрипт не молчит, если фасад перестроили.

*Отдел живого свидетеля.* Тест [TestSpecEnumsMatchLiveAPI](https://github.com/tonkeeper/tonapi-go/blob/e055d93bd26a091ddcf901b1f93720af6bcebd5c/spec_test.go#L112-L153) идет на `https://tonapi.io/v2/openapi.json`, сравнивает enum со своей копией и пропускает расхождение только по заранее записанной причине. Вечерний дозор, который сверяет газету с самой улицей.

## ВЕРДИКТ СЫЩИКА

Репозиторий [tonkeeper/tonapi-go](https://github.com/tonkeeper/tonapi-go) пойман не на обычном `TODO`, а на двухслойной канцелярии: в коммите [`e055d93`](https://github.com/tonkeeper/tonapi-go/commit/e055d93bd26a091ddcf901b1f93720af6bcebd5c) файл [`scripts/patch-openapi.sh`](https://github.com/tonkeeper/tonapi-go/blob/e055d93bd26a091ddcf901b1f93720af6bcebd5c/scripts/patch-openapi.sh) перед генерацией клиента добавляет пропавшую bearer-охрану и переименовывает `SetSignatureAllowedAction` в `SetSignatureAllowed`. А [`spec_test.go`](https://github.com/tonkeeper/tonapi-go/blob/e055d93bd26a091ddcf901b1f93720af6bcebd5c/spec_test.go#L71-L73) прямо признает: с неверным именем сгенерированный код все равно собирается, и ничего вокруг не замечает.

Сыщик не выносит приговор самому наличию локальной заплатки. Иногда upstream действительно приходится чинить до следующего релиза. Но жанр сцены безупречен: генератор кроит клиента по документу, документ чинят awk и sed, а затем тесты проверяют, что заплатка оставила следы. В тумане это называется не единой спецификацией, а семейным портретом, где каждый родственник пришел со своим именем.

Газовый рожок дернулся. На папке появилась печать: **«Схема доставлена. Перед употреблением подправить»**.

🐀
