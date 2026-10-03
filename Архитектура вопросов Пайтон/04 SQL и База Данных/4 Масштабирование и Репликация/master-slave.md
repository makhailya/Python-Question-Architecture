# 🔄 Master-Slave репликация БД

## 🎯 Ответ на собеседовании

**Master-Slave** — устаревающий термин для архитектуры репликации, где один узел является основным (**Master**), а другие получают его изменения и выступают как копии (**Slave**).

Современная терминология:

* **Master → Primary**
* **Slave → Replica / Standby**

Обычно:

```python id="r7m2kc"
                Application
                     │
              ┌──────┴──────┐
              │             │
            WRITE          READ
              │             │
              ▼             ▼
           Primary ─────► Replica
                         Replica
```

**Primary** принимает записи, а реплики получают изменения и могут использоваться для чтения.

---

## 🎤 Суперкоротко

> **Master-Slave — это схема Primary/Replica, где Primary является источником изменений, а Replica получает его данные. Она позволяет масштабировать чтение и повысить отказоустойчивость, но при асинхронной репликации возникает replication lag.**

---

# 🧠 Как работает

Допустим, приложение выполняет:

```python id="q8v4mx"
INSERT INTO users (name)
VALUES ('Ivan');
```

Запись попадает на Primary:

```python id="x5k9wp"
Application
     │
     ▼
 Primary
     │
     │ replication
     ▼
 Replica
```

Replica получает изменение и применяет его у себя.

В итоге:

```python id="n3c7qa"
Primary → user Ivan
Replica → user Ivan
```

---

# 1. 📝 Primary

Primary — основной узел.

Он обычно принимает:

```python id="a6m2rz"
INSERT
UPDATE
DELETE
```

И также может выполнять:

```python id="k9w4vc"
SELECT
```

Но при большой нагрузке чтение часто распределяют на реплики.

---

# 2. 📖 Replica

Replica получает изменения от Primary.

Основное применение:

```python id="p4x8nm"
SELECT
```

Например:

```python id="w7c2qa"
Primary
   │
   ├──► Replica 1
   ├──► Replica 2
   └──► Replica 3
```

Теперь чтение можно распределять:

```python id="m5r9vk"
SELECT → Replica 1
SELECT → Replica 2
SELECT → Replica 3
```

---

# 3. ⚖️ Read/Write Splitting

Типичная архитектура:

```python id="z3q7cx"
                 Application
                 /         \
              WRITE        READ
                │             │
                ▼             ▼
             Primary       Replicas
                            /     \
                           ▼       ▼
                       Replica   Replica
```

### WRITE

```python id="j8m4vp"
INSERT
UPDATE
DELETE
```

→ Primary

### READ

```python id="c6x9ka"
SELECT
```

→ Replica

Так можно масштабировать преимущественно **read-heavy workload**.

---

# 4. ⚠️ Replication Lag

Главный недостаток асинхронной репликации.

Например:

```python id="h2v7mq"
Primary:
balance = 500
```

Replica ещё содержит:

```python id="u5k9xc"
Replica:
balance = 1000
```

Причина:

```python id="p8r3nv"
Primary
   │
   │ replication delay
   ▼
Replica
```

Это называется:

**Replication Lag**.

---

# 5. 🔄 Read-After-Write

Особенно заметно, когда пользователь сначала изменяет данные, а затем сразу их читает.

```python id="n4q8ws"
UPDATE profile
SET name = 'Ivan';
```

Запись уже подтверждена Primary.

Но следующий запрос:

```python id="v6m2ka"
SELECT name
FROM profile;
```

если отправить на отстающую Replica, может вернуть старое значение.

Поэтому для критичного **read-after-write** иногда используют Primary.

---

# 6. 🔄 Синхронная репликация

Primary может ждать подтверждения Replica:

```python id="r9c4xm"
Application
     │
     ▼
 Primary
     │
     ▼
 Replica
     │
     ▼
 подтверждение
     │
     ▼
 COMMIT
```

### Плюс

Меньше вероятность потерять последние подтверждённые изменения при отказе Primary.

### Минус

Replica влияет на latency записи.

---

# 7. 🚀 Асинхронная репликация

Primary не ждёт полного применения изменения на Replica:

```python id="k3w7vp"
Application
     │
     ▼
 Primary
     │
     ├──► COMMIT
     │
     └────► Replica
```

### Плюсы

* меньшая задержка записи;
* высокая производительность;
* Primary меньше зависит от Replica.

### Минус

При внезапной потере Primary часть последних изменений может ещё не успеть попасть на Replica.

---

# 🆚 Синхронная vs асинхронная

