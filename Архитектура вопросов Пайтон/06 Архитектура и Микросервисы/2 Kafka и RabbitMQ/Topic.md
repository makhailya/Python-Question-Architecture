# Topic в Apache Kafka 📚

## 🎯 Ответ на собеседовании

**Topic в Kafka — это логический именованный поток событий, в который Producer записывает сообщения, а Consumer читает их.**

Topic физически разделён на **Partition**. Каждая Partition представляет собой упорядоченный append-only log и имеет собственные Offset.

Например:

```text
Topic: orders
│
├── Partition 0
├── Partition 1
├── Partition 2
├── Partition 3
└── Partition 4
```

Partition позволяют распределять нагрузку между Broker и Consumer'ами и обеспечивают горизонтальное масштабирование.

**Важно:** Topic — это логическое понятие. Данные физически хранятся в Partition на Broker'ах.

---

## 🎤 Суперкоротко

```text
Topic
  ↓
логический поток событий
  ↓
Partition
  ↓
упорядоченный log
  ↓
Offset
```

Например:

```text
orders
├── P0
├── P1
├── P2
└── P3
```

---

# Что такое Topic

Topic можно представить как **категорию или поток событий**.

Например:

```text
orders
payments
users
notifications
```

Producer публикует события в Topic:

```text
Order Service
      ↓
   orders
```

Consumer читает события:

```text
orders
   ↓
Analytics Service
```

---

# Topic не хранит сообщения напрямую

Это важный нюанс.

Логически мы говорим:

```text
Topic → orders
```

Но физически Kafka хранит данные в Partition:

```text
orders
│
├── Partition 0
├── Partition 1
└── Partition 2
```

А уже Partition хранится на Broker'ах.

Получается:

```text
Topic
  ↓
Partitions
  ↓
Brokers
  ↓
Disk
```

---

# Topic состоит из Partition

Допустим:

```text
Topic: orders
Partitions: 3
```

Получаем:

```text
orders
│
├── Partition 0
│     ├── offset 0
│     ├── offset 1
│     └── offset 2
│
├── Partition 1
│     ├── offset 0
│     ├── offset 1
│     └── offset 2
│
└── Partition 2
      ├── offset 0
      ├── offset 1
      └── offset 2
```

У каждой Partition свой Offset.

---

# Почему Topic делится на Partition

Главная причина — **масштабирование**.

Один огромный поток:

```text
orders
   ↓
одна Partition
   ↓
один Consumer
```

ограничивает параллелизм.

Если сделать:

```text
orders
├── P0
├── P1
├── P2
├── P3
└── P4
```

можно распределить обработку:

```text
P0 → Consumer 1
P1 → Consumer 2
P2 → Consumer 3
P3 → Consumer 4
P4 → Consumer 5
```

Получаем параллельную обработку.

---

# Topic и Consumer Group

Очень важно:

**Topic не привязан к одному Consumer.**

Один Topic может читать несколько Consumer Group.

Например:

```text
                 orders
                   │
          ┌────────┼────────┐
          ↓        ↓        ↓
       Group A   Group B   Group C
```

Каждая группа самостоятельно отслеживает свои Offset.

Например:

```text
orders
  │
  ├── analytics-group
  │
  ├── billing-group
  │
  └── notification-group
```

Каждая группа может независимо читать одни и те же события.

---

# Один Topic — несколько Consumer Group

Допустим, Producer отправил:

```text
order.created
```

В Topic:

```text
orders
```

Есть:

```text
analytics-group
billing-group
notification-group
```

Все три группы могут прочитать это событие:

```text
                orders
                   │
       ┌───────────┼───────────┐
       ↓           ↓           ↓
   Analytics    Billing    Notification
```

Это одна из причин, почему Kafka хорошо подходит для Event-Driven Architecture.

---

# Topic и Consumer Group — не одно и то же

Не путай:

```text
Topic
→ поток данных
```

и:

```text
Consumer Group
→ группа потребителей, совместно обрабатывающих этот поток
```

Например:

```text
Topic: orders

Consumer Groups:
├── analytics
├── billing
└── warehouse
```

---

# Topic и Partition

Это ещё одно важное различие.

### Topic

Логический поток:

```text
orders
```

### Partition

Физически разделённая часть Topic:

```text
orders
├── P0
├── P1
└── P2
```

Можно сказать:

> **Topic — логическая сущность, Partition — единица хранения, порядка и параллелизма.**

---

# Порядок сообщений в Topic

Kafka **не гарантирует глобальный порядок внутри всего Topic**, если Topic имеет несколько Partition.

Например:

```text
Topic: orders

P0:
A → B → C

P1:
X → Y → Z
```

