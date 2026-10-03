# Grafana 📊

## 🎤 Короткий ответ

**Grafana** — это платформа для **визуализации и анализа метрик и других данных**. Она подключается к источникам данных, например **[[Prometheus]]**, строит графики, дашборды и позволяет настраивать алерты.

Проще:

> **Prometheus собирает и хранит метрики → Grafana показывает их на дашбордах.**

---

## 🎯 Формула для собеседования

**Grafana = Data Source → Query → Dashboard → Visualization → Alert**

Например:

```python
Application
     ↓
 /metrics
     ↓
Prometheus
     ↓
  PromQL
     ↓
Grafana
     ↓
Dashboard
     ↓
Graphs / Tables / Alerts
```

---

## 1. Что такое Grafana

**Grafana** — инструмент для визуализации данных из различных источников.

Она сама обычно **не является основным хранилищем метрик**.

Grafana подключается к Data Source и выполняет запросы к нему.

Источниками могут быть:

* Prometheus
* PostgreSQL
* Loki
* Elasticsearch
* InfluxDB
* Tempo
* Jaeger
* и другие системы.

Например:

```text
Prometheus
     ↓
  metrics
     ↓
  Grafana
     ↓
график CPU
график памяти
RPS
ошибки
latency
```

---

# 2. Grafana и Prometheus

Это одна из самых важных связок для собеседования.

### Prometheus

Отвечает за:

* сбор метрик;
* хранение time series;
* запросы через PromQL;
* вычисление показателей.

### Grafana

Отвечает за:

* визуализацию;
* dashboards;
* графики;
* таблицы;
* gauges;
* переменные;
* отображение нескольких источников;
* визуальные настройки;
* часть alerting-функциональности.

```text
Application
     │
     │ metrics
     ▼
Prometheus
     │
     │ PromQL
     ▼
Grafana
     │
     ├── Graph
     ├── Table
     ├── Gauge
     └── Dashboard
```

### Главное

**Prometheus — собирает и хранит.
Grafana — визуализирует и анализирует.**

---

# 3. Data Source

**Data Source** — источник данных, из которого Grafana получает информацию.

Например, подключаем Prometheus:

```text
Grafana
   │
   ▼
Prometheus
```

Или PostgreSQL:

```text
Grafana
   │
   ▼
PostgreSQL
```

Или Loki:

```text
Grafana
   │
   ▼
Loki
```

Поэтому Grafana не ограничена только метриками.

---

# 4. Dashboard

**Dashboard** — набор визуализаций, собранных на одной странице.

Например, dashboard backend-приложения:

```text
┌─────────────────────────────────────┐
│         Backend Monitoring          │
├─────────────┬───────────┬───────────┤
│    RPS      │  Errors   │  P95      │
│    250      │   0.4%    │  180 ms   │
├─────────────┴───────────┴───────────┤
│                                     │
│       Requests over time            │
│       📈                            │
│                                     │
├─────────────────────────────────────┤
│       CPU / Memory                  │
│       📈                            │
├─────────────────────────────────────┤
│       Database Connections          │
│       📈                            │
└─────────────────────────────────────┘
```

Один dashboard может содержать много **панелей (Panels)**.

---

# 5. Panel

**Panel** — отдельный элемент визуализации на dashboard.

Например:

* график RPS;
* график CPU;
* таблица ошибок;
* Gauge с использованием RAM;
* график latency;
* количество активных пользователей.

```text
Dashboard
    │
    ├── Panel: RPS
    ├── Panel: Error Rate
    ├── Panel: P95 Latency
    ├── Panel: CPU
    └── Panel: Memory
```

---

# 6. Query

Каждая Panel обычно получает данные через **Query**.

Для Prometheus используется **PromQL**.

Например:

```python
rate(http_requests_total[5m])
```

Grafana отправляет этот запрос в Prometheus и получает временной ряд.

Затем строит его на графике.

```text
Panel
  ↓
PromQL
  ↓
Prometheus
  ↓
Time Series
  ↓
Grafana
  ↓
Graph
```

---

# 7. Визуализации

Grafana поддерживает разные способы отображения данных.

Основные:

| Visualization | Назначение                        |
| ------------- | --------------------------------- |
| Time series   | график изменения во времени       |
| Stat          | одно значение                     |
| Gauge         | показатель относительно диапазона |
| Table         | таблица                           |
| Bar chart     | столбцы                           |
| Pie chart     | доли                              |
| Heatmap       | распределение                     |
| Logs          | просмотр логов                    |

Например:

**CPU:**

```text
████████████░░░░ 75%
```

**RPS:**

```text
       ╭──╮
   ╭───╯  ╰──╮
───╯         ╰────
```

---

# 8. Labels и Variables

Grafana позволяет создавать **переменные dashboard**.

Например:

```text
Environment: [production ▼]
Service:     [api ▼]
Instance:    [server-01 ▼]
```

После выбора значения графики автоматически фильтруются.

Например:

```text
service="api"
environment="production"
```

Это позволяет использовать один dashboard для разных:

* серверов;
* сервисов;
* окружений;
* Kubernetes namespaces;
* Pod'ов.

---

# 9. Alerting

Grafana может использовать данные для создания **alert rules**.

Например:

```text
CPU > 90%
        ↓
    Alert
        ↓
Telegram / Email / Slack
```

Или:

```text
Error Rate > 5%
        ↓
      Alert
```

Важно различать:

**Metric** — данные.

**Dashboard** — визуализация.

**Alert** — правило, которое реагирует на условие.

---

# 10. Grafana Alerting

В современных системах Grafana имеет собственный механизм alerting.

Например:

```text
Prometheus
     ↓
metric
     ↓
Grafana Alert Rule
     ↓
condition
     ↓
Alert
     ↓
Contact Point
     ↓
Telegram / Email / Slack
```

Grafana может использовать данные из разных источников для построения правил.

---

# 11. Grafana + Loki

Grafana часто используется не только с Prometheus, но и с **Loki**.

Получается:

```text
Metrics → Prometheus → Grafana
Logs    → Loki       → Grafana
```

Таким образом, в одном интерфейсе можно смотреть:

* метрики;
* логи;
* графики;
* ошибки.

Например:

```text
P95 latency ↑
      ↓
Открываем логи
      ↓
Видим timeout PostgreSQL
```

Это уже полноценная часть observability.

---

# 12. Grafana + Tempo / Jaeger

Для distributed tracing можно использовать:

```text
Metrics → Prometheus
Logs    → Loki
Traces  → Tempo
             ↓
          Grafana
```

Получается единый интерфейс для трёх основных сигналов observability:

```text
        Observability
             │
    ┌────────┼────────┐
    ↓        ↓        ↓
 Metrics    Logs    Traces
    ↓        ↓        ↓
Prometheus  Loki    Tempo
    └────────┼────────┘
             ↓
          Grafana
```

---

# 13. Grafana в Kubernetes

Типичная схема:

```text
Kubernetes
     │
     ├── Pods
     │     ↓
     │   Metrics
     │
     └──────────────┐
                    ↓
                Prometheus
                    ↓
                 Grafana
                    ↓
                Dashboard
```

На dashboard можно смотреть:

* CPU Pods;
* RAM Pods;
* количество рестартов;
* состояние Nodes;
* количество Pod'ов;
* HTTP requests;
* latency;
* errors;
* PostgreSQL;
* Redis;
* Kafka.

---

# 14. RED + Grafana

Для backend API удобно использовать методику **RED**:

### R — Rate

Сколько запросов в секунду.

```text
requests/sec
```

### E — Errors

Количество/процент ошибок.

```text
5xx rate
```

### D — Duration

Время обработки запросов.

```text
P50
P95
P99
```

Например, dashboard:

```text
┌─────────────┬─────────────┬─────────────┐
│    Rate     │   Errors    │  Duration   │
│  350 req/s  │    0.3%     │   P95 180ms │
└─────────────┴─────────────┴─────────────┘
```

---

# 15. Grafana для Python Backend

Например, FastAPI-приложение:

```text
FastAPI
   │
   │ /metrics
   ▼
Prometheus
   │
   │ PromQL
   ▼
Grafana
```

