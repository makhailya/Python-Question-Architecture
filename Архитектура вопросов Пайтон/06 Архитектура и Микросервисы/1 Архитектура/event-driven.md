# Event-Driven Architecture (EDA) ⚡

## 🎯 Ответ на собеседовании

**Event-Driven Architecture (EDA)** — архитектурный подход, при котором компоненты системы взаимодействуют через **события (events)**.

Один сервис генерирует событие:

```text
OrderCreated
```

а другие сервисы независимо реагируют на него:

```text
                Order Service
                     │
                     │ OrderCreated
                     ↓
                  Broker
              ┌──────┼──────┐
              ↓      ↓      ↓
          Payment  Email   Analytics
           Service  Service  Service
```

Главная идея:

> **Сервис сообщает, что произошло, а не напрямую указывает другим сервисам, что им делать.**

Это уменьшает связанность (**coupling**) между компонентами и позволяет независимо масштабировать обработчики событий.

---

## 🎤 Суперкоротко

> Event-Driven Architecture — архитектура, где компоненты взаимодействуют через события. Producer публикует событие о произошедшем факте, а consumers независимо реагируют на него. Для доставки событий могут использоваться Kafka, RabbitMQ и другие брокеры.

---

# 1. Что такое Event?

**Event — событие, которое уже произошло.**

Например:

```text
UserRegistered
OrderCreated
PaymentCompleted
FileUploaded
DeliveryCreated
```

Важно различать:

```text
Command:
"Создай заказ"

Event:
"Заказ создан"
```

### Command

Это **намерение выполнить действие**:

```text
CreateOrder
SendEmail
ProcessPayment
```

### Event

Это **факт произошедшего действия**:

```text
OrderCreated
EmailSent
PaymentProcessed
```

---

# 2. Главный принцип EDA

В обычной архитектуре:

```text
Order Service
      │
      │ HTTP
      ↓
Payment Service
      │
      │ HTTP
      ↓
Email Service
```

Order Service знает:

* где находится Payment Service;
* как вызвать его API;
* какой endpoint использовать;
* какой формат запроса нужен.

Получается сильная связанность.

---

## Event-Driven подход

```text
Order Service
      │
      │ OrderCreated
      ↓
    Broker
   ┌──┼──┐
   ↓  ↓  ↓
Payment Email Analytics
```

Order Service не обязан знать, кто будет обрабатывать событие.

Он просто сообщает:

```text
"Заказ создан"
```

---

# 3. Producer

**Producer** — компонент, который публикует событие.

Например:

```text
Order Service
      │
      │ OrderCreated
      ↓
    Broker
```

Producer отвечает за создание и отправку события.

Упрощённо:

```python
event = {
    "type": "OrderCreated",
    "order_id": 123,
}
```

Затем событие отправляется в Kafka/RabbitMQ или другой брокер.

---

# 4. Consumer

**Consumer** — компонент, который получает событие и выполняет свою логику.

Например:

```text
OrderCreated
     ↓
Payment Service
```

Payment Service может:

```text
получить событие
      ↓
создать платёж
      ↓
сохранить результат
```

Другой consumer:

```text
OrderCreated
     ↓
Notification Service
     ↓
отправить email
```

---

# 5. Broker

**Broker** — промежуточная система доставки сообщений/событий.

Примеры:

* Kafka
* RabbitMQ
* NATS
* Amazon SQS/SNS

Схема:

```text
Producer
    ↓
 Broker
    ↓
Consumer
```

Broker позволяет отделить producer от consumers.

---

# 6. Основное преимущество — слабая связанность

Без EDA:

```text
A → HTTP → B
```

A должен знать о B.

С EDA:

```text
A → Event → Broker
              ↓
        ┌─────┼─────┐
        ↓     ↓     ↓
        B     C     D
```

A не обязан напрямую зависеть от B, C и D.

Это называется:

> **Loose Coupling — слабая связанность.**

---

# 7. Один Event — много Consumers

Это одно из ключевых преимуществ.

Например:

```text
OrderCreated
      ↓
    Kafka
      │
 ┌────┼────────┐
 ↓    ↓        ↓
Email Payment Analytics
```

Один producer публикует событие один раз.

Несколько независимых систем могут на него реагировать.

Например:

### Email Service

```text
OrderCreated
     ↓
отправить письмо
```

### Analytics Service

```text
OrderCreated
     ↓
обновить статистику
```

### Loyalty Service

```text
OrderCreated
     ↓
начислить бонусы
```

Order Service не нужно менять каждый раз, когда появляется новый consumer.

---

# 8. Синхронное vs Event-Driven взаимодействие

