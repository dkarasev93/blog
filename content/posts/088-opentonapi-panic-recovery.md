+++
title = "№ 88 — Паника, которую отправили на службу"
date = 2026-09-28T10:58:00+03:00
tags = ["opentonapi"]
+++

# ВЕЧЕРНИЙ ВАЛИДАТОР

*Газета газовых фонарей, пыльных мемпулов и бессонных нод*

**№ 88.** *Лондон. Утренний туман залег между домами, газовый рожок у редакции сипит, а сыщик получил из [tonkeeper/opentonapi](https://github.com/tonkeeper/opentonapi) срочную папку. В ней контора решила спасать HTTP от паник: обычному падению выдать 500, частично отправленному ответу оборвать связь, а специальному `ErrAbortHandler` вернуть его законную тишину. В соседней комнате тестовый горожанин записал `nil`-карту, а затем весь участок стал проверять, как именно должен звучать обрыв.*

## МЕСТО ПРОИСШЕСТВИЯ

Улика найдена в репозитории [tonkeeper/opentonapi](https://github.com/tonkeeper/opentonapi), в коммите [`4542e8e`](https://github.com/tonkeeper/opentonapi/commit/4542e8e98a8a413cc12623b7bae62d10f7fd87ac), названном `recover from panics in http handlers`. Главные комнаты дела — файл [`pkg/api/recovery.go`](https://github.com/tonkeeper/opentonapi/blob/4542e8e98a8a413cc12623b7bae62d10f7fd87ac/pkg/api/recovery.go) и его свидетель [`pkg/api/recovery_test.go`](https://github.com/tonkeeper/opentonapi/blob/4542e8e98a8a413cc12623b7bae62d10f7fd87ac/pkg/api/recovery_test.go). Цитаты сняты с этого коммита.

## ПЕРВЫЙ СЛЕД: ПАНИКЕ ВЫДАЛИ ФОРМЕННЫЙ ОТВЕТ

На строках [21–28](https://github.com/tonkeeper/opentonapi/blob/4542e8e98a8a413cc12623b7bae62d10f7fd87ac/pkg/api/recovery.go#L21-L28) служба сама описывает свою новую обязанность:

```go
// recoveryMiddleware catches a panic raised while serving a request, logs it and responds with HTTP 500
// so that a single broken request doesn't abort the connection with an empty reply (which a load balancer
// reports as 502).
//
// It only catches panics in the request goroutine.
// A panic in a goroutine spawned by a handler still crashes the process,
// so handlers must run such goroutines with conc.WaitGroup, which re-panics in the caller's goroutine on Wait().
func recoveryMiddleware(logger *zap.Logger, next http.Handler) http.Handler {
```

Картина достойна туманного Лондона. Один сломанный запрос раньше мог оставить пустую связь, которую балансировщик принимал за 502. Теперь обычная паника должна получить аккуратный HTTP 500. Но внизу примечания честно стоит вторая дверь: паника в отдельном потоке все еще валит процесс, если служащий не привел ее обратно через `conc.WaitGroup`.

Это не обещание бессмертия. Это обещание, что крушение в одной комнате хотя бы получит бланк, номер и печать.

## ВТОРОЙ СЛЕД: СНАЧАЛА ОШИБКА, ПОТОМ КРАСИВЫЙ JSON

В перехватчике, строки [31–53](https://github.com/tonkeeper/opentonapi/blob/4542e8e98a8a413cc12623b7bae62d10f7fd87ac/pkg/api/recovery.go#L31-L53), сыщик находит полный протокол дежурства:

```go
		defer func() {
			rec := recover()
			if rec == nil {
				return
			}
			if err, ok := rec.(error); ok && errors.Is(err, http.ErrAbortHandler) {
				// http.ErrAbortHandler is the documented way to abort a response, net/http handles it silently.
				panic(rec)
			}
			httpPanicsCounter.Inc()
			logger.Error("panic in http handler",
				zap.String("path", r.URL.Path),
				zap.String("panic", fmt.Sprint(rec)),
				zap.ByteString("stack", debug.Stack()))
			if rw.wroteHeader {
				// The response is partially sent, we can't replace it with an error.
				// Abort the connection so the client doesn't take a truncated response as a complete one.
				panic(http.ErrAbortHandler)
			}
			rw.Header().Set("Content-Type", "application/json")
			rw.WriteHeader(http.StatusInternalServerError)
			json.NewEncoder(rw).Encode(&errorJSON{Error: "internal server error"})
		}()
```

Сначала проверяется, была ли паника специальным `http.ErrAbortHandler`. Если да, ее не приручают, а возвращают обратно в `net/http`, где она должна исчезнуть без лишнего шума. Если нет, счетчик растет, в журнал летят путь, причина и стек, а клиенту выдают `internal server error`.

Но если заголовок уже ушел, поздно приклеивать поверх него новый 500. Тогда middleware снова вызывает `http.ErrAbortHandler`, чтобы клиент не принял обрезанный ответ за полный. Викторианская мораль сурова: если письмо уже отправлено, не пытайся вклеить новый первый лист поверх телеграммы.

## ТРЕТИЙ СЛЕД: ОКНО С ОТКРЫТЫМ ОТВЕТОМ

Для отслеживания этого ритуала контора оборачивает писателя ответа. В файле [`pkg/api/recovery.go`](https://github.com/tonkeeper/opentonapi/blob/4542e8e98a8a413cc12623b7bae62d10f7fd87ac/pkg/api/recovery.go) строки [58–76](https://github.com/tonkeeper/opentonapi/blob/4542e8e98a8a413cc12623b7bae62d10f7fd87ac/pkg/api/recovery.go#L58-L76) помечают каждый `WriteHeader`, `Write` и `Flush`:

```go
type recoveryResponseWriter struct {
	http.ResponseWriter
	wroteHeader bool
}

func (w *recoveryResponseWriter) WriteHeader(statusCode int) {
	w.wroteHeader = true
	w.ResponseWriter.WriteHeader(statusCode)
}

func (w *recoveryResponseWriter) Write(b []byte) (int, error) {
	w.wroteHeader = true
	return w.ResponseWriter.Write(b)
}

// Flush keeps streaming endpoints (SSE, GraphQL stream) working through the wrapper.
func (w *recoveryResponseWriter) Flush() {
	w.wroteHeader = true
	_ = http.NewResponseController(w.ResponseWriter).Flush()
}
```

Почерк точный: даже потоковый ответ не забыли, потому что SSE и GraphQL любят отправлять сведения порциями. Но за эту аккуратность платят особой логикой: после первого заголовка у паники уже нет права на приличный JSON. Остается только оборвать связь, пока обрыв не притворился нормальным финалом.

## ЧЕТВЕРТЫЙ СЛЕД: ТЕСТ С НУЛЕВОЙ КАРТОЙ

Самая сочная сцена лежит в [`pkg/api/recovery_test.go`](https://github.com/tonkeeper/opentonapi/blob/4542e8e98a8a413cc12623b7bae62d10f7fd87ac/pkg/api/recovery_test.go). На строках [15–27](https://github.com/tonkeeper/opentonapi/blob/4542e8e98a8a413cc12623b7bae62d10f7fd87ac/pkg/api/recovery_test.go#L15-L27) тестовый участок специально устраивает аварию:

```go
	mux.HandleFunc("/goroutine-panic", func(w http.ResponseWriter, r *http.Request) {
		var wg conc.WaitGroup
		wg.Go(func() {
			var m map[string]int
			m["x"] = 1
		})
		wg.Wait()
	})
```

Служащий пишет `var m map[string]int`, не выделяет для нее память и тут же обращается к `m["x"] = 1`. Это намеренная нулевая карта, поставленная на службу в качестве свидетеля. Паника возникает внутри отдельного потока, а `wg.Wait()` возвращает ее в поток запроса, где middleware уже может выдать 500.

Две соседние строки превращают тест в маленький театр абсурда: карта не готова принять запись, а `WaitGroup` готов принять ее крик. В большом сервисе такой прием нужен, чтобы отдельный поток не унес панику прямо в могилу процесса.

## ПЯТЫЙ СЛЕД: ЧАСТИЧНОЕ ПИСЬМО НЕ ДОЕЗЖАЕТ

На строках [28–33](https://github.com/tonkeeper/opentonapi/blob/4542e8e98a8a413cc12623b7bae62d10f7fd87ac/pkg/api/recovery_test.go#L28-L33) тот же свидетель сначала отправляет ответ, а потом падает:

```go
	mux.HandleFunc("/partial", func(w http.ResponseWriter, r *http.Request) {
		w.WriteHeader(http.StatusOK)
		w.Write([]byte("partial"))
		w.(http.Flusher).Flush()
		panic("boom")
	})
```

Тест требует для такого маршрута ошибку чтения `io.ErrUnexpectedEOF` на строках [62–66](https://github.com/tonkeeper/opentonapi/blob/4542e8e98a8a413cc12623b7bae62d10f7fd87ac/pkg/api/recovery_test.go#L62-L66). Это не самый уютный финал, зато честный: клиент не получает липовый успех после слова `partial`.

А на строках [67–70](https://github.com/tonkeeper/opentonapi/blob/4542e8e98a8a413cc12623b7bae62d10f7fd87ac/pkg/api/recovery_test.go#L67-L70) отдельно проверяется законный `ErrAbortHandler`. Паника не превращается в JSON, не попадает в счетчик и не получает лишней лекции от middleware. У каждого обрыва есть своя форма.

## ШЕСТОЙ СЛЕД: СОАВТОР В ПОЛИЦЕЙСКОМ ПРОТОКОЛЕ

Сам коммит [`4542e8e`](https://github.com/tonkeeper/opentonapi/commit/4542e8e98a8a413cc12623b7bae62d10f7fd87ac) оставил необычную подпись: `Co-Authored-By: Claude Opus 5.5 (1M context)`. В деле о восстановлении после паник искусственный соавтор стоит рядом с человеческой логикой, которая различает пустой ответ, оборванный поток и служебный `ErrAbortHandler`.

Редакция не делает из этой строки доказательство качества. Но зрелище примечательное: контора пишет защиту от аварий, тестирует намеренное падение нулевой карты, а внизу протокола ставит рядом имя модели с миллионным контекстом. Газовый рожок кашлянул так, будто получил ревью от привидения.

## ХРОНИКА МЕЛКОГО КРИНЖА

*Отдел счетчиков.* На строках [15–18](https://github.com/tonkeeper/opentonapi/blob/4542e8e98a8a413cc12623b7bae62d10f7fd87ac/pkg/api/recovery.go#L15-L18) паника получает метрику `handler_panics_total`. Даже скандал теперь идет в статистику.

*Отдел потоков.* Строки [73–80](https://github.com/tonkeeper/opentonapi/blob/4542e8e98a8a413cc12623b7bae62d10f7fd87ac/pkg/api/recovery.go#L73-L80) сохраняют `Flush` и `Unwrap`. Обертка не должна ломать двери, через которые ответ идет струей.

*Отдел спокойного участка.* После паники тест на строках [71–76](https://github.com/tonkeeper/opentonapi/blob/4542e8e98a8a413cc12623b7bae62d10f7fd87ac/pkg/api/recovery_test.go#L71-L76) требует, чтобы маршрут `/ok` все еще вернул 200 и `ok`. Один преступник не должен закрывать всю улицу.

## ВЕРДИКТ СЫЩИКА

Репозиторий [tonkeeper/opentonapi](https://github.com/tonkeeper/opentonapi) пойман в коммите [`4542e8e`](https://github.com/tonkeeper/opentonapi/commit/4542e8e98a8a413cc12623b7bae62d10f7fd87ac) на необычной профилактике: файл [`pkg/api/recovery.go`](https://github.com/tonkeeper/opentonapi/blob/4542e8e98a8a413cc12623b7bae62d10f7fd87ac/pkg/api/recovery.go) превращает обычную панику в 500, частично отправленный ответ обрывает, а специальный `ErrAbortHandler` пропускает обратно в `net/http`.

Файл [`pkg/api/recovery_test.go`](https://github.com/tonkeeper/opentonapi/blob/4542e8e98a8a413cc12623b7bae62d10f7fd87ac/pkg/api/recovery_test.go) проверяет это через намеренную запись в нулевую карту на строках [20–26](https://github.com/tonkeeper/opentonapi/blob/4542e8e98a8a413cc12623b7bae62d10f7fd87ac/pkg/api/recovery_test.go#L20-L26), через обрыв после `Flush` на строках [28–33](https://github.com/tonkeeper/opentonapi/blob/4542e8e98a8a413cc12623b7bae62d10f7fd87ac/pkg/api/recovery_test.go#L28-L33) и через последующую проверку живого `/ok`.

Приговор без тумана: паника — не всегда враг, иногда она служебный сигнал. Но если сигнал поднялся в отдельном потоке, его надо вернуть в приемную; если письмо уже начало путь, его нельзя выдавать за целое; если карта пуста, не делай вид, что она умеет хранить ключи.

Газовый рожок дернулся. Сыщик закрыл папку, оставил на ней печать `internal server error` и растворился в лондонском тумане.

🐀
