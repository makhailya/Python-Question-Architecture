# Publisher Return в RabbitMQ ↩️

## 🎯 Ответ на собеседовании

**Publisher Return — механизм RabbitMQ, позволяющий Producer узнать, что опубликованное сообщение не удалось маршрутизировать в Queue.**

Например, Producer отправил сообщение:

```python
routing_key = "order.created"
```

но у Exchange нет подходящего Binding.

Тогда сообщение может быть возвращено Producer'у через механизм **Publisher Returns**, если публикация выполнена с `mandatory=True`.

Главное различие:

* **Publisher Confirm** → RabbitMQ подтверждает принятие публикации;
* **Publisher Return** → RabbitMQ сообщает, что сообщение не удалось маршрутизировать в Queue.

---

## 🎤 Суперкоротко

```text
Producer
   │
   │ publish
   ▼
Exchange
   │
   ├── есть подходящий Binding → Queue
   │
   └── нет Binding + mandatory=True
                  ↓
              Publisher Return
                  ↓
               Producer
```

**Confirm ≠ Return.**

---

# Зачем нужен Publisher Return

Представим:

```text
Producer
   ↓
Exchange
   ↓
❌ нет подходящего Queue
```

Producer отправил сообщение, но оно фактически не попало ни в одну Queue.

Без обработки этой ситуации приложение может считать публикацию успешной и потерять событие с точки зрения бизнес-логики.

Publisher Return позволяет обнаружить такую проблему.

---

# Что такое unroutable message

**Unroutable message** — сообщение, которое Exchange не смог направить ни в одну Queue.

Например:

```text
Exchange: events

Bindings:
order.created → orders_queue
payment.*    → payments_queue
```

Producer отправляет:

```python
routing_key = "user.created"
```

Подходящего Binding нет:

```text
user.created
      ↓
   Exchange
      ↓
❌ no matching binding
```

Сообщение является **unroutable**.

---

# `mandatory`

Чтобы RabbitMQ вернул unroutable message Producer'у, при публикации используется:

```python
mandatory=True
```

Концептуально:

```python
channel.basic_publish(
    exchange="events",
    routing_key="user.created",
    body=b"event",
    mandatory=True,
)
```

Если сообщение не удалось маршрутизировать, Producer получает Return.

---

# Что происходит без `mandatory`

Упрощённо:

```text
Producer
   ↓
Exchange
   ↓
нет подходящего Binding
   ↓
сообщение не маршрутизировано
```

Producer не получает Publisher Return.

Поэтому для критичных сообщений важно продумать обработку unroutable сообщений.

---

# Что возвращает RabbitMQ

При Publisher Return Producer получает информацию о возвращённом сообщении.

В частности, можно получить:

* код ответа;
* текст причины;
* Exchange;
* Routing Key;
* само сообщение.

Например, причина может быть связана с отсутствием подходящего маршрута.

---

# Publisher Return vs Publisher Confirm

Это один из самых важных вопросов.

## Publisher Confirm

Отвечает:

> **RabbitMQ принял публикацию?**

Схема:

```text
Producer
   ↓
publish
   ↓
RabbitMQ
   ↓
Confirm
   ↓
Producer
```

## Publisher Return

Отвечает:

> **Удалось ли маршрутизировать сообщение в Queue?**

Схема:

```text
Producer
   ↓
Exchange
   ↓
нет маршрута
   ↓
Return
   ↓
Producer
```

---

# Почему Confirm и Return нужны одновременно

Эти механизмы проверяют разные этапы.

Например:

```text
Producer
   │
   │ publish
   ▼
RabbitMQ
   │
   │ Confirm
   ▼
Producer
```

RabbitMQ может подтвердить публикацию, но сообщение при этом не иметь подходящего маршрута.

Поэтому:

```text
Confirm
   ≠
Message routed to Queue
```

Для контроля публикации важно понимать оба уровня.

---

# Полная схема

```text
                         Producer
                            │
                            │ publish
                            ▼
                       ┌─────────┐
                       │ RabbitMQ│
                       └────┬────┘
                            │
                         Exchange
                            │
                 ┌──────────┴──────────┐
                 │                     │
          есть Binding             нет Binding
                 │                     │
                 ▼                     ▼
               Queue                Return
                 │                     │
                 ▼                     ▼
             Consumer              Producer
```

---

# Return и ACK — разные вещи

Не путай Publisher Return с Consumer ACK.

### Publisher Return

Относится к:

```text
Producer → RabbitMQ
```

и проблемам маршрутизации.

### Consumer ACK

Относится к:

```text
RabbitMQ → Consumer
```

и подтверждает обработку сообщения.

Схема:

```text
Producer
   │
   │ publish
   ▼
RabbitMQ
   │
   ▼
Consumer
   │
   │ ACK
   ▼
RabbitMQ
```

---

# Return и Publisher Confirm

Можно представить RabbitMQ как несколько этапов:

```text
1. Producer публикует
        ↓
2. RabbitMQ принимает публикацию
        ↓
3. Exchange маршрутизирует
        ↓
4. Queue получает сообщение
        ↓
5. Consumer получает сообщение
        ↓
6. Consumer обрабатывает
        ↓
7. Consumer отправляет ACK
```

Разные механизмы контролируют разные этапы:

| Этап                        | Механизм          |
| --------------------------- | ----------------- |
| Принятие публикации         | Publisher Confirm |
| Невозможность маршрутизации | Publisher Return  |
| Успешная обработка Consumer | Consumer ACK      |

---

# Пример с ошибкой Routing Key

Есть:

```text
Exchange:
events
```

Binding:

```text
order.created → orders_queue
```

Producer ошибся:

```python
routing_key = "orders.created"
```

Получаем:

