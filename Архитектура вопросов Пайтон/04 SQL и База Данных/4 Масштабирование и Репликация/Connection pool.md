# 🔌 Connection Pool

## 🎯 Формула для собеседования

> **Connection Pool — это пул заранее созданных или переиспользуемых соединений с БД. Приложение берёт свободное соединение из пула, выполняет запросы, а затем возвращает соединение обратно вместо создания нового TCP-соединения с БД. Это уменьшает latency и нагрузку на БД, а также позволяет ограничить количество одновременно используемых соединений.**

---

## 🎤 Суперкоротко

```text
Application
     │
     ▼
Connection Pool
 ┌───┬───┬───┬───┐
 │ C │ C │ C │ C │
 └───┴───┴───┴───┘
     │
     ▼
    DB
```

Вместо:

```text
Запрос
 ↓
создать connection
 ↓
SQL
 ↓
закрыть connection
```

делаем:

```text
Запрос
 ↓
взять connection из pool
 ↓
SQL
 ↓
вернуть connection в pool
```

---

# 1. 🤔 Зачем вообще нужен Connection Pool

Установление соединения с БД — не бесплатная операция.

При создании нового соединения могут происходить:

```text
TCP connection
      ↓
TLS handshake (если используется)
      ↓
аутентификация
      ↓
создание session
```

Если делать это для каждого запроса:

```text
Request 1 → connect → query → disconnect
Request 2 → connect → query → disconnect
Request 3 → connect → query → disconnect
```

это создаёт лишнюю нагрузку и увеличивает latency.

Connection Pool позволяет переиспользовать соединения:

```text
Request 1 ──┐
Request 2 ──┼──→ Connection Pool → DB
Request 3 ──┘
```

---

# 2. 🔄 Как работает Connection Pool

Допустим, pool настроен на 5 соединений:

```text
Pool

┌─────────────┐
│ Connection 1│
│ Connection 2│
│ Connection 3│
│ Connection 4│
│ Connection 5│
└─────────────┘
```

Приходит запрос:

```text
Request
   ↓
получить свободное соединение
   ↓
execute SQL
   ↓
вернуть connection
```

Само соединение при этом не обязательно закрывается.

---

# 3. 📦 Что происходит при нехватке соединений

Допустим:

```text
pool_size = 5
```

И одновременно пришло 10 запросов, которым нужна БД.

Первые 5 получают соединения:

```text
Request 1 → Connection 1
Request 2 → Connection 2
Request 3 → Connection 3
Request 4 → Connection 4
Request 5 → Connection 5
```

Остальные ждут:

```text
Request 6 → WAIT
Request 7 → WAIT
Request 8 → WAIT
...
```

Когда одно соединение освобождается:

```text
Connection 2
     ↓
вернулось в pool
     ↓
Request 6 получает его
```

Таким образом pool работает ещё и как **ограничитель количества одновременных соединений с БД**.

---

# 4. 🛡️ Почему ограничение соединений важно

Без ограничения приложение может попытаться создать огромное количество соединений:

```text
1000 requests
      ↓
1000 DB connections
      ↓
💥 PostgreSQL перегружен
```

У БД есть ограничение на количество соединений:

```text
max_connections
```

Поэтому Connection Pool помогает контролировать:

```text
Application
     ↓
  Pool limit
     ↓
 PostgreSQL
```

---

# 5. ⚙️ Основные параметры Pool

Конкретные названия зависят от библиотеки, но обычно есть такие параметры.

### `pool_size`

Количество постоянных соединений в пуле.

Например:

```text
pool_size = 10
```

---

### `max_overflow`

Количество дополнительных временных соединений сверх основного размера пула.

Например:

```text
pool_size = 10
max_overflow = 5
```

Максимально одновременно:

```text
10 + 5 = 15
```

Конкретная семантика зависит от реализации pool.

---

### `timeout`

Сколько времени ждать свободное соединение.

Например:

```text
pool exhausted
     ↓
wait
     ↓
timeout
     ↓
error
```

---

### `connection lifetime`

Максимальное время жизни соединения.

После достижения лимита соединение может быть закрыто и заменено новым.

---

### `idle timeout`

Время, после которого простаивающее соединение может быть закрыто.

---

# 6. 🔥 Pool не создаёт бесконечные соединения

Это одна из главных задач:

```text
Application
      ↓
Connection Pool
      ↓
ограниченное количество connections
      ↓
Database
```

Например:

