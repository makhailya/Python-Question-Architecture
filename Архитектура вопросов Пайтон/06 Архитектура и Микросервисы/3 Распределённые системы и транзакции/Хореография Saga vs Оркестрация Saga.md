# ⚖️ Choreography vs Orchestration

## 🎯 Ответ на собеседовании

**Choreography и Orchestration** — два основных способа реализации **Saga**.

* **Choreography** — сервисы самостоятельно взаимодействуют через события, центрального координатора нет.
* **Orchestration** — есть центральный **Saga Orchestrator**, который управляет последовательностью действий сервисов и компенсациями.

Главное различие — **где находится логика управления workflow**.

---

## 🎤 Суперкоротко

> В Choreography каждый сервис реагирует на события и запускает следующий шаг самостоятельно. В Orchestration центральный Orchestrator управляет всей Saga: вызывает сервисы, отслеживает состояние и запускает компенсации.

---

# 🔹 Choreography

Нет центрального координатора.

```text id="y8kn7r"
Order Service
      ↓
 OrderCreated
      ↓
    Kafka
      ↓
Payment Service
      ↓
PaymentCompleted
      ↓
    Kafka
      ↓
Inventory Service
```

Каждый сервис знает:

```text id="6cj2i1"
получил событие
      ↓
выполнил локальную транзакцию
      ↓
опубликовал событие
```

Логика Saga **распределена между сервисами**.

---

# 🔹 Orchestration

Есть центральный Orchestrator.

```text id="1v6e8c"
              Orchestrator
              /     |      \
             ↓      ↓       ↓
          Order  Payment  Inventory
```

Он управляет последовательностью:

```text id="g7qj9f"
Orchestrator
      ↓
Create Order
      ↓
Charge Payment
      ↓
Reserve Product
      ↓
Create Shipment
```

При ошибке:

```text id="u6x4yn"
Reserve Product ❌
       ↓
Orchestrator
       ↓
Refund Payment
       ↓
Cancel Order
```

Логика workflow находится **в Orchestrator**.

---

# 🔹 Главное различие

```text id="0z5xwu"
CHOREOGRAPHY

Service A
   ↓ event
Service B
   ↓ event
Service C
   ↓ event
```

vs.

```text id="7q0jye"
ORCHESTRATION

       Orchestrator
       /    |    \
      ↓     ↓     ↓
     A      B     C
```

То есть:

> **Choreography — сервисы договариваются через события.**
> **Orchestration — Orchestrator управляет сервисами.**

---

# 📊 Сравнение

| Критерий                 | Choreography                     | Orchestration                      |
| ------------------------ | -------------------------------- | ---------------------------------- |
| Центральный координатор  | ❌ Нет                            | ✅ Есть                             |
| Управление workflow      | Распределено                     | В Orchestrator                     |
| Взаимодействие           | Обычно события                   | Команды/вызовы + события           |
| Компенсации              | Реагирование на события          | Orchestrator запускает компенсации |
| Связанность              | Через event contracts            | Через контракты с Orchestrator     |
| Простая Saga             | ✅ Хорошо                         | ✅ Хорошо                           |
| Сложная Saga             | ❌ Может стать трудно управляемой | ✅ Обычно удобнее                   |
| Наблюдаемость workflow   | Сложнее                          | Проще                              |
| Отладка                  | Сложнее                          | Проще                              |
| Центральная точка отказа | ❌ Нет                            | ⚠️ Есть Orchestrator               |
| Масштабирование          | Хорошее                          | Хорошее                            |
| Риск усложнения          | Event spaghetti                  | God Orchestrator                   |

---

# 🔹 Пример: интернет-магазин

## Choreography

```text id="c8h5tv"
OrderCreated
     ↓
Payment Service
     ↓
PaymentCompleted
     ↓
Inventory Service
     ↓
InventoryReserved
     ↓
Shipping Service
     ↓
ShippingCreated
```

Если резервирование не удалось:

```text id="lq5z0m"
ReservationFailed
       ↓
Payment Service
       ↓
Refund
```

