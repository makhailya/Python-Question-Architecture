# 🔄 Airflow vs Saga Orchestration

## 🎯 Ответ на собеседовании

**[[Airflow]]** и **[[Оркестрация Saga]]** используют слово «оркестрация», но решают разные задачи.

**[[Airflow]]** оркестрирует **workflow и data-пайплайны**: запускает задачи, управляет зависимостями, расписанием, retries и мониторингом.

**Saga Orchestration**[[Оркестрация Saga]] управляет **распределённой бизнес-транзакцией** между микросервисами [[Микросервисы]]: запускает локальные транзакции и при ошибке инициирует компенсирующие операции.

> **Airflow управляет процессом выполнения задач, Saga Orchestrator — согласованностью бизнес-операции между сервисами.**

---

## 🎤 Суперкоротко

```text
Airflow
→ workflow / ETL / batch
→ DAG
→ Tasks
→ Scheduler
→ Executor

Saga Orchestration
→ distributed business transaction
→ Services
→ Local transactions
→ Compensation
```

---

# 🔹 Airflow

Airflow используется для управления **workflow**.

Типичный пример ETL:

```text
Extract
   ↓
Transform
   ↓
Validate
   ↓
Load
   ↓
Build report
```

Airflow:

* определяет зависимости;
* запускает задачи;
* выполняет их последовательно или параллельно;
* делает retries;
* хранит состояние;
* предоставляет UI и мониторинг;
* работает по расписанию.

Основной объект — **DAG**.

---

# 🔹 Saga Orchestration

Saga решает проблему **распределённых транзакций**.

Например, создание заказа:

```text
Order Service
      ↓
Payment Service
      ↓
Inventory Service
      ↓
Delivery Service
```

Каждый сервис выполняет **свою локальную транзакцию**.

Если последний шаг не удался:

```text
Order
  ↓
Payment
  ↓
Inventory ❌
```

Saga Orchestrator может запустить компенсации:

```text
Inventory ❌
    ↓
Refund Payment
    ↓
Cancel Order
```

Компенсация — это **бизнес-операция**, а не `ROLLBACK` общей БД.

---

# ⚖️ Основное различие

|                         | Airflow                    | Saga Orchestration          |
| ----------------------- | -------------------------- | --------------------------- |
| Основная задача         | Workflow orchestration     | Distributed transaction     |
| Типичные задачи         | ETL, ELT, batch            | Заказы, платежи, резервации |
| Основной объект         | DAG                        | Saga                        |
| Единица работы          | Task                       | Local transaction           |
| Ошибки                  | Retry / failure handling   | Compensation                |
| Согласованность         | Workflow execution         | Business consistency        |
| Расписание              | ✅                          | Обычно не основная задача   |
| Мониторинг              | ✅                          | Реализуется отдельно        |
| Eventual consistency    | Не является основной целью | ✅                           |
| Микросервисы            | Может работать с ними      | Основной сценарий           |
| Компенсирующие операции | ❌                          | ✅                           |

---

# 🧩 Пример: интернет-магазин

### Airflow

Допустим, каждую ночь нужно построить аналитический отчёт:

```text
Download orders
       ↓
Clean data
       ↓
Calculate metrics
       ↓
Load to warehouse
       ↓
Generate report
```

Это классический **Airflow workflow**.

---

### Saga

Пользователь оформляет заказ:

```text
Create Order
     ↓
Reserve Inventory
     ↓
Charge Payment
     ↓
Create Delivery
```

Если оплата прошла, но доставка не создалась:

```text
Create Order       ✅
Reserve Inventory  ✅
Charge Payment     ✅
Create Delivery    ❌
        ↓
Refund Payment
        ↓
Release Inventory
        ↓
Cancel Order
```

Это **Saga**.

---

# 🔥 Почему их нельзя считать одним и тем же

Airflow отвечает:

> **«Как выполнить workflow?»**

Saga отвечает:

> **«Как сохранить бизнес-согласованность, если распределённая операция частично завершилась?»**

Даже если оба используют центральный компонент-оркестратор, **цель разная**.

---

# 🚫 Важная ловушка на собеседовании

Не стоит говорить:

> «Airflow — это реализация Saga Orchestration».

Это неверно.

Airflow может **технически вызывать сервисы** и выглядеть как оркестратор, но сам по себе он не превращает workflow в Saga.

Для Saga нужны:

* локальные транзакции;
* состояние Saga;
* определение успешных/неуспешных шагов;
* компенсирующие операции;
* обработка частичных отказов;
* eventual consistency.

---

# 🔄 Можно ли использовать Airflow для бизнес-процесса?

Технически — **да**, но это не означает, что это хорошая реализация Saga.

Airflow хорошо подходит для:

```text
batch
ETL
ELT
data pipelines
scheduled workflows
```

Saga чаще используется для:

```text
orders
payments
inventory
booking
delivery
distributed business operations
```

Для долгоживущих бизнес-процессов обычно выбирают специализированный workflow/orchestration подход, а не используют Airflow как замену Saga.

---

# 🧠 Главное

```text
Airflow
        ↓
Оркестрация workflow
        ↓
DAG → Tasks → Dependencies
        ↓
ETL / ELT / Batch
```

```text
Saga Orchestration
        ↓
Оркестрация распределённой транзакции
        ↓
Local Transactions
        ↓
Compensating Operations
        ↓
Eventual Consistency
```

### Формула для собеседования

> **Airflow оркестрирует выполнение workflow, а Saga Orchestrator — распределённую бизнес-транзакцию и её компенсации.**