|                              | Синхронная                 | Асинхронная |
| ---------------------------- | -------------------------- | ----------- |
| Primary ждёт Replica         | ✅                          | ❌           |
| Latency записи               | Выше                       | Ниже        |
| Replication Lag              | Минимальный/контролируемый | Возможен    |
| Риск потери последних данных | Ниже                       | Выше        |
| Производительность           | Ниже                       | Выше        |

---

# 8. 🔥 Failover

Если Primary выходит из строя:

```python id="q5n8mc"
Primary ❌
    │
    ▼
Replica
```

одна из реплик может стать новым Primary:

```python id="x7m3ka"
Old Primary ❌

Replica 1
    ↓
New Primary
```

Это:

**Failover**.

Важно:

> **Репликация сама по себе не гарантирует автоматический failover.**

Для автоматического переключения нужен дополнительный механизм управления кластером.

---

# 9. 🛡️ High Availability

Master-Slave/Primary-Replica может использоваться как часть HA-архитектуры:

```python id="f4r8mw"
              Application
                   │
              HA / Proxy
                   │
            ┌──────┴──────┐
            ▼             ▼
         Primary        Replica
            │             │
            └─────────────┘
```

При отказе Primary система может переключить трафик на Replica.

---

# 10. 🗄️ Master-Slave ≠ Backup

Это принципиально.

Реплика:

```python id="w6q2pz"
Primary ─────► Replica
```

повторяет изменения.

Если приложение случайно выполнило:

```python id="s8m4yc"
DELETE FROM users;
```

изменение может распространиться и на Replica.

Поэтому:

> **Replica не заменяет backup.**

Backup нужен для восстановления данных после:

* случайного удаления;
* логической ошибки;
* повреждения;
* других сценариев восстановления.

---

# 11. 🆚 Master-Slave vs Sharding

### Master-Slave / Primary-Replica

```python id="c9v3ka"
Primary
   │
   ├──► Replica 1
   └──► Replica 2
```

Данные **копируются**.

### Sharding

```python id="r5x8mq"
Shard 1 → часть данных
Shard 2 → часть данных
Shard 3 → часть данных
```

Данные **разделяются**.

Главная формула:

```python id="m7q2vx"
Replication → Copy
Sharding    → Split
```

---

# 12. 📊 Что мониторить

Для такой архитектуры важны:

```python id="n4x7cw"
Replication Lag
Replica Health
WAL / replication position
Disk usage
Network
Replication errors
Failover status
```

Особенно важно следить за ростом lag:

```python id="p8m3qa"
0 → 10 → 100 → 1000 → 10000
```

Если lag постоянно увеличивается, Replica не успевает обрабатывать поток изменений.

---

# ⚠️ Главные проблемы

```python id="v6k9mr"
Master-Slave
     │
     ├── Replication Lag
     ├── Failover
     ├── Split Brain
     ├── Consistency
     └── Дополнительная инфраструктура
```

При этом Replica также требует:

* CPU;
* RAM;
* диска;
* сети.

---

# 🧩 Современная терминология

На собеседовании лучше использовать:

| Старый термин | Современный термин |
| ------------- | ------------------ |
| Master        | Primary            |
| Slave         | Replica            |
| Master-Slave  | Primary-Replica    |
| Master DB     | Primary DB         |
| Slave DB      | Replica DB         |

Термин **Master-Slave** всё ещё встречается в старой документации и разговорах, но в современной технической коммуникации предпочтительнее **Primary/Replica**.

---

# 🎯 Главное

```python id="a3n7xm"
Primary
   │
   ├──► Replica 1
   ├──► Replica 2
   └──► Replica 3
```

**Primary:**

```python id="q8v4kc"
WRITE
```

**Replica:**

```python id="m5x9pa"
READ
```

**Репликация:**

```python id="z6r2wn"
Primary → Replica
```

**Главная проблема:**

```python id="j4k7cx"
Replication Lag
```

**При отказе Primary:**

```python id="v9m3qa"
Replica → New Primary
```

---

# 🎤 Формулировка для собеседования

> **Master-Slave — историческое название архитектуры Primary-Replica. Primary принимает изменения, а Replica получает их через механизм репликации и может использоваться для чтения. Такая архитектура позволяет масштабировать read-heavy нагрузку и повысить отказоустойчивость. При асинхронной репликации возможен replication lag, поэтому Replica может временно отдавать устаревшие данные. При отказе Primary одну из реплик можно сделать новым Primary через failover. При этом репликация не заменяет backup и не является sharding: репликация копирует данные, а sharding их разделяет.**

---

## 📌 Формула

```python id="u7c4nx"
Master-Slave
     ↓
Primary + Replicas
     ↓
Replication
     ↓
Read Scaling + HA
     ↓
Replication Lag
     ↓
Failover
```

> В современной терминологии: **Primary-Replica**, а не Master-Slave.