| Синхронное                    | Event-Driven                      |
| ----------------------------- | --------------------------------- |
| HTTP/gRPC                     | Event + Broker                    |
| Есть прямой вызов             | Прямого вызова может не быть      |
| Часто ждём ответ              | Producer может не ждать обработку |
| Сильнее связанность           | Слабее связанность                |
| Проще понять flow             | Flow сложнее                      |
| Ошибка downstream сразу видна | Ошибка обрабатывается отдельно    |

### Синхронно

```text
A → HTTP → B
A ← response ← B
```

### Event-Driven

```text
A → Event → Broker
             ↓
             B
```

---

# 9. Асинхронность

EDA часто используется вместе с **асинхронным взаимодействием**.

Например:

```text
User
 ↓
Order API
 ↓
создать заказ
 ↓
вернуть response
```

А дальше:

```text
OrderCreated
 ↓
Broker
 ├── Email
 ├── Analytics
 └── Notification
```

API не обязательно ждать завершения всей этой работы.

Это позволяет:

* уменьшить latency основного запроса;
* вынести тяжёлые операции в background processing;
* независимо масштабировать consumers.

---

# 10. Event не обязательно означает Kafka

Это важный момент.

**EDA — архитектурный подход.**

Kafka — конкретная технология.

```text
EDA
├── Kafka
├── RabbitMQ
├── NATS
├── SQS/SNS
└── другие messaging systems
```

Поэтому неправильно говорить:

> EDA = Kafka.

Правильнее:

> Kafka — один из инструментов реализации event-driven архитектуры.

---

# 11. Event-Driven ≠ обязательно Microservices

EDA можно использовать в микросервисах:

```text
Microservice A
      ↓
    Event
      ↓
Microservice B
```

Но можно использовать события и внутри монолита.

Например:

```text
Django Monolith
      ↓
OrderCreated
      ↓
internal event handlers
```

То есть:

```text
EDA ≠ Microservices
```

Они часто используются вместе, но это разные концепции.

---

# 12. Event Choreography

В **choreography** каждый сервис самостоятельно реагирует на события.

Например:

```text
OrderCreated
     ↓
Payment Service
     ↓
PaymentCompleted
     ↓
Order Service
     ↓
OrderConfirmed
```

Никакого центрального orchestrator нет.

Сервисы реагируют на события друг друга.

```text
A
↓ event
B
↓ event
C
↓ event
D
```

### Плюсы

* слабая связанность;
* нет центрального координатора;
* легко добавлять независимых consumers.

### Минусы

При большом количестве сервисов flow становится сложнее понимать:

```text
A → B → C
    ↓
    D → E
        ↓
        F
```

Возникает сложность отслеживания бизнес-процесса.

---

# 13. Event Orchestration

При **orchestration** есть центральный координатор.

```text
             Orchestrator
          ┌──────┼──────┐
          ↓      ↓      ↓
       Payment Email Inventory
```

Orchestrator говорит сервисам, что делать.

Например:

```text
1. Создай платёж
2. Зарезервируй товар
3. Отправь уведомление
```

### Сравнение

```text
Choreography:

A → event → B → event → C

Orchestration:

       Orchestrator
       /     |     \
      A      B      C
```

---

# 14. Eventual Consistency

EDA часто приводит к **eventual consistency** — согласованности данных с некоторой задержкой.

Например:

```text
Order Service
     ↓
OrderCreated
     ↓
Kafka
     ↓
Analytics Service
```

Заказ уже создан:

```text
Order DB:
order = created
```

Но Analytics Service ещё не получил событие.

```text
Analytics DB:
данные пока отсутствуют
```

Через некоторое время:

```text
event processed
      ↓
Analytics DB updated
```

То есть система становится согласованной **не мгновенно, а со временем**.

---

# 15. Проблема повторной доставки

В распределённых системах событие может быть обработано повторно.

Например:

```text
OrderCreated
      ↓
Consumer
      ↓
обработал событие
      ↓
❌ не успел подтвердить offset
      ↓
Consumer restart
      ↓
OrderCreated снова
```

Получается:

```text
event → обработан
event → обработан повторно
```

Поэтому consumers часто должны быть **идемпотентными**.

Например:

```text
event_id = abc123
```

Перед обработкой:

```text
если abc123 уже обработан
    → пропустить
иначе
    → обработать
    → сохранить event_id
```

---

# 16. Event Schema

Событие должно иметь понятную структуру.

Например:

```python
event = {
    "event_id": "abc-123",
    "event_type": "OrderCreated",
    "timestamp": "2026-09-11T12:00:00",
    "order_id": 123,
    "user_id": 42,
}
```

Часто используются:

* `event_id`;
* `event_type`;
* timestamp;
* идентификатор сущности;
* данные события;
* metadata.

---

# 17. Версионирование событий

Схема события может меняться.

Было:

```text
OrderCreated v1
```

Появилось новое поле:

```text
OrderCreated v2
```

Consumers должны уметь корректно работать с изменениями схемы.

В больших системах для этого применяют:

* Schema Registry;
* Avro;
* Protobuf;
* JSON Schema;
* правила backward/forward compatibility.

---

# 18. Event-Driven архитектура с Kafka

Типичный вариант:

```text
                  Kafka
                    │
        ┌───────────┼───────────┐
        ↓           ↓           ↓
   OrderCreated Payment...   UserRegistered
        │
        ├───────────────┐
        ↓               ↓
 Payment Service    Email Service
        │
        ↓
PaymentCompleted
        │
        ↓
 Order Service
```

Kafka здесь выступает как **event streaming platform**, а сервисы — как producers/consumers.

---

# 19. Пример реальной бизнес-цепочки

Интернет-магазин:

```text
Пользователь оформил заказ
          ↓
      OrderCreated
          ↓
         Kafka
      ┌───┼────┐
      ↓   ↓    ↓
 Payment Email Stock
      │
      ↓
PaymentCompleted
      ↓
    Kafka
      ↓
Order Service
      ↓
OrderConfirmed
```

Каждый сервис отвечает за свою область.

---

# 20. Главные плюсы EDA

### ✅ Слабая связанность

Сервисы меньше зависят друг от друга.

### ✅ Масштабирование

Consumers можно масштабировать независимо.

### ✅ Асинхронность

Тяжёлые операции можно выполнять вне основного HTTP-запроса.

### ✅ Расширяемость

Можно добавить нового consumer без изменения producer.

```text
OrderCreated
     ↓
Kafka
 ┌───┼────┬────┐
 ↓   ↓    ↓    ↓
A    B    C    D ← новый сервис
```

### ✅ Устойчивость

Broker может временно хранить события, пока consumer недоступен.

---

# 21. Минусы EDA

### ❌ Сложнее отлаживать

Вместо:

```text
A → B
```

может быть:

```text
A
 ↓
Kafka
 ├→ B
 ├→ C
 │   ↓
 │   Kafka
 │    ↓
 │    D
 └→ E
```

### ❌ Eventual consistency

Данные могут быть временно несогласованными.

### ❌ Повторная обработка

Необходимо учитывать at-least-once delivery.

### ❌ Сложнее tracing

Нужны:

* correlation ID;
* distributed tracing;
* structured logging;
* monitoring.

### ❌ Сложнее схемы событий

Изменение event schema может затронуть множество consumers.

---

# 22. EDA vs REST

```text
REST:

Client
  ↓ HTTP
Service A
  ↓ HTTP
Service B
```

Связь:

```text
A знает B
```

---

```text
EDA:

Service A
   ↓
 Event
   ↓
 Broker
   ↓
Service B
```

Связь:

```text
A знает о событии,
но не обязан знать конкретных consumers.
```

---

# 23. Когда использовать EDA?

EDA особенно полезна, когда:

* много независимых сервисов;
* есть большое количество асинхронных задач;
* нужна высокая пропускная способность;
* нужно обрабатывать поток событий;
* несколько систем должны реагировать на один факт;
* нужна независимая масштабируемость consumers.

Например:

```text
Платёж завершён
      ↓
 ┌────┼─────┬──────┐
 ↓    ↓     ↓      ↓
Email CRM Analytics Loyalty
```

Один факт → много реакций.

---

# 24. Когда EDA может быть избыточной?

Для простого CRUD-приложения:

```text
Client
  ↓
Django
  ↓
PostgreSQL
```

не обязательно добавлять:

```text
Kafka
RabbitMQ
10 consumers
event schemas
distributed tracing
```

Если обычного REST достаточно, EDA может только усложнить систему.

---

# 25. EDA и Kafka — как связать на собеседовании

Хорошая формулировка:

> Event-Driven Architecture — это архитектурный подход, в котором компоненты взаимодействуют через события. Kafka может использоваться как транспорт и распределённое хранилище event stream. Producer публикует событие, а consumer groups независимо его обрабатывают. Это позволяет снизить связанность, масштабировать обработчики и реализовать асинхронную обработку.

---

# 26. Итоговая схема

```text
                 EVENT-DRIVEN ARCHITECTURE

                         Event
                           │
                           ↓
                      ┌─────────┐
                      │ Broker  │
                      │ Kafka   │
                      │ Rabbit  │
                      └────┬────┘
                           │
             ┌─────────────┼─────────────┐
             ↓             ↓             ↓
        Consumer A    Consumer B    Consumer C
             │             │             │
             ↓             ↓             ↓
          Service A     Service B     Service C
```

Основная идея:

```text
Producer
   ↓
"Что-то произошло"
   ↓
Event
   ↓
Broker
   ↓
Consumers
   ↓
реакция на событие
```

### Ключевое отличие

```text
Command:
"Сделай X"

Event:
"X произошло"
```

### Ключевая идея EDA

> **Не сообщать сервису, что он должен сделать, а публиковать факт, на который заинтересованные сервисы могут отреагировать.**
