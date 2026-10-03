# Partition в Kafka 🧩

## 🎯 Ответ на собеседовании

**Partition — это отдельная упорядоченная последовательность сообщений внутри Kafka Topic.**

Topic в Kafka разбивается на несколько partitions. Каждое сообщение внутри partition получает свой последовательный **offset**.

Partitions позволяют Kafka:

* распределять данные между брокерами;
* обрабатывать сообщения параллельно;
* масштабировать consumer groups;
* сохранять порядок сообщений внутри конкретной partition.

**Главное:** порядок гарантируется только **внутри одной partition**, а не во всём topic.

---

## 🎤 Суперкоротко

```text
Topic
│
├── Partition 0 → 0, 1, 2, 3, 4...
├── Partition 1 → 0, 1, 2, 3, 4...
└── Partition 2 → 0, 1, 2, 3, 4...
```

Каждая partition — это отдельный последовательный log.

**Больше partitions → больше потенциального параллелизма.**

---

# Что такое Partition

Kafka хранит сообщения не просто внутри Topic, а внутри его partitions.

Например:

```text
Topic: orders

Partition 0:
offset 0 → Order A
offset 1 → Order D
offset 2 → Order G

Partition 1:
offset 0 → Order B
offset 1 → Order E
offset 2 → Order H

Partition 2:
offset 0 → Order C
offset 1 → Order F
offset 2 → Order I
```

У каждой partition собственная последовательность offset.

Поэтому offset `5` в Partition 0 и offset `5` в Partition 1 — это **разные позиции**.

Идентифицировать запись концептуально можно так:

```text
(topic, partition, offset)
```

Например:

```text
orders, 2, 153
```

---

# Зачем нужны Partitions

Главная причина — **масштабирование и параллельная обработка**.

Представим Topic с одной partition:

```text
Topic
└── Partition 0
       ↓
   Consumer
```

Обработку этой partition нельзя одновременно распределить между несколькими consumers одной consumer group.

Добавляем partitions:

```text
Topic
├── Partition 0 → Consumer 1
├── Partition 1 → Consumer 2
├── Partition 2 → Consumer 3
└── Partition 3 → Consumer 4
```

Теперь обработка может выполняться параллельно.

---

# Partition и Consumer Group

Это одна из самых важных связей Kafka.

В рамках **одной Consumer Group** одна partition в конкретный момент времени назначается только одному consumer.

Например:

```text
Topic: orders
Partitions: 4

Consumer Group: order-service

Partition 0 → Consumer 1
Partition 1 → Consumer 2
Partition 2 → Consumer 3
Partition 3 → Consumer 4
```

Получаем параллельную обработку.

---

## Если consumers меньше partitions

```text
4 partitions
2 consumers

P0 ──┐
     ├── Consumer 1
P1 ──┘

P2 ──┐
     ├── Consumer 2
P3 ──┘
```

Каждый consumer обрабатывает несколько partitions.

---

## Если consumers больше partitions

```text
4 partitions
6 consumers

P0 → C1
P1 → C2
P2 → C3
P3 → C4

C5 → idle
C6 → idle
```

Два consumer'а будут простаивать.

Поэтому:

**максимальный параллелизм consumer group ограничен количеством partitions.**

Упрощённо:

```text
max parallelism ≈ количество partitions
```

---

# Как сообщение попадает в Partition

Producer отправляет сообщение в Topic.

Kafka должна определить, в какую partition его записать.

Один из важных механизмов — **key**.

Например:

```text
key = user_id
```

Kafka использует key для выбора partition.

Упрощённо:

```text
hash(key) → partition
```

Поэтому сообщения одного ключа обычно попадают в одну и ту же partition.

Например:

```text
user_id = 100

Order 1 → Partition 2
Order 2 → Partition 2
Order 3 → Partition 2
```

Это позволяет сохранить порядок событий для конкретного пользователя.

---

# Почему Key важен

Представим события:

```text
UserCreated
UserUpdated
UserDeleted
```

Для пользователя `42` важно обработать их именно в таком порядке:

```text
UserCreated
     ↓
UserUpdated
     ↓
UserDeleted
```

Если события этого пользователя попадут в разные partitions:

```text
P0 → UserCreated
P1 → UserUpdated
P2 → UserDeleted
```

Kafka не гарантирует общий порядок между partitions.

Поэтому часто используют:

```text
key = user_id
```

И получают:

```text
hash(user_id) → одна partition

P2:
UserCreated
UserUpdated
UserDeleted
```

Таким образом порядок для этого ключа сохраняется.

---

# Порядок сообщений

Kafka гарантирует порядок **внутри одной partition**.

Например:

