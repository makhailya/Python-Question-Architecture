# 🔌 PgBouncer

## 🎯 Формула для собеседования

> **PgBouncer — это лёгкий connection pooler для PostgreSQL, который находится между приложением и PostgreSQL и управляет пулом клиентских соединений. Он позволяет обслуживать большое количество клиентских подключений меньшим количеством реальных соединений с PostgreSQL, снижая нагрузку на БД. Основные режимы — session, transaction и statement pooling.**

---

## 🎤 Суперкоротко

```text
Application
     ↓
  PgBouncer
     ↓
PostgreSQL
```

Например:

```text
1000 client connections
        ↓
    PgBouncer
        ↓
50 PostgreSQL connections
```

**Главная идея:**

> PgBouncer принимает много клиентских подключений и переиспользует ограниченное количество соединений с PostgreSQL.

---

# 1. 🤔 Зачем нужен PgBouncer

PostgreSQL не рассчитан на бесконечное количество одновременных соединений.

Каждое подключение к PostgreSQL имеет определённую стоимость по:

* RAM;
* CPU;
* backend process;
* контексту сессии;
* обслуживанию connection.

Если приложение масштабируется:

```text
Application
 ├── Worker 1
 ├── Worker 2
 ├── Worker 3
 ├── ...
 └── Worker 100
```

каждый worker потенциально может создать свой пул соединений.

В результате количество connections может быстро вырасти.

PgBouncer позволяет отделить:

```text
количество клиентских подключений
```

от:

```text
количества реальных подключений к PostgreSQL
```

---

# 2. 🏗️ Где находится PgBouncer

Без PgBouncer:

```text
Application
    │
    ├──────────────→ PostgreSQL
    │
    ├──────────────→ PostgreSQL
    │
    └──────────────→ PostgreSQL
```

С PgBouncer:

```text
                 ┌───────────────┐
Application ────→│   PgBouncer   │
                 └───────┬───────┘
                         │
                         ▼
                    PostgreSQL
```

Если приложений несколько:

```text
App 1 ──┐
App 2 ──┤
App 3 ──┼──→ PgBouncer ──→ PostgreSQL
App 4 ──┤
App 5 ──┘
```

PgBouncer централизует управление соединениями.

---

# 3. 🔄 Как работает PgBouncer

Допустим:

```text
100 клиентов
```

подключились к PgBouncer.

При этом PgBouncer может иметь:

```text
20 реальных connections
```

к PostgreSQL.

Упрощённо:

```text
Client 1 ──┐
Client 2 ──┤
Client 3 ──┤
...        ├──→ PgBouncer ──→ PostgreSQL
Client 100─┘          │
                      └── 20 DB connections
```

Когда клиенту нужна БД:

```text
client
  ↓
PgBouncer
  ↓
свободное DB connection
  ↓
PostgreSQL
```

После завершения работы connection может быть передано другому клиенту — в зависимости от режима pooling.

---

# 4. 🎯 Главная задача PgBouncer

Важно понимать:

> **PgBouncer не является оптимизатором SQL.**

Он не делает:

```text
SELECT быстрее
```

сам по себе.

Он решает другую проблему:

```text
слишком много connections
        ↓
PgBouncer
        ↓
ограниченное количество
реальных DB connections
```

---

# 5. 📦 Режимы pooling

У PgBouncer есть три основных режима:

```text
Session pooling
Transaction pooling
Statement pooling
```

Это **очень важная тема для собеседования**.

---

# 6. 🔐 Session Pooling

При **session pooling** клиент получает конкретное соединение PostgreSQL на всю свою сессию.

```text
Client
  │
  ├── connect
  │
  ▼
PgBouncer
  │
  ▼
PostgreSQL connection #1
  │
  ├── query
  ├── query
  ├── query
  └── query
  │
  ▼
disconnect
```

После отключения connection возвращается в pool.

### Плюсы

* хорошая совместимость;
* состояние PostgreSQL-сессии сохраняется;
* меньше ограничений для приложений.

### Минус

Если клиент долго держит соединение:

```text
Client
   ↓
connection занято
   ↓
даже если клиент почти ничего не делает
```

то это connection нельзя эффективно передать другому клиенту.

---

# 7. 🔄 Transaction Pooling

При **transaction pooling** connection выдаётся клиенту **на время транзакции**.

Схема:

```text
Client
  ↓
BEGIN
  ↓
PgBouncer
  ↓
DB connection #1
  ↓
SQL
  ↓
COMMIT
  ↓
connection возвращается в pool
```