```text
orders.created
       ↓
    Exchange
       ↓
❌ нет подходящего Binding
       ↓
Publisher Return
```

Это позволяет обнаружить ошибку в routing key.

---

# Return не означает, что Consumer не обработал сообщение

Publisher Return происходит **до того, как сообщение попадёт в Queue**.

Поэтому это не ошибка обработки Consumer.

Нельзя путать:

```text
unroutable
```

и:

```text
processing failed
```

### Unroutable

```text
Exchange
   ↓
❌ Queue не найдена по маршруту
```

### Processing failed

```text
Exchange
   ↓
Queue
   ↓
Consumer
   ↓
❌ ошибка обработки
```

Во втором случае используются:

* NACK;
* Reject;
* Retry;
* DLX;
* DLQ.

---

# Return и Retry

Publisher Return может быть причиной для повторной публикации, если проблема временная или исправимая.

Например:

```text
Producer
   ↓
publish
   ↓
Exchange
   ↓
Return
   ↓
исправить routing/config
   ↓
publish снова
```

Но нельзя автоматически бесконечно повторять публикацию.

Иначе можно получить:

```text
Return
 ↓
Retry
 ↓
Return
 ↓
Retry
 ↓
...
```

Нужны:

* ограничение количества попыток;
* логирование;
* мониторинг;
* корректная обработка причины.

---

# Return и Idempotency

Если Producer повторно публикует сообщение после ошибки или неопределённого результата, могут появиться дубликаты.

Поэтому бизнес-обработка должна учитывать:

```text
Retry
+
At-least-once delivery
=
возможны дубликаты
```

Для критичных операций полезны:

* Idempotency Key;
* уникальные ограничения в БД;
* Deduplication.

---

# Publisher Return и Publisher Confirm вместе

Надёжная публикация может выглядеть примерно так:

```text
Producer
   │
   │ publish(mandatory=True)
   ▼
Exchange
   │
   ├──→ Queue
   │
   └──→ Return при отсутствии маршрута
   │
   ▼
Publisher Confirm
```

При этом важно понимать:

**Confirm не является подтверждением бизнес-обработки сообщения.**

Он не означает:

```text
Consumer обработал сообщение
```

---

# Publisher Confirm + Return + Consumer ACK

Полная картина RabbitMQ:

```text
                  Producer
                     │
                     │ publish
                     ▼
                 Exchange
                  │     │
         routed   │     │ unroutable
                  │     └────────→ Return
                  ▼
                Queue
                  │
                  ▼
               Consumer
                  │
               processing
                  │
                  ▼
                 ACK
```

Параллельно Producer может получить:

```text
Publisher Confirm
```

как подтверждение публикации RabbitMQ.

---

# Типичный backend-сценарий

Допустим, сервис заказов создаёт заказ в PostgreSQL и публикует событие:

```text
Order Service
     │
     ├── PostgreSQL
     │      └── order created
     │
     └── RabbitMQ
            └── order.created
```

Producer использует:

```text
Publisher Confirm
+
Publisher Return
```

чтобы контролировать публикацию.

Но здесь появляется важная проблема:

```text
DB COMMIT
   ↓
RabbitMQ publish
```

Если приложение упало между этими операциями:

```text
DB → успешно
RabbitMQ → событие не опубликовано
```

Это **dual-write problem**.

Publisher Confirm и Return сами по себе эту проблему не решают.

Для этого используют, например:

```text
Transactional Outbox
```

---

# Publisher Return vs DLQ

Это тоже разные механизмы.

### Publisher Return

Сообщение:

```text
Producer
   ↓
Exchange
   ↓
❌ не маршрутизировано
   ↓
Producer
```

### DLQ

Сообщение уже находится в системе обработки:

```text
Queue
   ↓
Consumer
   ↓
❌ processing failed
   ↓
DLX
   ↓
DLQ
```

То есть:

```text
Return → проблема маршрутизации

DLQ → проблема обработки / жизненного цикла сообщения
```

---

# Частые вопросы на собеседовании

### Что такое Publisher Return?

> Механизм RabbitMQ, позволяющий Producer получить сообщение обратно, если оно не удалось маршрутизировать в Queue.

### Когда возникает Return?

Например, когда нет подходящего Binding и публикация выполнена с `mandatory=True`.

### Что такое `mandatory=True`?

> Флаг, который заставляет RabbitMQ вернуть Producer'у сообщение, если его невозможно маршрутизировать в Queue.

### Publisher Confirm и Return — одно и то же?

Нет.

> **Confirm подтверждает принятие публикации RabbitMQ, Return сообщает о невозможности маршрутизации.**

### Return означает ошибку Consumer?

Нет.

Return возникает на этапе маршрутизации, до обработки Consumer.

### Return и Consumer ACK — одно и то же?

Нет.

* Return → Producer получает информацию о проблеме маршрутизации.
* ACK → RabbitMQ получает подтверждение успешной обработки от Consumer.

### Можно ли использовать Return для Retry?

Да, но Retry должен быть ограниченным и учитывать причину ошибки.

### Решает ли Publisher Confirm проблему dual-write?

Нет.

Для согласованной записи в БД и публикации события используют отдельные паттерны, например Transactional Outbox.

---

# 🧠 Итог

```text
Publisher Confirm
    ↓
RabbitMQ принял публикацию?

Publisher Return
    ↓
Сообщение не удалось маршрутизировать?

Consumer ACK
    ↓
Consumer успешно обработал?
```

### Формула для собеседования

> **Publisher Confirm подтверждает Producer'у принятие публикации RabbitMQ, а Publisher Return сообщает о невозможности маршрутизировать сообщение в Queue. Return обычно используется вместе с `mandatory=True`. Confirm и Return решают разные задачи и не гарантируют успешную бизнес-обработку сообщения Consumer'ом.**
