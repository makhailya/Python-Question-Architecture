# 🔄 ETL / ELT

## 🎯 Ответ на собеседовании

**ETL (Extract, Transform, Load)** — процесс извлечения данных из источников, их преобразования и последующей загрузки в целевую систему.

**ELT (Extract, Load, Transform)** — сначала данные загружаются в целевое хранилище, а преобразование выполняется уже внутри него.

Airflow часто используется для **оркестрации ETL/ELT-пайплайнов**: он управляет порядком выполнения задач, зависимостями, расписанием, retry и мониторингом.

---

## 🎤 Суперкоротко

> ETL: **извлекли → преобразовали → загрузили**.
> ELT: **извлекли → загрузили → преобразовали**.

Главное отличие — **где и когда происходит Transform**.

---

# 🔹 ETL

Расшифровка:

```text
E — Extract
T — Transform
L — Load
```

Схема:

```text
Источник
   ↓
Extract
   ↓
Transform
   ↓
Load
   ↓
Data Warehouse
```

Например:

```text
PostgreSQL
    ↓
Extract
    ↓
очистка данных
    ↓
преобразование
    ↓
агрегация
    ↓
Data Warehouse
```

---

# 🔹 Extract

**Extract** — извлечение данных из источника.

Источниками могут быть:

* PostgreSQL;
* MySQL;
* REST API;
* CSV;
* JSON;
* Excel;
* S3/object storage;
* внешние сервисы.

Например:

```text
API
 ↓
GET /orders
 ↓
JSON
```

или:

```text
PostgreSQL
 ↓
SELECT ...
```

---

# 🔹 Transform

**Transform** — преобразование данных.

Например:

```text
грязные данные
      ↓
удалить дубликаты
      ↓
исправить типы
      ↓
заполнить пропуски
      ↓
нормализовать значения
      ↓
агрегировать
```

Пример на Python:

```python
def transform(data):
    data = remove_duplicates(data)
    data = normalize_dates(data)
    data = calculate_total(data)

    return data
```

---

# 🔹 Load

**Load** — загрузка обработанных данных в целевую систему.

Например:

```text
Python
  ↓
PostgreSQL
```

или:

```text
ETL
 ↓
Data Warehouse
```

---

# 🔹 Полный ETL

```text
             EXTRACT
                ↓
        API / PostgreSQL
                ↓
             TRANSFORM
                ↓
      очистка / агрегация
                ↓
               LOAD
                ↓
         Data Warehouse
```

---

# 🔹 ELT

В ELT порядок другой:

```text
E — Extract
L — Load
T — Transform
```

Схема:

```text
Источник
   ↓
Extract
   ↓
Load
   ↓
Data Warehouse
   ↓
Transform
   ↓
готовые данные
```

Сначала загружаем данные практически в исходном виде:

```text
API
 ↓
Raw data
 ↓
Data Warehouse
```

А затем выполняем преобразования уже внутри хранилища.

---

# 🔹 Почему появился ELT

Современные аналитические хранилища способны эффективно выполнять большие объёмы SQL-трансформаций.

Поэтому можно:

```text
загрузить данные
      ↓
Data Warehouse
      ↓
SQL
      ↓
Transform
```

Вместо:

```text
загрузить данные
      ↓
Python server
      ↓
Transform
      ↓
Data Warehouse
```

---

# 🔹 ETL vs ELT

| ETL                                            | ELT                                              |
| ---------------------------------------------- | ------------------------------------------------ |
| Extract → Transform → Load                     | Extract → Load → Transform                       |
| Transform до загрузки                          | Transform после загрузки                         |
| Часто используется Python/ETL-инструмент       | Часто используется SQL в хранилище               |
| В хранилище попадают обработанные данные       | Можно сохранять raw data                         |
| Меньше данных попадает в target                | В target сначала попадает больше исходных данных |
| Подходит для сложной предварительной обработки | Хорош для мощных аналитических хранилищ          |

---

# 🔹 Пример ETL

Есть API с заказами:

```text
API
 ↓
100 000 orders
 ↓
Python
 ↓
очистка
 ↓
агрегация
 ↓
PostgreSQL
```

Например, Python выполняет:

```python
orders = extract()
orders = transform(orders)
load(orders)
```

---

