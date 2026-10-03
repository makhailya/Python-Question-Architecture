# 🔌 Пул соединений (Connection Pooling)

## 🎯 Формула для собеседования

> **Connection Pooling — это механизм переиспользования соединений с БД. Вместо создания нового соединения для каждого запроса приложение берёт свободное соединение из пула, выполняет работу и возвращает его обратно. Это уменьшает задержки и нагрузку на БД, а также позволяет ограничить количество одновременно открытых соединений.**

---

## 🎤 Суперкоротко

```text id="q1s8xk"
Без Pool:

Request
  ↓
CREATE CONNECTION
  ↓
SQL
  ↓
CLOSE CONNECTION

Каждый запрос → новое соединение
```

С Pool:

```text id="g9a2m4"
              Connection Pool
             ┌───┬───┬───┬───┐
Request ────→│ C │ C │ C │ C │
             └───┴───┴───┴───┘
                  ↓
              PostgreSQL
```

```text id="k2m4d7"
acquire
   ↓
использовать connection
   ↓
release
   ↓
connection обратно в pool
```

---

# 1. 🤔 Зачем нужен Connection Pooling

Создание соединения с PostgreSQL требует ресурсов.

Упрощённо:

```text id="w7m0q3"
Application
    ↓
TCP connection
    ↓
аутентификация
    ↓
PostgreSQL session
```

Если создавать connection для каждого запроса:

```text id="v8x2n1"
Request 1 → connect → SQL → disconnect
Request 2 → connect → SQL → disconnect
Request 3 → connect → SQL → disconnect
```

возникают лишние:

* сетевые операции;
* аутентификация;
* создание серверной session;
* использование CPU и RAM;
* задержки.

Pooling позволяет **переиспользовать уже существующие соединения**.

---

# 2. 🔄 Как работает Pool

Допустим:

```text id="r4k6p8"
pool_size = 5
```

Пул содержит:

```text id="t5y7u2"
Connection 1
Connection 2
Connection 3
Connection 4
Connection 5
```

Приходит запрос:

```text id="n6c3v9"
Request
   ↓
acquire()
   ↓
Connection 2
   ↓
SQL
   ↓
release()
   ↓
Pool
```

Соединение **не обязательно закрывается** после запроса.

Оно возвращается в пул и может быть использовано следующим запросом.

---

# 3. 📦 Что такое Pool

**Pool** — это управляемый набор соединений.

Например:

```text id="e4h8s2"
Connection Pool
│
├── C1 → idle
├── C2 → busy
├── C3 → idle
├── C4 → busy
└── C5 → idle
```

Состояния условно:

```text id="y6k2p1"
idle
→ свободно

busy
→ используется
```

Приложение не создаёт connection самостоятельно для каждой операции, а получает его через pool.

---

# 4. 🛡️ Pool ограничивает нагрузку на БД

Представим:

```text id="p3v8n6"
1000 HTTP requests
```

Без ограничения приложение потенциально может попытаться открыть огромное количество соединений.

С pool:

```text id="m2q7x4"
1000 requests
      ↓
Connection Pool
      ↓
20 connections
      ↓
PostgreSQL
```

Только ограниченное количество запросов одновременно работает с БД через эти connections.

Остальные могут ждать освобождения connection.

---

# 5. ⏳ Что происходит, если все connections заняты

Допустим:

```text id="a8f4c2"
pool_size = 3
```

Все три заняты:

```text id="z3m9w7"
C1 → busy
C2 → busy
C3 → busy
```

Приходит четвёртый запрос:

```text id="s5n2k8"
Request 4
    ↓
Pool
    ↓
нет свободного connection
    ↓
WAIT
```

Когда один connection освобождается:

```text id="u7c4p1"
C2
 ↓
release
 ↓
Request 4 получает C2
```

Если свободного connection слишком долго нет, может произойти:

```text id="r9d3m5"
pool timeout
```

---

# 6. ⚙️ Основные параметры Pool

Названия зависят от конкретной библиотеки, но концепции обычно одинаковые.

### `pool_size`

Количество соединений основного пула.

```text id="f2k7v4"
pool_size = 10
```

→ 10 соединений в пуле.

---

### `max_overflow`

