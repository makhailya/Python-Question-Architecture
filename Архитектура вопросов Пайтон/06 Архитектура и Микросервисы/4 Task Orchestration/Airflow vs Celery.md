# ⚖️ Airflow vs Celery

## 🎯 Ответ на собеседовании

**Airflow и Celery решают разные задачи.**

**[[Airflow]]** — платформа для **оркестрации workflow**: описывает зависимости между задачами, запускает их по расписанию, отслеживает состояние, retry и предоставляет мониторинг.

[[Celery]] — распределённая система для **асинхронного и фонового выполнения задач** через очередь сообщений.

Главное различие:

> **Airflow управляет workflow, Celery выполняет фоновые задачи.**

---

## 🎤 Суперкоротко

```text
Airflow → "Какой workflow и в каком порядке выполнить?"
Celery  → "Выполни эту задачу асинхронно."
```

---

# 🔹 Airflow

Airflow строит workflow в виде **DAG**:

```text
Extract
   ↓
Transform
   ↓
Validate
   ↓
Load
```

Он умеет:

* планировать запуск;
* учитывать зависимости;
* выполнять задачи;
* делать retry;
* хранить состояние;
* показывать логи;
* мониторить workflow.

Типичный сценарий:

```text
Каждый день
    ↓
Airflow DAG
    ↓
Extract
    ↓
Transform
    ↓
Load
```

---

# 🔹 Celery

Celery используется для выполнения фоновых задач.

Например, пользователь отправил запрос:

```text
API
 ↓
Celery
 ↓
Queue
 ↓
Worker
 ↓
долгая задача
```

Пользователь при этом не обязан ждать завершения операции.

Например:

```python
@celery_app.task
def generate_report():
    ...
```

Задача отправляется worker'у:

```text
Producer
   ↓
Message Broker
   ↓
Celery Worker
   ↓
Task
```

В качестве брокера могут использоваться, например:

* RabbitMQ;
* Redis.

---

# 🔹 Основное отличие

| Airflow                    | Celery                  |
| -------------------------- | ----------------------- |
| Workflow orchestration     | Distributed task queue  |
| DAG                        | Очередь задач           |
| Зависимости между Tasks    | Отправка задач Worker   |
| Планирование workflow      | Асинхронное выполнение  |
| ETL / Data pipelines       | Background jobs         |
| Мониторинг DAG             | Мониторинг задач        |
| Scheduler                  | Broker + Workers        |
| Хорош для сложных workflow | Хорош для фоновых задач |

---

# 🔹 Пример: Celery

Допустим, API должен отправить email.

Без Celery:

```text
Client
  ↓
FastAPI
  ↓
Send email
  ↓
Response
```

Если отправка занимает несколько секунд, пользователь ждёт.

С Celery:

```text
Client
  ↓
FastAPI
  ↓
Celery task
  ↓
Response
```

А worker отдельно:

```text
Celery Worker
     ↓
Send email
```

Получается асинхронное выполнение фоновой задачи.

---

# 🔹 Пример: Airflow

Нужно каждый день обрабатывать данные:

```text
API
 ↓
Extract
 ↓
Transform
 ↓
Validate
 ↓
Load PostgreSQL
```

Здесь важен **workflow и зависимости**:

```text
Extract → Transform → Validate → Load
```

Airflow подходит лучше:

```text
Airflow
   ↓
DAG
   ↓
Tasks
```

---

# 🔹 Главное различие на примере

### Celery

```text
"Сделай эту задачу в фоне."
```

Например:

```text
generate_report
send_email
resize_image
process_file
```

### Airflow

```text
"Каждый день выполни этот workflow:
A → B → C → D."
```

Например:

```text
Extract
   ↓
Clean
   ↓
Transform
   ↓
Load
```

---

# 🔹 Airflow + Celery вместе

Они **не являются взаимоисключающими технологиями**.

Celery может использоваться как механизм распределённого выполнения задач Airflow — например, через **CeleryExecutor**.

Схема:

```text
              Airflow
                 │
            Scheduler
                 │
          CeleryExecutor
                 │
          Message Broker
           /           \
          ↓             ↓
     Celery Worker  Celery Worker
          ↓             ↓
       Task A          Task B
```