Порядок гарантируется:

```text
P0: A → B → C
P1: X → Y → Z
```

Но глобального порядка между:

```text
A, B, C, X, Y, Z
```

нет.

---

# Почему Partition важнее Topic для порядка

Порядок гарантируется **внутри Partition**.

Поэтому:

```text
Topic
  ↓
Partition
  ↓
Ordering
```

Если события одной сущности должны обрабатываться последовательно, их обычно направляют в одну Partition с помощью Kafka Key.

Например:

```text
user_id = 123
```

может использоваться как Key.

Тогда события пользователя:

```text
user.created
user.updated
user.deleted
```

попадут в одну Partition и сохранят порядок внутри неё.

---

# Topic и Kafka Key

Producer может отправить:

```python
key = "user-123"
value = "user.updated"
```

Kafka использует Key при выборе Partition в соответствии с используемым partitioner.

Упрощённо:

```text
key
 ↓
partitioner
 ↓
Partition
```

Это позволяет направлять события одного ключа в одну Partition.

---

# Topic и Offset

Каждая Partition имеет собственную последовательность Offset:

```text
P0:
offset 0
offset 1
offset 2

P1:
offset 0
offset 1
offset 2
```

Поэтому Offset не является глобальным идентификатором сообщения.

Идентичность записи концептуально:

```text
(topic, partition, offset)
```

Например:

```text
(orders, 2, 157)
```

---

# Topic и Retention

Kafka не удаляет сообщение просто потому, что Consumer его прочитал.

Сообщение остаётся в Partition согласно политике **Retention**.

Например:

```text
orders
   ↓
Partition
   ↓
events
   ↓
Retention = 7 days
```

После истечения retention старые данные удаляются согласно политике хранения.

---

# Поэтому несколько Consumer Group могут читать один Topic

Например:

```text
orders
   │
   ├── Group A читает сейчас
   ├── Group B подключилась позже
   └── Group C перечитывает историю
```

Это возможно благодаря тому, что Kafka хранит события независимо от факта их прочтения конкретным Consumer.

---

# Topic и Replay

Одна из сильных сторон Kafka:

**Consumer может перечитать старые сообщения, если они ещё находятся в Retention.**

Например:

```text
orders

offset:
100
101
102
103
104
```

Consumer был на:

```text
offset = 102
```

Можно изменить позицию и снова прочитать:

```text
102
103
104
```

Это называется **replay**.

---

# Topic и Log Compaction

Для Topic можно использовать не только обычный retention, но и **Log Compaction**.

Например, события:

```text
user_id=123 → name=Ilya
user_id=123 → name=Alex
user_id=123 → name=Bob
```

При compaction Kafka может оставить последнее значение для ключа:

```text
user_id=123 → name=Bob
```

Это отличается от обычного удаления по времени.

---

# Topic и Tombstone

При log compaction специальное сообщение:

```python
key = "user-123"
value = None
```

может выступать как **tombstone** — маркер удаления ключа.

Упрощённо:

```text
user-123 → Bob
user-123 → None
```

После compaction запись о ключе может быть удалена.

---

# Topic и Broker

Topic может иметь Partition, распределённые между разными Broker.

Например:

```text
Kafka Cluster

Broker 1
 ├── orders-P0
 └── orders-P3

Broker 2
 ├── orders-P1
 └── orders-P4

Broker 3
 └── orders-P2
```

Таким образом нагрузка распределяется между узлами.

---

# Topic и Replication

Partition может иметь несколько реплик.

Например:

```text
orders-P0

Broker 1 → Leader
Broker 2 → Follower
Broker 3 → Follower
```

То есть:

```text
Topic
 ↓
Partition
 ↓
Replicas
 ↓
Brokers
```

Это уже относится к отказоустойчивости Kafka.

---

# Сколько Partition нужно Topic

Количество Partition — архитектурное решение.

Больше Partition:

```text
+
больше потенциального параллелизма
+
больше возможностей для масштабирования
```

Но:

```text
-
больше метаданных
-
больше ресурсов
-
сложнее управление
```

Поэтому нельзя просто делать огромное количество Partition без необходимости.

---

# Можно ли уменьшить количество Partition?

Важный практический момент:

**Количество Partition существующего Topic нельзя просто уменьшить обычным изменением настройки.**

Увеличить количество Partition можно, но это может повлиять на распределение Key и связанные с ним гарантии порядка.

Поэтому количество Partition желательно планировать заранее.

---

# Topic и Scaling

Допустим:

```text
Topic = orders
Partitions = 5
```

Consumer Group:

```text
Consumer 1 → P0
Consumer 2 → P1
Consumer 3 → P2
Consumer 4 → P3
Consumer 5 → P4
```

Максимальный параллелизм этой группы примерно:

```text
5 Consumer
```

Если добавить шестого:

```text
Consumer 6 → idle
```

Потому что Partition всего пять.

---

# Topic и несколько групп

Важно, что ограничение относится к **одной Consumer Group**.

Например:

```text
Topic: orders
Partitions: 5

Group A → 5 Consumer
Group B → 5 Consumer
Group C → 5 Consumer
```

Каждая группа независимо читает Topic.

---

# Topic — это не Queue

Очень частый вопрос на собеседовании.

### RabbitMQ Queue

```text
Queue
 ↓
Consumer
```

После ACK сообщение обычно больше не нужно Queue.

### Kafka Topic

```text
Topic
 ↓
Partition
 ↓
Consumer Group
```

Consumer читает запись, но запись остаётся в Kafka согласно Retention.

---

# Topic vs RabbitMQ Queue

| Kafka Topic                     | RabbitMQ Queue                      |
| ------------------------------- | ----------------------------------- |
| Логический поток событий        | Очередь сообщений                   |
| Делится на Partition            | Не имеет Kafka-подобных Partition   |
| Сообщения хранятся по Retention | Обычно удаляются после ACK          |
| Есть Offset                     | Есть ACK                            |
| Replay возможен                 | Обычно не является основной моделью |
| Несколько Consumer Group        | Несколько независимых Queue         |

---

# Жизненный цикл сообщения

Упрощённо:

```text
Producer
   ↓
Topic
   ↓
Partition
   ↓
Broker
   ↓
Disk
   ↓
Consumer
   ↓
Commit Offset
```

При этом:

```text
Commit Offset
    ≠
Delete Message
```

Сообщение продолжает храниться согласно Retention.

---

# Типичный пример

Есть интернет-магазин.

Создаём:

```text
orders
```

В него отправляются:

```text
order.created
order.paid
order.shipped
order.cancelled
```

Topic:

```text
orders
│
├── P0
├── P1
├── P2
└── P3
```

Consumer Groups:

```text
orders
│
├── billing-group
├── analytics-group
└── notification-group
```

Получаем:

```text
                orders
                   │
       ┌───────────┼───────────┐
       ↓           ↓           ↓
   Billing      Analytics   Notification
```

Каждая группа самостоятельно читает события.

---

# 🎯 Частые вопросы на собеседовании

### Что такое Topic?

> Логический именованный поток событий в Kafka, который состоит из Partition.

### Topic — это физический файл?

Нет. Topic — логическая сущность. Физически данные хранятся в Partition на Broker'ах.

### Зачем Topic нужны Partition?

Для распределения данных, горизонтального масштабирования и параллельной обработки.

### Гарантируется ли порядок во всём Topic?

Нет. Порядок гарантируется только внутри одной Partition.

### Может ли один Topic читать несколько Consumer Group?

Да. Каждая группа имеет собственные Offset и может независимо читать события.

### Удаляется ли сообщение после чтения?

Нет. Kafka хранит его согласно Retention независимо от того, прочитал его Consumer или нет.

### Можно ли перечитать сообщения?

Да, если они ещё доступны в соответствии с политикой хранения.

### Что такое Topic с точки зрения масштабирования?

> Topic делится на Partition, а Partition распределяются между Broker и Consumer'ами, что позволяет масштабировать хранение и обработку.

### Можно ли уменьшить число Partition?

Обычным изменением конфигурации — нет. Увеличение Partition возможно, но может повлиять на распределение Key и порядок связанных событий.

---

# 🧠 Полная схема

```text
                         Kafka Cluster
                              │
                           Topic
                         "orders"
                              │
              ┌───────────────┼───────────────┐
              ↓               ↓               ↓
          Partition 0     Partition 1     Partition 2
              │               │               │
           offsets         offsets         offsets
              │               │               │
              └───────────────┼───────────────┘
                              │
                       Consumer Group
                              │
                 ┌────────────┼────────────┐
                 ↓            ↓            ↓
             Consumer 1   Consumer 2   Consumer 3
```

### Ключевая цепочка

```text
Topic
  ↓
Partition
  ↓
Offset
  ↓
Consumer Group
  ↓
Consumer
```

### Формула для собеседования

> **Topic — это логический поток событий Kafka. Он состоит из Partition, которые являются единицами хранения, порядка и параллелизма. Producer записывает сообщения в Topic, Consumer читает их через Consumer Group, а каждая Partition имеет собственные Offset. Сообщения не удаляются после чтения — их хранение определяется Retention.**
