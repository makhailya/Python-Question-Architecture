# ⚡ Redis

## 🎤 Короткий ответ

**Redis** — это высокопроизводительное in-memory хранилище данных типа **[[Key-Value]]**. Основные данные хранятся в оперативной памяти, поэтому Redis очень быстрый.

В Backend его часто используют для:

* кэширования;
* хранения сессий;
* rate limiting;
* очередей и временных данных;
* брокера сообщений;
* распределённых блокировок;
* хранения счётчиков.

---

## 🎯 Формула для собеседования

> **Redis = in-memory key-value storage → очень быстрый доступ к данным → cache, sessions, counters, queues и другие задачи.**

```python id="r8m3kp"
Application
     ↓
   Redis
     ↓
RAM
```

---

# 🧠 Что такое Redis

Redis расшифровывается как **Remote Dictionary Server**.

Классическая модель:

```python id="v4x7qs"
key → value
```

Например:

```python id="n3p8dw"
user:1001 → "Ilya"
```

Можно представить Redis как очень быстрый словарь, доступный по сети.

---

# ⚡ Почему Redis быстрый

Основная причина:

> **Большинство операций выполняется непосредственно с данными в оперативной памяти.**

Условно:

```text id="z6q2mr"
RAM
 ↓
Redis
 ↓
быстрый доступ
```

По сравнению с дисковой БД путь до данных обычно короче.

Но Redis всё равно может использовать диск для **персистентности** — это не означает, что Redis является исключительно RAM-only системой.

---

# 🧩 Типы данных Redis

Redis поддерживает не только обычные строки.

Основные структуры:

| Тип             | Пример использования                        |
| --------------- | ------------------------------------------- |
| **String**      | cache, counters                             |
| **Hash**        | объект/набор полей                          |
| **List**        | списки, очереди                             |
| **Set**         | уникальные значения                         |
| **Sorted Set**  | рейтинги                                    |
| **Stream**      | поток сообщений                             |
| **Bitmap**      | битовые состояния                           |
| **HyperLogLog** | приблизительный подсчёт уникальных значений |

---

# 🔤 String

Самый простой тип:

```python id="q2w6hm"
SET user:1 "Ilya"
GET user:1
```

Результат:

```text id="j7k3nf"
Ilya
```

Можно использовать как счётчик:

```python id="b9x4sp"
INCR page_views
```

---

# 🗂️ Hash

Hash удобно использовать для объекта:

```python id="k8m2vc"
HSET user:1 name "Ilya" age 31
```

Получить поле:

```python id="s6q9tr"
HGET user:1 name
```

Получить всё:

```python id="d4n7px"
HGETALL user:1
```

Концептуально:

```text id="w3r5ka"
user:1
 ├── name → Ilya
 └── age  → 31
```

---

# 📋 List

List — упорядоченная коллекция.

Например:

```python id="p5v8mx"
LPUSH queue task1
LPUSH queue task2
```

Получение:

```python id="e2q6wd"
RPOP queue
```

Может использоваться для простых очередей.

Но для сложных production-сценариев очередей выбор Redis Lists зависит от требований; также существуют Redis Streams и специализированные брокеры.

---

# 🎯 Set

Set хранит **уникальные значения**.

```python id="j4m8sz"
SADD tags python
SADD tags redis
SADD tags python
```

`python` второй раз не добавится.

Получить:

```python id="x7q2nc"
SMEMBERS tags
```

Подходит для:

* уникальных идентификаторов;
* множества тегов;
* проверки принадлежности.

---

# 🏆 Sorted Set

Sorted Set хранит элементы с числовым score.

Например:

```python id="h5k9rq"
ZADD leaderboard 100 user1
ZADD leaderboard 250 user2
ZADD leaderboard 180 user3
```

Можно использовать для:

* рейтингов;
* leaderboard;
* приоритетов;
* задач с числовым score.

---

# ⏱️ TTL

Одна из важнейших возможностей Redis — автоматическое истечение ключей.

Например:

```python id="c8m3vz"
SET verification_code "123456" EX 300
```

Ключ будет автоматически удалён после истечения TTL.

Посмотреть TTL:

```python id="r2w7kp"
TTL verification_code
```

TTL особенно полезен для:

* кэша;
* OTP-кодов;
* временных токенов;
* rate limiting;
* временных блокировок.

---

# 🗑️ DELETE

Удалить ключ:

```python id="f6q2mz"
DEL user:1
```

Проверить существование:

```python id="a8v4nx"
EXISTS user:1
```

---

# 💾 Redis как Cache

