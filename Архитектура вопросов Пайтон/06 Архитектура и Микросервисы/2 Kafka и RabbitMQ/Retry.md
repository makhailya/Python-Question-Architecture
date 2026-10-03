# 🔄 Retry

## 🎯 Ответ на собеседовании

**Retry** — механизм повторного выполнения операции после временной ошибки.

Он используется, когда есть вероятность, что ошибка временная:

* сетевой сбой;
* timeout;
* временная недоступность сервиса;
* перегрузка;
* временная ошибка базы данных;
* временный сбой message broker.

Главная проблема Retry — **повторная операция может выполниться дважды**, поэтому Retry часто должен использоваться вместе с **идемпотентностью**.

> **Retry повышает надёжность системы, а Idempotency защищает от последствий повторного выполнения.**

---

## 🎤 Суперкоротко

```text
Ошибка
  ↓
Retry
  ↓
повторить операцию
  ↓
успех → готово
ошибка → следующий retry
```

Но:

```text
Retry + неидемпотентная операция
        ↓
возможный дубль
```

Поэтому:

```text
Retry + Idempotency
        ↓
безопасный повтор
```

---

# 🔹 Зачем нужен Retry

Представим:

```text
Client
  ↓
Payment Service
  ↓
Timeout
```

Timeout не обязательно означает, что операция не выполнилась.

Возможны два сценария.

### Сценарий 1

```text
Request
  ↓
Service недоступен
  ↓
Error
```

Retry может помочь:

```text
Request
  ↓
Retry
  ↓
Service доступен
  ↓
Success
```

### Сценарий 2

```text
Request
  ↓
Payment создан
  ↓
Response потерян
  ↓
Client получает timeout
```

Клиент делает Retry:

```text
Retry
  ↓
Payment уже создан
```

И здесь нужна **идемпотентность**.

---

# 🔹 Базовый алгоритм

Упрощённо:

```python
for attempt in range(3):
    try:
        result = do_operation()
        return result
    except TemporaryError:
        if attempt == 2:
            raise
```

Например:

```text
Attempt 1 → ошибка
Attempt 2 → ошибка
Attempt 3 → успех
```

---

# 🔢 Количество попыток

Обычно задают ограничение:

```text
max_retries = 3
```

Например:

```text
1-я попытка
   ↓ ошибка
2-я попытка
   ↓ ошибка
3-я попытка
   ↓ ошибка
STOP
```

Нельзя делать бесконечные retry.

Иначе система может получить:

```text
Retry
 ↓
Retry
 ↓
Retry
 ↓
Retry
 ↓
...
```

и создать дополнительную нагрузку.

---

# ⏱️ Retry Delay

Между попытками обычно делают задержку.

Простейший вариант:

```text
Request
 ↓
ошибка
 ↓
1 сек
 ↓
Retry
 ↓
ошибка
 ↓
1 сек
 ↓
Retry
```

Но лучше использовать **backoff**.

---

# 📈 Exponential Backoff

**Exponential Backoff** — увеличение задержки между попытками.

Например:

```text
1 сек
2 сек
4 сек
8 сек
16 сек
```

Формула:

```text
delay = base × 2^attempt
```

Например:

```text
attempt 0 → 1 сек
attempt 1 → 2 сек
attempt 2 → 4 сек
attempt 3 → 8 сек
```

Это уменьшает нагрузку на временно недоступный сервис.

---

# 🎲 Jitter

Если тысячи клиентов одновременно делают Retry:

```text
10 000 клиентов
      ↓
ошибка
      ↓
через 1 секунду
      ↓
10 000 запросов одновременно
```

Это может создать **thundering herd**.

Поэтому к backoff добавляют случайную составляющую — **jitter**.

Например:

```text
Client A → 1.2 сек
Client B → 1.7 сек
Client C → 1.1 сек
Client D → 1.9 сек
```

Запросы распределяются во времени.

---

# 🔥 Exponential Backoff + Jitter

Типичная схема:

```text
Ошибка
  ↓
Retry
  ↓
1–2 сек
  ↓
Retry
  ↓
2–4 сек
  ↓
Retry
  ↓
4–8 сек
```

Jitter делает задержку немного случайной.

---

# 🚫 Какие ошибки не нужно повторять

Не каждую ошибку имеет смысл retry.

Например:

```text
400 Bad Request
```

Обычно означает ошибку самого запроса.

Повторение того же запроса:

```text
400
400
400
400
```

не поможет.

То же самое часто относится к:

```text
401 Unauthorized
403 Forbidden
404 Not Found
```

Retry прежде всего нужен для **временных ошибок**.

Например:

```text
timeout
connection reset
temporary unavailable
429 Too Many Requests
5xx
```

Но конкретная политика зависит от API и типа операции.

---

# ⚠️ Retry и 429

`429 Too Many Requests` означает, что клиент отправил слишком много запросов.

Сервер может вернуть:

```text
Retry-After: 10
```

Тогда клиент должен подождать указанное время:

```text
429
 ↓
Retry-After: 10
 ↓
10 секунд
 ↓
Retry
```

---

# ⚠️ Retry и 5xx

Например:

```text
503 Service Unavailable
```

