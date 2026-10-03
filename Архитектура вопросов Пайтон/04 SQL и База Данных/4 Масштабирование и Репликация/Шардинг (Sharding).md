# 🧩 Шардинг (Sharding)

## 🎯 Ответ на собеседовании

**Шардинг** — это горизонтальное распределение данных между несколькими независимыми узлами — **shards**.

Каждый shard хранит только **часть общего набора данных**.

Например:

```python id="3f8k2m"
Shard 1 → users 1–100000
Shard 2 → users 100001–200000
Shard 3 → users 200001–300000
```

Главная цель — выйти за ограничения одного сервера и масштабировать **объём данных и нагрузку горизонтально**.

---

## 🎤 Суперкоротко

> **Sharding — делим данные между несколькими серверами. Каждый shard хранит свою часть данных. Важнейшее решение — правильный shard key, по которому определяется, куда попадёт запись.**

```python id="7k2p9x"
                Application
                     │
              Sharding Router
             /       |       \
            ▼        ▼        ▼
        Shard 1   Shard 2   Shard 3
```

---

# 🧠 Зачем нужен Sharding

Представим PostgreSQL на одном сервере:

```python id="4n7xqp"
Application
     │
     ▼
PostgreSQL
     │
     ├── CPU 🔥
     ├── RAM 🔥
     ├── Disk 🔥
     └── Storage → 100 TB
```

Если один сервер уже не справляется, можно разделить данные:

```python id="8v3mka"
                 Application
                      │
               Sharding Router
                      │
          ┌───────────┼───────────┐
          ▼           ▼           ▼
       Shard 1     Shard 2     Shard 3
```

Теперь каждый сервер обслуживает только свою часть данных.

---

# 1. 📦 Что такое Shard

**Shard** — отдельный узел или логическая часть распределённой БД, содержащая определённую часть общего набора данных.

Например:

```python id="w2c6zr"
users

user_id 1–1 000 000
```

Можно распределить:

```python id="a4m9qt"
Shard 1 → 1–250 000
Shard 2 → 250 001–500 000
Shard 3 → 500 001–750 000
Shard 4 → 750 001–1 000 000
```

Каждый shard хранит только свою часть пользователей.

---

# 2. 🔑 Shard Key

**Shard Key** — ключ, по которому определяется, в какой shard попадёт запись.

Например:

```python id="f5q8nb"
user_id
```

Условно:

```python id="m7r3kc"
shard = hash(user_id) % 4
```

Получаем:

```python id="v2x6pd"
user_id
   ↓
  hash
   ↓
shard number
   ↓
Shard 1 / 2 / 3 / 4
```

---

# 🎯 Хороший Shard Key

Хороший shard key должен обеспечивать:

### 1. Равномерное распределение

```python id="q9w4ms"
Shard 1 → 25%
Shard 2 → 25%
Shard 3 → 25%
Shard 4 → 25%
```

А не:

```python id="e6t2kp"
Shard 1 → 90% 🔥
Shard 2 → 4%
Shard 3 → 3%
Shard 4 → 3%
```

Последний вариант создаёт **Hot Shard**.

---

### 2. Хорошую маршрутизацию запросов

Желательно, чтобы приложение часто знало shard key:

```python id="h3n8vy"
SELECT *
FROM users
WHERE user_id = 12345;
```

Тогда можно обратиться непосредственно к нужному shard.

---

### 3. Минимум cross-shard операций

Если запрос постоянно требует данных со всех shards:

```python id="k7p2xd"
Query
  │
  ├──► Shard 1
  ├──► Shard 2
  ├──► Shard 3
  └──► Shard 4
```

преимущество шардинга частично теряется.

---

# 3. 🔀 Виды шардирования

## Range Sharding

Данные распределяются по диапазонам.

```python id="m4z8qs"
user_id 1–100000      → Shard 1
user_id 100001–200000 → Shard 2
user_id 200001–300000 → Shard 3
```

### Плюсы

* удобно выполнять range queries;
* логика понятна;
* можно легко определить shard.

### Минус

Неравномерное распределение.

Например, если все новые записи имеют большие ID:

```python id="n6x2br"
Shard 1
Shard 2
Shard 3 🔥🔥🔥
```

Получаем **Hot Shard**.

---

# 4. #️⃣ Hash Sharding

Используется hash от shard key:

```python id="r5c9kw"
hash(user_id) % N
```

Например:

```python id="b7m3qa"
hash(101) % 4 → Shard 1
hash(102) % 4 → Shard 3
hash(103) % 4 → Shard 2
```

### Плюс

Обычно обеспечивает более равномерное распределение.

### Минус

Range queries становятся сложнее.

Например:

```python id="t8v4nx"
WHERE user_id BETWEEN 1000 AND 2000
```

может потребовать обращения к нескольким shards.

---

# 5. 🗂️ Sharding по географии