Получается:

```text
Airflow → управляет workflow
Celery  → помогает распределённо выполнять задачи
```

---

# 🔹 Airflow + Celery для ETL

Например:

```text
              Airflow DAG
                   │
             Scheduler
                   ↓
                Tasks
              /   |   \
             ↓    ↓    ↓
         Extract Transform Load
             \    |    /
                  ↓
           CeleryExecutor
             /        \
            ↓          ↓
        Worker 1    Worker 2
```

Airflow определяет зависимости:

```text
Extract → Transform → Load
```

А Celery workers могут выполнять отдельные Tasks.

---

# 🔹 Celery не является заменой Airflow

Можно попытаться построить workflow на Celery:

```text
Task A
 ↓
Task B
 ↓
Task C
```

Celery поддерживает chains, groups, chords и другие примитивы.

Но Airflow предоставляет специализированные возможности для **управления и мониторинга долгоживущих workflow**, особенно в data engineering.

Например:

```text
DAG
 ↓
schedule
 ↓
dependencies
 ↓
retries
 ↓
history
 ↓
logs
 ↓
monitoring
```

---

# 🔹 Airflow не является заменой Celery

Если FastAPI получает:

```text
"Сгенерировать PDF"
```

и нужно просто выполнить тяжёлую операцию в фоне:

```text
FastAPI
   ↓
Celery
   ↓
Worker
```

Airflow для такой задачи обычно избыточен.

---

# 🔹 Сравнение архитектур

### Celery

```text
FastAPI
   ↓
Celery
   ↓
RabbitMQ / Redis
   ↓
Worker
   ↓
Task
```

### Airflow

```text
Airflow
   ↓
DAG
   ↓
Scheduler
   ↓
Executor
   ↓
Task
```

### Airflow + CeleryExecutor

```text
Airflow
   ↓
Scheduler
   ↓
CeleryExecutor
   ↓
Broker
   ↓
Celery Workers
   ↓
Tasks
```

---

# 🔹 Когда выбирать Airflow

Используй Airflow, когда есть:

* сложный workflow;
* зависимости между задачами;
* ETL/ELT;
* data pipelines;
* периодическое расписание;
* необходимость видеть историю запусков;
* мониторинг всего pipeline.

Пример:

```text
Каждый день:

Extract
   ↓
Transform
   ↓
Validate
   ↓
Load
   ↓
Generate Report
```

---

# 🔹 Когда выбирать Celery

Используй Celery, когда нужны:

* фоновые задачи;
* асинхронное выполнение;
* распределённая обработка;
* очереди задач;
* retry задач;
* выполнение тяжёлых операций вне HTTP request.

Примеры:

```text
send_email
generate_pdf
resize_image
process_video
send_notification
```

---

# ⚠️ Важный нюанс

**Celery — не брокер сообщений.**

Celery использует брокер:

```text
Celery
   ↓
RabbitMQ / Redis
```

Упрощённо:

```text
Producer
   ↓
Broker
   ↓
Celery Worker
```

Celery — система выполнения задач поверх очереди.

---

# ⚠️ Airflow — не просто scheduler

Airflow действительно имеет **Scheduler**, но сам Airflow гораздо шире:

```text
Airflow
├── DAG
├── Tasks
├── Scheduler
├── Executor
├── Workers
├── Metadata DB
└── Web UI
```

Поэтому:

```text
Airflow ≠ cron
Airflow ≠ просто scheduler
```

---

# 🔥 Главное

```text
                    Задача
                       │
              ┌────────┴────────┐
              ↓                 ↓
           Workflow         Background job
              ↓                 ↓
           Airflow            Celery
              ↓                 ↓
             DAG              Queue
              ↓                 ↓
           Scheduler          Worker
```

### Ключевая фраза для собеседования

> Airflow и Celery решают разные задачи. Airflow предназначен для оркестрации workflow: DAG, зависимости, расписание, retry и мониторинг. Celery предназначен для распределённого и асинхронного выполнения фоновых задач через брокер сообщений. При этом Celery может использоваться внутри Airflow через CeleryExecutor для распределённого выполнения Tasks.

### Формула

```text
Airflow → Workflow orchestration
Celery  → Distributed task execution
```
