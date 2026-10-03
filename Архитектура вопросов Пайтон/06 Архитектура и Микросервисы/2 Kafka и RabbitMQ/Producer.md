# 📤 Producer

## 🎯 Ответ на собеседовании

**Producer** — это приложение или компонент, который **публикует сообщения или события в брокер сообщений**.

В Kafka Producer отправляет сообщения в **Topic**. Kafka определяет, в какую **Partition** попадёт сообщение.

```text
Producer
   ↓
Kafka
   ↓
Topic
   ↓
Partition
```

Producer не отправляет сообщение напрямую Consumer'у. Он передаёт его Kafka, а Consumer уже читает его оттуда.

---

## 🎤 Суперкоротко

**Producer — это отправитель сообщений. В Kafka Producer публикует события в Topic.**

---

## 🔄 Как работает Producer

Упрощённо:

```text
Producer
   │
   │ message
   ↓
Kafka Broker
   │
   ↓
Topic
   │
   ↓
Partition
```

Например, сервис заказов создал заказ:

```text
Order Service
      ↓
Producer
      ↓
Kafka
      ↓
orders
```

Producer отправляет событие:

```python
{
    "event": "order_created",
    "order_id": 123
}
```

---

## 🧩 Что отправляет Producer

Producer может отправлять:

* события;
* команды;
* сообщения с данными.

Например:

```text
order.created
order.paid
user.registered
payment.completed
```

В event-driven архитектуре чаще говорят именно о **событиях**.

---

## 🔑 Key сообщения

Producer может указать **key**.

Например:

```python
key = "user_123"
```

Kafka использует key при выборе Partition.

Сообщения с одинаковым key обычно попадают в одну Partition:

```text
user_123 → Partition 0
user_123 → Partition 0
user_123 → Partition 0

user_456 → Partition 1
user_456 → Partition 1
```

Это важно, если нужно сохранить порядок событий для конкретного пользователя или заказа.

---

## 📌 Producer и Partition

Допустим, Topic имеет три Partition:

```text
orders

Partition 0
Partition 1
Partition 2
```

Producer отправляет сообщения, а Kafka распределяет их между Partition.

```text
Producer
   │
   ├──→ Partition 0
   ├──→ Partition 1
   └──→ Partition 2
```

Благодаря Partition Kafka может обрабатывать данные параллельно.

---

## 🚀 Producer и производительность

Producer может отправлять сообщения не по одному, а **пакетами (batch)**.

```text
Message 1 ─┐
Message 2 ─┼──→ Batch → Kafka
Message 3 ─┘
```

Batching уменьшает количество сетевых операций и повышает производительность.

Также Producer может использовать сжатие сообщений.

---

## 🛡️ Надёжность доставки

Producer должен учитывать, насколько надёжно сообщение должно быть записано в Kafka.

Одна из важных настроек Kafka — **acks**.

Упрощённо:

```text
acks=0
```

Producer не ждёт подтверждения от Kafka.

Быстрее, но возможна потеря сообщения.

```text
acks=1
```

Producer ждёт подтверждения от Leader Partition.

```text
acks=all
```

Producer ждёт подтверждения от всех необходимых реплик согласно настройкам ISR.

Надёжнее, но потенциально выше задержка.

---

## 🔁 Producer Retry

При временной ошибке Producer может повторить отправку сообщения.

```text
Producer
   ↓
Kafka
   ↓
❌ ошибка
   ↓
Retry
   ↓
Kafka
   ↓
✅
```

При использовании retries нужно учитывать возможность появления дубликатов, поэтому важна идемпотентность.

---

## ♻️ Idempotent Producer

**Идемпотентный Producer** помогает избежать дубликатов сообщений при повторных отправках.

Упрощённо:

```text
Producer
   ↓
Message
   ↓
Kafka
   ↓
временная ошибка
   ↓
Retry
```

Без правильной настройки повторная отправка может привести к дубликату.

Kafka поддерживает **idempotent producer**, который помогает сделать запись безопаснее при retries.

---

## 🐍 Producer в Python

Пример с Kafka-клиентом:

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

Здесь:

```text
KafkaProducer
      ↓
orders
      ↓
Kafka
```

---

## 🎯 Producer в микросервисах

Например:

```text
Order Service
      │
      ↓
   Producer
      │
      ↓
    Kafka
      │
      ↓
orders topic
```

После создания заказа:

```text
Order Service
      ↓
order.created
      ↓
Kafka
```

Другие сервисы уже самостоятельно читают это событие.

---

## 🆚 Producer и Consumer

| Producer                 | Consumer                 |
| ------------------------ | ------------------------ |
| Отправляет сообщения     | Получает сообщения       |
| Пишет в Kafka            | Читает из Kafka          |
| Публикует события        | Обрабатывает события     |
| Может задавать key       | Работает с Offset        |
| Не обязан знать Consumer | Не обязан знать Producer |

---

## 🧠 Главное

```text
Producer
   ↓
создаёт сообщение
   ↓
отправляет в Kafka
   ↓
Topic
   ↓
Partition
```

Producer отвечает именно за **публикацию сообщений**.

### Формула для собеседования

**Producer = создаёт и отправляет сообщения → Kafka Topic → Partition.**

Если спросят **«Зачем Producer нужен в Kafka?»**:

> **Producer — это клиент, который публикует сообщения в Kafka. Он отправляет события в Topic, может задавать key для выбора Partition, а также настраивать подтверждение доставки, retries, batching и идемпотентность.**