В некоторых реализациях позволяет временно создать дополнительные соединения сверх основного размера.

Например:

```text id="q8m3s6"
pool_size = 10
max_overflow = 5
```

Потенциальный максимум:

```text id="j4p9x2"
10 + 5 = 15
```

Конкретная семантика зависит от pool-библиотеки.

---

### `timeout`

Сколько ждать свободного connection:

```text id="c6v2n8"
Pool exhausted
      ↓
wait
      ↓
timeout
```

---

### `idle timeout`

Сколько времени неиспользуемое соединение может оставаться открытым.

---

### `connection lifetime`

Максимальное время жизни connection.

После достижения лимита соединение может быть заменено новым.

---

# 7. 🔄 Жизненный цикл соединения

Типичный сценарий:

```text id="m8r2q5"
Pool
 ↓
acquire()
 ↓
Connection
 ↓
BEGIN
 ↓
SQL
 ↓
COMMIT / ROLLBACK
 ↓
release()
 ↓
Pool
```

Ключевой момент:

> `release()` возвращает connection в pool, а не обязательно закрывает его физически.

---

# 8. ⚠️ Connection Leak

Если приложение получило connection и не вернуло его:

```text id="n4x7c1"
acquire()
   ↓
connection
   ↓
❌ release() не вызван
```

это **connection leak**.

Если таких случаев много:

```text id="w5p9k3"
C1 → leaked
C2 → leaked
C3 → leaked
...
```

пул постепенно исчерпывается:

```text id="q2m8v6"
Pool exhausted
      ↓
новые запросы ждут
      ↓
timeout
```

Поэтому connection нужно возвращать в pool даже при исключении.

---

# 9. 🧯 Connection Pool и исключения

Надёжный подход:

```python id="k8r4m2"
connection = pool.acquire()

try:
    execute_query()
    commit()
except Exception:
    rollback()
    raise
finally:
    pool.release(connection)
```

`finally` гарантирует попытку вернуть connection в pool.

В реальных библиотеках обычно используются context managers, которые делают это безопаснее.

---

# 10. 🔐 Connection Pool и транзакции

Нельзя оставить connection в pool с незавершённой транзакцией.

Плохой сценарий:

```text id="d7m2p9"
Request 1
   ↓
BEGIN
   ↓
UPDATE
   ↓
❌ connection возвращён в pool
```

Следующий запрос может получить connection с неожиданным состоянием.

Правильная логика:

```text id="u3k8v5"
BEGIN
 ↓
SQL
 ↓
COMMIT / ROLLBACK
 ↓
reset state
 ↓
release
```

---

# 11. 🌐 Connection Pool в FastAPI / Django

Типичная архитектура:

```text id="x9f3m7"
HTTP Requests
      ↓
FastAPI / Django
      ↓
Connection Pool
      ↓
PostgreSQL
```

Например, ORM или драйвер может самостоятельно управлять пулом соединений.

Разработчику важно понимать:

```text id="r4v8c2"
Request
   ↓
get connection
   ↓
query
   ↓
return connection
```

---

# 12. 👷 Pool и Workers

Очень важный момент.

Допустим, приложение имеет:

```text id="z6m2p8"
4 workers
```

и каждый worker имеет:

```text id="c3v7n5"
pool_size = 10
```

Если каждый worker создаёт собственный pool:

```text id="h8q4s1"
4 × 10 = 40 connections
```

То есть `pool_size=10` не обязательно означает **10 соединений на всё приложение**.

Нужно учитывать архитектуру процессов:

```text id="w2n9k6"
workers
×
pool_size
+
overflow
```

И сравнивать это с возможностями PostgreSQL.

---

# 13. 🧠 Pooling и PostgreSQL

PostgreSQL использует отдельный серверный процесс для клиентского подключения.

Поэтому большое количество connections может потреблять существенные ресурсы.

Pooling позволяет:

```text id="p7c3m8"
много application requests
        ↓
ограниченное количество connections
        ↓
PostgreSQL
```

Это особенно важно для приложений с большим количеством concurrent requests.

---

# 14. 🔌 Application Pool и PgBouncer

Не следует полностью отождествлять эти понятия.

### Application-level pool