Сервис временно недоступен.

Retry может помочь:

```text
503
 ↓
backoff
 ↓
retry
 ↓
200 OK
```

Но бесконечно повторять запрос нельзя.

---

# 🔑 Retry + Idempotency

Это одна из самых важных связок.

Плохой вариант:

```text
POST /payment
      ↓
Payment created
      ↓
Timeout
      ↓
Retry
      ↓
Payment created AGAIN
```

Хороший вариант:

```text
POST /payment
Idempotency-Key: abc-123
      ↓
Payment created
      ↓
Timeout
      ↓
Retry + same key
      ↓
Payment already exists
      ↓
return previous result
```

---

# 📨 Retry в Kafka

Consumer может получить сообщение:

```text
Event #123
   ↓
processing
   ↓
error
```

Можно повторить обработку:

```text
Event #123
   ↓
retry
   ↓
success
```

Но consumer должен учитывать возможность повторной обработки.

Поэтому важна:

```text
Kafka
 ↓
Retry
 ↓
Idempotent Consumer
```

---

# 🐇 Retry в RabbitMQ

Сообщение может быть обработано:

```text
Consumer
   ↓
error
```

и отправлено на повторную обработку.

Часто используют:

```text
Queue
 ↓
Consumer
 ↓
Error
 ↓
Retry / Dead Letter
```

При большом количестве неудачных попыток сообщение можно отправить в **DLQ — Dead Letter Queue**.

---

# 🐍 Retry в Celery

В Celery задача может быть повторена:

```python
@app.task(bind=True, max_retries=3)
def send_email(self):
    try:
        send()
    except TemporaryError as exc:
        raise self.retry(exc=exc, countdown=10)
```

Схема:

```text
Task
 ↓
Worker
 ↓
ошибка
 ↓
retry
 ↓
Worker
 ↓
ошибка
 ↓
retry
```

После достижения лимита:

```text
max_retries
     ↓
Task FAILED
```

---

# 🌐 Retry в HTTP-клиенте

Например, сервис A вызывает сервис B:

```text
Service A
   ↓ HTTP
Service B
```

Если B временно недоступен:

```text
Service A
   ↓
timeout
   ↓
retry
   ↓
Service B
   ↓
200 OK
```

Но если запрос:

```text
POST /payment
```

то необходимо учитывать идемпотентность.

---

# 💥 Опасность Retry Storm

Если сервис B упал:

```text
Service A
Service C
Service D
Service E
   ↓
все делают retry
   ↓
Service B
```

После восстановления B получает огромное количество запросов.

Это называется **Retry Storm**.

Поэтому используют:

* ограничение количества retries;
* exponential backoff;
* jitter;
* timeout;
* circuit breaker;
* rate limiting.

---

# 🔌 Retry vs Circuit Breaker

Это разные механизмы.

### Retry

> «Попробуй ещё раз».

```text
Error
 ↓
Retry
```

### Circuit Breaker

> «Сервис сейчас явно недоступен — временно перестань его вызывать».

```text
Service B
   ↓
много ошибок
   ↓
Circuit OPEN
   ↓
не отправлять запросы
```

Через некоторое время Circuit Breaker проверяет:

```text
HALF-OPEN
   ↓
тестовый запрос
   ↓
успех → CLOSED
```

---

# 🧠 Типичная архитектура

```text
              ┌──────────────┐
              │   Service B  │
              └──────▲───────┘
                     │
               HTTP request
                     │
              ┌──────┴───────┐
              │   Service A  │
              └──────────────┘
                     │
              Timeout / 5xx
                     ↓
                  Retry
                     ↓
             Exponential Backoff
                     ↓
                  Retry
                     ↓
                Idempotency
```

---

# 📊 Retry-политика

Хорошая Retry Policy обычно определяет:

| Параметр         | Пример            |
| ---------------- | ----------------- |
| Max retries      | 3                 |
| Initial delay    | 1 сек             |
| Backoff          | Exponential       |
| Jitter           | Да                |
| Timeout          | 5 сек             |
| Retryable errors | 429, 5xx, timeout |
| Non-retryable    | 400, 401, 403     |
| После retries    | FAILED / DLQ      |

---

# 🔗 Связь с предыдущими темами

```text
Временная ошибка
       ↓
      Retry
       ↓
повторное выполнение
       ↓
┌──────────────────┐
│ операция должна  │
│ быть идемпотентной│
└──────────────────┘
       ↓
Idempotency Key
       ↓
защита от дублей
```

В распределённых системах:

```text
Retry
  ↓
At-Least-Once
  ↓
Duplicate
  ↓
Idempotency / Deduplication
```

---

## 🧠 Главное

> **Retry — это повторное выполнение операции после временной ошибки.**

Хороший Retry обычно включает:

```text
Max Retries
     +
Timeout
     +
Exponential Backoff
     +
Jitter
     +
Retryable Errors
     +
Idempotency
```

### Формула для собеседования

> **Retry используется для восстановления после временных ошибок. Количество попыток ограничивают, между попытками используют backoff и часто jitter. При повторении операции необходимо учитывать идемпотентность, чтобы retry не привёл к дублированию бизнес-операции.**