Следующая транзакция может получить уже другое connection:

```text
Transaction 1 → DB connection #1
Transaction 2 → DB connection #7
Transaction 3 → DB connection #3
```

Это позволяет значительно эффективнее переиспользовать соединения.

---

# 8. ⭐ Почему Transaction Pooling эффективнее

Допустим:

```text
1000 клиентов
```

но одновременно реально выполняется только:

```text
20 транзакций
```

PgBouncer может держать условно:

```text
20 PostgreSQL connections
```

вместо:

```text
1000 PostgreSQL connections
```

Получается:

```text
1000 clients
      ↓
PgBouncer
      ↓
20 DB connections
      ↓
PostgreSQL
```

Это одна из главных причин использования PgBouncer.

---

# 9. ⚠️ Ограничения Transaction Pooling

Поскольку клиент не закреплён за одним PostgreSQL connection на всю сессию, нельзя бездумно рассчитывать на состояние конкретной session.

Проблемными могут быть механизмы, завязанные на конкретное connection/session.

Например:

```text
session-level state
temporary tables
session-level settings
```

Также нужно учитывать prepared statements и другие connection-specific возможности в зависимости от версии PostgreSQL, PgBouncer и настроек.

Поэтому transaction pooling требует, чтобы приложение корректно работало при смене backend connection между транзакциями.

---

# 10. ⚡ Statement Pooling

В режиме **statement pooling** connection может переиспользоваться после каждого SQL statement.

Упрощённо:

```text
Statement 1
    ↓
DB connection #1
    ↓
release

Statement 2
    ↓
DB connection #5
    ↓
release
```

Это самый агрессивный уровень переиспользования connections.

Но и ограничений больше.

Он плохо подходит для сценариев, где SQL-операции должны выполняться в рамках одной транзакции:

```text
BEGIN
UPDATE ...
UPDATE ...
COMMIT
```

Поэтому для большинства современных приложений обычно рассматривают **session или transaction pooling** в зависимости от требований.

---

# 11. 📊 Режимы PgBouncer

| Режим           | Connection закреплён      | Переиспользование | Совместимость      |
| --------------- | ------------------------- | ----------------- | ------------------ |
| **Session**     | За клиентом на всю сессию | Ниже              | Высокая            |
| **Transaction** | На транзакцию             | Высокая           | Ниже               |
| **Statement**   | На statement              | Очень высокая     | Самая ограниченная |

Главная мысль:

```text
Session
→ connection на всю session

Transaction
→ connection на одну transaction

Statement
→ connection на один statement
```

---

# 12. 🧩 PgBouncer и Connection Pool приложения

У тебя уже есть понятие **Connection Pool**.

Важно различать:

### Application Pool

```text
FastAPI
   ↓
SQLAlchemy Pool
   ↓
PostgreSQL
```

### PgBouncer

```text
FastAPI
   ↓
PgBouncer
   ↓
PostgreSQL
```

### Вместе

```text
FastAPI
   ↓
Application Pool
   ↓
PgBouncer
   ↓
PostgreSQL
```

Они решают похожую задачу, но находятся на разных уровнях.

---

# 13. ⚠️ Почему два pool подряд требуют внимания

Если сделать:

```text
Application Pool = 50
```

и:

```text
PgBouncer = 50
```

это не означает автоматически:

```text
идеальная производительность
```

Можно получить лишнее количество клиентских connections к PgBouncer и неоптимальные настройки.

Например:

```text
10 application workers
×
20 connections
=
200 application connections
```

А PgBouncer может ограничить реальные connections к PostgreSQL:

```text
200 clients
      ↓
PgBouncer
      ↓
30 PostgreSQL connections
```

Поэтому нужно считать всю цепочку.

---

# 14. 🆚 PgBouncer vs PostgreSQL max_connections

PostgreSQL имеет:

```python id="5fzj3k"
max_connections
```

Это ограничение количества подключений к PostgreSQL.

PgBouncer позволяет держать:

```text
много клиентских connections
```

при этом ограничивать:

```text
реальные PostgreSQL connections
```

Например:

```text
Clients
1000
  ↓
PgBouncer
  ↓
50
  ↓
PostgreSQL
```

То есть:

```text
client connections
≠
server connections
```

Это ключевая идея PgBouncer.

---

# 15. 🚨 Что происходит при нехватке DB connections

Допустим:

```text
PgBouncer
max DB connections = 20
```

и все 20 заняты.

