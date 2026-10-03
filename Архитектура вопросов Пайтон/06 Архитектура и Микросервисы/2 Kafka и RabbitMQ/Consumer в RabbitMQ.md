# Consumer в RabbitMQ 👷

## 🎯 Ответ на собеседовании

**Consumer — это приложение или процесс, который получает сообщения из RabbitMQ Queue и выполняет их обработку.**

Consumer подключается к RabbitMQ, подписывается на Queue и получает сообщения. После успешной обработки он подтверждает сообщение через **ACK**.

В RabbitMQ несколько Consumers могут одновременно читать одну Queue, что позволяет масштабировать обработку.

Основная схема:

```python
Producer
    ↓
Exchange
    ↓
Queue
    ↓
Consumer
    ↓
Business Logic
    ↓
ACK
```

**Главное:** Consumer отвечает за обработку сообщения, а RabbitMQ — за доставку и управление очередью.

---

## 🎤 Суперкоротко

> **Consumer читает сообщения из Queue, обрабатывает их и подтверждает успешную обработку через ACK.**

```text
Queue → Consumer → обработка → ACK
```

---

# Что такое Consumer

Consumer — это приложение, которое получает сообщения из Queue.

Например:

```python
RabbitMQ
   ↓
orders_queue
   ↓
Order Worker
```

Worker является Consumer'ом.

Он получает:

```python
OrderCreated
```

и выполняет бизнес-логику:

```python
create_order()
```

После успешной обработки:

```python
ACK
```

---

# Producer и Consumer

Producer и Consumer выполняют противоположные задачи.

### Producer

Создаёт и отправляет сообщения.

```text
Producer
    ↓
message
```

### Consumer

Получает и обрабатывает сообщения.

```text
message
    ↓
Consumer
    ↓
business logic
```

Между ними находится RabbitMQ:

```text
Producer
   ↓
Exchange
   ↓
Queue
   ↓
Consumer
```

Producer и Consumer не обязаны напрямую знать друг о друге.

---

# Как Consumer получает сообщение

Типичный процесс:

```text
1. Consumer подключается к RabbitMQ
2. Consumer подписывается на Queue
3. RabbitMQ доставляет сообщение
4. Consumer обрабатывает сообщение
5. Consumer отправляет ACK
```

Схема:

```text
RabbitMQ
   ↓
Queue
   ↓
Consumer
   ↓
обработка
   ↓
ACK
```

---

# Consumer не удаляет сообщение самостоятельно

Consumer не должен просто считать:

> «Я получил сообщение — значит, оно обработано».

Получение и успешная обработка — разные вещи.

Например:

```text
Queue
   ↓
Consumer
   ↓
получил Message
   ↓
обработка
   ↓
ERROR
```

В этом случае ACK отправлять нельзя, если задача не выполнена.

---

# ACK

После успешной обработки:

```text
Message
   ↓
Consumer
   ↓
Business Logic
   ↓
SUCCESS
   ↓
ACK
```

RabbitMQ получает подтверждение.

```text
ACK = сообщение успешно обработано
```

Если Consumer упадёт до ACK, сообщение может быть доставлено повторно.

---

# At-least-once

RabbitMQ при использовании manual acknowledgements часто используют в модели **at-least-once delivery**.

Это означает:

> Сообщение должно быть доставлено как минимум один раз, но при сбое возможно повторное получение.

Например:

```text
Message
   ↓
Consumer
   ↓
DB update
   ↓
COMMIT
   ↓
💥 Consumer crash
```

ACK не успел уйти.

RabbitMQ может доставить сообщение повторно:

```text
Message
   ↓
Consumer 2
```

Поэтому Consumer должен учитывать возможность дубликатов.

---

# Идемпотентный Consumer

Если сообщение может прийти повторно, обработка должна быть безопасной при повторном выполнении.

Например:

```text
Message ID = 123
```

Первый раз:

```text
123 → обработан
```

Повторно:

```text
123 → уже обработан
```

Consumer не должен повторно выполнять опасную операцию.

Для этого можно использовать:

* уникальный `message_id`;
* `idempotency_key`;
* UNIQUE constraint;
* таблицу обработанных сообщений;
* UPSERT.

---

# Несколько Consumers

Одну Queue могут обслуживать несколько Consumers.

```text
                Queue
             /    |    \
            ↓     ↓     ↓
           C1    C2    C3
```

RabbitMQ распределяет сообщения между ними.

Например:

```text
M1 → C1
M2 → C2
M3 → C3
M4 → C1
M5 → C2
M6 → C3
```

