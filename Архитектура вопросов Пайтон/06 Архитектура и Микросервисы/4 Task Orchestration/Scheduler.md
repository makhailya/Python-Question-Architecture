# ⏱️ Scheduler в Apache Airflow

## 🎯 Ответ на собеседовании

**Scheduler** — это компонент Apache Airflow, который следит за [[DAG]] и определяет, **какие задачи и когда можно запустить**.

Он учитывает расписание DAG, зависимости между Tasks и их текущее состояние, после чего передаёт готовые задачи на выполнение.

---

## 🎤 Суперкоротко

> Scheduler — компонент Airflow, который анализирует DAG, расписание и зависимости и определяет, какие Task готовы к запуску.

---

# 🔹 Где находится Scheduler

Упрощённая архитектура:

```text id="5s7w2c"
                DAG
                 ↓
             Scheduler
                 ↓
              Executor
                 ↓
              Worker
                 ↓
                Task
```

Логика:

```text id="9q3m1v"
DAG
 ↓
что и в каком порядке выполнять

Scheduler
 ↓
когда Task готова к запуску

Executor
 ↓
как организовать выполнение

Worker
 ↓
фактически выполняет Task
```

---

# 🔹 Что делает Scheduler

Основные задачи Scheduler:

* проверяет расписание DAG;
* анализирует зависимости между Tasks;
* определяет готовые Tasks;
* учитывает состояние предыдущих запусков;
* создаёт/планирует Task Instances;
* передаёт задачи на выполнение через Executor.

Упрощённо:

```text id="w7m2kd"
Scheduler
    ↓
"Наступило время запуска DAG?"
    ↓
"Выполнены ли зависимости?"
    ↓
"Можно ли запускать Task?"
    ↓
Executor
```

---

# 🔹 Scheduler и зависимости

Допустим:

```text id="x5n8qc"
Extract
   ↓
Transform
   ↓
Load
```

Scheduler не запустит:

```text id="8q4v2p"
Transform
```

пока `Extract` не удовлетворяет необходимым условиям для запуска.

После успешного выполнения:

```text id="5m9zq1"
Extract → SUCCESS
             ↓
          Scheduler
             ↓
         Transform
```

---

# 🔹 Scheduler и расписание

Допустим DAG настроен:

```python id="e8x3pq"
with DAG(
    dag_id="daily_etl",
    schedule="@daily",
    ...
):
    ...
```

Scheduler отслеживает это расписание и создаёт план запуска DAG.

Условно:

```text id="c5w2ha"
Каждый день
    ↓
Scheduler
    ↓
DAG Run
    ↓
Tasks
```

---

# 🔹 Scheduler не выполняет бизнес-код

Это важное различие.

Scheduler **не должен сам выполнять**:

```python id="x4k7nv"
def process_data():
    ...
```

Он решает:

> «Эту Task пора запускать».

А непосредственное выполнение организуется через **Executor** и соответствующую инфраструктуру.

```text id="r8p2lm"
Scheduler
    ↓
Executor
    ↓
Worker
    ↓
Python code
```

---

# 🔹 Scheduler vs Executor

Это одна из самых частых путаниц.

| Scheduler                         | Executor                         |
| --------------------------------- | -------------------------------- |
| Решает, **что и когда запускать** | Определяет, **как запускать**    |
| Анализирует DAG                   | Организует механизм выполнения   |
| Проверяет dependencies            | Передаёт задачи исполнителям     |
| Планирует Task Instances          | Определяет execution environment |

Формула:

```text id="k2m8xq"
Scheduler = WHEN / WHAT
Executor  = HOW
```

Упрощённо:

> Scheduler принимает решение о запуске, Executor отвечает за механизм выполнения.

---

# 🔹 Scheduler vs Worker

```text id="m5v9cr"
Scheduler
   ↓
решает запустить Task
   ↓
Executor
   ↓
передаёт Task
   ↓
Worker
   ↓
выполняет Task
```

Поэтому:

```text id="z7q3mp"
Scheduler ≠ Worker
```

Scheduler не является исполнителем бизнес-кода.

---

# 🔹 Scheduler и DAG Run

Scheduler участвует в создании и планировании запусков DAG.