Можно распределять данные по региону:

```python id="p3q7mc"
Europe → Shard 1
Asia   → Shard 2
USA    → Shard 3
```

Это называется **Geographic Sharding**.

Может быть полезно, если важно:

* уменьшить latency;
* хранить данные ближе к пользователям;
* соблюдать требования к размещению данных.

Но распределение должно оставаться достаточно равномерным.

---

# 6. 🔥 Hot Shard

Одна из главных проблем шардирования.

Например:

```python id="u4n9qs"
Shard 1 → 25%
Shard 2 → 25%
Shard 3 → 25%
Shard 4 → 25%
```

По объёму всё хорошо.

Но по нагрузке:

```python id="c8m2rx"
Shard 1 → 10%
Shard 2 → 10%
Shard 3 → 10%
Shard 4 → 70% 🔥
```

Один shard становится bottleneck.

Причины:

* плохой shard key;
* популярный пользователь;
* популярный tenant;
* неравномерный workload.

---

# 7. 🔄 Scatter-Gather

Если приложение не знает, на каком shard находятся данные, запрос приходится отправлять нескольким shards.

```python id="x6k3mv"
                 Query
                   │
          ┌────────┼────────┐
          ▼        ▼        ▼
       Shard 1  Shard 2  Shard 3
          │        │        │
          └────────┼────────┘
                   ▼
                 Merge
```

Это называется:

**Scatter-Gather Query.**

Например:

```python id="j8p5cz"
SELECT COUNT(*)
FROM orders
WHERE status = 'paid';
```

Если `status` не является shard key, придётся спрашивать несколько shards.

---

# ⚠️ Почему Scatter-Gather дорогой

Один запрос превращается в несколько:

```python id="s4n7yw"
1 request
   ↓
3 requests
   ↓
3 результата
   ↓
aggregation / merge
   ↓
final result
```

При большом количестве shards стоимость и latency могут расти.

Поэтому архитектуру стараются строить так, чтобы наиболее важные запросы могли быть направлены **в конкретный shard**.

---

# 8. 🔗 Cross-Shard JOIN

Обычный JOIN:

```python id="k2m8vd"
SELECT *
FROM users u
JOIN orders o
  ON o.user_id = u.id;
```

В одной БД это относительно просто.

При шардинге:

```python id="w5q3xa"
Shard 1
  users
  orders

Shard 2
  users
  orders
```

Если связанные данные оказались на разных shards, появляется необходимость выполнять **cross-shard JOIN**.

Это значительно сложнее и дороже.

Поэтому shard key часто выбирают так, чтобы связанные данные находились вместе.

---

# 9. 💳 Cross-Shard Transaction

Ещё одна проблема — транзакция, которая затрагивает несколько shards.

Например:

```python id="d7x4mn"
Transaction
   │
   ├──► Shard 1
   │
   └──► Shard 2
```

Обычная локальная транзакция БД:

```python id="z9q2kp"
BEGIN
   ↓
UPDATE
   ↓
COMMIT
```

гораздо проще.

Когда участвуют несколько независимых узлов, возникает проблема **распределённой транзакции**.

Могут потребоваться:

* 2PC;
* Saga;
* Outbox;
* идемпотентность;
* компенсационные операции.

---

# 10. 🆔 Генерация ID

При нескольких shards становится сложнее использовать простой последовательный:

```python id="r4m8cy"
SERIAL
```

потому что несколько узлов могут одновременно создавать ID.

Например:

```python id="n3q6vb"
Shard 1 → id = 100
Shard 2 → id = 100
```

Для распределённых систем используют различные стратегии:

* UUID;
* ULID;
* Snowflake-подобные ID;
* составные ключи.

Выбор зависит от требований системы.

---

# 11. 🔄 Rebalancing

Со временем shards могут заполниться неравномерно:

```python id="y8c4mk"
Shard 1 → 90 GB
Shard 2 → 92 GB
Shard 3 → 900 GB 🔥
```

Нужно перераспределить данные.

Это называется:

**Rebalancing**.

Например:

```python id="q6m2vz"
До:

Shard 1 → 30%
Shard 2 → 30%
Shard 3 → 40%

После:

Shard 1 → 33%
Shard 2 → 33%
Shard 3 → 34%
```

Rebalancing может быть сложным, особенно для production-систем с большим объёмом данных.

---

# 12. 🔢 Количество shards

Количество shards — важное архитектурное решение.

Например:

```python id="f3k7np"
4 shards
```

может быть недостаточно через несколько лет.

Но слишком большое количество shards тоже создаёт проблемы:

```python id="w9m5cx"
1000 shards
```

Увеличиваются:

* количество соединений;
* операционная сложность;
* routing;
* monitoring;
* backup;
* migrations;
* cross-shard операции.

Поэтому количество shards выбирают с учётом **объёма данных, нагрузки и прогнозируемого роста**.

