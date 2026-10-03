# Routing Key и Binding в RabbitMQ 🔀

## 🎯 Ответ на собеседовании

**Routing Key — строка, которую Producer указывает при публикации сообщения и которую Exchange использует для маршрутизации.**

**Binding — правило связи между Exchange и Queue, определяющее, какие сообщения должны попасть в Queue.**

Упрощённо:

```python
Producer
    ↓
message + routing_key
    ↓
Exchange
    ↓
Binding
    ↓
Queue
    ↓
Consumer
```

Правила зависят от типа Exchange:

* **Direct** → точное совпадение routing key;
* **Topic** → совпадение по шаблону;
* **Fanout** → routing key не используется для выбора Queue;
* **Headers** → маршрутизация по headers.

**Главное:** Producer указывает `routing key`, а Exchange сравнивает его с bindings и решает, в какие Queue направить сообщение.

---

## 🎤 Суперкоротко

```text
Routing Key → ЧТО указал Producer
Binding     → ПРАВИЛО маршрутизации
Exchange    → КУДА отправить
Queue       → ГДЕ ждать Consumer
```

---

# Общая схема

```text
Producer
   │
   │ routing_key
   ▼
Exchange
   │
   │ bindings
   ▼
Queue
   │
   ▼
Consumer
```

Например:

```python
routing_key = "order.created"
```

Exchange ищет Queue, у которых есть подходящий binding.

---

# Что такое Routing Key

Routing Key — это строка, связанная с опубликованным сообщением.

Например:

```python
order.created
order.updated
order.cancelled
payment.created
payment.failed
```

Producer отправляет:

```python
message = "Order #123 created"
routing_key = "order.created"
```

После этого Exchange использует `routing_key` для маршрутизации.

---

# Что такое Binding

Binding связывает:

```text
Exchange
   ↓
Queue
```

и содержит правила маршрутизации.

Например:

```text
orders_exchange
      │
      │ binding = "order.created"
      ▼
orders_queue
```

Это можно прочитать:

> Сообщения с подходящим routing key направлять в `orders_queue`.

---

# Direct Exchange

Для Direct Exchange routing key должен точно совпасть с binding key.

Например:

```text
Exchange
│
├── order.created → orders_queue
├── order.cancelled → cancelled_queue
└── payment.created → payments_queue
```

Producer:

```python
routing_key = "order.created"
```

Результат:

```text
order.created
      ↓
orders_queue
```

---

## Другой routing key

Producer:

```python
routing_key = "payment.created"
```

Получим:

```text
payment.created
      ↓
payments_queue
```

Потому что:

```text
payment.created == payment.created
```

---

# Direct: точное совпадение

```text
routing_key:
order.created

binding:
order.created

        ↓

       MATCH
```

Но:

```text
routing_key:
order.updated

binding:
order.created

        ↓

       NO MATCH
```

Именно поэтому:

> **Direct = exact match.**

---

# Topic Exchange

Topic Exchange работает с шаблонами.

Routing keys обычно имеют несколько слов, разделённых точкой:

```python
order.created
order.updated
order.cancelled
payment.created
payment.failed
```

Bindings могут использовать:

```text
*
#
```

---

# Символ `*`

`*` соответствует **одному слову**.

Например:

```text
binding:
order.*
```

Подойдут:

```text
order.created
order.updated
order.cancelled
```

Но не:

```text
order.payment.created
```

потому что здесь после `order` два слова:

```text
order.payment.created
     ↑       ↑
   слово   слово
```

---

# Символ `#`

`#` соответствует **нулю или большему количеству слов**.

Например:

```text
binding:
order.#
```

может соответствовать:

```text
order.created
order.updated
order.payment.created
order.payment.success
```

---

# Сравнение `*` и `#`

| Binding   | Подходит                  |
| --------- | ------------------------- |
| `order.*` | `order.created`           |
| `order.*` | `order.updated`           |
| `order.*` | `order.payment`           |
| `order.*` | ❌ `order.payment.created` |
| `order.#` | `order.created`           |
| `order.#` | `order.payment.created`   |
| `order.#` | `order.payment.success`   |

