# 🦁 Apache Kafka

## 🎯 Ответ на собеседовании

**Apache Kafka** — это распределённая платформа для передачи и хранения потоков событий. Она позволяет сервисам публиковать события в **топики**, а другим сервисам — читать их и обрабатывать.

Kafka используется для **event streaming**, построения асинхронного взаимодействия между микросервисами, аналитики и обработки больших потоков данных.

Основные понятия Kafka:

**Producer → Topic → Partition → Consumer Group → Consumer**

---

## 🎤 Суперкоротко

**Kafka — это распределённая платформа для потоковой передачи событий. Producer записывает сообщения в Topic, Topic разделён на Partition, а Consumer читает сообщения и отслеживает свою позицию через Offset.**

---

## 🧩 Основная схема

```text
Producer
    │
    │ event
    ↓
  Kafka
    │
    ↓
  Topic
    │
    ├── Partition 0
    ├── Partition 1
    └── Partition 2
    │
    ↓
Consumer Group
    ├── Consumer 1
    ├── Consumer 2
    └── Consumer 3
```

---

## 📌 Topic

**Topic** — логическая категория, в которую Producer записывает события.

Например:

```text
orders
payments
users
notifications
```

Producer может отправить событие:

```python
{
    "event": "order_created",
    "order_id": 123
}
```

в Topic:

```text
orders
```

---

## 🧱 Partition

**Partition** — отдельная последовательность сообщений внутри Topic.

Topic может состоять из нескольких Partition:

```text
Topic: orders

Partition 0 → [M1] [M4] [M7]
Partition 1 → [M2] [M5] [M8]
Partition 2 → [M3] [M6] [M9]
```

Именно Partition позволяют Kafka **масштабировать обработку и хранение данных**.

---

## 🔢 Offset

Каждое сообщение внутри Partition имеет **Offset** — позицию сообщения в этой Partition.

```text
Partition 0

Offset:
  0 → Message A
  1 → Message B
  2 → Message C
  3 → Message D
```

Consumer использует Offset, чтобы понимать, **до какого места он обработал сообщения**.

Важно:

**Offset относится к конкретной Partition.**

---

## 👥 Consumer

**Consumer** — приложение, которое читает сообщения из Kafka.

```text
Kafka
  ↓
Topic
  ↓
Partition
  ↓
Consumer
```

Consumer самостоятельно читает сообщения и обрабатывает их.

---

## 👥 Consumer Group

**Consumer Group** — группа Consumer'ов, совместно обрабатывающих сообщения одного Topic.

Например:

```text
Topic: orders

Partition 0 ──→ Consumer 1
Partition 1 ──→ Consumer 2
Partition 2 ──→ Consumer 3
```

В рамках одной Consumer Group **одна Partition в конкретный момент времени обрабатывается одним Consumer**.

Это позволяет распределять нагрузку.

---

## ⚖️ Масштабирование Consumer'ов

Допустим, есть 3 Partition:

```text
P0
P1
P2
```

И 3 Consumer:

```text
C1
C2
C3
```

Распределение:

```text
P0 → C1
P1 → C2
P2 → C3
```

Если Consumer'ов больше, чем Partition:

```text
P0 → C1
P1 → C2
P2 → C3
C4 → без Partition
```

Поэтому внутри одной Consumer Group количество параллельно работающих Consumer'ов ограничено количеством Partition.

---

## 🔄 Consumer Group ≠ Consumer

Это важно различать.

```text
Consumer Group A
├── Consumer 1
├── Consumer 2
└── Consumer 3
```

Другая группа:

```text
Consumer Group B
├── Consumer 4
└── Consumer 5
```

Каждая группа самостоятельно читает Topic.

Поэтому одно событие может быть обработано **разными Consumer Group**.

Например:

```text
             orders
                │
       ┌────────┴────────┐
       ↓                 ↓
Order Group        Analytics Group
       ↓                 ↓
Order Service      Analytics Service
```

