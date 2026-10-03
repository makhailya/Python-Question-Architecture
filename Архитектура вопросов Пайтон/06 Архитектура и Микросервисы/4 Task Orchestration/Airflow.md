# 🌬️ Apache Airflow

## 🎯 Ответ на собеседовании

**Apache Airflow** — это платформа для программного определения, планирования и мониторинга **workflow** — последовательностей связанных задач.

Workflow описывается в виде **[[DAG]] (Directed Acyclic Graph)**, где узлы — задачи, а зависимости между ними определяют порядок выполнения.

Airflow особенно часто используется для **ETL/ELT, data pipelines, batch-задач и периодических workflow**.

---

## 🎤 Суперкоротко

> Airflow — инструмент оркестрации workflow. Мы описываем DAG из задач и зависимостей между ними, а Airflow планирует их выполнение, следит за статусами, retry и ошибками.

---

# 🔹 Что такое DAG

**DAG — Directed Acyclic Graph**, ориентированный ациклический граф.

Например:

```text id="qj0u9m"
Extract
   ↓
Transform
   ↓
Load
```

Или более сложный workflow:

```text id="9x2v5d"
        Extract
        /     \
       ↓       ↓
   Clean     Validate
       \       /
        ↓     ↓
         Load
```

### Directed

У зависимостей есть направление:

```text
A → B
```

### Acyclic

Нет циклов:

```text
A → B → C → A  ❌
```

---

# 🔹 Основные компоненты

```text id="x7t8pm"
DAG
 ↓
Tasks
 ↓
Scheduler
 ↓
Executor
 ↓
Workers
```

### DAG

Описывает workflow:

```python id="j9d2ks"
from airflow import DAG
```

### Task

Отдельная задача:

```text id="1j3k8f"
Extract
Transform
Load
```

### Scheduler

Определяет, **какие задачи и когда нужно запустить**.

### Executor

Определяет механизм выполнения задач.

### Worker

Фактически выполняет задачи в соответствующей конфигурации Airflow.

### Web UI

Позволяет смотреть:

* DAG;
* состояние задач;
* логи;
* retries;
* длительность выполнения;
* историю запусков.

---

# 🔹 Пример DAG

Упрощённо:

```python id="m7zq4x"
from airflow import DAG
from airflow.operators.python import PythonOperator
from datetime import datetime


def extract():
    print("Extract data")


def transform():
    print("Transform data")


def load():
    print("Load data")


with DAG(
    dag_id="etl_pipeline",
    start_date=datetime(2026, 1, 1),
    schedule="@daily",
    catchup=False,
) as dag:

    extract_task = PythonOperator(
        task_id="extract",
        python_callable=extract,
    )

    transform_task = PythonOperator(
        task_id="transform",
        python_callable=transform,
    )

    load_task = PythonOperator(
        task_id="load",
        python_callable=load,
    )

    extract_task >> transform_task >> load_task
```

Зависимость:

```text id="r1c5v8"
extract
   ↓
transform
   ↓
load
```

---

# 🔹 Что такое Operator

**Operator** определяет, **что должна делать задача**.

Например:

* `PythonOperator` — выполнить Python-код;
* `BashOperator` — выполнить shell-команду;
* операторы для SQL;
* операторы для Kubernetes;
* операторы для облачных сервисов.

Упрощённо:

```text id="8t6s4v"
Operator
   ↓
Task
```

Operator — это шаблон/тип выполняемой операции, а Task — конкретный экземпляр этой операции в DAG.

---

# 🔹 Зависимости между задачами

Airflow позволяет явно описывать порядок:

```python id="p5q8fz"
extract >> transform >> load
```

Можно сделать параллельное выполнение:

```python id="q1j3vx"
extract >> [clean, validate]
```

Получится:

```text id="8x3n1r"
        Extract
        /     \
       ↓       ↓
    Clean   Validate
```

А затем объединить:

```python id="w8s2nk"
[clean, validate] >> load
```

---

# 🔹 Retry

Если задача временно завершилась ошибкой, Airflow может автоматически повторить её.

Например:

```python id="y2m7pf"
from datetime import timedelta

default_args = {
    "retries": 3,
    "retry_delay": timedelta(minutes=5),
}
```

Схема:

```text id="x4h7nb"
Task
 ↓
❌
 ↓
Retry
 ↓
❌
 ↓
Retry
 ↓
✅
```

---

# 🔹 Планирование

Airflow позволяет запускать DAG по расписанию:

```python id="5n7g1a"
schedule="@daily"
```

Например:

```text id="v3p9qk"
каждый день
     ↓
DAG
     ↓
Extract
     ↓
Transform
     ↓
Load
```

Также можно использовать cron-выражения.

---

# 🔹 Airflow и ETL

Один из классических сценариев:

```text id="e3f7zc"
API / Database / Files
          ↓
       Extract
          ↓
      Transform
          ↓
         Load
          ↓
      PostgreSQL
```

Airflow в данном случае **не обязательно сам обрабатывает все данные**.

Он скорее **управляет процессом**:

```text id="b7w4sq"
"Запусти Extract"
        ↓
"После успешного Extract запусти Transform"
        ↓
"После Transform запусти Load"
```

---

# 🔹 Airflow и Saga

Здесь важно провести границу.

**Airflow может быть workflow orchestrator, но это не означает, что Airflow автоматически является Saga Orchestrator.**

Saga:

```text id="j8h4mq"
Order
  ↓
Payment
  ↓
Inventory
  ↓
Shipping
```

решает задачу **распределённой бизнес-транзакции** и компенсаций.

Airflow:

```text id="k5r2wn"
Task A
  ↓
Task B
  ↓
Task C
```

решает задачу **оркестрации workflow**.

### Поэтому:

```text id="g1s6jc"
Airflow
   ≠
Saga
```

Теоретически Airflow можно использовать для координации определённого бизнес-workflow, но для обычных микросервисных Saga это не самый типичный выбор.

---

# 🔹 Airflow vs Saga Orchestrator

| Airflow                                      | Saga Orchestrator                     |
| -------------------------------------------- | ------------------------------------- |
| Workflow orchestration                       | Distributed transaction orchestration |
| Часто ETL/Data Engineering                   | Бизнес-процессы микросервисов         |
| DAG                                          | Saga state/workflow                   |
| Scheduling                                   | Управление бизнес-шагами              |
| Retry задач                                  | Retry бизнес-операций                 |
| Мониторинг workflow                          | Мониторинг Saga                       |
| Не решает автоматически проблему компенсаций | Компенсации — важная часть Saga       |

---

# 🔹 Airflow vs Celery

Это тоже полезное сравнение для Python Backend.

| Airflow                            | Celery                           |
| ---------------------------------- | -------------------------------- |
| Оркестрация workflow               | Очередь фоновых задач            |
| DAG                                | Отдельные задачи/цепочки         |
| Планирование workflow              | Асинхронное выполнение           |
| Хорош для ETL/Data pipelines       | Хорош для background jobs        |
| Web UI и мониторинг DAG            | Celery Flower/другие инструменты |
| Сложные зависимости между задачами | Простые и сложные task chains    |

Например:

### Celery

```text id="m4v7kc"
API
 ↓
Celery
 ↓
Background Task
```

### Airflow

```text id="n8x2qa"
DAG
 ├── Extract
 ├── Transform
 ├── Validate
 └── Load
```

---

# 🔹 Почему именно DAG

Airflow должен знать зависимости:

```text id="c2k5wy"
A → B → C
```

Тогда он может определить:

* что можно запускать;
* что должно ждать;
* какие задачи можно выполнить параллельно;
* что делать после ошибки.

Например:

```text id="7p3h2s"
        A
       / \
      B   C
       \ /
        D
```

`B` и `C` могут выполняться параллельно после завершения `A`.

`D` ждёт завершения обеих.

---

# ⚠️ Важный момент: Airflow не просто «cron»

`cron`:

```text id="r9f4yc"
каждый день в 03:00
      ↓
запустить скрипт
```

Airflow:

```text id="u6w2za"
DAG
 ├── Task A
 │
 ├── Task B
 │
 ├── Task C
 │
 └── Task D
```

Он понимает зависимости, хранит состояние выполнения, предоставляет UI, логи, retries и управление workflow.

Поэтому:

> **Cron отвечает в основном за расписание, Airflow — за управление workflow.**

---

# 🔥 Главное

```text id="w5q8kn"
              Airflow
                 ↓
                DAG
                 ↓
        ┌────────┼────────┐
        ↓        ↓        ↓
      Task     Task     Task
        │        │        │
        └────────┼────────┘
                 ↓
          Scheduler
                 ↓
             Executor
                 ↓
             Workers
```

**Ключевая фраза для собеседования:**

> Apache Airflow — платформа для оркестрации workflow. Workflow описывается в виде DAG, состоящего из задач и зависимостей между ними. Scheduler определяет, когда запускать задачи, executor определяет механизм их выполнения, а Airflow предоставляет мониторинг, логи и retry. Чаще всего Airflow применяется для ETL/ELT и data pipelines.

### Связь с предыдущими темами

```text id="n2c7vx"
Saga
 ├── Choreography
 │     └── Events
 │
 └── Orchestration
       └── Saga Orchestrator

Workflow orchestration
 └── Airflow
       └── DAG
            └── Tasks
```

**Не смешивай:** `Saga Orchestration` и `Airflow` — это связанные по идее понятия оркестрации, но решают разные классы задач.