Это позволяет масштабировать обработку.

---

# Competing Consumers

Такая модель называется **Competing Consumers**.

Несколько workers конкурируют за сообщения одной Queue.

```text
                Queue
                  │
        ┌─────────┼─────────┐
        ↓         ↓         ↓
       C1        C2        C3
```

Каждое сообщение обычно обрабатывается одним Consumer'ом.

Это отличается от Fanout-сценария.

---

# Один Queue и несколько сервисов

Если несколько независимых сервисов должны получить **каждую копию события**, обычно используют разные Queue.

Например:

```text
                  Exchange
                     │
          ┌──────────┼──────────┐
          ↓          ↓          ↓
      email_queue  crm_queue  analytics_queue
          ↓          ↓          ↓
      Email Svc    CRM Svc   Analytics Svc
```

Если же несколько workers должны **делить работу**, они могут использовать одну Queue:

```text
orders_queue
     │
 ┌───┼───┐
 ↓   ↓   ↓
C1  C2  C3
```

Это важное различие.

---

# Consumer и Prefetch

Consumer работает вместе с Prefetch.

Например:

```python
prefetch = 1
```

Consumer получает:

```text
M1
```

После:

```python
ACK(M1)
```

RabbitMQ может передать:

```text
M2
```

При:

```python
prefetch = 10
```

Consumer может иметь несколько неподтверждённых сообщений.

```text
C1:
M1
M2
M3
...
M10
```

Поэтому:

```text
Prefetch → сколько сообщений Consumer может держать без ACK
```

---

# Consumer и NACK

Если обработка завершилась ошибкой:

```text
Message
   ↓
Consumer
   ↓
ERROR
```

Consumer может отправить:

```text
NACK
```

Например:

```text
NACK + requeue
```

Сообщение вернётся в Queue.

Или:

```text
NACK + no requeue
```

При настроенном DLX сообщение может перейти в dead-letter flow.

---

# Consumer и Retry

Типичная цепочка:

```text
Queue
   ↓
Consumer
   ↓
ERROR
   ↓
Retry
   ↓
Consumer
```

Если ошибка временная:

```text
API timeout
```

повторная попытка может помочь.

Если ошибка постоянная:

```text
Invalid message
```

лучше после ограниченного числа попыток отправить сообщение в DLQ.

---

# Consumer и DLQ

Полная схема:

```text
Main Queue
    ↓
Consumer
    ↓
ERROR
    ↓
Retry
    ↓
Retry
    ↓
Retry
    ↓
DLQ
```

Consumer не должен бесконечно повторять неудачную операцию.

---

# Consumer и транзакция БД

Очень важная последовательность:

```text
Message
   ↓
Consumer
   ↓
BEGIN
   ↓
DB operation
   ↓
COMMIT
   ↓
ACK
```

Если DB operation завершилась ошибкой:

```text
Message
   ↓
Consumer
   ↓
BEGIN
   ↓
DB operation
   ↓
ERROR
   ↓
ROLLBACK
   ↓
Retry / NACK
```

Так мы не подтверждаем сообщение до успешного завершения бизнес-операции.

---

# Что будет при падении Consumer

Допустим:

```text
Queue
   ↓
Consumer 1
   ↓
M1
   ↓
M2
   ↓
💥 Consumer 1
```

Если сообщения не были ACK:

```text
M1
M2
```

могут быть повторно доставлены.

Если есть другой Consumer:

```text
Queue
   ↓
Consumer 2
```

он сможет продолжить обработку.

Это одна из причин, почему Queue используется как буфер между producers и workers.

---

# Consumer и RabbitMQ Connection

В production Consumer обычно поддерживает соединение с RabbitMQ и получает сообщения через канал.

Концептуально:

```text
Consumer Application
       │
       │ TCP connection
       ▼
    RabbitMQ
       │
      Channel
       │
       ▼
      Queue
```

**Connection** — сетевое соединение с RabbitMQ.

**Channel** — логический канал внутри connection.

---

# Connection vs Channel

На собеседовании достаточно понимать:

```text
Application
    ↓
Connection
    ↓
Channel
    ↓
Queue
```

Обычно не создают отдельное TCP-соединение на каждое сообщение.

Используются долгоживущие соединения и каналы.

---

# Consumer и балансировка

Несколько Consumers позволяют горизонтально масштабировать обработку:

```text
1 Consumer
    ↓
100 msg/s
```

Можно добавить workers:

```text
3 Consumers
    ↓
потенциально больше throughput
```

Но это не означает автоматическое трёхкратное ускорение.

Узким местом может быть:

```text
Consumer
   ↓
PostgreSQL
```

или:

```text
Consumer
   ↓
External API
```

Поэтому масштабировать нужно всю цепочку.

---

# Consumer и Queue Depth

Для мониторинга полезно смотреть:

```text
Ready
Unacked
Consumer Count
```

Например:

```text
Ready = 10 000
Unacked = 30
Consumers = 3
```

Если Ready постоянно растёт:

```text
1000
2000
5000
10000
```

это может означать, что Consumer'ы не успевают обрабатывать сообщения.

Тогда нужно анализировать:

* скорость Producer;
* количество Consumers;
* Prefetch;
* время обработки;
* БД;
* внешние API.

---

# Consumer Lag в RabbitMQ

В RabbitMQ обычно не используют термин **Consumer Lag** так же, как в Kafka.

Вместо этого часто смотрят на:

```text
Ready messages
Unacked messages
Queue depth
```

Для Kafka характерен:

```text
Consumer Lag
```

Для RabbitMQ полезнее думать:

```text
Queue depth
```

---

# RabbitMQ Consumer vs Kafka Consumer

|                 | RabbitMQ Consumer         | Kafka Consumer              |
| --------------- | ------------------------- | --------------------------- |
| Получает данные | Из Queue                  | Из Partition                |
| Модель          | Broker push / delivery    | Consumer pull               |
| Позиция         | ACK state                 | Offset                      |
| Подтверждение   | ACK/NACK                  | Commit offset               |
| Replay          | Не основная модель        | Основная возможность        |
| Масштабирование | Несколько consumers Queue | Consumer Group + partitions |
| Хранение        | Queue                     | Event log по retention      |

---

# Пример на Python

Упрощённый Consumer:

```python
def process_message(message):
    try:
        process_business_logic(message)
        ack(message)
    except TemporaryError:
        nack(message, requeue=True)
    except PermanentError:
        reject(message, requeue=False)
```

Логика:

```text
успех
  ↓
ACK

временная ошибка
  ↓
Retry / requeue

постоянная ошибка
  ↓
DLQ
```

В реальном приложении retry, транзакции, DLQ и идемпотентность обычно реализуются более аккуратно.

---

# 🎯 Частые вопросы на собеседовании

### Что такое Consumer?

Приложение или процесс, который получает сообщения из Queue и обрабатывает их.

### Может ли быть несколько Consumers у одной Queue?

Да. Они могут совместно обрабатывать сообщения.

### Что такое Competing Consumers?

Несколько Consumers, которые совместно потребляют сообщения из одной Queue.

### Что произойдёт при падении Consumer?

Неподтверждённые сообщения могут быть повторно доставлены.

### Зачем нужен ACK?

Чтобы подтвердить успешную обработку сообщения.

### Что такое Prefetch?

Количество сообщений, которые Consumer может получить до подтверждения предыдущих.

### Почему Consumer должен быть идемпотентным?

Потому что сообщение может быть доставлено повторно.

### Как обрабатывать временные ошибки?

Использовать Retry с ограниченным количеством попыток и backoff.

### Что делать с постоянными ошибками?

После заданного количества попыток отправлять сообщение в DLQ.

### Как несколько сервисов могут получить одно событие?

Использовать отдельные Queue, связанные с одним Exchange.

### Чем RabbitMQ Consumer отличается от Kafka Consumer?

RabbitMQ Consumer получает сообщения из Queue и подтверждает обработку ACK, а Kafka Consumer читает записи из partitions и управляет своим прогрессом через offsets.

---

# 🧠 Итоговая схема

```text
                    Producer
                       │
                       ▼
                   Exchange
                       │
                       ▼
                     Queue
                       │
             ┌─────────┼─────────┐
             ↓         ↓         ↓
            C1        C2        C3
             │         │         │
             └─────────┼─────────┘
                       ↓
                  обработка
                       │
                ┌──────┴──────┐
                ↓             ↓
              успех         ошибка
                ↓             ↓
               ACK        Retry / NACK
                              │
                         после N попыток
                              ↓
                             DLQ
```

### Формула для собеседования

> **Consumer получает сообщение из Queue, выполняет бизнес-логику и после успешной обработки отправляет ACK. Несколько Consumers могут совместно обрабатывать одну Queue. При ошибках используются NACK, Retry и DLQ. Поскольку возможна повторная доставка, Consumer должен быть идемпотентным.**

```text
Queue
  ↓
Consumer
  ↓
обработка
  ↓
ACK

ERROR
  ↓
Retry
  ↓
DLQ
```
