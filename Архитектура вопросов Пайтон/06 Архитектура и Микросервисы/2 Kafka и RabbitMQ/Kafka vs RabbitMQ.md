# Kafka vs RabbitMQ ⚡

## 🎯 Ответ на собеседовании

**[[Kafka]]** и **[[RabbitMQ]]** — брокеры сообщений, но они оптимизированы под разные задачи.

**[[RabbitMQ]]** — классический message broker: сообщения отправляются через **exchange → queue → consumer** и после обработки обычно удаляются из очереди.

**[[Kafka]]** — распределённая event streaming platform: сообщения записываются в **topics/partitions**, хранятся определённое время и могут быть повторно прочитаны несколькими consumer groups.

Ключевое различие:

> **RabbitMQ — доставка сообщений потребителям. Kafka — хранение и потоковая обработка событий.**

---

## 🎤 Суперкоротко

```text
RabbitMQ
Producer
   ↓
Exchange
   ↓
Queue
   ↓
Consumer
```

```text
Kafka
Producer
   ↓
Topic
   ↓
Partition
   ↓
Consumer Group
```

|                  | RabbitMQ                   | Kafka                      |
| ---------------- | -------------------------- | -------------------------- |
| Основная идея    | Message broker             | Event streaming            |
| Модель           | Очереди                    | Topics + partitions        |
| Сообщение        | Обычно удаляется после ACK | Хранится по retention      |
| Повторное чтение | Не основная модель         | Да                         |
| Порядок          | В пределах очереди         | В пределах partition       |
| Масштабирование  | Очереди/consumers          | Partitions/consumer groups |
| Routing          | Exchange                   | Topic/partition            |
| Хорош для        | Task queues, commands      | Events, streaming, logs    |
| Consumer groups  | Нет как основной механизм  | Да                         |
| Replay событий   | Ограниченно                | Основная возможность       |

---

# 1. RabbitMQ 🐇

RabbitMQ — **message broker**, построенный вокруг модели очередей.

Типичный путь сообщения:

```text
Producer
   ↓
Exchange
   ↓
Queue
   ↓
Consumer
```

Producer обычно отправляет сообщение в **Exchange**, а Exchange маршрутизирует его в одну или несколько очередей.

Consumer получает сообщение из Queue.

После успешной обработки Consumer отправляет:

```text
ACK
```

После этого RabbitMQ может удалить сообщение из очереди.

---

## Exchange

RabbitMQ использует Exchange для маршрутизации сообщений.

Основные типы:

```text
Direct
Topic
Fanout
Headers
```

Например:

```text
Producer
   ↓
Exchange
   ├── Queue A
   ├── Queue B
   └── Queue C
```

Это одна из сильных сторон RabbitMQ — **гибкая маршрутизация сообщений**.

---

# 2. Kafka 🟠

Kafka работает с **topics**.

```text
Producer
    ↓
  Topic
    ↓
Partition 0
Partition 1
Partition 2
```

Topic разбивается на **partitions**.

Каждое сообщение получает **offset**.

Например:

```text
Partition 0

offset 0 → event A
offset 1 → event B
offset 2 → event C
offset 3 → event D
```

Consumer читает сообщения по offset.

---

# 3. Главное отличие: удаление сообщения

### RabbitMQ

Упрощённо:

```text
Queue
 ↓
Consumer
 ↓
ACK
 ↓
сообщение удаляется
```

### Kafka

```text
Topic
 ↓
Consumer
 ↓
offset = 125
```

Само сообщение продолжает находиться в Kafka до окончания **retention**.

Другой consumer может прочитать его независимо.

Например:

```text
Kafka Topic
      ↓
 ┌────┼────┐
 ↓    ↓    ↓
CG1  CG2  CG3
```

Каждая **Consumer Group** имеет собственные offsets.

---

# 4. Consumer Group в Kafka 👥

Consumer Group позволяет распределять partitions между consumer'ами.

Например:

