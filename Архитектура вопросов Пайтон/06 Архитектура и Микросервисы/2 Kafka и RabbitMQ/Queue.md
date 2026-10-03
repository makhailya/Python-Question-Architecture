# Queue в RabbitMQ 📬

## 🎯 Ответ на собеседовании

**Queue — это очередь сообщений в RabbitMQ, из которой Consumer получает сообщения для обработки.**

Exchange маршрутизирует сообщения в Queue, а Queue хранит их до момента обработки и подтверждения.

Queue позволяет:

* буферизировать сообщения;
* отделять Producer от Consumer;
* обрабатывать сообщения асинхронно;
* масштабировать обработку несколькими Consumers;
* переживать временную недоступность Consumer.

Основная схема:

```text
Producer
   ↓
Exchange
   ↓
Queue
   ↓
Consumer
   ↓
ACK
```

**Главное:** Exchange отвечает за маршрутизацию, Queue — за хранение и доставку сообщений Consumer.

---

## 🎤 Суперкоротко

> **Queue — это очередь, где RabbitMQ хранит сообщения, ожидающие обработки Consumer.**

```text
Exchange → Queue → Consumer
```

---

# Что такое Queue

Queue — это именованная очередь сообщений.

Например:

```text
orders_queue

┌─────────────────────────────────┐
│ Message 1                       │
│ Message 2                       │
│ Message 3                       │
│ Message 4                       │
└─────────────────────────────────┘
                 ↓
             Consumer
```

Producer не должен знать, какой именно Consumer будет обрабатывать сообщение.

Он публикует сообщение в Exchange:

```text
Producer
   ↓
Exchange
   ↓
orders_queue
   ↓
Consumer
```

---

# Зачем нужна Queue

Основная задача Queue — **буферизация между Producer и Consumer**.

Представим:

```text
Producer
   ↓
1000 сообщений
   ↓
Queue
   ↓
Consumer
```

Если Consumer способен обработать только 100 сообщений в секунду, остальные сообщения могут временно находиться в Queue.

```text
Producer: 1000 msg/s
Consumer: 100 msg/s

        ↓

       Queue
   накопление сообщений
```

Это позволяет Producer и Consumer работать с разной скоростью.

---

# Queue и асинхронность

Queue позволяет не ждать непосредственного выполнения операции.

Например, пользователь оформляет заказ:

```text
HTTP Request
     ↓
Order Service
     ↓
RabbitMQ
     ↓
email_queue
```

Order Service может быстро завершить HTTP-запрос, а отправка email произойдёт позже:

```text
email_queue
     ↓
Email Worker
     ↓
Отправка email
```

Получается:

```text
HTTP → быстро

тяжёлая работа → асинхронно
```

---

# Exchange и Queue

Это ключевое различие.

### Exchange

Решает:

> **Куда направить сообщение?**

### Queue

Решает:

> **Где сообщение будет ждать обработки?**

Схема:

```text
Producer
   │
   │ publish
   ▼
Exchange
   │
   │ routing
   ▼
Queue
   │
   │ delivery
   ▼
Consumer
```

---

# Queue не получает сообщение «сама»

Producer обычно публикует сообщение в Exchange.

Например:

```text
Producer
   │
   │ routing_key = "order.created"
   ▼
Exchange
   │
   │ binding
   ▼
orders_queue
```

Exchange определяет, подходит ли Queue под правила маршрутизации.

---

# Binding

Queue связывается с Exchange через **binding**.

Например:

```text
Exchange
   │
   │ binding
   │ "order.created"
   ▼
orders_queue
```

Это означает:

```text
order.created
       ↓
orders_queue
```

Для Topic Exchange binding может содержать шаблон:

```text
order.*
```

---

# Consumer получает сообщения из Queue

Consumer подписывается на Queue и получает сообщения:

```text
Queue
 │
 ├── Message 1
 ├── Message 2
 ├── Message 3
 └── Message 4
        ↓
    Consumer
```

После обработки Consumer подтверждает сообщение:

```text
Consumer
   │
   │ ACK
   ▼
RabbitMQ
```

После успешного ACK сообщение может быть удалено из Queue.

---

# ACK

**ACK — подтверждение успешной обработки сообщения Consumer'ом.**

Типичный сценарий:

```text
1. RabbitMQ → отправляет Message
2. Consumer → обрабатывает Message
3. Consumer → отправляет ACK
4. RabbitMQ → удаляет сообщение
```

Схема:

```text
Queue
  ↓
Message
  ↓
Consumer
  ↓
обработка
  ↓
ACK
  ↓
сообщение больше не нужно хранить
```

Это отличается от Kafka.

В Kafka:

```text
прочитал
   ↓
commit offset
   ↓
сообщение всё равно остаётся в log
```

В RabbitMQ:

```text
получил
   ↓
ACK
   ↓
сообщение обычно удаляется из Queue
```