```text
1000 HTTP requests
        ↓
pool = 20
        ↓
не более ~20 одновременно используемых
соединений из этого pool
```

Это не означает, что 1000 запросов не могут обрабатываться одновременно.

Просто операции, которым нужна БД, будут ждать свободное соединение.

---

# 7. 🌐 Connection Pool в веб-приложении

Типичная архитектура:

```text
                 HTTP Requests
                 ↓ ↓ ↓ ↓ ↓
              ┌───────────┐
              │ FastAPI   │
              │ / Django  │
              └─────┬─────┘
                    ↓
             Connection Pool
             ┌──┬──┬──┬──┐
             │C1│C2│C3│C4│
             └──┴──┴──┴──┘
                    ↓
               PostgreSQL
```

Каждый запрос приложения получает connection на время работы с БД.

---

# 8. 🧩 Connection Pool и транзакция

Очень важно правильно возвращать соединение в pool.

Например:

```python
connection = pool.acquire()

try:
    transaction_begin()
    execute_query()
    transaction_commit()
finally:
    pool.release(connection)
```

Если произошла ошибка:

```python
connection = pool.acquire()

try:
    execute_query()
except Exception:
    transaction_rollback()
    raise
finally:
    pool.release(connection)
```

Главная идея:

> **Соединение нужно вернуть в pool после завершения работы и привести его в корректное состояние.**

---

# 9. ⚠️ Что будет, если connection не вернуть

Допустим:

```text
pool_size = 10
```

Приложение получает соединение:

```text
Request
 ↓
Connection 1
```

Но забывает вернуть его.

Через некоторое время:

```text
Connection 1 → leaked
Connection 2 → leaked
Connection 3 → leaked
...
```

Пул постепенно исчерпается:

```text
Pool exhausted
     ↓
новые запросы ждут
     ↓
timeout
     ↓
ошибки
```

Это называется **connection leak**.

---

# 10. 🧠 Pooling ≠ постоянная транзакция

Connection Pool хранит соединения, но это не означает, что каждое соединение постоянно находится внутри одной транзакции.

Обычно жизненный цикл выглядит примерно так:

```text
connection
    ↓
acquire
    ↓
transaction
    ↓
COMMIT / ROLLBACK
    ↓
reset connection state
    ↓
release
    ↓
pool
```

Это важно, потому что состояние предыдущего запроса не должно случайно влиять на следующий.

---

# 11. 🔄 Connection Pool и PostgreSQL

PostgreSQL использует модель:

```text
1 connection
≈
1 backend process
```

Поэтому огромное количество соединений может быть дорогим для PostgreSQL.

Connection Pool позволяет держать контролируемое количество соединений:

```text
Web application
      ↓
Pool: 20 connections
      ↓
PostgreSQL
```

а не создавать отдельное соединение для каждого HTTP-запроса.

---

# 12. 🏗️ Connection Pool при нескольких workers

Очень важный момент для backend-разработчика.

Допустим:

```text
Gunicorn
 ├── Worker 1 → pool 10
 ├── Worker 2 → pool 10
 ├── Worker 3 → pool 10
 └── Worker 4 → pool 10
```

Это потенциально:

```text
4 × 10 = 40 connections
```

То есть `pool_size=10` **не обязательно означает 10 соединений на всё приложение**.

Если pool создаётся отдельно внутри каждого процесса, лимит применяется к каждому pool.

Поэтому при настройке нужно учитывать:

```text
workers
×
pool size
+
overflow
```

и сопоставлять результат с возможностями PostgreSQL.

---

# 13. 🔀 Connection Pool и PgBouncer

**PgBouncer** — отдельный connection pooler для PostgreSQL.

Архитектура:

```text
Application
     ↓
Connection Pool
     ↓
PgBouncer
     ↓
PostgreSQL
```

PgBouncer позволяет централизованно управлять соединениями между большим количеством клиентов и PostgreSQL.

Основные режимы pooling:

```text
Session pooling
Transaction pooling
Statement pooling
```

Для backend-разработчика важно понимать концепцию:

> Pool может находиться непосредственно в приложении, а PgBouncer — отдельным инфраструктурным компонентом перед PostgreSQL.

---

# 14. 📊 Application Pool vs PgBouncer

|                                               | Application Pool | PgBouncer        |
| --------------------------------------------- | ---------------- | ---------------- |
| Где находится                                 | В приложении     | Отдельный сервис |
| Управляет соединениями                        | Да               | Да               |
| Общий для нескольких приложений               | Обычно нет       | Да               |
| Может уменьшить число реальных DB connections | Да               | Да               |
| Полезен при множестве workers                 | Да               | Особенно полезен |
| Требует отдельной инфраструктуры              | Нет              | Да               |