Одно из самых распространённых применений.

Без Redis:

```text id="u2r7km"
Client
  ↓
FastAPI
  ↓
PostgreSQL
  ↓
Response
```

Если запрос повторяется:

```text id="q5m8dx"
Client
  ↓
FastAPI
  ↓
PostgreSQL
```

База снова выполняет запрос.

С Redis:

```text id="v9k3ps"
Client
  ↓
FastAPI
  ↓
Redis
  ↓
Cache Hit
  ↓
Response
```

Если данных нет:

```text id="n6w2yc"
FastAPI
   ↓
Redis
   ↓
Cache Miss
   ↓
PostgreSQL
   ↓
Redis SET
   ↓
Response
```

---

# 🎯 Cache Hit / Cache Miss

### Cache Hit

Данные найдены:

```text id="j4v8sq"
Request
  ↓
Redis
  ↓
✅ Data found
```

### Cache Miss

Данных нет:

```text id="x3k7mp"
Request
  ↓
Redis
  ↓
❌ Not found
  ↓
PostgreSQL
```

---

# 🔄 Cache-Aside

Один из распространённых вариантов работы с кэшем:

```text id="m9q2fz"
Request
   ↓
Redis
   │
   ├── HIT → return
   │
   └── MISS
         ↓
      PostgreSQL
         ↓
      Redis SET
         ↓
       return
```

Например:

```python id="n8c5rx"
user = redis.get(f"user:{user_id}")

if user is None:
    user = database.get_user(user_id)
    redis.set(f"user:{user_id}", user, ex=300)
```

---

# ⚠️ Проблема устаревшего кэша

Предположим:

```text id="q7p3mx"
PostgreSQL:
name = "Ilya"

Redis:
name = "Ilya"
```

Изменили БД:

```text id="v6n2kc"
PostgreSQL:
name = "Ivan"
```

Но Redis всё ещё содержит:

```text id="w4m8sq"
Redis:
name = "Ilya"
```

Получается **stale data**.

Поэтому при изменении данных нужно продумать:

* удаление ключа;
* обновление кэша;
* TTL;
* стратегию invalidation.

---

# 🧹 Cache Invalidation

При изменении данных:

```python id="e7r3kp"
UPDATE database
      ↓
DEL cache:user:1
```

При следующем запросе:

```text id="p5m8vx"
Redis MISS
   ↓
DB
   ↓
Redis SET
```

Это один из типичных подходов.

Фраза:

> **Cache invalidation is one of the hard problems in computer science**

часто используется именно из-за сложности поддержания согласованности кэша.

---

# 🔐 Redis для сессий

Redis удобно использовать для хранения session data:

```text id="d8q4ny"
session:abc123
       ↓
{
  user_id: 42,
  expires: ...
}
```

Поскольку данные могут иметь TTL:

```python id="z6m2wp"
SET session:abc123 data EX 3600
```

сессия автоматически исчезнет через час.

---

# 🚦 Redis для Rate Limiting

Redis хорошо подходит для счётчиков.

Например:

```text id="k3v7ps"
IP: 192.0.2.10
Requests: 47
Window: 60 sec
```

Можно использовать:

```python id="n4x8qm"
INCR rate:192.0.2.10
EXPIRE rate:192.0.2.10 60
```

И ограничивать:

```text id="c7m2zr"
≤ 100 requests/min → OK
> 100 requests/min → 429
```

На практике точная реализация rate limiting может использовать Lua scripts, sorted sets или другие алгоритмы/структуры.

---

# 🔢 Redis для счётчиков

Например:

```python id="q9w4mx"
INCR likes:post:100
```

Получаем:

```text id="u5k8np"
likes:post:100 = 124
```

Атомарные операции Redis удобны для подобных сценариев.

---

# 🔒 Distributed Lock

Redis можно использовать для распределённой блокировки.

Например, есть несколько экземпляров приложения:

```text id="v3n7qs"
API 1 ─┐
API 2 ─┼→ Redis Lock
API 3 ─┘
```

Нужно, чтобы только один экземпляр выполнял критическую операцию.

Концептуально:

```text id="r8m2kc"
Acquire Lock
     ↓
Execute
     ↓
Release Lock
```

Для production-распределённых блокировок нужно учитывать TTL, отказ клиентов и корректность алгоритма; простого `SETNX` без дополнительных гарантий недостаточно для всех сценариев.

---

# 📨 Redis как брокер

Redis может использоваться для передачи сообщений.

Например:

```text id="k5q9vx"
Producer
   ↓
Redis
   ↓
Consumer
```