---

# Что происходит без ACK

Если Consumer получил сообщение, но не подтвердил его обработку, RabbitMQ считает сообщение **неподтверждённым**.

Например:

```text
Queue
  ↓
Consumer
  ↓
обработка
  ↓
❌ ошибка / Consumer умер
```

Сообщение не считается успешно обработанным.

В зависимости от режима и настроек оно может быть повторно доставлено.

Это позволяет избежать простой потери сообщения из-за падения Consumer.

---

# ACK, NACK и Reject

Основные варианты подтверждения:

| Механизм | Назначение                     |
| -------- | ------------------------------ |
| `ACK`    | Сообщение успешно обработано   |
| `NACK`   | Обработка не удалась           |
| `Reject` | Отклонить конкретное сообщение |

При отрицательном подтверждении можно настроить, например, повторную постановку сообщения в очередь:

```text
Message
   ↓
Consumer
   ↓
ошибка
   ↓
NACK + requeue
   ↓
Queue
```

Но бесконечные retry могут привести к циклу:

```text
Queue
 ↓
Consumer
 ↓
ERROR
 ↓
Queue
 ↓
Consumer
 ↓
ERROR
 ↓
...
```

Поэтому для production часто используют **retry policy и Dead Letter Queue**.

---

# Dead Letter Queue

**DLQ (Dead Letter Queue)** — очередь для сообщений, которые не удалось нормально обработать.

Например:

```text
orders_queue
      ↓
   Consumer
      ↓
    ERROR
      ↓
   retries
      ↓
     DLQ
```

После нескольких неудачных попыток сообщение можно отправить в отдельную очередь:

```text
orders_dlq
```

Это позволяет не блокировать основную обработку.

---

# Несколько Consumers

Одну Queue могут обрабатывать несколько Consumers.

Например:

```text
             orders_queue
            /      |      \
           ↓       ↓       ↓
         C1       C2       C3
```

RabbitMQ распределяет сообщения между Consumers.

Это позволяет увеличить производительность обработки.

Например:

```text
1 Consumer
    ↓
100 msg/s

3 Consumers
    ↓
потенциально ≈ 300 msg/s
```

Но фактическая производительность зависит от:

* CPU;
* I/O;
* БД;
* внешних API;
* размера сообщений;
* настроек prefetch;
* времени обработки.

---

# Queue и Consumer Group в Kafka

Здесь важно не проводить слишком прямую аналогию.

В Kafka:

```text
Topic
   ↓
Partitions
   ↓
Consumer Group
   ↓
Consumers
```

В RabbitMQ:

```text
Exchange
   ↓
Queue
   ↓
Consumers
```

Queue — не точный аналог Kafka Consumer Group.

Но концептуально несколько Consumers могут совместно обрабатывать сообщения одной Queue.

---

# Competing Consumers

Несколько Consumers, читающих одну Queue, часто называют моделью **Competing Consumers**.

```text
             Queue
          /    |    \
         ↓     ↓     ↓
        C1    C2    C3
```

Каждое сообщение обрабатывается одним из Consumers.

Например:

```text
Queue:

M1
M2
M3
M4
M5
M6

        ↓

C1 → M1, M4
C2 → M2, M5
C3 → M3, M6
```

Это позволяет горизонтально масштабировать workers.

---

# Prefetch

**Prefetch** определяет, сколько неподтверждённых сообщений RabbitMQ может передать Consumer заранее.

Например:

```text
prefetch = 10
```

Consumer может получить до 10 сообщений, которые ещё не подтверждены.

Схематично:

```text
Queue
 │
 ├── M1 ──┐
 ├── M2   │
 ├── M3   │
 ├── ...  ├──→ Consumer
 └── M10 ─┘
```

Prefetch влияет на:

* распределение нагрузки;
* throughput;
* latency;
* количество сообщений, находящихся в обработке.

Слишком большой prefetch может привести к тому, что один Consumer накопит много сообщений, пока другие простаивают.

---

# Durable Queue

Queue может быть **durable**.

Durable Queue сохраняется после перезапуска RabbitMQ.

Например:

```text
durable queue
      ↓
RabbitMQ restart
      ↓
Queue существует
```

Но важно:

**durable Queue сама по себе не гарантирует сохранность каждого сообщения.**

Для надёжности также важны настройки самого сообщения, в частности его **persistence**.

---

# Persistent Message

Сообщение можно сделать persistent.

Упрощённо:

```text
Durable Queue
+
Persistent Message
+
корректная конфигурация RabbitMQ
```

дают более высокую устойчивость к перезапуску брокера.

Не стоит говорить на собеседовании:

> Durable Queue гарантирует, что все сообщения переживут любой сбой.

Это слишком сильное утверждение.

---

# Queue и порядок сообщений

RabbitMQ может сохранять порядок сообщений в Queue, но в реальной системе порядок обработки может нарушаться.