---

## 🔥 Kafka — это не обычная Queue

Это принципиальное отличие от классической очереди.

В RabbitMQ после успешной обработки сообщение обычно удаляется из очереди.

В Kafka сообщение сохраняется согласно **retention policy**.

```text
Topic

M1 → M2 → M3 → M4 → M5
```

Consumer Group может прочитать:

```text
M1 → M2 → M3
```

а другая группа:

```text
M1 → M2 → M3 → M4 → M5
```

При этом сообщения не исчезают только потому, что один Consumer их прочитал.

---

## 🔁 Повторное чтение

Поскольку Kafka хранит сообщения определённое время, Consumer может повторно прочитать их, изменив свою позицию — Offset.

Например:

```text
Offset:

0   1   2   3   4
↓   ↓   ↓   ↓   ↓
M1  M2  M3  M4  M5
        ↑
     Consumer
```

Consumer может продолжить с Offset 3 или перечитать более ранние сообщения.

---

## 📝 Producer

**Producer** публикует сообщения в Topic.

```text
Application
    ↓
Producer
    ↓
Kafka Topic
```

Producer может использовать **key**, по которому Kafka определяет Partition.

Например:

```python
key = "user_123"
```

Сообщения с одинаковым key обычно направляются в одну и ту же Partition, что позволяет сохранять порядок сообщений для данного ключа.

---

## 🔢 Порядок сообщений

Kafka гарантирует порядок сообщений **внутри одной Partition**.

```text
Partition 0

M1 → M2 → M3 → M4
```

Но между разными Partition общего порядка нет.

```text
Partition 0 → M1 → M3
Partition 1 → M2 → M4
```

Нельзя гарантировать:

```text
M1 → M2 → M3 → M4
```

между Partition без дополнительной логики.

---

## 🖥️ Broker

**Kafka Broker** — сервер Kafka, который хранит и обслуживает данные.

Kafka-кластер состоит из нескольких Broker:

```text
Kafka Cluster

┌──────────┐
│ Broker 1 │
└──────────┘

┌──────────┐
│ Broker 2 │
└──────────┘

┌──────────┐
│ Broker 3 │
└──────────┘
```

Partition распределяются между Broker'ами.

---

## 🛡️ Репликация

Kafka может хранить несколько копий Partition.

Например:

```text
Partition 0

Leader
  ↓
Replica 1
  ↓
Replica 2
```

Если Broker с Leader выйдет из строя, Kafka может выбрать другую реплику Leader'ом.

Это повышает отказоустойчивость.

---

## 👑 Leader и Replica

Для каждой Partition существует **Leader** и могут существовать **Follower replicas**.

```text
Partition 0

Broker 1 → Leader
Broker 2 → Follower
Broker 3 → Follower
```

Producer и Consumer взаимодействуют с Leader Partition, а реплики поддерживают копии данных.

---

## 🚀 Почему Kafka производительная

Основные причины:

* последовательная запись;
* Partitioning;
* горизонтальное масштабирование;
* batch processing;
* эффективная работа с диском;
* возможность параллельной обработки;
* zero-copy и эффективная передача данных;
* репликация между Broker'ами.

Главный механизм масштабирования:

```text
Topic
  ↓
Partitions
  ↓
Parallel processing
```

---

## 🔄 Kafka в микросервисах

Например, пользователь оформил заказ:

```text
Order Service
      │
      ↓
Kafka
      │
      ↓
orders topic
      │
 ┌────┼──────────┐
 ↓    ↓          ↓
Billing  Analytics  Notification
```

Order Service публикует:

```text
order.created
```

А другие сервисы самостоятельно реагируют на событие.

Это называется **event-driven architecture**.

---

## 📨 Event-driven архитектура

Вместо:

```text
Order Service
      │
      ├──→ Billing Service
      ├──→ Email Service
      └──→ Analytics Service
```

можно:

```text
Order Service
      │
      ↓
order.created
      ↓
    Kafka
      │
 ┌────┼──────────┐
 ↓    ↓          ↓
Billing Email  Analytics
```

Producer не обязан напрямую знать о каждом Consumer.

---

## 🆚 Kafka vs RabbitMQ

| Kafka                                     | RabbitMQ                                             |
| ----------------------------------------- | ---------------------------------------------------- |
| Event streaming                           | Message broker                                       |
| Topic + Partition                         | Exchange + Queue                                     |
| Сообщения хранятся по retention           | Сообщение обычно удаляется после ACK                 |
| Consumer отслеживает Offset               | Consumer подтверждает сообщение ACK                  |
| Отлично масштабируется через Partition    | Масштабирование через очереди/Consumer'ы             |
| Хорош для event streaming                 | Хорош для task queues                                |
| Высокая пропускная способность            | Сильная маршрутизация                                |
| Повторное чтение удобно встроено в модель | Повторная доставка требует соответствующей настройки |

Упрощённо:

```text
RabbitMQ → "сделай эту задачу"

Kafka → "произошло это событие"
```

Это упрощение, но для собеседования хорошо передаёт основную разницу.

---

## 🐍 Kafka и Python

В Python для Kafka используются клиенты, например:

```python
from kafka import KafkaProducer

producer = KafkaProducer(
    bootstrap_servers="localhost:9092"
)

producer.send(
    "orders",
    b"order_created"
)

producer.flush()
```

Consumer:

```python
from kafka import KafkaConsumer

consumer = KafkaConsumer(
    "orders",
    bootstrap_servers="localhost:9092",
    group_id="order-service"
)

for message in consumer:
    print(message.value)
```

В production обычно отдельно продумываются сериализация, обработка ошибок, commit Offset, retries и идемпотентность.

---

## ⚠️ Важные проблемы Kafka

При работе с Kafka нужно учитывать:

* дубликаты сообщений;
* повторную обработку;
* порядок сообщений;
* Consumer Lag;
* управление Offset;
* количество Partition;
* репликацию;
* retention;
* retry;
* идемпотентность;
* мониторинг.

### Consumer Lag

**Consumer Lag** — отставание Consumer от последних сообщений в Partition.

```text
Последнее сообщение:
Offset 1000

Consumer обработал:
Offset 900

Lag = 100
```

Большой Lag означает, что Consumer не успевает обрабатывать поступающие сообщения.

---

## 🔐 Delivery semantics

Kafka может использовать разные модели гарантии обработки:

```text
At most once
At least once
Exactly once
```

### At most once

Сообщение обрабатывается максимум один раз.

Возможна потеря сообщения.

### At least once

Сообщение не должно потеряться, но возможна повторная обработка.

Поэтому особенно важна **идемпотентность Consumer'а**.

### Exactly once

Обработка стремится обеспечить семантику "ровно один раз", но это сложный сценарий, который требует правильной настройки Kafka и всей цепочки обработки.

---

## 💡 Главное

```text
Kafka
 │
 ├── Broker
 │
 ├── Topic
 │      ↓
 │   Partition
 │      ↓
 │    Offset
 │
 ├── Producer
 │
 └── Consumer
        ↓
   Consumer Group
```

Самая важная цепочка:

**Producer → Topic → Partition → Consumer → Offset**

И самое важное отличие:

**Kafka хранит поток событий и позволяет Consumer Group независимо читать его, используя Offset.**

### Формула для собеседования

**Kafka = distributed event streaming + Topic + Partition + Offset + Consumer Group + replication**

Если спросят, **почему Kafka хорошо подходит для микросервисов**, хороший ответ:

> **Kafka позволяет сервисам обмениваться событиями асинхронно, хранить поток событий и независимо подключать несколько Consumer Group. За счёт Partition можно масштабировать обработку горизонтально, а репликация обеспечивает отказоустойчивость.**
