# ⚙️ Executor в Apache Airflow

## 🎯 Ответ на собеседовании

**Executor** — компонент Apache Airflow, который определяет **как и где будут выполняться [[Task]]**.

[[Scheduler]] определяет, какие задачи готовы к запуску, а Executor организует их фактическое выполнение с учётом выбранной модели исполнения.

---

## 🎤 Суперкоротко

> Executor определяет механизм выполнения Task: где и каким способом Airflow будет запускать задачи.

---

# 🔹 Scheduler vs Executor

Это главное различие:

```text id="e8x4mc"
DAG
 ↓
Scheduler
 ↓
"Task готова к запуску"
 ↓
Executor
 ↓
"Как её выполнить?"
 ↓
Worker / execution environment
```

Формула:

```text id="a4m8qp"
Scheduler → ЧТО и КОГДА запускать
Executor  → КАК выполнять
```

---

# 🔹 Зачем нужен Executor

Один и тот же DAG можно выполнять в разной инфраструктуре.

Например:

```text id="v7q2kn"
Один DAG
   │
   ├── локально
   ├── несколько процессов
   ├── Celery workers
   └── Kubernetes pods
```

Executor позволяет выбрать соответствующую модель выполнения.

---

# 🔹 Основные типы Executor

Для собеседования полезно знать несколько основных вариантов:

| Executor             | Идея                                                |
| -------------------- | --------------------------------------------------- |
| `SequentialExecutor` | Задачи выполняются последовательно                  |
| `LocalExecutor`      | Задачи выполняются локально с параллелизмом         |
| `CeleryExecutor`     | Задачи выполняются на распределённых Celery workers |
| `KubernetesExecutor` | Для задач создаются Kubernetes Pods                 |

---

# 🔹 SequentialExecutor

Самый простой вариант.

```text id="w8k3mz"
Task A
  ↓
Task B
  ↓
Task C
```

Следующая задача выполняется после предыдущей.

Плюсы:

* простой;
* удобен для минимальных установок/тестов.

Минус:

```text id="p4r7vc"
нет нормального параллельного выполнения
```

Для production с большим количеством задач обычно не подходит.

---

# 🔹 LocalExecutor

Задачи выполняются **на той же инфраструктуре, где работает Airflow**, с возможностью параллельного выполнения.

Например:

```text id="x2n9qk"
              Airflow
                 │
           LocalExecutor
          /       |       \
         ↓        ↓        ↓
      Task A   Task B   Task C
```

Если задачи независимы:

```text id="k6m3vp"
        Extract
        /     \
       ↓       ↓
    Clean   Validate
```

`Clean` и `Validate` могут выполняться параллельно.

---

# 🔹 CeleryExecutor

Используется для распределённого выполнения задач через **Celery workers**.

Упрощённая схема:

```text id="r8w4jc"
             Scheduler
                 ↓
              Executor
                 ↓
          Celery infrastructure
                 ↓
       ┌─────────┼─────────┐
       ↓         ↓         ↓
    Worker 1  Worker 2  Worker 3
       ↓         ↓         ↓
     Task A    Task B    Task C
```

Это позволяет распределять выполнение задач между несколькими worker-машинами.

Полезно для:

* большого количества задач;
* горизонтального масштабирования;
* распределённой инфраструктуры.

---

# 🔹 KubernetesExecutor

В Kubernetes задача может выполняться в отдельном **Pod**.

Упрощённо:

```text id="g4m8zs"
Airflow
   ↓
KubernetesExecutor
   ↓
┌───────────────┐
│ Pod           │
│ Task A        │
└───────────────┘

┌───────────────┐
│ Pod           │
│ Task B        │
└───────────────┘
```

Каждая Task может получить изолированное окружение.

Преимущества:

* изоляция;
* динамическое создание execution environment;
* хорошо подходит для Kubernetes-инфраструктуры;
* удобное масштабирование.

---

# 🔹 Executor и Worker

Не все Executor используют отдельные worker-машины одинаковым образом.

Например:

```text id="v6q2ps"
CeleryExecutor
      ↓
Celery Workers
      ↓
Tasks
```

А при KubernetesExecutor:

```text id="s8n4kc"
KubernetesExecutor
      ↓
Kubernetes Pods
      ↓
Tasks
```

Поэтому нельзя говорить:

> «Executor — это Worker».

Правильнее:

> **Executor определяет механизм выполнения и взаимодействия с execution environment.**

---

# 🔹 Executor и Operator

Не путай эти понятия.

### Operator

Определяет:

> **Что должна делать Task?**

Например:

```python id="c7m2vz"
PythonOperator(
    task_id="extract",
    python_callable=extract,
)
```

### Executor

Определяет:

> **Как организовать выполнение этой Task?**

Упрощённо:

```text id="n5x8qa"
Operator
   ↓
Что делать?
   ↓
Task

Executor
   ↓
Как выполнять?
   ↓
Worker / Pod / Process
```

---

# 🔹 Executor и Scheduler

Ещё одно важное сравнение:

```text id="y7q4nm"
Scheduler
   ↓
планирует Task
   ↓
Executor
   ↓
организует выполнение
   ↓
Execution environment
```

Например:

```text id="f2k8vc"
Scheduler
    ↓
"extract готова"
    ↓
CeleryExecutor
    ↓
Worker 2
    ↓
extract
```

---

# 🔹 Executor и масштабирование

Выбор Executor влияет на возможности масштабирования.

Условно:

```text id="z4p7mc"
Sequential
    ↓
один поток выполнения

Local
    ↓
несколько локальных задач

Celery
    ↓
несколько Worker

Kubernetes
    ↓
динамические Pods
```

Поэтому при росте нагрузки можно использовать распределённые модели выполнения.

---

# 🔹 Пример

Допустим, есть DAG:

```text id="j8q2ws"
             Extract
             /     \
            ↓       ↓
         Clean   Validate
            \       /
             ↓     ↓
              Load
```

Scheduler определяет:

```text id="c6v9py"
Clean и Validate готовы
```

Executor организует их выполнение:

```text id="m2k7vx"
          Executor
          /      \
         ↓        ↓
     Worker 1  Worker 2
       Clean    Validate
```

После завершения обеих:

```text id="a5r8nz"
Clean ────┐
          ├──→ Load
Validate ─┘
```

---

# ⚠️ Executor не выбирает бизнес-логику

Executor не знает:

```text id="n7k3pz"
что такое заказ
что такое платёж
что такое ETL
что такое пользователь
```

Его задача инфраструктурная:

```text id="q8m4xc"
получить готовую Task
        ↓
организовать её выполнение
```

---

# 🔥 Главное

```text id="w4q8mn"
                   DAG
                    ↓
                Scheduler
                    ↓
             готовая Task
                    ↓
                Executor
                    ↓
        ┌───────────┼───────────┐
        ↓           ↓           ↓
      Local       Celery     Kubernetes
        ↓           ↓           ↓
      Task       Worker       Pod
```

### Ключевая фраза для собеседования

> Executor в Airflow отвечает за механизм выполнения задач. Scheduler определяет, какие Task готовы к запуску, а Executor передаёт их соответствующей execution environment. В зависимости от выбранного Executor задачи могут выполняться локально, на распределённых Celery workers или в Kubernetes Pods.

### Цепочка Airflow

```text id="p7v3ka"
DAG
 ↓
Task
 ↓
Operator → что делать
 ↓
Scheduler → когда запускать
 ↓
Executor → как выполнять
 ↓
Worker / Pod → фактическое выполнение
```