Например:

```text
Queue:

M1
M2
M3
```

Есть два Consumer:

```text
C1
C2
```

Может произойти:

```text
M1 → C1
M2 → C2
```

Если C1 обработает M1 медленнее:

```text
M2 завершён
M1 ещё выполняется
```

Поэтому при параллельной обработке нельзя автоматически считать, что **порядок завершения обработки = порядок поступления**.

---

# Queue как буфер

Одна из самых полезных моделей:

```text
Producer
   ↓
  Queue
   ↓
Workers
```

Например:

```text
API
 ↓
RabbitMQ
 ↓
image_processing_queue
 ↓
Worker 1
Worker 2
Worker 3
```

API не занимается тяжёлой обработкой самостоятельно.

Queue выступает буфером между API и workers.

---

# RabbitMQ + Celery

Это типичный backend-сценарий.

Например:

```text
Django / FastAPI
      ↓
    Celery
      ↓
  RabbitMQ
      ↓
    Queue
      ↓
 Celery Worker
```

Или концептуально:

```text
Web Application
      ↓
     Task
      ↓
   RabbitMQ
      ↓
     Queue
      ↓
Celery Workers
```

RabbitMQ используется как broker для передачи задач workers.

---

# Пример

Представим интернет-магазин.

Пользователь оформил заказ:

```text
POST /orders
```

Backend создаёт задачу:

```text
send_order_email
```

Producer:

```text
Order Service
     ↓
Exchange
     ↓
email_queue
```

Worker:

```text
email_queue
     ↓
Email Worker
     ↓
отправка письма
     ↓
ACK
```

Если Worker временно недоступен:

```text
email_queue
     ↓
сообщения ждут
     ↓
Worker возвращается
     ↓
обрабатывает сообщения
```

---

# Queue vs Kafka Partition

|                     | RabbitMQ Queue                       | Kafka Partition                    |
| ------------------- | ------------------------------------ | ---------------------------------- |
| Основная идея       | Очередь сообщений                    | Часть event log                    |
| Хранение            | До обработки/ACK согласно настройкам | По retention                       |
| Чтение              | Broker доставляет Consumer           | Consumer читает сам                |
| Replay              | Не основная модель                   | Основная возможность               |
| Позиция             | Нет Kafka-style offset               | Offset                             |
| Параллелизм         | Несколько Consumers                  | Consumer Group + partitions        |
| Основное назначение | Доставка задач/сообщений             | Потоки событий и обработка истории |

---

# 🎯 Частые вопросы на собеседовании

### Что такое Queue?

Queue — очередь сообщений RabbitMQ, в которой сообщения ожидают доставки и обработки Consumer.

### Зачем нужна Queue?

Для буферизации сообщений, асинхронной обработки и отделения Producer от Consumer.

### Кто отправляет сообщение в Queue?

Обычно Producer отправляет сообщение в Exchange, а Exchange маршрутизирует его в Queue.

### Что происходит после ACK?

RabbitMQ считает сообщение подтверждённым и может удалить его из Queue.

### Что будет, если Consumer упадёт до ACK?

Сообщение не считается успешно обработанным и при соответствующей конфигурации может быть доставлено повторно.

### Можно ли иметь несколько Consumers у одной Queue?

Да. Они могут совместно обрабатывать сообщения, реализуя competing consumers.

### Что такое Prefetch?

Ограничение на количество сообщений, которые Consumer может получить до подтверждения предыдущих.

### Что такое DLQ?

Dead Letter Queue — отдельная очередь для сообщений, которые не удалось успешно обработать по установленным правилам.

### Чем Queue отличается от Exchange?

> **Exchange маршрутизирует, Queue хранит и доставляет.**

### Чем Queue отличается от Kafka Partition?

> **Queue — механизм очереди и доставки сообщений в RabbitMQ, а Partition — упорядоченная часть event log в Kafka.**

---

# 🧠 Итоговая схема RabbitMQ

```text
                    Producer
                       │
                       │ publish
                       ▼
                  ┌──────────┐
                  │ Exchange │
                  └────┬─────┘
                       │
                  routing rules
                       │
              ┌────────┼────────┐
              ▼        ▼        ▼
           Queue 1   Queue 2   Queue 3
              │        │        │
              ▼        ▼        ▼
           Worker 1 Worker 2 Worker 3
              │        │        │
              └────────┼────────┘
                       ▼
                      ACK
```

### Формула для собеседования

> **Exchange отвечает за маршрутизацию → Queue буферизирует и хранит сообщения → Consumer обрабатывает → ACK подтверждает успешную обработку.**

```text
Exchange → маршрутизация
Queue    → хранение/буфер
Consumer → обработка
ACK      → подтверждение
DLQ      → проблемные сообщения
Prefetch → сколько сообщений выдавать заранее
```