Для более сложных потоков Redis предоставляет **Streams**.

Но Redis не является полной заменой Kafka или RabbitMQ во всех сценариях.

---

# 🌊 Redis Streams

Streams позволяют хранить последовательность сообщений:

```text id="x7m3qp"
event1
event2
event3
event4
```

Есть:

* message ID;
* consumer groups;
* подтверждение обработки;
* чтение с определённой позиции.

Это делает Streams более подходящими для некоторых event-processing сценариев, чем обычные Lists.

---

# 💾 Persistence

Хотя Redis в основном работает с RAM, данные можно сохранять на диск.

Основные механизмы:

### RDB

Периодические снимки состояния.

```text id="c4n8yp"
RAM
 ↓
Snapshot
 ↓
Disk
```

Плюсы:

* компактный файл;
* удобен для backup/snapshot-подхода;
* восстановление обычно проще.

Минус:

* можно потерять изменения между снимками.

### AOF

**Append Only File** — журнал операций изменения.

```text id="s2w6mq"
SET a 10
INCR a
DEL b
...
```

Команды записываются в журнал, который можно использовать для восстановления состояния.

AOF обычно даёт более детальный журнал изменений, но имеет свои накладные расходы.

---

# 🆚 RDB vs AOF

| RDB                                           | AOF                                         |
| --------------------------------------------- | ------------------------------------------- |
| Snapshot                                      | Журнал операций                             |
| Периодическая запись                          | Более непрерывная фиксация                  |
| Компактнее                                    | Обычно больше                               |
| Возможна потеря изменений между snapshots     | Меньше окно потери при подходящей настройке |
| Быстрое восстановление в подходящих сценариях | Более подробное восстановление              |

Redis также позволяет использовать оба механизма.

---

# 🧠 Redis vs PostgreSQL

| Redis                                                            | PostgreSQL                           |
| ---------------------------------------------------------------- | ------------------------------------ |
| In-memory ориентирован                                           | Дисковая реляционная БД              |
| Key-value + структуры данных                                     | Реляционная модель                   |
| Очень быстрые операции                                           | Богатый SQL                          |
| Cache / sessions / counters                                      | Основные бизнес-данные               |
| TTL                                                              | Полноценные транзакции и ограничения |
| Нет необходимости использовать для всей persistent business data | Основное durable storage             |

Упрощённая архитектура:

```text id="k7m3vx"
FastAPI
   │
   ├── Redis
   │     └── Cache
   │
   └── PostgreSQL
         └── Source of Truth
```

В таком сценарии PostgreSQL остаётся **источником истины**, а Redis — быстрым вспомогательным хранилищем.

---

# 🆚 Redis vs Memcached

| Redis                     | Memcached                                     |
| ------------------------- | --------------------------------------------- |
| Богатые структуры данных  | В основном key-value                          |
| Persistence               | Обычно используется как cache без persistence |
| Lists/Sets/Hashes/Streams | Более простой cache                           |
| TTL                       | TTL                                           |
| Pub/Sub, Streams и др.    | Более минималистичный                         |
| Более широкий набор задач | Преимущественно caching                       |

---

# 🐳 Redis в Docker Compose

Для Python Backend:

```python id="f4k8mz"
services:
  api:
    build: .
    depends_on:
      - redis

  redis:
    image: redis:7
```

API обращается к Redis по имени сервиса:

```python id="j8q3vx"
redis://redis:6379
```

Не:

```python id="n2w7kp"
redis://localhost:6379
```

Потому что внутри контейнера `localhost` указывает на сам контейнер API.

---

# 🐍 Redis в Python

Популярный клиент:

```python id="c5m9rq"
pip install redis
```

Пример:

```python id="w7k2mx"
import redis


client = redis.Redis(
    host="localhost",
    port=6379,
    decode_responses=True,
)

client.set("name", "Ilya", ex=300)

name = client.get("name")

print(name)
```

---

# ⚡ Атомарность операций

Некоторые операции Redis атомарны на уровне команды.

Например:

```python id="p4x8nz"
INCR counter
```

Это удобнее для счётчиков, чем схема:

```text id="m6q2vr"
GET
 ↓
Python + 1
 ↓
SET
```

потому что между `GET` и `SET` возможна гонка.

---

# ⚠️ Redis не решает всё

Redis не должен автоматически становиться заменой PostgreSQL.

Например, сложные данные:

```text id="y8m3qk"
Orders
Users
Payments
Products
```

обычно требуют полноценной БД с:

* транзакциями;
* ограничениями;
* связ