Каждый сервис самостоятельно знает, на какие события реагировать.

---

## Orchestration

```text id="x2kw7n"
             Orchestrator
                  │
        ┌─────────┼─────────┐
        ↓         ↓         ↓
      Order    Payment   Inventory
```

Orchestrator:

```text id="8tq3kg"
1. Create Order
2. Charge Payment
3. Reserve Inventory
4. Create Shipment
```

Если Inventory вернул ошибку:

```text id="h4q5jv"
Inventory ❌
    ↓
Orchestrator
    ↓
Refund Payment
    ↓
Cancel Order
```

---

# 🔹 Плюсы и минусы Choreography

### ✅ Плюсы

* нет центрального координатора;
* сервисы слабее связаны между собой;
* естественно подходит для event-driven архитектуры;
* нет единой точки управления Saga.

### ❌ Минусы

* сложнее понять весь workflow;
* сложнее отлаживать длинные цепочки;
* трудно контролировать сложные зависимости;
* большое количество событий может привести к **event spaghetti**.

---

# 🔹 Плюсы и минусы Orchestration

### ✅ Плюсы

* workflow находится в одном месте;
* проще понимать последовательность операций;
* проще управлять компенсациями;
* проще мониторить состояние Saga;
* хорошо подходит для сложных бизнес-процессов.

### ❌ Минусы

* появляется центральный компонент;
* Orchestrator требует отказоустойчивости;
* слишком большой Orchestrator может превратиться в **God Service**;
* сервисы зависят от команд и контрактов Orchestrator.

---

# 🔹 Что выбрать?

Условно:

```text id="u5j3sz"
Простая Saga
     ↓
Choreography
```

Например:

```text id="v8j6zq"
OrderCreated
     ↓
Payment
     ↓
PaymentCompleted
     ↓
Inventory
```

Если workflow становится сложным:

```text id="q3e8lm"
много шагов
+ ветвления
+ компенсации
+ retry
+ разные сценарии
        ↓
Orchestration
```

---

# ⚠️ Важный нюанс

**Choreography не означает «только Kafka».**

И **Orchestration не означает «обязательно REST».**

Это способы организации управления Saga.

Транспорт может быть разным:

```text id="r7r2dw"
Kafka
RabbitMQ
gRPC
REST
```

Например, Orchestrator может отправлять команды через Kafka:

```text id="3p5m6e"
Orchestrator
      ↓
Kafka
      ↓
Payment Service
```

А результат возвращается событием:

```text id="f8k2sp"
PaymentCompleted
      ↓
Kafka
      ↓
Orchestrator
```

---

# 🔗 Связь с Outbox Pattern

Оба подхода часто используют **Outbox Pattern** для надёжной публикации событий.

```text id="c4b7kw"
Local Transaction
      ↓
Business Data + Outbox
      ↓
Publisher
      ↓
Kafka
```

Дальше:

### Choreography

```text id="w3v8dy"
Kafka
 ↓
Service B
 ↓
event
 ↓
Service C
```

### Orchestration

```text id="j5m9zt"
Kafka
 ↓
Orchestrator
 ↓
command
 ↓
Service B
```

---

# 🔥 Главное

```text id="q2z9mp"
                 SAGA
                  │
          ┌───────┴───────┐
          ↓               ↓
    CHOREOGRAPHY     ORCHESTRATION
          │               │
          ↓               ↓
    Events между      Orchestrator
     сервисами        управляет flow
          │               │
          ↓               ↓
   Нет центра          Есть центр
```

### Формула для собеседования

> **Choreography** — децентрализованная Saga: сервисы реагируют на события и самостоятельно запускают следующие шаги и компенсации.
> **Orchestration** — централизованная Saga: Orchestrator управляет workflow, вызывает сервисы, отслеживает состояние и запускает компенсации.

### Самое важное отличие

> **Choreography: «сервисы знают, что делать после события».**
> **Orchestration: «Orchestrator знает, что должен делать каждый сервис».**