```text
Partition 0:

offset 10 → A
offset 11 → B
offset 12 → C
offset 13 → D
```

Consumer прочитает их в этом порядке:

```text
A → B → C → D
```

Но между partitions порядка нет.

```text
Partition 0:
A → B → C

Partition 1:
X → Y → Z
```

Нельзя утверждать, что глобальный порядок был:

```text
A → X → B → Y → C → Z
```

Kafka этого не гарантирует.

---

# Partition и Broker

Partitions распределяются между Kafka brokers.

Например:

```text
Broker 1
├── Partition 0
└── Partition 3

Broker 2
├── Partition 1
└── Partition 4

Broker 3
├── Partition 2
└── Partition 5
```

Это позволяет распределять нагрузку:

* CPU;
* RAM;
* диск;
* network.

Поэтому Kafka масштабируется горизонтально.

---

# Replication Factor

Partition может иметь несколько реплик.

Например:

```text
Partition 0

Broker 1 → Leader
Broker 2 → Replica
Broker 3 → Replica
```

Это уже связано не с параллелизмом, а прежде всего с **отказоустойчивостью**.

Например:

```text
Replication Factor = 3
```

означает, что partition имеет три копии.

Важно различать:

| Понятие            | Для чего                                |
| ------------------ | --------------------------------------- |
| Partition          | Параллелизм и распределение данных      |
| Consumer           | Обработка данных                        |
| Consumer Group     | Параллельная обработка одним сервисом   |
| Replication Factor | Отказоустойчивость                      |
| Broker             | Инфраструктура хранения/обработки Kafka |

---

# Partition ≠ Consumer

Это часто спрашивают на собеседовании.

Partition — это **часть данных / log**.

Consumer — это **процесс, который читает данные**.

Например:

```text
3 partitions
2 consumers

P0 ──┐
P1 ──┼── Consumer 1
     │
P2 ──┴── Consumer 2
```

Consumer может читать несколько partitions.

Но две consumers одной группы не могут одновременно обрабатывать одну partition.

---

# Что произойдёт при добавлении Consumer

Допустим:

```text
3 partitions
3 consumers

P0 → C1
P1 → C2
P2 → C3
```

Добавляем четвёртого:

```text
3 partitions
4 consumers

P0 → C1
P1 → C2
P2 → C3
C4 → idle
```

Производительность не увеличится только из-за появления четвёртого consumer.

Чтобы увеличить параллелизм, может понадобиться увеличить количество partitions.

---

# Увеличение количества Partitions

Например:

```text
Было:

Topic
├── P0
└── P1
```

Стало:

```text
Topic
├── P0
├── P1
├── P2
└── P3
```

Теперь больше consumers могут работать параллельно.

Но количество partitions нельзя бездумно увеличивать.

Почему?

### 1. Больше ресурсов

Каждая partition требует ресурсов Kafka.

### 2. Больше метаданных

Kafka должна управлять большим количеством partitions.

### 3. Изменение распределения ключей

При изменении количества partitions распределение ключей может измениться.

Поэтому количество partitions желательно планировать заранее.

---

# Partition и Retention

Partition является частью Kafka log.

Сообщения записываются последовательно:

```text
P0:

0
1
2
3
4
5
6
...
```

Kafka не удаляет сообщение просто потому, что consumer его прочитал.

Удаление происходит согласно политике retention.

Например:

```text
Retention = 7 days
```

После истечения срока старые данные могут быть удалены.

Поэтому consumer может прочитать старые события повторно, если они ещё находятся в retention.

---

# Partition и Replay

Это одно из важных преимуществ Kafka.

Consumer может вернуться к более раннему offset и повторно обработать события.

Например:

```text
P0:

100
101
102
103
104
```

Consumer уже обработал:

```text
100 → 101 → 102 → 103
```

Но может снова начать чтение с:

```text
102
```

и получить:

```text
102 → 103 → 104
```

Это называется **replay**.

---

# Partition и Offset

У каждой partition собственный offset.

```text
Partition 0:
0
1
2
3

Partition 1:
0
1
2
3

Partition 2:
0
1
2
3
```

Offset не является глобальным идентификатором сообщения.

Поэтому:

```text
offset = 10
```

сам по себе недостаточен.

Нужно знать:

```text
topic + partition + offset
```

---

# Partition и Consumer Lag

Lag считается относительно конкретных partitions.

Например:

```text
Partition 0:
Log End Offset = 1000
Consumer Offset = 900

Lag = 100
```

Другой partition:

```text
Partition 1:
Log End Offset = 500
Consumer Offset = 450

Lag = 50
```

Общий lag consumer group можно рассматривать как сумму lag по partitions:

```text
100 + 50 = 150
```