```text
Topic
├── Partition 0
├── Partition 1
├── Partition 2
└── Partition 3

Consumer Group
├── Consumer 1 → P0
├── Consumer 2 → P1
├── Consumer 3 → P2
└── Consumer 4 → P3
```

Если consumers больше, чем partitions:

```text
4 partitions
6 consumers
```

два consumer'а будут простаивать.

В одной consumer group **один partition в конкретный момент обрабатывается одним consumer'ом**.

---

# 5. Масштабирование

## RabbitMQ

Можно добавить consumers:

```text
Queue
 ├── Consumer 1
 ├── Consumer 2
 └── Consumer 3
```

Сообщения распределяются между ними.

---

## Kafka

Основной механизм масштабирования — **partitions**:

```text
Topic
├── P0 → Consumer 1
├── P1 → Consumer 2
├── P2 → Consumer 3
└── P3 → Consumer 4
```

Больше partitions → потенциально больше параллелизма.

---

# 6. Порядок сообщений 🔢

### RabbitMQ

Порядок может сохраняться в пределах очереди, но различные настройки, конкурирующие consumers и повторная доставка могут влиять на фактический порядок обработки.

### Kafka

Kafka гарантирует порядок сообщений **внутри одной partition**.

Например:

```text
Partition 0

0 → A
1 → B
2 → C
3 → D
```

Consumer читает:

```text
A → B → C → D
```

Но между разными partitions глобального порядка нет:

```text
Partition 0: A B C
Partition 1: X Y Z

Нет гарантии:

A B C X Y Z
```

### На собеседовании

> **В Kafka порядок гарантируется только внутри partition.**

---

# 7. Replay 🔄

Это одно из главных преимуществ Kafka.

Представим:

```text
10:00 → OrderCreated
10:01 → PaymentCreated
10:02 → OrderShipped
```

Consumer обработал события.

Позже мы захотели заново обработать историю.

В Kafka можно переместить offset назад:

```text
offset 150
   ↓
offset 100
```

и перечитать события.

В RabbitMQ основная модель другая: сообщение после успешной обработки и ACK обычно покидает очередь.

---

# 8. Когда использовать RabbitMQ 🐇

RabbitMQ хорошо подходит для:

* фоновых задач;
* task queues;
* команд;
* RPC;
* сложной маршрутизации;
* распределения задач между workers;
* Celery.

Например:

```text
FastAPI
   ↓
RabbitMQ
   ↓
Celery Worker
   ↓
send_email()
```

---

# 9. Когда использовать Kafka 🟠

Kafka хорошо подходит для:

* event-driven architecture;
* event streaming;
* больших потоков событий;
* логов;
* аналитики;
* ETL/ELT;
* интеграции большого количества сервисов;
* event sourcing;
* повторной обработки истории событий.

Например:

```text
Order Service
      ↓
    Kafka
   /  |  \
  ↓   ↓   ↓
Analytics
Billing
Notifications
```

Каждая система может иметь свою Consumer Group.

---

# 10. Kafka vs RabbitMQ на примере заказа

Допустим, пользователь создал заказ.

## RabbitMQ

```text
Order Service
     ↓
   Queue
     ↓
Payment Worker
     ↓
  ACK
```

Сообщение используется как **задача**.

---

## Kafka

```text
Order Service
     ↓
OrderCreated
     ↓
Kafka Topic
   /   |   \
  ↓    ↓    ↓
Billing Analytics Notifications
```

Событие становится частью потока данных.

Разные системы могут самостоятельно прочитать одно и то же событие.

---

# 11. Надёжность доставки

Оба брокера имеют механизмы надёжной доставки, но модели разные.

В RabbitMQ важны:

* ACK/NACK;
* durable queues;
* persistent messages;
* acknowledgements;
* dead-letter exchanges.

В Kafka важны:

* replication;
* acknowledgements;
* consumer offsets;
* consumer groups;
* retention.

Важно понимать:

> **«Брокер надёжный» не означает автоматически «сообщение обработается ровно один раз».**

На практике часто используется:

```text
At-least-once
      ↓
возможны дубликаты
      ↓
идемпотентный consumer
```

---

# 12. RabbitMQ + Celery

Для Python Backend это особенно важно.

Celery может использовать RabbitMQ как broker:

```text
Python Application
       ↓
     Celery
       ↓
   RabbitMQ
       ↓
    Worker
```

Например:

```python id="d3r8bx"
from celery import Celery

app = Celery(
    "tasks",
    broker="amqp://localhost",
)
```

Задача:

```python id="f9j2qk"
@app.task
def send_email(email):
    print(f"Send email to {email}")
```

В таком сценарии RabbitMQ выступает как транспорт задач.

---

# 13. Kafka в микросервисах

Типичный сценарий:

```text
                 Kafka
                   │
       ┌───────────┼───────────┐
       ↓           ↓           ↓
 Order Service  Payment    Analytics
       │         Service       │
       └─────────→ Events ←────┘
```

Например:

```text
OrderCreated
PaymentCompleted
OrderShipped
```

Сервисы реагируют на события независимо.

Это хорошо сочетается с:

* Event-Driven Architecture;
* Saga;
* Outbox Pattern;
* CQRS.

---

# 14. Kafka и RabbitMQ — не «кто лучше»

Неправильный вопрос:

> Что лучше — Kafka или RabbitMQ?

Правильный:

> Какая модель взаимодействия нужна системе?

### Если нужна очередь задач:

```text
Task
 ↓
Queue
 ↓
Worker
```

→ **RabbitMQ**

### Если нужен поток событий:

```text
Event
 ↓
Topic
 ↓
много независимых consumers
```

→ **Kafka**

---

# 15. RabbitMQ vs Kafka — ключевое сравнение

| Характеристика            | RabbitMQ               | Kafka             |
| ------------------------- | ---------------------- | ----------------- |
| Модель                    | Message broker         | Event streaming   |
| Основная сущность         | Queue                  | Topic             |
| Маршрутизация             | Exchange               | Topic + partition |
| Хранение                  | Очередь                | Log               |
| Replay                    | Не основной сценарий   | Да                |
| Offset                    | Нет Kafka-style offset | Есть              |
| Consumer Group            | Нет                    | Да                |
| Порядок                   | Обычно очередь         | Partition         |
| Масштабирование           | Consumers/queues       | Partitions        |
| Routing                   | Очень гибкий           | Более простой     |
| Task queue                | ⭐⭐⭐                    | ⭐⭐                |
| Event streaming           | ⭐⭐                     | ⭐⭐⭐               |
| Большие потоки данных     | ⭐⭐                     | ⭐⭐⭐               |
| Background jobs           | ⭐⭐⭐                    | ⭐⭐                |
| Event-driven architecture | ⭐⭐                     | ⭐⭐⭐               |

---

# ⚠️ Частая ошибка на собеседовании

Не говори:

> «Kafka — это просто более быстрый RabbitMQ».

Это разные модели.

Лучше:

> **RabbitMQ ориентирован на доставку и маршрутизацию сообщений через очереди, а Kafka — на распределённое хранение и обработку потоков событий. Kafka сохраняет события и позволяет разным consumer groups независимо читать их и повторно обрабатывать.**

---

# 🧠 Формула для запоминания

```text
RabbitMQ
→ Message Broker
→ Exchange
→ Queue
→ Consumer
→ ACK
→ Task / Command
```

```text
Kafka
→ Event Streaming
→ Topic
→ Partition
→ Offset
→ Consumer Group
→ Replay
→ Event
```

## Самое главное

```text
RabbitMQ
"Кому доставить сообщение?"

Kafka
"Как хранить и распространять поток событий?"
```

И ещё одна формула:

> **RabbitMQ — очередь задач. Kafka — журнал событий.**

При этом это упрощение: RabbitMQ умеет гораздо больше, а Kafka может использоваться не только для событий. Но для **Junior Python Backend** такая модель хорошо показывает понимание принципиальной разницы.
