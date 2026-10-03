# 👁️ Observability

## 🎤 Короткий ответ

**Observability (наблюдаемость)** — это способность системы по её внешним данным понять, **что внутри неё происходит и почему возникла проблема**.

Основные источники данных:

> **Metrics + Logs + Traces = три основных столпа Observability.**

Например, если API стало медленным:

```text
Metrics → показывают, что latency выросла
Logs    → показывают ошибку
Traces  → показывают, какой сервис/запрос тормозит
```

## 🎯 Формула для собеседования

> **Observability = [[Метрики]] + [[Логи]] + [[Трейсинг]] → понимание состояния системы и поиск причины проблем.**

---

# 🔹 Что такое Observability

В распределённой системе недостаточно знать:

```text
"Сервис не работает"
```

Нужно понять:

```text
Что сломалось?
        ↓
Где сломалось?
        ↓
Почему сломалось?
        ↓
Какой запрос затронут?
        ↓
Какой сервис является причиной?
```

Observability предоставляет данные, необходимые для ответа на эти вопросы.

---

# 🔹 Три основных столпа

```text id="m5q8vx"
             Observability
                  │
       ┌──────────┼──────────┐
       ↓          ↓          ↓
    Metrics      Logs      Traces
       │          │          │
    числа       события    путь запроса
```

---

# 🔹 Metrics

**Metrics (метрики)** — числовые показатели состояния системы, обычно собираемые во времени.

Примеры:

```text id="q7m3kc"
CPU usage
Memory usage
Request rate
Error rate
Request latency
Database connections
Kafka consumer lag
```

Например:

```text id="x4n8vp"
HTTP requests/sec = 1500
Error rate       = 2.4%
P95 latency      = 850 ms
CPU              = 78%
```

По метрикам можно быстро понять:

> **Что происходит с системой прямо сейчас и как это меняется со временем.**

---

# 🔹 RED

Для HTTP-сервисов часто используют методику **RED**:

```text
R — Rate
E — Errors
D — Duration
```

### Rate

Количество запросов:

```text
requests/sec
```

### Errors

Количество/доля ошибок:

```text
5xx rate
```

### Duration

Время обработки:

```text
P50
P95
P99
```

Например:

```text id="a8m3qx"
Rate     → 1000 req/s
Errors   → 3%
P95      → 700 ms
```

---

# 🔹 USE

Для инфраструктуры часто используют **USE**:

```text
U — Utilization
S — Saturation
E — Errors
```

Например для CPU:

```text
Utilization → 90%
Saturation  → очередь задач растёт
Errors      → ошибки CPU/системы
```

---

# 🔹 Logs

**Logs (логи)** — записи о событиях, происходящих в системе.

Например:

```python id="r6m9vc"
2026-09-16 20:40:12 ERROR
Database connection failed
```

Логи могут содержать:

* timestamp;
* уровень (`INFO`, `WARNING`, `ERROR`);
* сообщение;
* service;
* request ID;
* stack trace;
* дополнительные поля.

---

# 🔹 Structured Logs

Лучше использовать структурированные логи.

Например:

```python id="k3q8mx"
{
    "level": "ERROR",
    "service": "payment",
    "request_id": "abc123",
    "error": "database_timeout"
}
```

В production структурированные логи проще:

* фильтровать;
* искать;
* агрегировать;
* анализировать автоматически.

---

# 🔹 Log Levels

Типичные уровни:

```text id="v7m4qx"
DEBUG
INFO
WARNING
ERROR
CRITICAL
```

Условно:

| Level      | Назначение                           |
| ---------- | ------------------------------------ |
| `DEBUG`    | подробная информация для диагностики |
| `INFO`     | обычные события                      |
| `WARNING`  | потенциальная проблема               |
| `ERROR`    | ошибка                               |
| `CRITICAL` | критическая проблема                 |

Конкретный набор уровней зависит от logging framework.

---

# 🔹 Traces

**Distributed Trace** показывает путь одного запроса через систему.

Например:

```text id="p8m3vx"
Client
  ↓
API Gateway
  ↓
FastAPI
  ↓
Payment Service
  ↓
PostgreSQL
  ↓
Redis
```

Trace позволяет увидеть:

```text id="q4n7kc"
Request: abc123

API Gateway      20 ms
FastAPI          50 ms
Payment Service  800 ms
PostgreSQL       750 ms
```

И становится видно:

> Основная задержка находится при обращении к PostgreSQL.

---

# 🔹 Span

**Span** — отдельная операция внутри Trace.

Например:

```text id="m6q2vx"
Trace
 │
 ├── HTTP request
 │
 ├── DB query
 │
 └── Redis GET
```

Каждая операция может быть отдельным Span.

У Span обычно есть:

