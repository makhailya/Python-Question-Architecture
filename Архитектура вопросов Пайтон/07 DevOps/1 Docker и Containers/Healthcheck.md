# ❤️‍🩹 Healthcheck в Docker

## 🎤 Короткий ответ

**Healthcheck** — это механизм Docker для проверки, **готов ли контейнер нормально работать**. Docker периодически выполняет указанную команду внутри контейнера и по её результату определяет состояние: `starting`, `healthy` или `unhealthy`.

Важно: **контейнер может быть запущен (`running`), но при этом быть `unhealthy`**.

## 🎯 Формула для собеседования

> **Healthcheck = периодическая проверка → команда успешно выполнилась → `healthy`; ошибка → `unhealthy`.**

---

# 🔹 Зачем нужен Healthcheck

Сам факт запуска процесса ещё не означает, что приложение готово принимать запросы.

Например, PostgreSQL:

```text
Container started
       ↓
PostgreSQL process started
       ↓
инициализация БД...
       ↓
БД готова принимать подключения
```

Docker может видеть:

```text
STATUS: Up
```

но приложение внутри ещё не готово.

Healthcheck позволяет проверять именно **работоспособность/готовность**, а не только факт существования процесса.

---

# 🔹 Состояния

У контейнера с Healthcheck есть три основных состояния:

```text
starting
   ↓
healthy
```

или:

```text
starting
   ↓
unhealthy
```

### `starting`

Проверка ещё не прошла успешно необходимое количество раз.

### `healthy`

Последняя проверка успешна.

### `unhealthy`

Проверки завершаются ошибкой заданное количество раз.

---

# 🔹 Простой пример

В `compose.yaml`:

```python id="h7k3qm"
services:
  db:
    image: postgres:15
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user -d app"]
      interval: 5s
      timeout: 5s
      retries: 5
```

Docker примерно каждые 5 секунд выполняет:

```text
pg_isready -U user -d app
```

Если PostgreSQL готов:

```text
healthy
```

Если несколько проверок подряд не проходят:

```text
unhealthy
```

---

# 🔹 Основные параметры

## `test`

Команда, которая выполняет проверку.

```python id="m8p4vc"
healthcheck:
  test: ["CMD-SHELL", "pg_isready -U user -d app"]
```

Можно использовать:

```python id="n3q6wx"
test: ["CMD", "pg_isready", "-U", "user", "-d", "app"]
```

или:

```python id="x5r9ka"
test: ["CMD-SHELL", "curl -f http://localhost:8000/health || exit 1"]
```

Главная идея:

> Команда должна завершиться с кодом `0`, если сервис здоров.

---

# 🔹 Exit Code

Healthcheck ориентируется прежде всего на **код завершения команды**.

```text id="p7c2mv"
exit 0
   ↓
успешно
   ↓
healthy
```

А:

```text id="v4n8qx"
exit 1
   ↓
ошибка
   ↓
неуспешная проверка
```

Например:

```python id="q2m7sd"
curl -f http://localhost:8000/health
```

Если endpoint отвечает успешно → `0`.

Если запрос завершился ошибкой → ненулевой exit code.

---

# 🔹 `interval`

Как часто выполнять проверку:

```python id="j8k4pr"
interval: 10s
```

То есть примерно раз в 10 секунд.

---

# 🔹 `timeout`

Сколько максимум ждать выполнения одной проверки:

```python id="w3m9xc"
timeout: 5s
```

Если команда не завершилась за 5 секунд, проверка считается неуспешной.

---

# 🔹 `retries`

Сколько последовательных неудачных проверок требуется для состояния `unhealthy`:

```python id="f6q2vn"
retries: 3
```

Например:

```text id="r8m5kc"
Check 1 → ❌
Check 2 → ❌
Check 3 → ❌
          ↓
      unhealthy
```

Одна случайная ошибка не обязательно сразу означает `unhealthy`.

---

# 🔹 `start_period`

Даёт приложению время на первоначальный запуск:

```python id="c4x7mz"
start_period: 20s
```

Это особенно полезно для приложений, которые долго запускаются.

Например:

```text id="y5q8nv"
Container start
      ↓
20 секунд на инициализацию
      ↓
Healthcheck
```

---

# 🔹 Полный пример

```python id="u7m3qx"
services:
  app:
    build: .
    ports:
      - "8000:8000"
    healthcheck:
      test: ["CMD-SHELL", "curl -f http://localhost:8000/health || exit 1"]
      interval: 10s
      timeout: 5s
      retries: 3
      start_period: 20s
```

Схема:

```text
Container
    ↓
каждые 10 секунд
    ↓
GET /health
    ↓
┌───────────────┐
│ HTTP 200       │ → healthy
│ HTTP error     │ → failed check
└───────────────┘
```

---

# 🔹 Healthcheck для FastAPI

В приложении можно сделать endpoint:

```python id="k6v2pd"
from fastapi import FastAPI

app = FastAPI()


@app.get("/health")
def health():
    return {"status": "ok"}
```

Dockerfile должен содержать инструмент, которым выполняется проверка. Например, если используется `curl`, его необходимо установить в Image.

После этого:

```python id="s9m4wx"
healthcheck:
  test: ["CMD-SHELL", "curl -f http://localhost:8000/health || exit 1"]
  interval: 10s
  timeout: 5s
  retries: 3
```

---

# 🔹 Liveness vs Readiness

Это важное понятие для Backend/DevOps.

### Liveness

Отвечает на вопрос:

> **Процесс вообще жив?**

Например:

```text
GET /health/live
```

```json
{"status": "ok"}
```

### Readiness

Отвечает:

> **Приложение готово принимать запросы?**

Например, приложение может проверять доступность:

```text
PostgreSQL
Redis
внешних критичных зависимостей
```

Условно:

```text id="b3q7mx"
Liveness
   ↓
процесс жив

Readiness
   ↓
приложение готово работать
```

В Docker Compose чаще всего просто используют один Healthcheck, а в Kubernetes понятия `livenessProbe` и `readinessProbe` разделены явно.

---

# 🔹 Healthcheck и `depends_on`

Это особенно важно в Docker Compose.

Без проверки:

```python id="m8q2vx"
services:
  app:
    depends_on:
      - db
```

Это означает:

> Сначала запусти контейнер `db`, потом `app`.

Но **не означает**, что PostgreSQL уже готов принимать подключения.

Поэтому можно использовать:

```python id="x4n7kc"
services:
  app:
    depends_on:
      db:
        condition: service_healthy

  db:
    image: postgres:15
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user -d app"]
      interval: 5s
      timeout: 5s
      retries: 5
```

Получаем:

```text
db container started
        ↓
Healthcheck
        ↓
PostgreSQL ready
        ↓
healthy
        ↓
app starts
```

Это уже проверка **готовности сервиса**, а не просто порядка запуска контейнеров.

---

# 🔹 Healthcheck не перезапускает контейнер

Очень важный момент:

> **Healthcheck сам по себе не перезапускает контейнер.**

Например:

```text
Container: running
Healthcheck: unhealthy
```

Контейнер может продолжать работать.

За перезапуск отвечают другие механизмы, например:

```python id="r5k9wp"
restart: unless-stopped
```

Но `restart` и `healthcheck` — **разные механизмы**.

---

# 🔹 `restart` vs `healthcheck`

| Healthcheck                  | Restart                                       |
| ---------------------------- | --------------------------------------------- |
| Проверяет состояние          | Определяет политику перезапуска               |
| `healthy/unhealthy`          | Перезапуск контейнера                         |
| Не перезапускает сам по себе | Может перезапустить после завершения процесса |
| Контролирует health          | Контролирует lifecycle                        |

Например:

```text id="n6q3vx"
Application
    ↓
Healthcheck
    ↓
unhealthy
```

не означает автоматически:

```text
unhealthy
   ↓
restart
```

---

# 🔹 Как посмотреть состояние

