# 🐇 RabbitMQ

## 🎯 Ответ на собеседовании

**RabbitMQ** — это брокер сообщений, который позволяет сервисам обмениваться сообщениями асинхронно.

Producer отправляет сообщение в RabbitMQ, RabbitMQ маршрутизирует его в очередь, а Consumer получает сообщение и обрабатывает его.

```text
Producer
    ↓
RabbitMQ
    ↓
Queue
    ↓
Consumer
```

RabbitMQ часто используется в микросервисной архитектуре и, например, вместе с Celery для выполнения фоновых задач.

---

## 🎤 Суперкоротко

**RabbitMQ — это брокер сообщений. Producer отправляет сообщение в Exchange, Exchange маршрутизирует его в Queue, а Consumer забирает сообщение из очереди и обрабатывает его.**

---

## 🧩 Основные компоненты

Главные понятия RabbitMQ:

```text
Producer
   ↓
Exchange
   ↓
Queue
   ↓
Consumer
```

### Producer

**Producer** — приложение, которое публикует сообщение.

Например:

```text
FastAPI
   ↓
"Отправить email пользователю"
```

---

### Exchange

**Exchange** — компонент RabbitMQ, который принимает сообщения от Producer и определяет, в какие очереди их направить.

Важно:

**Producer обычно отправляет сообщение не напрямую в Queue, а в Exchange.**

```text
Producer
   ↓
Exchange
   ↓
Queue
```

---

### Queue

**Queue** — очередь, в которой сообщения ожидают обработки Consumer'ом.

```text
Queue:

[Message 1]
[Message 2]
[Message 3]
```

Consumer забирает сообщения из очереди.

---

### Consumer

**Consumer** — приложение, которое получает сообщения из Queue и выполняет их обработку.

```text
Queue
  ↓
Consumer
  ↓
обработка
```

---

## 🔀 Exchange

Exchange отвечает за **маршрутизацию сообщений**.

Основные типы Exchange:

```text
Direct
Fanout
Topic
Headers
```

---

## 1️⃣ Direct Exchange

Маршрутизация происходит по точному совпадению **routing key**.

```text
Producer
   ↓
Direct Exchange
   │
   ├── routing key: email → Email Queue
   │
   └── routing key: sms   → SMS Queue
```

Например:

```text
routing_key = "email"
```

Сообщение попадёт в очередь, связанную с этим ключом.

---

## 2️⃣ Fanout Exchange

Отправляет сообщение **во все связанные очереди**.

```text
              ┌──→ Queue 1
              │
Producer → Fanout Exchange
              │
              ├──→ Queue 2
              │
              └──→ Queue 3
```

Routing key при такой маршрутизации не играет основной роли.

Используется для **broadcast / Pub/Sub** сценариев.

---

## 3️⃣ Topic Exchange

Маршрутизация происходит по шаблонам routing key.

Например:

```text
user.created
user.deleted
order.created
order.paid
```

Можно подписаться на определённый шаблон.

Например:

```text
user.*
```

получит:

```text
user.created
user.deleted
```

Но не:

```text
order.created
```

---

## 4️⃣ Headers Exchange

Маршрутизация выполняется на основе **headers сообщения**, а не routing key.

Используется реже.

---

## 🔗 Binding

**Binding** — связь между Exchange и Queue.

```text
Exchange
    │
    │ Binding
    ↓
  Queue
```

Binding определяет правила, по которым сообщения попадают из Exchange в Queue.

---

## 🔑 Routing Key

**Routing key** — строка, которую Producer передаёт вместе с сообщением и которая используется Exchange для маршрутизации.

Например:

```text
order.created
```

Для Direct:

```text
"email"
```

Для Topic:

```text
"order.created"
```

---

## 📨 Полный путь сообщения

Это одна из самых важных схем для собеседования:

```text
Producer
   │
   │ message + routing key
   ↓
Exchange
   │
   │ routing
   ↓
Queue
   │
   ↓
Consumer
```

Например:

```text
FastAPI
   │
   │ order.created
   ↓
Topic Exchange
   │
   ├──→ Order Queue
   │
   └──→ Analytics Queue
          ↓
       Consumer
```

---

## ✅ ACK

**ACK (acknowledgement)** — подтверждение от Consumer, что сообщение успешно обработано.

```text
RabbitMQ
   ↓
Message
   ↓
Consumer
   ↓
обработка
   ↓
ACK
   ↓
RabbitMQ
```

После успешного подтверждения RabbitMQ может удалить сообщение из очереди.

---

## ❌ Что если Consumer упал?

Допустим:

```text
Queue
   ↓
Consumer
   ↓
❌ ошибка
```

Если сообщение не было подтверждено ACK и используется соответствующая настройка доставки, RabbitMQ может вернуть сообщение в очередь для повторной обработки.

Поэтому сообщения не обязательно теряются при падении Consumer.

---

## 🔁 Retry