Например:

```text id="a3k6fy"
DAG: daily_etl

        Scheduler
             ↓
      DAG Run #1
             ↓
        Tasks
```

На следующий период:

```text id="q8n4vw"
        Scheduler
             ↓
      DAG Run #2
             ↓
        Tasks
```

---

# 🔹 Scheduler и Task Instance

Scheduler определяет, когда конкретный Task Instance может перейти к выполнению.

Например:

```text id="c7j2px"
Task:
extract

DAG Run:
2026-09-11

Task Instance:
extract @ 2026-09-11
```

Scheduler проверяет её состояние и зависимости:

```text id="u4m8zs"
Task Instance
      ↓
dependencies satisfied?
      ↓
     YES
      ↓
Executor
```

---

# 🔹 Что такое «готовая» Task

Task может быть запущена, когда необходимые условия выполнены.

Например:

```text id="w2r6cy"
Extract → SUCCESS
     ↓
Transform
     ↓
можно запускать
```

Если:

```text id="z5m1qp"
Extract → FAILED
     ↓
Transform
     ↓
не запускается
```

Если предусмотрен retry:

```text id="f9k3vw"
Extract → FAILED
     ↓
retry
     ↓
SUCCESS
     ↓
Transform
```

Scheduler учитывает эти состояния.

---

# 🔹 Scheduler и параллельность

DAG может содержать независимые Tasks:

```text id="n6t4xz"
        Extract
        /     \
       ↓       ↓
    Clean   Validate
```

Scheduler может запланировать обе задачи, если их зависимости выполнены и ограничения параллельности позволяют это сделать.

То есть Scheduler не просто идёт по DAG сверху вниз.

Он анализирует:

```text id="r3k7qp"
Dependencies
+
Task states
+
Scheduling
+
Concurrency limits
```

и определяет, какие задачи можно запускать.

---

# 🔹 Несколько Scheduler

В современных конфигурациях Airflow можно использовать несколько экземпляров Scheduler для повышения отказоустойчивости и производительности.

Упрощённо:

```text id="v8m2sx"
Scheduler 1 ─┐
             ├── Airflow metadata DB
Scheduler 2 ─┘
```

Это позволяет уменьшить зависимость от одного экземпляра Scheduler.

Для Junior достаточно знать сам принцип:

> Scheduler является критически важным компонентом, поэтому в production его можно запускать в отказоустойчивой конфигурации.

---

# 🔹 Scheduler и Metadata Database

Airflow хранит состояние DAG и Task Instances в **metadata database**.

Упрощённо:

```text id="b4x8nm"
             Scheduler
                 ↓
       Metadata Database
                 ↑
                 │
              Web UI
```

В БД хранится информация о:

* DAG Runs;
* Task Instances;
* состояниях задач;
* расписаниях;
* других объектах Airflow.

Scheduler использует эту информацию для принятия решений.

---

# ⚠️ Scheduler не является обычным cron

Сравнение:

```text id="m7x3qp"
Cron
 ↓
"Запусти скрипт в 03:00"
```

Scheduler:

```text id="q4n8vz"
DAG
 ↓
Schedule
 ↓
Dependencies
 ↓
Task states
 ↓
Concurrency
 ↓
готовые Tasks
```

То есть Scheduler учитывает **состояние и зависимости workflow**, а не только время.

---

# 🔥 Главное

```text id="w3p9ks"
                 DAG
                  ↓
              Scheduler
                  ↓
        ┌─────────┴─────────┐
        ↓                   ↓
  Dependencies          Schedule
        │                   │
        └─────────┬─────────┘
                  ↓
           Ready Tasks
                  ↓
              Executor
                  ↓
               Worker
```

### Ключевая фраза для собеседования

> Scheduler в Airflow — это компонент, который анализирует DAG, расписание, зависимости и состояния Task Instances и определяет, какие задачи готовы к запуску. Он не выполняет бизнес-код сам, а передаёт готовые задачи Executor, который организует их выполнение.

### Формула

```text id="a9f2mc"
Scheduler → ЧТО и КОГДА запускать
Executor  → КАК запускать
Worker    → ГДЕ фактически выполнить
```