```python id="c7m4qa"
docker ps
```

В колонке `STATUS` можно увидеть примерно:

```text
Up 2 minutes (healthy)
```

или:

```text
Up 2 minutes (unhealthy)
```

Подробности:

```python id="v8k2mx"
docker inspect app
```

В Compose:

```python id="p4q7nc"
docker compose ps
```

---

# 🔹 Что именно проверять

Хороший Healthcheck должен проверять **реальную готовность сервиса**, а не просто наличие процесса.

Например для PostgreSQL:

```python id="h3m8qx"
pg_isready
```

Для HTTP-сервиса:

```python id="a7k4wp"
curl -f http://localhost:8000/health
```

Для Redis:

```python id="z5q2nv"
redis-cli ping
```

---

# 🔹 Плохой Healthcheck

Например:

```python id="d8m3kc"
test: ["CMD", "echo", "ok"]
```

Команда всегда возвращает успех:

```text
exit 0
```

Docker будет считать контейнер:

```text
healthy
```

даже если само приложение не работает.

Поэтому Healthcheck должен проверять **существенную часть работоспособности сервиса**.

---

# 🔹 Важный нюанс: глубокая проверка зависимостей

Не всегда стоит делать endpoint `/health`, который проверяет абсолютно всё:

```text
FastAPI
 ├── PostgreSQL
 ├── Redis
 ├── Kafka
 ├── внешний API
 └── ещё 10 сервисов
```

Если одна второстепенная зависимость временно недоступна, приложение может стать `unhealthy`, хотя оно способно выполнять большинство операций.

Поэтому проверки проектируют в соответствии с тем, **что именно означает "готов" для конкретного сервиса**.

---

# 🔹 Healthcheck в архитектуре

Типичный Python Backend:

```text id="w2n8mf"
                 Docker Compose
                       │
        ┌──────────────┼──────────────┐
        ↓              ↓              ↓
     FastAPI       PostgreSQL       Redis
        │              │
        ↓              ↓
   /health         pg_isready
        │              │
        ↓              ↓
    healthy        healthy
```

Compose может использовать состояние `healthy` при организации запуска зависимых сервисов.

---

# 🔹 Частые вопросы на собеседовании

### Что такое Healthcheck?

> Механизм Docker для периодической проверки состояния контейнера с помощью команды.

### Какие состояния бывают?

> `starting`, `healthy`, `unhealthy`.

### Что определяет результат Healthcheck?

> В первую очередь exit code проверочной команды: `0` означает успех, ненулевой код — ошибку.

### Что делает `interval`?

> Определяет интервал между проверками.

### Что делает `timeout`?

> Максимальное время выполнения одной проверки.

### Что делает `retries`?

> Количество последовательных неудачных проверок перед переходом в `unhealthy`.

### Что делает `start_period`?

> Даёт приложению время на первоначальный запуск перед тем, как ошибки healthcheck начнут учитываться обычным образом.

### Запускает ли Healthcheck контейнер заново?

> Нет. Healthcheck только определяет состояние `healthy/unhealthy`; перезапуск — отдельный механизм.

### Чем Healthcheck отличается от `depends_on`?

> `depends_on` управляет зависимостями запуска, а Healthcheck проверяет состояние сервиса.

### Почему `depends_on` без Healthcheck недостаточно?

> Потому что запуск контейнера не гарантирует готовность приложения внутри него.

---

## 🔑 Главное

```text id="q8m4vx"
Docker Container
       │
       ↓
  Healthcheck
       │
       ├── exit 0
       │      ↓
       │   healthy
       │
       └── ошибки
              ↓
          unhealthy
```

### 🎤 Финальная формула

> **Healthcheck — это периодическая проверка работоспособности контейнера. Docker выполняет заданную команду и по её exit code определяет состояние `starting`, `healthy` или `unhealthy`. В Docker Compose Healthcheck особенно полезен вместе с `depends_on`, когда важно дождаться готовности сервиса, а не просто его запуска.**
