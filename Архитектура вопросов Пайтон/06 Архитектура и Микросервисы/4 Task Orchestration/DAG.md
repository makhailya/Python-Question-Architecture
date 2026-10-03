# 🔗 DAG в Apache Airflow

## 🎯 Ответ на собеседовании

**DAG (Directed Acyclic Graph)** — это ориентированный ациклический граф, который описывает workflow в Apache Airflow.

В DAG находятся **задачи (Tasks)** и зависимости между ними. Airflow использует эти зависимости, чтобы определить **порядок и возможность параллельного выполнения задач**.

---

## 🎤 Суперкоротко

> DAG — это описание workflow в Airflow в виде ориентированного ациклического графа: вершины — задачи, рёбра — зависимости между ними.

---

# 🔹 Расшифровка DAG

**D — Directed** — ориентированный.

Связь имеет направление:

```text
A → B
```

Это означает:

> Сначала выполняется A, затем B.

**A — Acyclic** — ациклический.

В графе не должно быть циклов:

```text
A → B → C → A  ❌
```

**G — Graph** — граф.

Он состоит из:

* вершин;
* связей между вершинами.

В Airflow:

```text
Vertex → Task
Edge   → Dependency
```

---

# 🔹 Пример DAG

Простой ETL:

```text
Extract
   ↓
Transform
   ↓
Load
```

Здесь:

```text
Extract → Task
Transform → Task
Load → Task
```

А стрелки:

```text
Extract → Transform
Transform → Load
```

— это **dependencies**.

---

# 🔹 Более сложный DAG

Некоторые задачи могут выполняться параллельно:

```text
             Extract
             /     \
            ↓       ↓
         Clean    Validate
            \       /
             ↓     ↓
              Load
```

Логика:

```text
Extract
   ↓
 ┌─┴───────┐
 ↓         ↓
Clean   Validate
 └────┬────┘
      ↓
     Load
```

После завершения `Extract` задачи `Clean` и `Validate` могут выполняться независимо друг от друга.

`Load` ждёт завершения обеих.

---

# 🔹 Создание DAG

В Airflow DAG можно описать в Python:

```python
from airflow import DAG
from datetime import datetime


with DAG(
    dag_id="etl_pipeline",
    start_date=datetime(2026, 1, 1),
    schedule="@daily",
    catchup=False,
) as dag:
    pass
```

Здесь:

```text
dag_id
```

— уникальный идентификатор DAG.

```text
start_date
```

— дата, с которой Airflow рассматривает DAG для планирования.

```text
schedule
```

— расписание запуска.

```text
catchup
```

— определяет, нужно ли выполнять пропущенные плановые запуски.

---

# 🔹 Добавление Tasks

DAG сам по себе не выполняет бизнес-операции.

В него добавляются **Tasks**:

```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from datetime import datetime


def extract():
    print("Extract")


def transform():
    print("Transform")


def load():
    print("Load")


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

Получается:

```text
extract
   ↓
transform
   ↓
load
```

---

# 🔹 DAG определяет зависимости, а не просто список задач

Например:

```python
extract >> transform >> load
```

означает:

```text
extract
   ↓
transform
   ↓
load
```

Но:

```python
extract >> [clean, validate]
```

означает:

```text
       extract
       /     \
      ↓       ↓
   clean   validate
```

Таким образом DAG позволяет описывать **граф выполнения**, а не просто последовательность функций.

---

# 🔹 Параллельное выполнение

Если задачи не зависят друг от друга:

```text
          Extract
         /       \
        ↓         ↓
     Clean     Validate
```

то `Clean` и `Validate` потенциально могут выполняться параллельно.

Это одно из преимуществ DAG-подхода:

> Airflow может использовать зависимости между задачами для организации последовательного и параллельного выполнения workflow.

---

# 🔹 DAG Run

**DAG** — это описание workflow.

**DAG Run** — конкретный запуск этого workflow.

Например:

```text
DAG:
daily_etl
```

может запускаться каждый день:

```text
11 сентября → DAG Run
12 сентября → DAG Run
13 сентября → DAG Run
```

То есть:

```text
DAG
 ↓
определение workflow

DAG Run
 ↓
конкретный запуск workflow
```

---

# 🔹 Task и Task Instance

Аналогично:

```text
Task
 ↓
описание задачи
```

Конкретный запуск задачи:

```text
Task Instance
 ↓
конкретное выполнение Task
```

Например:

```text
DAG: daily_etl

Task:
    extract

DAG Run:
    2026-09-11

Task Instance:
    extract для запуска 2026-09-11
```

---

# 🔹 Почему DAG должен быть ациклическим

Airflow должен иметь возможность определить порядок выполнения.

Если создать:

```text
A → B → C → A
```

возникает бесконечная зависимость:

```text
A ждёт C
C ждёт B
B ждёт A
```

Поэтому workflow в Airflow представляет собой **DAG**, а не произвольный граф.

---

# 🔹 DAG ≠ Task

Это важное различие.

| DAG                         | Task                         |
| --------------------------- | ---------------------------- |
| Описывает весь workflow     | Описывает отдельную операцию |
| Содержит Tasks              | Находится внутри DAG         |
| Определяет зависимости      | Выполняет конкретную работу  |
| Может содержать много задач | Является частью workflow     |

Например:

```text
DAG: daily_etl

├── extract
├── transform
├── validate
└── load
```

---

# 🔹 DAG ≠ Scheduler

Также не нужно путать:

```text
DAG
 ↓
описывает ЧТО и в каком порядке делать
```

```text
Scheduler
 ↓
решает КОГДА задачи можно запустить
```

Упрощённо:

```text
DAG
 ↓
workflow + dependencies
        ↓
Scheduler
        ↓
запуск готовых Tasks
```

---

# 🔥 Главное

```text
                 DAG
                  │
          workflow definition
                  │
       ┌──────────┼──────────┐
       ↓          ↓          ↓
     Task       Task       Task
       │          │          │
       └──── dependencies ───┘
```

**Ключевая фраза для собеседования:**

> DAG в Airflow — это ориентированный ациклический граф, описывающий workflow. Вершины графа — задачи, а рёбра — зависимости между ними. DAG позволяет определить порядок выполнения задач и организовать их параллельное выполнение там, где нет зависимостей.