Если обработка завершилась ошибкой, сообщение можно обработать повторно.

```text
Message
   ↓
Consumer
   ↓
Ошибка
   ↓
Retry
   ↓
Consumer
```

После нескольких неудачных попыток сообщение можно отправить в **Dead Letter Queue**.

---

## 💀 Dead Letter Queue

**DLQ (Dead Letter Queue)** — очередь для сообщений, которые не удалось нормально обработать.

```text
Main Queue
    ↓
 Consumer
    ↓
   ❌
    ↓
 Retry
    ↓
   ❌
    ↓
   DLQ
```

DLQ позволяет не блокировать основную обработку и отдельно разбирать проблемные сообщения.

---

## 🚦 Prefetch

**Prefetch** определяет, сколько сообщений RabbitMQ может выдать Consumer'у до получения подтверждений.

Например:

```text
prefetch = 10
```

Consumer может получить до 10 неподтверждённых сообщений.

Это позволяет управлять распределением нагрузки между Consumer'ами.

---

## 👥 Несколько Consumer

Можно запустить несколько экземпляров Consumer:

```text
             ┌──→ Consumer 1
             │
Queue ───────┼──→ Consumer 2
             │
             └──→ Consumer 3
```

RabbitMQ распределяет сообщения между ними.

Это позволяет увеличить производительность обработки.

---

## 🐍 RabbitMQ + Celery

Очень распространённый сценарий в Python.

```text
                 ┌──────────────┐
                 │    FastAPI   │
                 └──────┬───────┘
                        │
                        ↓
                   RabbitMQ
                        │
                        ↓
                 Celery Worker
                        │
                        ↓
                  Выполнение
```

Например, пользователь зарегистрировался.

FastAPI:

```text
POST /register
```

После регистрации нужно отправить email.

Вместо ожидания:

```text
FastAPI
   ↓
Email Service
   ↓
response
```

можно:

```text
FastAPI
   ↓
RabbitMQ
   ↓
Celery Worker
   ↓
Отправка email
```

HTTP-запрос возвращается быстрее, а тяжёлая задача выполняется в фоне.

---

## 🆚 RabbitMQ vs Redis

Redis — это прежде всего **in-memory data store**, а RabbitMQ — специализированный **message broker**.

Redis также может использоваться для очередей и сообщений, например через Redis Streams.

Но их архитектурные модели и основные сценарии использования различаются.

---

## 🆚 RabbitMQ vs Kafka

| RabbitMQ                                   | Kafka                                  |
| ------------------------------------------ | -------------------------------------- |
| Message broker                             | Event streaming platform               |
| Queue / Exchange                           | Topic / Partition                      |
| Сильная маршрутизация                      | Высокая пропускная способность         |
| ACK и delivery-модель                      | Consumer offsets                       |
| Хорош для task queues                      | Хорош для event streaming              |
| Сообщения обычно удаляются после обработки | Сообщения хранятся по retention policy |

Упрощённо:

```text
RabbitMQ → задачи и очереди

Kafka → потоки событий и event streaming
```

---

## 🔥 Пример архитектуры

Интернет-магазин:

```text
                     Client
                        ↓
                     FastAPI
                        ↓
                   RabbitMQ
                        ↓
              ┌─────────┼─────────┐
              ↓         ↓         ↓
          Email      Analytics   Billing
          Worker      Worker      Worker
```

После создания заказа FastAPI публикует событие:

```text
order.created
```

Дальше разные Consumer'ы выполняют свои задачи независимо.

---

## ⚠️ Проблемы, которые нужно учитывать

При использовании RabbitMQ важно учитывать:

* повторную доставку сообщений;
* дубликаты;
* идемпотентность Consumer'ов;
* ACK;
* retry;
* DLQ;
* порядок сообщений;
* prefetch;
* мониторинг очередей;
* отказ Consumer;
* переполнение очереди.

### Идемпотентность

Consumer должен корректно переживать повторную обработку одного сообщения.

Например:

```text
Message: payment_created

Consumer получил его дважды
        ↓
Не должен создать две оплаты
```

Поэтому обработчик должен быть **идемпотентным**.

---

## 🧠 Важное различие

Не путай:

```text
RabbitMQ
   ↓
Broker

Exchange
   ↓
маршрутизирует сообщения

Queue
   ↓
хранит сообщения до обработки

Consumer
   ↓
обрабатывает сообщения
```

То есть **Exchange — не очередь**.

Это одна из частых ошибок на собеседовании.

---

## 🎯 Главное

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
    ↓
ACK
```

Основные Exchange:

```text
Direct  → точное совпадение routing key
Fanout  → всем связанным очередям
Topic   → маршрутизация по шаблону
Headers → маршрутизация по headers
```

### Формула для собеседования

**RabbitMQ = Message Broker + Exchange + Queue + Consumer + Routing + ACK**

А типичный Python-сценарий:

**FastAPI → RabbitMQ → Celery Worker → фоновая задача**