Их также можно использовать вместе, но тогда нужно внимательно рассчитывать лимиты.

---

# 15. ⚠️ Слишком большой Pool — тоже проблема

Кажется логичным:

```text
чем больше pool → тем быстрее
```

Но это неверно.

Например:

```text
pool_size = 500
```

может создать огромную нагрузку на PostgreSQL.

Если база реально эффективно обрабатывает только ограниченное число одновременных операций, увеличение connections может привести к:

```text
больше конкуренции
      ↓
больше CPU / RAM
      ↓
больше context switching
      ↓
больше ожиданий
      ↓
меньше производительность
```

Поэтому размер pool подбирают по:

* нагрузке;
* количеству workers;
* CPU/RAM БД;
* характеру запросов;
* latency;
* `max_connections`;
* метрикам.

---

# 16. 🔐 Connection Pool и состояние соединения

Connection может сохранять состояние:

```text
SET
temporary tables
session variables
prepared statements
transaction state
```

Поэтому перед возвратом соединения в pool важно, чтобы оно не оставалось в неожиданном состоянии.

Хорошая pool-библиотека обычно выполняет необходимые проверки/reset, но конкретное поведение зависит от реализации.

Особенно важно:

```text
COMMIT
или
ROLLBACK
```

перед возвратом соединения после работы с транзакцией.

---

# 17. 🚨 Connection Pool и ошибки

Если connection оказался сломанным:

```text
network error
connection reset
database restart
```

его нельзя просто вернуть в pool как здоровое соединение.

Обычно pool должен:

```text
обнаружить неисправность
       ↓
закрыть connection
       ↓
создать новый при необходимости
```

Именно поэтому pool обычно имеет механизмы:

```text
health check
connection recycling
pre-ping
timeout
```

Названия зависят от библиотеки.

---

# 18. 🎯 Связь с производительностью

Connection Pool решает сразу несколько задач:

```text
Connection Pool
│
├── уменьшает latency
│
├── переиспользует connections
│
├── уменьшает стоимость создания connections
│
├── ограничивает concurrency к БД
│
└── защищает БД от слишком большого
    количества соединений
```

Но:

> Connection Pool не ускоряет сам SQL-запрос.

Если запрос выполняется 5 секунд:

```text
Pool
 ↓
SQL
 ↓
5 секунд
```

pool не сделает его автоматически быстрым.

Для этого нужны:

```text
indexes
EXPLAIN
query optimization
caching
schema optimization
```

---

# 19. 🧠 Connection Pool и N+1

Pool **не исправляет N+1 проблему**.

Например:

```text
100 пользователей
      ↓
101 SQL-запрос
```

Даже если используется Connection Pool:

```text
101 запрос
      ↓
Connection Pool
      ↓
Database
```

Количество запросов осталось 101.

Pool только эффективно управляет соединениями.

N+1 решается через:

```text
JOIN Load
Selection Load
batch loading
prefetching
```

---

# 20. 🧩 Connection Pool в общей архитектуре

```text
                    Clients
                       │
                       ▼
                    Nginx
                       │
                       ▼
                  FastAPI
                       │
                ┌──────┴──────┐
                ▼             ▼
             Redis       Connection Pool
                              │
                              ▼
                         PostgreSQL
```

Pool находится между application code и БД и управляет жизненным циклом соединений.

---

# 🎯 Главное

```text
Connection Pool
│
├── набор переиспользуемых DB connections
│
├── acquire
│     ↓
│   получить свободное соединение
│
├── execute
│     ↓
│   выполнить SQL
│
├── release
│     ↓
│   вернуть соединение
│
├── уменьшает стоимость создания connections
│
├── ограничивает количество connections
│
└── защищает БД от чрезмерной конкуренции
```

### Ключевая схема

```text
HTTP Request
     ↓
Connection Pool
     ↓
acquire connection
     ↓
SQL / Transaction
     ↓
COMMIT / ROLLBACK
     ↓
release connection
     ↓
Connection Pool
```

### Важное различие

```text
Connection Pool
→ управляет соединениями

N+1
→ проблема количества SQL-запросов

Индекс
→ ускоряет поиск данных

PgBouncer
→ отдельный pooler между приложением
  и PostgreSQL
```