Упрощённо:

```text
* → одно слово
# → любое количество слов
```

---

# Пример Topic Exchange

Есть:

```text
events_exchange
```

Queue:

```text
orders_queue
```

Binding:

```text
order.*
```

Producer отправляет:

```python
routing_key = "order.created"
```

Получаем:

```text
Producer
   ↓
order.created
   ↓
Topic Exchange
   ↓
order.*
   ↓
orders_queue
```

Сообщение попадает в Queue.

---

# Несколько Bindings

У одной Queue может быть несколько bindings.

Например:

```text
orders_queue

bindings:
order.created
order.updated
order.cancelled
```

Получается:

```text
              Exchange
                  │
       ┌──────────┼──────────┐
       ↓          ↓          ↓
order.created order.updated order.cancelled
       │          │          │
       └──────────┼──────────┘
                  ↓
             orders_queue
```

Queue получает сообщения, соответствующие любому подходящему binding.

---

# Одна Queue — несколько Exchanges

В RabbitMQ Queue может быть связана с несколькими Exchanges.

Например:

```text
Exchange A ──┐
             │
             ▼
          Queue
             ▲
             │
Exchange B ──┘
```

Таким образом одна Queue может получать сообщения из разных источников маршрутизации.

---

# Один Exchange — несколько Queue

Это ещё более распространённый сценарий.

```text
                  Exchange
                 /    |    \
                ↓     ↓     ↓
              Queue1 Queue2 Queue3
```

Например:

```text
order.created
      ↓
Exchange
 ├──→ email_queue
 ├──→ analytics_queue
 └──→ warehouse_queue
```

Одно событие могут получить несколько независимых сервисов.

---

# Fanout Exchange

Для Fanout routing key не используется для выбора Queue.

```text
Producer
   ↓
Fanout Exchange
   ├──→ Queue 1
   ├──→ Queue 2
   └──→ Queue 3
```

Например:

```python
routing_key = "anything"
```

сообщение всё равно будет отправлено связанным Queue.

Поэтому:

> **Fanout = broadcast.**

---

# Headers Exchange

Headers Exchange маршрутизирует сообщения на основе headers.

Например:

```text
headers:
    type = "order"
    region = "eu"
```

Exchange проверяет эти значения и определяет Queue.

```text
Producer
   ↓
Message + Headers
   ↓
Headers Exchange
   ↓
Queue
```

В этом случае routing key не является главным механизмом маршрутизации.

---

# Что происходит, если нет подходящего Binding

Представим:

```text
routing_key = "payment.created"
```

Но Exchange не имеет подходящего binding.

```text
Producer
   ↓
payment.created
   ↓
Exchange
   ↓
❌ no matching binding
```

Если сообщение опубликовано с соответствующей настройкой, Producer может получить **Return**.

Это позволяет обнаружить сообщение, которое не было маршрутизировано в Queue.

---

# Routing Key ≠ Queue Name

Это важное различие.

Например:

```text
routing_key = "order.created"
```

Queue может называться:

```text
orders_processing
```

То есть:

```text
routing_key
     ≠
queue name
```

Routing key — значение для маршрутизации.

Queue name — имя очереди.

---

# Routing Key ≠ Binding

Тоже важно.

Producer отправляет:

```python
routing_key = "order.created"
```

А Queue имеет:

```text
binding = "order.*"
```

Это разные сущности:

```text
Producer
   │
   │ routing key
   ▼
Exchange
   │
   │ binding
   ▼
Queue
```

Exchange сравнивает routing key с правилами binding.

---

# Routing Key и Event-Driven Architecture

Routing key особенно удобен для событий.

Например:

```text
order.created
order.updated
order.cancelled
payment.created
payment.failed
```

Можно настроить:

```text
order.* → Order-related Service
payment.* → Payment-related Service
```

Получается слабая связанность:

```text
Producer
   ↓
Exchange
   ↓
разные Queue
   ↓
разные сервисы
```

Producer не обязан знать, какие конкретно сервисы сейчас подписаны на события.

---

# Routing Key и микросервисы

Например:

```text
Order Service
      ↓
order.created
      ↓
RabbitMQ Exchange
      │
      ├──→ Notification Service
      ├──→ Analytics Service
      └──→ Warehouse Service
```

Каждый сервис имеет собственную Queue и собственные bindings.

---

# Пример

Допустим, есть Topic Exchange:

```text
events
```

Bindings:

```text
notification_queue → order.*
analytics_queue    → #
payment_queue      → payment.*
```

Producer публикует:

```python
routing_key = "order.created"
```

Получаем:

```text
order.created
      │
      ▼
   events
      │
      ├──→ notification_queue
      │
      └──→ analytics_queue
```

`payment_queue` сообщение не получает.

---

# Почему Binding важен

Binding позволяет отделить:

```text
что произошло
```

от:

```text
кто должен это обработать
```

Producer сообщает:

```text
order.created
```

Exchange через bindings определяет:

```text
кто должен получить событие
```

Это один из механизмов слабой связанности в Event-Driven Architecture.

---

# Routing Key и RabbitMQ vs Kafka

Здесь важно не смешивать модели.

### RabbitMQ

```text
Producer
   ↓
routing key
   ↓
Exchange
   ↓
binding
   ↓
Queue
```

### Kafka

```text
Producer
   ↓
key
   ↓
Topic
   ↓
Partition
```

Kafka тоже использует key для выбора partition, но это **не тот же механизм**, что routing key + binding в RabbitMQ.

---

# Routing Key и Kafka Key

Оба значения могут влиять на распределение, но задачи разные.

### RabbitMQ Routing Key

Используется Exchange для маршрутизации:

```text
routing key
    ↓
Exchange
    ↓
Queue
```

### Kafka Key

Используется, в частности, для определения partition:

```text
key
 ↓
partition selection
 ↓
Partition
```

Kafka key особенно важен для сохранения порядка событий одного ключа.

---

# 🎯 Частые вопросы на собеседовании

### Что такое Routing Key?

Routing key — значение сообщения, которое Exchange использует для маршрутизации.

### Что такое Binding?

Binding — связь между Exchange и Queue с правилами маршрутизации.

### Чем Routing Key отличается от Binding?

> **Routing Key приходит вместе с сообщением, Binding задаётся на стороне Exchange/Queue как правило маршрутизации.**

### Как работает Direct Exchange?

Требует точного совпадения routing key и binding key.

### Как работает Topic Exchange?

Сопоставляет routing key с шаблонами, используя `*` и `#`.

### Что означает `*`?

Одно слово в Topic routing key.

### Что означает `#`?

Ноль или больше слов.

### Использует ли Fanout routing key?

Routing key не используется для выбора Queue — сообщение направляется во все связанные Queue.

### Может ли одна Queue иметь несколько bindings?

Да.

### Может ли один Exchange отправить сообщение в несколько Queue?

Да.

### Что произойдёт, если нет подходящего binding?

Сообщение не будет маршрутизировано в Queue; при соответствующей настройке Producer может получить Return.

### Routing Key — это имя Queue?

Нет.

### Routing Key и Kafka Key — одно и то же?

Нет. Они используются в разных механизмах маршрутизации/распределения.

---

# 🧠 Итоговая схема

```text
                         Producer
                            │
                     routing_key
                            │
                            ▼
                       ┌─────────┐
                       │Exchange │
                       └────┬────┘
                            │
                         bindings
                            │
              ┌─────────────┼─────────────┐
              ↓             ↓             ↓
           Queue 1       Queue 2       Queue 3
              ↓             ↓             ↓
          Consumer 1    Consumer 2    Consumer 3
```

### Для Direct

```text
routing_key == binding
```

### Для Topic

```text
routing_key ↔ pattern

* → одно слово
# → ноль или больше слов
```

### Для Fanout

```text
Exchange → все связанные Queue
```

### Формула для собеседования

> **Producer публикует сообщение с routing key → Exchange сопоставляет его с bindings → подходящие Queue получают сообщение → Consumer обрабатывает его. Routing Key — это значение сообщения, Binding — правило маршрутизации.**