Можно мониторить:

* количество запросов;
* HTTP status codes;
* latency;
* количество ошибок;
* CPU;
* RAM;
* количество активных соединений;
* PostgreSQL;
* Redis;
* Kafka.

---

# 16. Grafana vs Prometheus

|                  | Prometheus              | Grafana              |
| ---------------- | ----------------------- | -------------------- |
| Сбор метрик      | ✅                       | ❌                    |
| Хранение metrics | ✅                       | ❌                    |
| PromQL           | ✅                       | ❌                    |
| Графики          | ограниченно             | ✅                    |
| Dashboard        | ❌                       | ✅                    |
| Visualization    | ❌                       | ✅                    |
| Data Source      | сам является источником | подключает источники |
| Alerting         | Alertmanager / rules    | Grafana Alerting     |

Упрощённо:

```text
Prometheus = данные

Grafana = интерфейс для просмотра данных
```

---

# 17. Grafana ≠ мониторинг целиком

Важно на собеседовании не говорить:

> «Grafana — это система мониторинга».

Точнее:

> **Grafana — платформа визуализации и observability, которая получает данные из различных источников и представляет их в виде dashboards, панелей и alerting.**

Сам мониторинг обычно состоит из нескольких компонентов:

```text
Application
    │
    ├── Metrics ──→ Prometheus
    │
    ├── Logs ─────→ Loki
    │
    └── Traces ───→ Tempo
                       │
                       ▼
                    Grafana
```

---

# 18. Grafana и Alertmanager

Для Prometheus часто используется связка:

```text
Prometheus
    │
    │ alert rule
    ▼
Alertmanager
    │
    ├── Telegram
    ├── Email
    ├── Slack
    └── PagerDuty
```

Grafana тоже имеет собственный механизм alerting.

Поэтому в конкретной архитектуре может быть:

```text
Prometheus → Alertmanager
       ↓
    Grafana
```

или alerting может быть организован через Grafana.

---

# 19. Главное различие компонентов Observability

```text
Prometheus
    ↓
Metrics

Loki
    ↓
Logs

Tempo / Jaeger
    ↓
Traces

Grafana
    ↓
Visualization / Dashboards
```

---

# 20. Пример архитектуры Production

```text
                 ┌───────────────┐
                 │   FastAPI     │
                 └───────┬───────┘
                         │
          ┌──────────────┼──────────────┐
          ↓              ↓              ↓
       Metrics          Logs          Traces
          ↓              ↓              ↓
     Prometheus         Loki          Tempo
          │              │              │
          └──────────────┼──────────────┘
                         ↓
                     Grafana
                         │
              ┌──────────┴──────────┐
              ↓                     ↓
          Dashboard               Alert
```

---

# 21. Частые ошибки

### ❌ «Grafana собирает метрики»

Нет.

Обычно сбор выполняет Prometheus или другой Data Source.

### ❌ «Grafana хранит все метрики»

Grafana в типичной связке является визуализационным слоем, а метрики хранятся в Prometheus или другом backend.

### ❌ «Prometheus нужен только для Grafana»

Нет.

Prometheus может использоваться самостоятельно: PromQL, правила, Alertmanager и API.

### ❌ «Grafana работает только с Prometheus»

Нет.

Она поддерживает множество Data Source.

### ❌ «Dashboard = metric»

Нет.

Metric — данные.

Dashboard — набор визуализаций этих данных.

---

# 22. 🎤 Как ответить на собеседовании

> **Grafana — это платформа визуализации и observability. Она подключается к источникам данных, например Prometheus, выполняет запросы и отображает результаты в виде dashboards и panels. В связке с Prometheus обычно получается так: Prometheus собирает и хранит метрики, а Grafana визуализирует их и позволяет создавать dashboards и alert rules.**

---

## 🎯 Главное

```text
Grafana
   ↓
Visualization / Dashboards

Prometheus
   ↓
Metrics / Time Series

Loki
   ↓
Logs

Tempo / Jaeger
   ↓
Traces
```

**Ключевая формула:**

> **Prometheus собирает метрики → Grafana визуализирует → Alerting сообщает о проблемах.**
