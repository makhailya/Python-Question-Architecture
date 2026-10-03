# Exchange в RabbitMQ 🔀

## 🎯 Ответ на собеседовании

**Exchange — это компонент RabbitMQ, который принимает сообщения от Producer и определяет, в какие Queue их направить.**

Producer обычно отправляет сообщение **не напрямую в Queue, а в Exchange**.

Exchange использует правила маршрутизации — **bindings** и `routing key` — чтобы определить, куда доставить сообщение.

Основные типы Exchange:

* **Direct** — точное совпадение `routing key`;
* **Topic** — маршрутизация по шаблону `routing key`;
* **Fanout** — отправка во все связанные очереди;
* **Headers** — маршрутизация по headers.

Схема:

```text
Producer
   ↓
Exchange
   ↓
Routing
   ↓
Queue
   ↓
Consumer
```

---

## 🎤 Суперкоротко

> **Exchange принимает сообщение и маршрутизирует его в одну или несколько Queue согласно правилам.**

Важно:

**Exchange не хранит сообщения как основное место хранения.**

Сообщение после маршрутизации попадает в Queue.

---

# Как работает Exchange

Общий путь сообщения:

```text
Producer
   │
   │ message + routing key
   ▼
Exchange
   │
   │ routing rules
   ▼
Queue
   │
   ▼
Consumer
```

Например:

```text
Producer
   │
   │ routing_key = "order.created"
   ▼
Exchange
   │
   ├──→ orders_queue
   │
   └──→ analytics_queue
```

Одно сообщение может быть доставлено в несколько очередей.

Это особенно полезно, когда несколько сервисов должны независимо отреагировать на одно событие.

---

# Exchange и Queue — не одно и то же

Это важно не путать.

### Exchange

Отвечает за:

> **Куда отправить сообщение?**

### Queue

Отвечает за:

> **Где сообщение будет ожидать Consumer?**

Схема:

```text
Producer
   ↓
Exchange       ← маршрутизация
   ↓
Queue         ← хранение/ожидание
   ↓
Consumer      ← обработка
```

---

# Binding

**Binding** — это правило связи между Exchange и Queue.

Например:

```text
Exchange
   │
   │ binding: "order.created"
   ▼
orders_queue
```

Можно представить это как правило:

> Если сообщение соответствует этому routing key — отправить его в эту Queue.

---

# Routing Key

**Routing key** — строка, которую Producer передаёт вместе с сообщением и которая используется Exchange для маршрутизации.

Например:

```text
order.created
order.updated
order.cancelled
```

Producer:

```text
message
routing_key = "order.created"
```

Exchange смотрит на routing key и bindings и принимает решение о маршруте.

---

# Типы Exchange

## 1. Direct Exchange

Маршрутизация происходит по **точному совпадению routing key**.

Например:

```text
Exchange
│
├── "order.created" → orders_queue
├── "order.cancelled" → cancelled_queue
└── "payment.created" → payments_queue
```

Сообщение:

```text
routing_key = "order.created"
```

попадёт в:

```text
orders_queue
```

Но не в:

```text
payments_queue
```

### Схема

```text
Producer
   │
   │ "order.created"
   ▼
Direct Exchange
   │
   ├── order.created → orders_queue
   └── payment.created → payments_queue
```

**Direct = точное совпадение.**

---

# 2. Fanout Exchange

Fanout **игнорирует routing key** и отправляет сообщение во все связанные Queue.

```text
             ┌──→ Queue 1
             │
Exchange ────┼──→ Queue 2
             │
             └──→ Queue 3
```

Например, событие:

```text
UserRegistered
```

может одновременно понадобиться:

```text
Email Service
Analytics Service
CRM Service
```

Тогда:

```text
Producer
   ↓
Fanout Exchange
   ├──→ email_queue
   ├──→ analytics_queue
   └──→ crm_queue
```

**Fanout = всем связанным очередям.**

---

# 3. Topic Exchange

Topic позволяет использовать **шаблоны routing key**.

Например:

```text
order.created
order.updated
order.cancelled
payment.created
payment.failed
```

Можно создать binding:

```text
order.*
```

Он будет соответствовать:

```text
order.created
order.updated
order.cancelled
```

Но не:

```text
payment.created
```

---

## Символы Topic

В Topic Exchange используются:

* `*` — ровно одно слово;
* `#` — ноль или больше слов.

Например:

```text
order.*
```

совпадает:

```text
order.created
order.updated
```

Но не:

```text
order.payment.created
```

А:

```text
order.#
```

может соответствовать:

```text
order.created
order.payment.created
order.payment.success
```

### Схема

```text
                 ┌──→ order_queue
                 │    binding: order.*
                 │
Producer → Topic Exchange
                 │
                 └──→ payment_queue
                      binding: payment.*
```

