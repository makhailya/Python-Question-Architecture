# 📤 Outbox Pattern

## 🎯 Ответ на собеседовании

**Outbox Pattern** — это паттерн надёжной публикации событий из базы данных в брокер сообщений.

Идея: **изменение бизнес-данных и запись события в специальную Outbox-таблицу выполняются в одной локальной транзакции БД**. Затем отдельный процесс читает события из Outbox и отправляет их в брокер.

Это позволяет избежать ситуации, когда данные в БД сохранились, а сообщение в [[Kafka]]/[[RabbitMQ]] не отправилось.

---

## 🎤 Суперкоротко

> Outbox Pattern решает проблему атомарности между БД и брокером: бизнес-изменение и событие записываются в одну транзакцию БД, после чего отдельный воркер публикует событие в брокер.

---

## 🔹 Какая проблема решается

Представим:

```text
Order Service
     ↓
PostgreSQL
     ↓
Kafka
```

Нужно:

1. Создать заказ в PostgreSQL.
2. Отправить `OrderCreated` в Kafka.

Наивный вариант:

```python
create_order()
publish_event()
```

Но между этими операциями может произойти ошибка:

```text
create_order()
     ↓
SUCCESS
     ↓
❌ Kafka недоступна
     ↓
publish_event() failed
```

В результате:

```text
PostgreSQL → заказ создан ✅
Kafka      → события нет ❌
```

Сервисы, подписанные на событие, не узнают о создании заказа.

---

# 🔹 Как работает Outbox

Добавляем специальную таблицу:

```text
PostgreSQL
├── orders
└── outbox
```

При создании заказа:

```text
BEGIN
   ↓
Создать заказ
   ↓
Записать событие в outbox
   ↓
COMMIT
```

Обе операции находятся **в одной транзакции**.

Например:

```python
with transaction():
    order = create_order()

    create_outbox_event(
        event_type="OrderCreated",
        payload={"order_id": order.id}
    )
```

Если транзакция успешно завершилась:

```text
orders       → запись есть ✅
outbox       → событие есть ✅
```

Если произошла ошибка:

```text
orders       → ROLLBACK ❌
outbox       → ROLLBACK ❌
```

---

# 🔹 Таблица Outbox

Упрощённый пример:

```python
CREATE TABLE outbox (
    id UUID PRIMARY KEY,
    event_type VARCHAR(100),
    payload JSONB,
    created_at TIMESTAMP,
    processed_at TIMESTAMP NULL
);
```

В ней могут храниться:

| Поле           | Назначение                 |
| -------------- | -------------------------- |
| `id`           | уникальный ID события      |
| `event_type`   | тип события                |
| `payload`      | данные события             |
| `created_at`   | время создания             |
| `processed_at` | время публикации/обработки |

---

# 🔹 Отправка события

Отдельный worker периодически читает Outbox:

```text
PostgreSQL
    ↓
Outbox
    ↓
Worker
    ↓
Kafka
    ↓
Consumer
```

Например:

```python
events = get_unprocessed_events()

for event in events:
    publish_to_kafka(event)

    mark_as_processed(event)
```

Если Kafka временно недоступна:

```text
Outbox
   ↓
Kafka ❌
   ↓
событие остаётся в Outbox
   ↓
повторная попытка
```

Поэтому событие не теряется только из-за временной недоступности брокера.

---

# 🔹 Почему это называется Transactional Outbox

Потому что:

```text
Business Data
      +
Outbox Event
      ↓
Одна транзакция БД
```

Это ключевая идея паттерна.

Например:

```python
BEGIN;

INSERT INTO orders (...);

INSERT INTO outbox (
    event_type,
    payload
) VALUES (
    'OrderCreated',
    '...'
);

COMMIT;
```

База гарантирует атомарность этих двух операций.

---

# 🔹 Проблема двойной отправки

Есть важный нюанс.

Worker может сделать:

```text
1. Отправить событие в Kafka ✅
2. ❌ Упасть до отметки processed
```

После перезапуска:

```text
3. Прочитать то же событие
4. Отправить его снова
```

Получится:

```text
Kafka:
OrderCreated
OrderCreated
```

То есть Outbox обычно обеспечивает не строгий `exactly-once`, а **at-least-once delivery**.

Поэтому consumer должен быть **идемпотентным**.

---

# 🔹 Outbox + Idempotency

