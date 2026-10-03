# ⚙️ Operator в Apache Airflow

## 🎯 Ответ на собеседовании

**Operator** — это класс, который определяет, **какую операцию должна выполнить Task и каким способом она будет выполняться**.

Когда Operator используется внутри [[DAG]], создаётся конкретный **Task**.

Например, `PythonOperator` позволяет выполнить Python-функцию, а `BashOperator` — shell-команду.

---

## 🎤 Суперкоротко

> Operator определяет, что должна делать Task и как это делать. Task — это конкретный экземпляр Operator внутри DAG.

---

# 🔹 Связь DAG → Task → Operator

Основная структура:

```text id="h9x2va"
DAG
 ↓
Task
 ↓
Operator
```

Например:

```python id="7k4m2a"
task = PythonOperator(
    task_id="extract",
    python_callable=extract,
)
```

Здесь:

```text id="2k1x8c"
PythonOperator → Operator
task           → Task
```

---

# 🔹 Основные Operator

В Airflow существует множество Operator.

Наиболее важные:

| Operator             | Назначение                    |
| -------------------- | ----------------------------- |
| `PythonOperator`     | Выполнить Python-функцию      |
| `BashOperator`       | Выполнить shell-команду       |
| SQL Operators        | Выполнить SQL                 |
| HTTP Operators       | Взаимодействовать с HTTP/API  |
| Kubernetes Operators | Запускать задачи в Kubernetes |

На собеседовании важно понимать **концепцию**, а не запоминать все существующие Operator.

---

# 🔹 PythonOperator

Используется для выполнения Python-кода.

```python id="v8p3kq"
from airflow.operators.python import PythonOperator


def extract():
    print("Extract data")


extract_task = PythonOperator(
    task_id="extract",
    python_callable=extract,
)
```

Здесь:

```text id="c6z0s1"
python_callable
      ↓
Python-функция
      ↓
Task выполняет функцию
```

---

# 🔹 BashOperator

Позволяет выполнить shell-команду.

```python id="g5n2rv"
from airflow.operators.bash import BashOperator


task = BashOperator(
    task_id="show_date",
    bash_command="date",
)
```

Airflow запустит:

```text id="h3j7kn"
date
```

как shell-команду.

---

# 🔹 Operator создаёт Task

Важно понимать направление:

```python id="f9r2qk"
task = PythonOperator(
    task_id="extract",
    python_callable=extract,
)
```

Мы используем класс:

```text id="k8v5ma"
PythonOperator
```

и создаём его экземпляр:

```text id="q2j7ws"
task
```

Этот экземпляр становится **Task внутри DAG**.

То есть:

```text id="z1w6cx"
PythonOperator
      ↓
экземпляр
      ↓
Task
      ↓
DAG
```

---

# 🔹 Один Operator → много Tasks

Можно создать несколько Tasks одного типа:

```python id="u4x9dp"
extract = PythonOperator(
    task_id="extract",
    python_callable=extract_data,
)

transform = PythonOperator(
    task_id="transform",
    python_callable=transform_data,
)

validate = PythonOperator(
    task_id="validate",
    python_callable=validate_data,
)
```

Все они используют:

```text id="2m7rqa"
PythonOperator
```

но являются разными Tasks:

```text id="s4j1cy"
extract
transform
validate
```

---

# 🔹 Operator не определяет зависимости

Operator отвечает за **саму операцию**.

А зависимости задаются отдельно:

```python id="g7c2ne"
extract >> transform >> validate
```

Получается:

```text id="x6q3fz"
extract
   ↓
transform
   ↓
validate
```

То есть:

```text id="2x5g8c"
Operator
   ↓
как выполнить Task

Dependency
   ↓
когда выполнить Task
```

---

# 🔹 Operator и Python-функция

Например:

```python id="y4n8hs"
def extract():
    print("Extract data")


task = PythonOperator(
    task_id="extract",
    python_callable=extract,
)
```

Здесь:

```text id="q8w3jk"
extract()
   ↓
бизнес-логика / код

PythonOperator
   ↓
механизм запуска функции

Task
   ↓
конкретная задача в DAG
```

Это полезное разделение ответственности.

---

# 🔹 Operator и Hook — не одно и то же

В Airflow можно встретить ещё **Hook**.

Упрощённо:

```text id="x9k3qm"
Operator
   ↓
выполняет задачу

Hook
   ↓
предоставляет интерфейс
для работы с внешней системой
```

Например:

```text id="j5r7bc"
PostgresHook
   ↓
PostgreSQL
```

Operator может использовать Hook для подключения к внешней системе.

---

# 🔹 Operator и Sensor

Есть ещё **Sensor** — специальный тип задачи, который ждёт наступления определённого условия.

Например:

```text id="p3c6wf"
Sensor
 ↓
ждёт файл
 ↓
файл появился
 ↓
следующая Task
```

Или:

```text id="a7m2xe"
Sensor
 ↓
ждёт API/data
 ↓
условие выполнено
 ↓
продолжение DAG
```

Поэтому:

```text id="d4x8sq"
Operator
 ├── PythonOperator
 ├── BashOperator
 ├── SQL Operator
 └── ...

Sensor
 └── ждёт условие
```

---

# 🔹 Operator не равен Executor

Это частая путаница.

### Operator

Отвечает на вопрос:

> **Что нужно сделать?**

Например:

```text id="v3j8ma"
PythonOperator
→ выполнить Python-функцию
```

### Executor

Отвечает на вопрос:

> **Как и где выполнять задачи?**

Например:

```text id="k5p1rz"
Executor
→ определяет механизм выполнения Task
```

Поэтому:

```text id="y7q4nd"
Operator
     ↓
что выполнить

Executor
     ↓
как выполнить
```

---

# 🔹 Пример полного DAG

```python id="r8m4ty"
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

Структура:

```text id="q2m8vp"
DAG
 │
 ├── extract_task
 │      └── PythonOperator
 │
 ├── transform_task
 │      └── PythonOperator
 │
 └── load_task
        └── PythonOperator
```

Зависимости:

```text id="v7x1cz"
extract
   ↓
transform
   ↓
load
```

---

# ⚠️ Operator — это не сама выполняемая работа

Например:

```python id="s2w6jk"
PythonOperator(
    task_id="extract",
    python_callable=extract,
)
```

`PythonOperator` не является бизнес-данными и не является самой функцией `extract`.

Он задаёт **способ выполнения** этой функции в рамках Airflow Task.

---

# 🔥 Главное

```text id="f4q8nc"
                DAG
                 ↓
                Task
                 ↓
             Operator
            /        \
           ↓          ↓
 PythonOperator   BashOperator
      ↓                 ↓
 Python code        Shell command
```

### Ключевая фраза для собеседования

> Operator в Airflow — это класс, определяющий тип и способ выполнения задачи. На основе Operator создаётся конкретная Task внутри DAG. Например, `PythonOperator` выполняет Python-функцию, а `BashOperator` — shell-команду. Operator определяет, что делать, а зависимости между Tasks определяют порядок выполнения.
