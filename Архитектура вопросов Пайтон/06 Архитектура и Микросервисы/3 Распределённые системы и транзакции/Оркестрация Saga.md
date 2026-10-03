# 🎼 Оркестрация Saga

## 🎯 Ответ на собеседовании

**Оркестрация Saga** — это способ реализации [[Saga]], при котором **центральный Saga Orchestrator управляет последовательностью локальных транзакций [[Локальные транзакции]]в микросервисах**. [[Микросервисы]]

Оркестратор знает порядок выполнения операций, вызывает нужные сервисы и при ошибке запускает соответствующие **компенсирующие операции**.

---

## 🎤 Суперкоротко

> Оркестрация Saga — это подход, при котором центральный Orchestrator управляет всей распределённой бизнес-транзакцией: запускает шаги, обрабатывает результаты и при ошибке запускает компенсации.

---

# 🔹 Как работает

Допустим, оформление заказа состоит из нескольких шагов:

```text
             Saga Orchestrator
                    │
        ┌───────────┼───────────┐
        ↓           ↓           ↓
     Order       Payment     Inventory
     Service     Service      Service
```

Оркестратор управляет последовательностью:

```text
Orchestrator
     ↓
Create Order
     ↓
Charge Payment
     ↓
Reserve Product
     ↓
Create Shipment
     ↓
SUCCESS
```

В отличие от хореографии, **логика workflow находится в одном месте**.

---

# 🔹 Пример

Пусть пользователь покупает товар за 5000 ₽.

### Шаг 1 — создать заказ

```text
Orchestrator
      ↓
Order Service
      ↓
Create Order
      ↓
SUCCESS
```

### Шаг 2 — списать деньги

```text
Orchestrator
      ↓
Payment Service
      ↓
Charge 5000 ₽
      ↓
SUCCESS
```

### Шаг 3 — зарезервировать товар

```text
Orchestrator
      ↓
Inventory Service
      ↓
Reserve Product
      ↓
SUCCESS
```

### Шаг 4 — создать доставку

```text
Orchestrator
      ↓
Shipping Service
      ↓
Create Shipment
      ↓
SUCCESS
```

Вся Saga:

```text
Order
  ↓
Payment
  ↓
Inventory
  ↓
Shipping
  ↓
Completed
```

---

# 🔹 Что происходит при ошибке

Предположим:

```text
Order       → SUCCESS
Payment     → SUCCESS
Inventory   → ❌ ERROR
```

Orchestrator знает, какие операции уже были выполнены и какие компенсации необходимо запустить.

Например:

```text
             Orchestrator
                  ↓
              Inventory
                  ↓
                 ❌
                  ↓
            Compensation
             /          \
            ↓            ↓
     Refund Payment   Cancel Order
```

Получается:

```text
Create Order
      ↓
Charge Payment
      ↓
Reserve Product ❌
      ↓
Refund Payment
      ↓
Cancel Order
```

---

# 🔹 Главная роль Orchestrator

Orchestrator хранит **состояние и логику workflow**.

Условно:

```python
class OrderSaga:
    def execute(self):
        create_order()

        charge_payment()

        reserve_product()

        create_shipment()
```

Если шаг завершается ошибкой:

```python
class OrderSaga:
    def execute(self):
        create_order()

        try:
            charge_payment()
            reserve_product()
            create_shipment()

        except Exception:
            refund_payment()
            cancel_order()
```

В реальной системе это, конечно, не обязательно один Python-метод — Saga может быть долгоживущим процессом с сохранением состояния и асинхронными сообщениями.

---

# 🔹 Взаимодействие с сервисами

Оркестратор может взаимодействовать с микросервисами:

* через HTTP/REST;
* через gRPC;
* через сообщения в Kafka/RabbitMQ.

Например:

```text
Orchestrator
      │
      │ REST/gRPC
      ↓
Payment Service
```

или:

```text
Orchestrator
      │
      │ event/message
      ↓
Kafka
      ↓
Payment Service
```

Сам принцип оркестрации от конкретного транспорта не зависит.

---

# 🔹 Состояние Saga

Для долгих процессов Orchestrator обычно должен знать:

```text
Saga ID
Order ID
Current Step
Completed Steps
Failed Step
Status
```

Например:

```text
Saga ID: 123
Order: 456

Order       → COMPLETED
Payment     → COMPLETED
Inventory   → FAILED
Refund      → COMPLETED
OrderCancel → COMPLETED

Saga Status → COMPENSATED
```

Это позволяет восстановить выполнение после сбоя самого Orchestrator.