* start time;
* duration;
* operation name;
* service;
* attributes;
* status;
* trace ID.

---

# 🔹 Trace ID и Correlation

Представим запрос:

```text id="x8m4qp"
Client
   ↓
FastAPI
   ↓
Order Service
   ↓
Payment Service
```

Всем компонентам можно передавать один:

```text
trace_id = abc123
```

Тогда можно связать:

```text id="k5n7vc"
Trace abc123
      │
      ├── FastAPI log
      ├── Order Service log
      ├── Payment Service log
      └── DB span
```

Это значительно упрощает диагностику распределённых систем.

---

# 🔹 Metrics vs Logs vs Traces

|                   | Metrics             | Logs                | Traces                 |
| ----------------- | ------------------- | ------------------- | ---------------------- |
| Формат            | Числа               | События             | Путь запроса           |
| Хорошо показывает | Состояние/тренд     | Детали ошибки       | Где задержка           |
| Пример            | P95 = 500 ms        | DB timeout          | DB span = 450 ms       |
| Объём данных      | Обычно небольшой    | Большой             | Средний/большой        |
| Основная задача   | Обнаружить проблему | Исследовать событие | Найти путь/узкое место |

Они **дополняют друг друга**, а не заменяют.

---

# 🔹 Пример диагностики

Пользователь говорит:

> API работает медленно.

### Шаг 1 — Metrics

Смотрим:

```text id="r3m8qx"
P95 latency:
200 ms → 900 ms
```

Проблема подтверждена.

### Шаг 2 — Logs

Находим:

```text id="v7q4kc"
database timeout
```

### Шаг 3 — Trace

Видим:

```text id="m8n2vx"
FastAPI      30 ms
Redis        10 ms
PostgreSQL  850 ms
```

Получаем цепочку:

```text id="x5q7mc"
Latency ↑
   ↓
DB timeout
   ↓
PostgreSQL operation = 850 ms
```

---

# 🔹 Observability vs Monitoring

Эти понятия связаны, но не полностью идентичны.

### Monitoring

Отвечает преимущественно:

> **Всё нормально или есть проблема?**

Например:

```text
CPU > 90%
Error rate > 5%
```

Можно создать alert.

### Observability

Помогает ответить:

> **Почему возникла проблема?**

Например:

```text
Error rate ↑
   ↓
Trace
   ↓
Payment Service
   ↓
PostgreSQL
   ↓
Slow query
```

Упрощённо:

> **Monitoring обнаруживает проблему, Observability помогает исследовать её причину.**

---

# 🔹 Observability в Kubernetes

Для Kubernetes Observability особенно важна, потому что Pods постоянно создаются, удаляются и перемещаются между Nodes.

Например:

```text id="q8m3vx"
Cluster
 ├── Node 1
 │    ├── Pod
 │    └── Pod
 │
 ├── Node 2
 │    ├── Pod
 │    └── Pod
 │
 └── Node 3
      └── Pod
```

Нужно понимать:

* состояние Nodes;
* состояние Pods;
* CPU/RAM;
* рестарты контейнеров;
* ошибки;
* latency;
* network;
* application metrics.

---

# 🔹 Типичный стек Observability

Один из распространённых вариантов:

```text id="m4q8vx"
Applications
     │
     ├────────→ Metrics
     │
     ├────────→ Logs
     │
     └────────→ Traces
                   │
          ┌────────┼────────┐
          ↓        ↓        ↓
      Prometheus   Loki    Tempo
          │        │        │
          └────────┼────────┘
                   ↓
                Grafana
```

Это только один из вариантов архитектуры.

---

# 🔹 Prometheus

**Prometheus** — система мониторинга и хранения временных рядов, широко используемая для metrics.

Например:

```text
http_requests_total
http_request_duration_seconds
process_cpu_seconds_total
```

Prometheus обычно периодически **scrape'ит** metrics endpoints.

---

# 🔹 Grafana

**Grafana** используется для визуализации данных.

Например:

```text
Grafana
   ↓
Dashboard
   ├── CPU
   ├── RAM
   ├── RPS
   ├── Error rate
   ├── P95
   └── Kafka Lag
```

Grafana может работать с различными источниками данных.

---

# 🔹 Loki

**Loki** — система хранения и поиска логов, часто используемая вместе с Grafana.

Схема:

```text
Application
    ↓
Logs
    ↓
Loki
    ↓
Grafana
```

---

# 🔹 OpenTelemetry

**OpenTelemetry (OTel)** — открытый набор стандартов, API и SDK для сбора telemetry:

* metrics;
* logs;
* traces.

Упрощённо:

```text id="x7m3qp"
Application
      ↓
OpenTelemetry
      ↓
Telemetry
 ┌────┼────┐
 ↓    ↓    ↓
Logs Metrics Traces
```