**Topic = маршрутизация по шаблонам.**

---

# 4. Headers Exchange

Headers Exchange использует **headers сообщения**, а не routing key.

Например, сообщение содержит:

```text
type = "order"
region = "eu"
```

Exchange может маршрутизировать сообщение на основании этих значений.

Схематично:

```text
Producer
   │
   │ headers
   ▼
Headers Exchange
   │
   ├──→ Queue 1
   └──→ Queue 2
```

На практике Direct и Topic используются значительно чаще.

---

# Сравнение типов Exchange

| Exchange    | Как маршрутизирует                |
| ----------- | --------------------------------- |
| **Direct**  | Точное совпадение routing key     |
| **Topic**   | Совпадение routing key с шаблоном |
| **Fanout**  | Во все связанные Queue            |
| **Headers** | По headers сообщения              |

Запомнить можно так:

```text
Direct  → точно
Topic   → шаблон
Fanout  → всем
Headers → по заголовкам
```

---

# Один Exchange → несколько Queue

Это одна из главных возможностей RabbitMQ.

Например:

```text
                    Exchange
                       │
          ┌────────────┼────────────┐
          ↓            ↓            ↓
       Queue A      Queue B      Queue C
          ↓            ↓            ↓
       Service A    Service B    Service C
```

Каждый сервис получает **свою копию сообщения** через свою Queue.

Например, событие:

```text
order.created
```

может одновременно обработать:

```text
Order Service
Analytics Service
Notification Service
```

---

# Exchange → Queue → Consumer

Важно понимать роли компонентов:

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
   │ deliver
   ▼
Consumer
```

### Producer

Создаёт и публикует сообщение.

### Exchange

Определяет маршрут.

### Queue

Хранит сообщение до обработки.

### Consumer

Получает и обрабатывает сообщение.

---

# Что происходит с ACK

После получения сообщения Consumer должен подтвердить обработку.

```text
Queue
  ↓
Consumer
  ↓
обработка
  ↓
ACK
```

При успешной обработке Consumer отправляет:

```text
ACK
```

RabbitMQ получает подтверждение и может удалить сообщение из Queue.

Если обработка не удалась, возможны:

```text
NACK
```

или:

```text
REJECT
```

В зависимости от настроек сообщение может быть:

* возвращено в очередь;
* перенаправлено;
* отброшено.

---

# Exchange не заменяет Queue

Неправильная формулировка:

> Exchange хранит сообщения.

Лучше:

> **Exchange маршрутизирует сообщения, а Queue используется для их хранения и последующей доставки Consumer.**

Схема:

```text
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

# Exchange и Kafka Topic

Здесь часто возникает путаница.

### RabbitMQ

```text
Producer
   ↓
Exchange
   ↓
Queue
   ↓
Consumer
```

### Kafka

```text
Producer
   ↓
Topic
   ↓
Partition
   ↓
Consumer
```

Kafka Topic и RabbitMQ Exchange **не являются прямыми аналогами**.

Kafka Topic — это логическая сущность, содержащая partitions и хранящая event log.

RabbitMQ Exchange — механизм маршрутизации сообщений в Queue.

---

# Пример из backend

Представим интернет-магазин.

После создания заказа появляется событие:

```text
order.created
```

Producer публикует его:

```text
Producer
   │
   │ order.created
   ▼
orders_exchange
```

Exchange маршрутизирует сообщение:

```text
orders_exchange
   │
   ├──→ email_queue
   │       ↓
   │   Email Service
   │
   ├──→ analytics_queue
   │       ↓
   │   Analytics Service
   │
   └──→ warehouse_queue
           ↓
       Warehouse Service
```

В итоге один факт:

```text
OrderCreated
```

может вызвать несколько независимых обработчиков.

---

# Exchange и Event-Driven Architecture

Exchange хорошо подходит для Event-Driven Architecture.

Например:

```text
Order Service
     │
     │ OrderCreated
     ▼
  Exchange
     │
     ├──→ Email Service
     ├──→ Analytics Service
     └──→ Warehouse Service
```

Сервисы не обязаны напрямую знать друг о друге.

Это уменьшает связанность компонентов.

---

# Частые вопросы на собеседовании

### Что такое Exchange?

Exchange — компонент RabbitMQ, который принимает сообщения от Producer и маршрутизирует их в Queue согласно правилам маршрутизации.

### Producer отправляет сообщение напрямую в Queue?

Обычно Producer публикует сообщение в Exchange, а Exchange направляет его в Queue.

### Что такое Binding?

Binding — связь и правило маршрутизации между Exchange и Queue.

### Что такое Routing Key?

Routing key — значение, используемое Exchange для определения маршрута сообщения.

### Чем Direct отличается от Topic?

**Direct** требует точного совпадения routing key.

**Topic** позволяет исп