# 🔹 Пример ELT

```text
API
 ↓
100 000 orders
 ↓
PostgreSQL
 ↓
SQL transformation
 ↓
analytics_orders
```

Например:

```python
orders = extract()

load_to_database(orders)

run_sql_transformations()
```

---

# 🔹 ETL / ELT и Airflow

Airflow обычно **не является самим ETL-инструментом**.

Он **оркестрирует pipeline**.

Например:

```text
             Airflow DAG
                  │
       ┌──────────┼──────────┐
       ↓          ↓          ↓
    Extract   Transform     Load
       │          │          │
       ↓          ↓          ↓
      API       Python    PostgreSQL
```

Airflow отвечает за:

* порядок;
* зависимости;
* расписание;
* retry;
* мониторинг;
* запуск задач.

А сами задачи могут выполняться:

* Python;
* SQL;
* Spark;
* dbt;
* Bash;
* внешними сервисами.

---

# 🔹 ETL Pipeline в Airflow

Например:

```python
extract_task >> transform_task >> load_task
```

Получаем:

```text
Extract
   ↓
Transform
   ↓
Load
```

Для ELT:

```python
extract_task >> load_task >> transform_task
```

Получаем:

```text
Extract
   ↓
Load
   ↓
Transform
```

---

# 🔹 Batch Processing

ETL часто выполняется пакетами.

Например:

```text
Каждый день в 02:00
        ↓
Extract yesterday's data
        ↓
Transform
        ↓
Load
```

Это называется **batch processing**.

Airflow хорошо подходит для таких периодических workflow.

---

# 🔹 ETL и Streaming

Важно не путать batch и streaming.

### Batch

```text
Каждый час
    ↓
забрать 1 млн записей
    ↓
обработать
```

### Streaming

```text
Event 1 → обработка
Event 2 → обработка
Event 3 → обработка
Event 4 → обработка
```

Для streaming часто используют:

* Kafka;
* Flink;
* Spark Streaming;
* другие stream-processing системы.

Airflow преимущественно используется для **оркестрации batch/workflow**, а не как streaming engine.

---

# 🔹 Пример из Python Backend

Предположим, нужно каждый день собирать вакансии.

```text
HH API
   ↓
Extract
   ↓
JSON
   ↓
Transform
   ├── очистка
   ├── нормализация
   └── фильтрация
   ↓
Load
   ↓
PostgreSQL
```

Airflow может организовать это:

```text
Daily DAG
   ↓
Extract vacancies
   ↓
Transform vacancies
   ↓
Load PostgreSQL
```

---

# 🔹 ETL vs ELT — где происходит Transform

Самая важная формулировка:

```text
ETL:

Source
  ↓
Transform
  ↓
Target
```

```text
ELT:

Source
  ↓
Target
  ↓
Transform
```

Именно это отличие нужно уверенно объяснять на собеседовании.

---

# ⚠️ ETL — это не обязательно Airflow

Нельзя говорить:

> «ETL — это Airflow».

Правильно:

> **ETL — это процесс обработки данных, а Airflow — инструмент для оркестрации workflow, который может управлять ETL-процессом.**

---

# ⚠️ Airflow не обязательно сам преобразует данные

Например:

```text
Airflow
   ↓
запускает Python
   ↓
Python transform
```

или:

```text
Airflow
   ↓
запускает SQL
   ↓
Database transform
```

или:

```text
Airflow
   ↓
запускает Spark job
   ↓
Spark transform
```

Airflow управляет процессом, а специализированный инструмент выполняет обработку.

---

# 🔥 Главное

```text
ETL
Extract → Transform → Load

ELT
Extract → Load → Transform
```

### Ключевая фраза для собеседования

> ETL — это процесс, при котором данные сначала извлекаются из источника, затем преобразуются и после этого загружаются в целевую систему. В ELT сначала выполняется загрузка исходных данных, а преобразование происходит уже в целевой системе. Airflow может использоваться для оркестрации обоих процессов: управлять расписанием, зависимостями, retry и мониторингом задач.

### Связь с предыдущими темами

```text
Airflow
   ↓
DAG
   ↓
Tasks
   ↓
Operators
   ↓
Scheduler
   ↓
Executor
   ↓
ETL / ELT Pipeline
```