Пул находится внутри приложения:

```text id="m4x8q2"
FastAPI
   ↓
SQLAlchemy Pool
   ↓
PostgreSQL
```

### PgBouncer

Отдельный сервис:

```text id="v6k3p9"
FastAPI
   ↓
PgBouncer
   ↓
PostgreSQL
```

### Вместе

```text id="a2r7m5"
FastAPI
   ↓
Application Pool
   ↓
PgBouncer
   ↓
PostgreSQL
```

В таком случае нужно правильно настроить оба уровня.

---

# 15. 📊 Connection Pool vs PgBouncer

|                                    | Connection Pool          | PgBouncer                    |
| ---------------------------------- | ------------------------ | ---------------------------- |
| Что это                            | Механизм pooling         | PostgreSQL connection pooler |
| Где находится                      | Обычно внутри приложения | Отдельный сервис             |
| Переиспользует connections         | ✅                        | ✅                            |
| Может быть общим для приложений    | Обычно нет               | ✅                            |
| Управляет connections к PostgreSQL | ✅                        | ✅                            |
| Session/Transaction pooling        | Зависит от реализации    | ✅                            |

---

# 16. ⚠️ Слишком большой Pool

Большой pool не означает автоматически высокую производительность.

Например:

```text id="p5x8m2"
pool_size = 500
```

может привести к:

```text id="v7n3c9"
слишком много connections
        ↓
больше конкуренции
        ↓
больше RAM / CPU
        ↓
больше переключений и ожиданий
        ↓
нагрузка на PostgreSQL
```

Поэтому размер пула подбирают на основе:

* количества workers;
* CPU и RAM PostgreSQL;
* характера запросов;
* concurrency;
* latency;
* `max_connections`;
* метрик реальной нагрузки.

---

# 17. 🧩 Pool не исправляет N+1

Важно не путать две разные проблемы.

### Connection Pool

Решает:

```text id="q3m7v1"
слишком много
созданий/соединений с БД
```

### N+1

Решает:

```text id="k8p4s2"
слишком много SQL-запросов
```

Например:

```text id="m6v9x3"
100 пользователей

N+1:
101 SQL-запрос
```

Connection Pool:

```text id="r2c7n5"
101 запрос
      ↓
Pool
      ↓
PostgreSQL
```

Запросов всё равно 101.

Для N+1 нужны:

```text id="b4k8m1"
JOIN Load
Selection Load
Batch Loading
Prefetching
```

---

# 18. 📈 Connection Pool и производительность

Pooling уменьшает:

```text id="z5n2c8"
connection setup overhead
```

и помогает контролировать:

```text id="q7m4v3"
DB concurrency
```

Но он не ускоряет сам SQL:

```text id="x8p3k6"
SELECT
   ↓
5 секунд
```

останется примерно таким же запросом.

Для ускорения SQL нужны другие инструменты:

```text id="m9c4v7"
EXPLAIN
Indexes
Query optimization
Caching
Schema optimization
```

---

# 19. 🏗️ Общая схема

```text id="k4r8m2"
                  HTTP Requests
                 /      |      \
                ↓       ↓       ↓
             FastAPI / Django
                    │
                    ▼
             Connection Pool
          ┌─────────┼─────────┐
          │         │         │
         C1        C2        C3
          │         │         │
          └─────────┼─────────┘
                    ▼
               PostgreSQL
```

---

# 🎯 Главное

```text id="v8m3q6"
Connection Pooling
│
├── переиспользование DB connections
│
├── acquire()
│     ↓
│   получить connection
│
├── выполнить SQL / transaction
│
├── commit / rollback
│
├── release()
│     ↓
│   вернуть connection
│
├── уменьшает стоимость создания connections
│
├── ограничивает concurrency к БД
│
└── защищает PostgreSQL
    от чрезмерного количества connections
```

### Формула

> **Connection Pooling — механизм переиспользования соединений с БД. Приложение берёт свободное соединение из пула, выполняет работу и возвращает его обратно. Это уменьшает overhead создания соединений и позволяет контролировать количество одновременно используемых connections.**

```text id="j6q2m9"
Pool
→ acquire
→ use
→ commit / rollback
→ release
→ reuse
```