Если одна partition сильно отстаёт, она может стать узким местом.

---

# Hot Partition

**Hot Partition** — partition, которая получает значительно больше нагрузки, чем остальные.

Например:

```text
P0 → 90 000 сообщений
P1 →  5 000 сообщений
P2 →  3 000 сообщений
P3 →  2 000 сообщений
```

Причиной может быть неравномерное распределение ключей.

Например, если огромное количество событий связано с одним популярным `user_id` или другим ключом.

В результате:

```text
P0 → перегружена
P1 → свободна
P2 → свободна
P3 → свободна
```

Добавление consumers не всегда решит проблему, потому что конкретную partition всё равно читает только один consumer в рамках группы.

---

# Главное отличие Partition от Topic

```text
Topic
  ↓
логическая категория событий

Partition
  ↓
физически отдельный упорядоченный log внутри Topic
```

Например:

```text
Topic: payments

├── Partition 0
├── Partition 1
├── Partition 2
└── Partition 3
```

Topic объединяет partitions под одним именем.

---

# Как всё связано

```text
                    Kafka Cluster
                         │
                    ┌────▼────┐
                    │  Topic  │
                    └────┬────┘
                         │
             ┌───────────┼───────────┐
             ▼           ▼           ▼
           P0            P1          P2
           │             │           │
        offsets       offsets     offsets
           │             │           │
           └─────────────┼───────────┘
                         │
                  Consumer Group
                         │
             ┌───────────┼───────────┐
             ▼           ▼           ▼
            C1           C2          C3
```

Связь:

```text
Topic
  ↓
Partitions
  ↓
Offsets
  ↓
Consumer Group
  ↓
Consumers
```

А replication обеспечивает отказоустойчивость:

```text
Partition
   ↓
Leader + Replicas
   ↓
разные Brokers
```

---

# Kafka Partition vs RabbitMQ Queue

|                 | Kafka Partition                  | RabbitMQ Queue                                                               |
| --------------- | -------------------------------- | ---------------------------------------------------------------------------- |
| Основная роль   | Часть event log                  | Очередь сообщений                                                            |
| Хранение        | По retention                     | Обычно до ACK/удаления                                                       |
| Порядок         | Внутри partition                 | В рамках очереди есть порядок, но фактическая обработка зависит от consumers |
| Масштабирование | Через partitions                 | Через очереди/consumers и архитектуру RabbitMQ                               |
| Replay          | Да, через offset                 | Не является основной моделью                                                 |
| Consumer Group  | Да                               | Аналогии есть, но модель другая                                              |
| Параллелизм     | Ограничен количеством partitions | Через consumers/очереди                                                      |
| Главная идея    | Распределённый журнал событий    | Доставка сообщений через broker                                              |

---

# 🎯 Частые вопросы на собеседовании

### Что такое Partition?

Partition — это отдельный упорядоченный log внутри Kafka Topic, содержащий последовательность сообщений с offset.

### Зачем нужны partitions?

Для горизонтального масштабирования Kafka и параллельной обработки сообщений consumer groups.

### Гарантируется ли порядок сообщений?

Да, но только **внутри одной partition**.

### Сколько consumers может одновременно обрабатывать одну partition?

В рамках одной consumer group — **один consumer**.

### Что будет, если consumers больше partitions?

Часть consumers будет простаивать.

### Что будет, если partitions больше consumers?

Некоторые consumers будут обрабатывать несколько partitions.

### Как сохранить порядок событий конкретного пользователя?

Обычно использовать `user_id` как key, чтобы события пользователя попадали в одну partition.

### Можно ли увеличить количество partitions?

Да, но делать это нужно осознанно: это влияет на распределение ключей и потенциально на порядок событий для ключей.

### Partition отвечает за отказоустойчивость?

Нет. За отказоустойчивость отвечает прежде всего **Replication Factor** и репликация partitions.

### Что такое Hot Partition?

Partition, которая получает непропорционально большую нагрузку относительно остальных partitions.

---

# 🧠 Итоговая схема

```text
Topic
│
├── Partition 0 ──→ Broker 1
│      └── offsets 0, 1, 2, 3...
│
├── Partition 1 ──→ Broker 2
│      └── offsets 0, 1, 2, 3...
│
└── Partition 2 ──→ Broker 3
       └── offsets 0, 1, 2, 3...

              ↓

       Consumer Group

        ┌─────┼─────┐
        ↓     ↓     ↓
       C1    C2    C3
```

**Формула для собеседования:**

> **Partitions = параллелизм.
> Replication = отказоустойчивость.
> Offset = позиция в partition.
> Consumer Group = совместная обработка partitions.
> Key = способ управлять распределением и сохранять порядок событий для сущности.**