---

# 🔹 Оркестратор — не обязательно отдельный микросервис

Это важный нюанс.

Orchestrator может быть:

* отдельным сервисом;
* компонентом внутри существующего сервиса;
* workflow engine;
* специализированным orchestration-компонентом.

Главное условие:

> существует центральный компонент, который управляет последовательностью Saga.

---

# 🔹 Оркестрация vs хореография

| Хореография                                     | Оркестрация                              |
| ----------------------------------------------- | ---------------------------------------- |
| Нет центрального координатора                   | Есть Orchestrator                        |
| Управление через события                        | Центр управляет workflow                 |
| Логика распределена по сервисам                 | Логика Saga сосредоточена в Orchestrator |
| Сложнее увидеть общий flow                      | Flow проще понять                        |
| Сложнее централизованно управлять компенсациями | Компенсации управляются Orchestrator     |
| Нет отдельной точки управления                  | Есть центральная точка управления        |
| Хороша для относительно простых процессов       | Хороша для сложных бизнес-процессов      |

---

# 🔹 Преимущество №1 — понятный workflow

При оркестрации можно посмотреть на Orchestrator и увидеть:

```text
1. Create Order
2. Charge Payment
3. Reserve Product
4. Create Shipment
```

В хореографии эти зависимости могут быть распределены:

```text
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
```

При большом количестве событий разобраться становится сложнее.

---

# 🔹 Преимущество №2 — централизованные компенсации

Orchestrator знает:

```text
что уже выполнено
       ↓
какие шаги нужно компенсировать
       ↓
в каком порядке
```

Например:

```text
A → SUCCESS
B → SUCCESS
C → ERROR

Compensate:
B → compensation
A → compensation
```

---

# 🔹 Недостатки

### 1. Центральная точка управления

Orchestrator становится критически важным компонентом.

Если он недоступен:

```text
Orchestrator ❌
      ↓
Saga не управляется
```

Поэтому его нужно делать отказоустойчивым.

---

### 2. Риск усложнения Orchestrator

Если поместить туда слишком много бизнес-логики:

```text
Orchestrator
├── Order logic
├── Payment logic
├── Inventory logic
├── Shipping logic
└── Business rules
```

он может превратиться в **God Object / God Service**.

Лучше, чтобы Orchestrator управлял **workflow**, а бизнес-логика оставалась внутри соответствующих сервисов.

---

### 3. Более сильная зависимость от Orchestrator

Сервисы становятся зависимыми от команд/контрактов, которые определяет Orchestrator.

---

# 🔹 Оркестрация и eventual consistency

Как и хореография, оркестрация Saga обычно **не делает все операции одной ACID-транзакцией**.

Например:

```text
Order → CREATED
Payment → PAID
Inventory → FAILED
```

Некоторое время система может находиться в таком состоянии.

Затем Orchestrator запускает компенсацию:

```text
Refund Payment
      ↓
Cancel Order
```

И система приходит к:

```text
Order → CANCELLED
Payment → REFUNDED
Inventory → NOT_RESERVED
```

Это **eventual consistency**.

---

# 🔹 Оркестрация + Outbox

Если Orchestrator и сервисы используют сообщения, полезно применять **Outbox Pattern**.

Например:

```text
Orchestrator
      ↓
записать команду
      ↓
Outbox
      ↓
Kafka
      ↓
Payment Service
```

А Payment Service:

```text
получил команду
      ↓
Local Transaction
      ├── списать деньги
      └── записать событие в Outbox
                         ↓
                       Kafka
                         ↓
                    Orchestrator
```

Таким образом:

```text
Saga Orchestration
        +
Local Transactions
        +
Outbox
        +
Message Broker
```

образуют типичный набор для надёжной реализации распределённого workflow.

---

# 🔥 Главное

```text
                  Orchestrator
                       │
          ┌────────────┼────────────┐
          ↓            ↓            ↓
       Order        Payment      Inventory
          │            │            │
          └────────────┼────────────┘
                       ↓
                    Shipping
```

При ошибке:

```text
A → B → C ❌
    ↓
Orchestrator
    ↓
Compensate B
    ↓
Compensate A
```

**Ключевая фраза для собеседования:**

> При оркестрации Saga есть центральный Orchestrator, который управляет последовательностью локальных транзакций микросервисов, отслеживает состояние workflow и при ошибке запускает компенсирующие операции. В отличие от хореографии, логика управления Saga находится в одном месте, поэтому сложные процессы проще контролировать и отлаживать, но сам Orchestrator становится критически важным компонентом.