---

# 13. 🔁 Sharding + Replication

В production часто используют оба подхода:

```python id="p8x4mr"
                 Application
                      │
          ┌───────────┼───────────┐
          ▼           ▼           ▼
       Shard 1     Shard 2     Shard 3
          │           │           │
        ┌─┴─┐       ┌─┴─┐       ┌─┴─┐
        ▼   ▼       ▼   ▼       ▼   ▼
      Rep1 Rep2   Rep1 Rep2   Rep1 Rep2
```

Здесь:

**Sharding**:

```python id="k4n8ys"
распределяет данные
```

**Replication**:

```python id="v2m6qa"
создаёт копии каждого shard
```

Это позволяет одновременно решать задачи масштабирования и отказоустойчивости.

---

# 🆚 Sharding vs Partitioning

Эти понятия часто путают.

### Partitioning

Делим таблицу на части внутри одной БД:

```python id="a6q9mw"
PostgreSQL
    │
    └── orders
         ├── 2024
         ├── 2025
         └── 2026
```

### Sharding

Распределяем данные между узлами:

```python id="s7c3xn"
          Database Cluster
          /      |      \
         ▼       ▼       ▼
      Shard 1 Shard 2 Shard 3
```

Главное:

> **Partitioning — разделение данных внутри БД. Sharding — распределение данных между узлами.**

---

# 🆚 Sharding vs Replication

|                        | Sharding         | Replication  |
| ---------------------- | ---------------- | ------------ |
| Данные                 | Разделяются      | Копируются   |
| Каждый узел хранит     | Часть данных     | Копию данных |
| Масштабирование объёма | ✅                | Ограниченно  |
| Масштабирование чтения | ✅                | ✅            |
| Отказоустойчивость     | Само по себе нет | ✅            |
| Cross-node запросы     | Возможны         | Обычно проще |

---

# 🧭 Как выглядит запрос

Хороший сценарий:

```python id="m9v2cx"
SELECT *
FROM users
WHERE user_id = 123;
```

Если `user_id` — shard key:

```python id="h5k7qa"
user_id = 123
      ↓
Shard Router
      ↓
Shard 2
      ↓
result
```

Плохой для шардинга сценарий:

```python id="u3x8nz"
SELECT *
FROM users
WHERE email = 'user@example.com';
```

Если shard key — `user_id`, может потребоваться:

```python id="d6q4mw"
Shard 1 ─┐
Shard 2 ─┼─► Search
Shard 3 ─┤
Shard 4 ─┘
```

---

# ⚠️ Главные проблемы Sharding

```python id="q2m7vc"
Sharding
   │
   ├── выбор shard key
   ├── Hot Shards
   ├── Scatter-Gather
   ├── Cross-Shard JOIN
   ├── Distributed Transactions
   ├── Rebalancing
   ├── генерация ID
   └── операционная сложность
```

Поэтому:

> **Sharding — мощный, но дорогой с точки зрения сложности архитектуры инструмент.**

Не стоит использовать его только потому, что система «большая».

---

# 🎯 Когда нужен Sharding

Sharding рассматривают, когда:

* один сервер не справляется с объёмом данных;
* одного сервера недостаточно по нагрузке;
* данные физически не помещаются на одном узле;
* требуется горизонтальное масштабирование;
* необходимо распределить workload между несколькими узлами.

До этого обычно рассматривают:

```python id="b4x9ms"
SQL optimization
       ↓
Indexes
       ↓
Caching
       ↓
Connection Pooling
       ↓
Scale Up
       ↓
Read Replicas
       ↓
Partitioning
       ↓
Sharding
```

Это не обязательная последовательность, а общий инженерный подход.

---

# 🎯 Главное

```python id="x7m3qp"
Sharding
    ↓
Разделяем данные
    ↓
Несколько узлов
    ↓
Каждый хранит свою часть
```

Ключевые понятия:

```python id="n5q8cw"
Shard
Shard Key
Hash Sharding
Range Sharding
Hot Shard
Scatter-Gather
Cross-Shard JOIN
Cross-Shard Transaction
Rebalancing
```

---

# 🎤 Формулировка для собеседования

> **Sharding — это горизонтальное распределение данных между несколькими узлами. Каждый shard хранит только часть общего набора данных. Ключевое решение — выбор shard key: он должен обеспечивать равномерное распределение данных и нагрузки и позволять эффективно маршрутизировать запросы. Основные проблемы шардинга — hot shards, cross-shard запросы и JOIN, распределённые транзакции и rebalancing. Sharding отличается от репликации тем, что репликация копирует данные, а sharding их разделяет.**

---

## 📌 Формула

```python id="c8v2mq"
Sharding
=
Split Data
+
Multiple Nodes
+
Shard Key
+
Horizontal Scaling
-
Cross-Shard Complexity
-
Hot Shards
-
Rebalancing
```