OpenTelemetry не является просто «одной системой хранения». Он предоставляет стандартные инструменты и протоколы для instrumentation и передачи telemetry в backend'ы.

---

# 🔹 Alerting

Observability должна помогать не только смотреть dashboards, но и обнаруживать проблемы.

Например:

```text id="p6q8mx"
Error rate > 5%
       ↓
    Alert
       ↓
Engineer
```

Другие примеры:

```text
P95 latency > 1 sec
CPU > 90%
Disk > 85%
Kafka Consumer Lag растёт
Pod restart count ↑
```

---

# 🔹 SLI, SLO, SLA

Эти понятия часто спрашивают вместе с Observability.

### SLI

**Service Level Indicator** — измеримый показатель качества сервиса.

Например:

```text
99.95% запросов успешны
```

### SLO

**Service Level Objective** — целевое значение SLI.

Например:

```text
SLO:
99.9% запросов должны завершаться успешно
```

### SLA

**Service Level Agreement** — договорное обязательство перед клиентом.

Например:

```text
SLA:
99.9% availability
```

Упрощённо:

```text id="w3m7qx"
SLI → что измеряем
SLO → какую цель ставим
SLA → что обещаем по договору
```

---

# 🔹 Error Budget

Если SLO:

```text
99.9% availability
```

то допустимый уровень недоступности:

```text
0.1%
```

Это и есть основа **Error Budget** — допустимый объём нарушения SLO.

Если Error Budget быстро расходуется, команда может уделить больше внимания надёжности.

---

# 🔹 Cardinality

В Observability есть важное понятие **cardinality** — количество уникальных значений label/attribute.

Например, опасно использовать:

```text
user_id
request_id
```

как labels в метриках.

Если у нас:

```text
10 000 000 пользователей
```

получаем огромное количество уникальных time series.

Это может сильно увеличить потребление памяти и стоимость monitoring-системы.

Поэтому:

```text
Metrics:
service=payment
status=500
method=POST
```

обычно лучше, чем:

```text
user_id=12345678
```

Для высококардинальных данных чаще подходят logs или traces.

---

# 🔹 Observability и Performance

Observability сама по себе тоже имеет стоимость.

Например:

* слишком много логов → больше storage;
* слишком подробные traces → больше overhead;
* высокая cardinality metrics → больше memory/storage;
* stack traces на каждый запрос → большой объём данных.

Поэтому telemetry тоже проектируют.

---

# 🔹 Частые вопросы на собеседовании

### Что такое Observability?

> Способность понимать внутреннее состояние системы по внешним telemetry-данным и находить причины проблем.

### Какие три основных столпа Observability?

> Metrics, Logs и Traces.

### Чем Metrics отличаются от Logs?

> Metrics — числовые агрегированные показатели, Logs — записи отдельных событий и их деталей.

### Что такое Trace?

> Представление полного пути одного запроса через компоненты распределённой системы.

### Что такое Span?

> Отдельная операция внутри Trace.

### Что такое Trace ID?

> Идентификатор, позволяющий связать операции одного распределённого запроса между сервисами.

### Monitoring vs Observability?

> Monitoring помогает обнаружить проблему, Observability помогает исследовать её причины.

### Что такое OpenTelemetry?

> Открытый стандартный набор API, SDK и инструментов для instrumentation и сбора/передачи telemetry.

### Зачем Prometheus?

> Для сбора и хранения metrics как временных рядов и выполнения запросов к ним.

### Зачем Grafana?

> Для визуализации metrics и других данных в dashboards.

### Что такое SLI?

> Измеримый показатель качества сервиса.

### Что такое SLO?

> Целевое значение SLI.

### Что такое SLA?

> Договорное обязательство по уровню сервиса.

### Что такое Cardinality?

> Количество уникальных комбинаций значений labels/attributes; высокая cardinality особенно опасна для metrics.

---

## 🔑 Главное

```text id="q5m8vx"
                    Observability
                         │
          ┌──────────────┼──────────────┐
          ↓              ↓              ↓
       Metrics          Logs          Traces
          │              │              │
      "Что?"          "Что было?"    "Где?"
          │              │              │
          └──────────────┼──────────────┘
                         ↓
                    Diagnosis
                         ↓
                 "Почему сломалось?"
```

### 🎤 Финальная формула

> **Observability — это способность понять состояние и причины поведения системы по telemetry-данным. Три основных компонента — Metrics, Logs и Traces: metrics показывают числовые показатели и тренды, logs — детали событий, traces — путь конкретного запроса через систему. В Kubernetes Observability особенно важна из-за динамики Pods и Nodes; типичный стек может включать OpenTelemetry, Prometheus, Grafana и систему хранения логов/трейсов.**