Приходит ещё запрос:

```text
Client 21
    ↓
PgBouncer
    ↓
нет свободного DB connection
    ↓
ожидание
```

Если ожидание превышает соответствующий timeout:

```text
timeout
```

Таким образом PgBouncer может выступать как **буфер и контролируемая очередь перед PostgreSQL**.

---

# 16. 🔐 PgBouncer и транзакции

При transaction pooling важно понимать границу:

```text
BEGIN
   ↓
transaction
   ↓
COMMIT
```

Именно на этой границе connection может вернуться в pool.

Схематично:

```text
Client
  │
  ├── BEGIN
  │
  ├── SELECT
  │
  ├── UPDATE
  │
  └── COMMIT
           ↓
       release
```

Это позволяет одному PostgreSQL connection обслужить много разных клиентов последовательно.

---

# 17. 📈 Где PgBouncer особенно полезен

Типичные сценарии:

### Много коротких соединений

```text
Serverless
   ↓
много краткоживущих клиентов
   ↓
PostgreSQL
```

PgBouncer позволяет сгладить эту нагрузку.

### Много application workers

```text
Kubernetes
 ├── Pod 1
 ├── Pod 2
 ├── Pod 3
 ├── ...
 └── Pod 100
```

Каждый pod может иметь свои connections.

PgBouncer позволяет централизованно контролировать backend connections.

### Несколько приложений

```text
API
Worker
Admin
ETL
Celery
   ↓
PgBouncer
   ↓
PostgreSQL
```

---

# 18. 🧠 PgBouncer не является репликацией

Не путать:

```text
PgBouncer
→ connection pooling
```

и:

```text
Replication
→ копирование данных между PostgreSQL instances
```

PgBouncer:

```text
Application
    ↓
PgBouncer
    ↓
один PostgreSQL
```

Репликация:

```text
Primary
   ↓
Replica
```

Это совершенно разные задачи.

---

# 19. 🧠 PgBouncer не является балансировщиком SQL

PgBouncer не распределяет запросы между:

```text
PostgreSQL A
PostgreSQL B
PostgreSQL C
```

как обычный load balancer между независимыми БД.

Его основная задача:

```text
clients
   ↓
connection pooling
   ↓
PostgreSQL
```

Для HA, routing к Primary/Replica и распределения read/write используются другие компоненты и архитектурные решения.

---

# 20. 🏗️ Пример production-архитектуры

```text
                    Internet
                       │
                       ▼
                     Nginx
                       │
                       ▼
                  FastAPI Pods
              ┌────────┼────────┐
              ▼        ▼        ▼
            Pool      Pool     Pool
              └────────┼────────┘
                       ▼
                   PgBouncer
                       │
                       ▼
                  PostgreSQL
```

PgBouncer находится ближе к БД и централизует управление backend connections.

---

# 21. 🔥 Главные преимущества

```text
PgBouncer
│
├── уменьшает количество реальных DB connections
├── переиспользует connections
├── снижает стоимость частого подключения
├── защищает PostgreSQL от connection storm
├── полезен при большом количестве клиентов
└── особенно эффективен с transaction pooling
```

---

# 22. ⚠️ Недостатки и ограничения

```text
PgBouncer
│
├── дополнительный компонент
├── дополнительная конфигурация
├── ограничения transaction/statement pooling
├── нужно учитывать session state
├── нужен мониторинг
└── неправильные настройки могут создать bottleneck
```

Главное:

> PgBouncer не делает систему автоматически быстрее. Его задача — эффективно управлять соединениями с PostgreSQL.

---

# 🎯 Главное

```text
PgBouncer
│
├── connection pooler для PostgreSQL
│
├── находится между приложением и PostgreSQL
│
├── принимает много client connections
│
├── переиспользует ограниченное количество
│   PostgreSQL connections
│
├── Session Pooling
│     → connection на всю session
│
├── Transaction Pooling
│     → connection на transaction
│
└── Statement Pooling
      → connection на statement
```

### Ключевая схема

```text
Много клиентов
      ↓
┌─────────────┐
│  PgBouncer  │
└──────┬──────┘
       ↓
Меньше реальных connections
       ↓
┌─────────────┐
│ PostgreSQL  │
└─────────────┘
```

### Связь с Connection Pool

```text
Connection Pool
→ общее понятие управления DB connections

PgBouncer
→ специализированный внешний pooler
  для PostgreSQL

PostgreSQL
→ сама БД, которой нужно ограничивать
  количество реальных connections
```