Типичная схема:

```text
Database
   ↓
Outbox
   ↓
Publisher
   ↓
Kafka
   ↓
Consumer
   ↓
Idempotent processing
```

Consumer может хранить ID уже обработанных событий:

```python
if event.id in processed_events:
    return

process_event(event)

save_processed_event(event.id)
```

Повторная доставка тогда не приведёт к повторному бизнес-действию.

---

# 🔹 Outbox и Saga

Outbox особенно полезен вместе с **Saga**.

Например:

```text
Order Service
     ↓
Local Transaction
     ├── create order
     └── write OrderCreated → Outbox
                         ↓
                       Kafka
                         ↓
                  Payment Service
                         ↓
                  Local Transaction
                         ├── charge payment
                         └── write PaymentCompleted → Outbox
```

Каждый сервис использует **свою локальную транзакцию** и свой Outbox.

```text
Order DB
   ↓
Order Outbox
   ↓
Kafka
   ↓
Payment DB
   ↓
Payment Outbox
   ↓
Kafka
```

---

# 🔹 Outbox vs 2PC

Это важное сравнение на собеседовании.

| Outbox                              | 2PC                               |
| ----------------------------------- | --------------------------------- |
| Использует локальные транзакции     | Распределённая транзакция         |
| БД + Outbox в одной транзакции      | Coordinator управляет участниками |
| Асинхронная публикация              | Синхронный протокол               |
| Eventual consistency                | Более сильная согласованность     |
| Не требует распределённого `COMMIT` | Требует Prepare/Commit            |
| Хорошо подходит для микросервисов   | Более тяжёлый и дорогой механизм  |

Outbox не делает БД и Kafka одной транзакцией.

Он делает надёжной связку:

```text
DB transaction
     ↓
Outbox
     ↓
асинхронная доставка
     ↓
Broker
```

---

# 🔹 Outbox vs прямой publish

### Без Outbox

```text
BEGIN
 ↓
UPDATE DB
 ↓
COMMIT
 ↓
publish Kafka
 ↓
❌ ошибка
```

Событие потеряно.

### С Outbox

```text
BEGIN
 ↓
UPDATE DB
 ↓
INSERT Outbox
 ↓
COMMIT
 ↓
Worker
 ↓
Kafka
```

Если Kafka недоступна:

```text
Outbox
 ↓
retry
 ↓
Kafka
```

---

# 🔹 Polling Publisher

Самый простой вариант — периодически опрашивать таблицу:

```text
каждые N секунд
       ↓
SELECT events FROM outbox
WHERE processed_at IS NULL
       ↓
publish
       ↓
mark processed
```

Плюсы:

* простая реализация;
* легко контролировать retry;
* не требует дополнительной инфраструктуры.

Минус:

* задержка между записью события и публикацией;
* дополнительная нагрузка на БД.

---

# 🔹 CDC / Debezium

Другой вариант — использовать **Change Data Capture (CDC)**.

Например:

```text
PostgreSQL
    ↓
WAL
    ↓
Debezium
    ↓
Kafka
```

Debezium отслеживает изменения в БД и публикует их в Kafka.

В таком случае не обязательно постоянно делать polling Outbox-таблицы.

---

# ⚠️ Что важно помнить

Outbox **не гарантирует**, что consumer обработает событие ровно один раз.

Возможна повторная доставка:

```text
Event
 ↓
Kafka
 ↓
Consumer
 ↓
обработка
 ↓
❌ consumer упал
 ↓
повторная доставка
```

Поэтому обычно нужны:

* **idempotency**;
* уникальный `event_id`;
* retry;
* обработка ошибок;
* иногда Dead Letter Queue.

---

# 🔥 Главное

```text
Бизнес-изменение
       +
Outbox Event
       ↓
Одна локальная транзакция БД
       ↓
     Outbox
       ↓
    Publisher
       ↓
 Kafka / RabbitMQ
       ↓
    Consumer
```

**Ключевая фраза для собеседования:**

> Outbox Pattern позволяет надёжно публиковать события из микросервиса. Изменение бизнес-данных и запись события в Outbox выполняются в одной локальной транзакции БД. После этого отдельный publisher отправляет событие в брокер. Это предотвращает потерю события при сбое между изменением БД и публикацией в брокер. При этом возможна повторная доставка, поэтому consumer должен быть идемпотентным.
