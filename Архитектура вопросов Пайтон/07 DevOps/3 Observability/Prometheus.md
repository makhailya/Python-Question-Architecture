# 📊 Prometheus

## 🎤 Короткий ответ

**Prometheus** — система мониторинга и сбора **метрик**. Она периодически забирает числовые показатели с приложений и инфраструктуры, хранит их как временные ряды и позволяет анализировать их с помощью языка запросов **PromQL**.

Типичная связка:

```python id="m4x8qz"
Application
    ↓
/metrics
    ↓
Prometheus
    ↓
PromQL
    ↓
Grafana
```

---

## 🎯 Формула для собеседования

> **Prometheus = собирает + хранит + позволяет запрашивать метрики → Grafana визуализирует их.**

```python id="q7n3vx"
Exporter / Application
        ↓
      Metrics
        ↓
   Prometheus
        ↓
      PromQL
        ↓
     Grafana
```

---

# 🧠 Что такое Prometheus

**Prometheus** — система мониторинга и временных рядов (**time series database**), созданная для сбора метрик.

Он используется, чтобы понимать состояние системы:

* сколько запросов приходит;
* сколько ошибок;
* какая latency;
* сколько CPU/RAM используется;
* сколько активных соединений;
* какой consumer lag;
* сколько сообщений обработано.

---

# 📈 Что такое метрика

Метрика — числовое значение, которое описывает состояние или поведение системы.

Например:

```text id="e6q2mw"
http_requests_total = 152340
```

Или:

```text id="w8k4pn"
process_cpu_usage = 0.72
```

Метрика обычно представляет собой **time series**:

```text id="s5m7qx"
metric + labels + timestamp + value
```

---

# 🏷️ Labels

Prometheus использует labels для разделения одной метрики на разные временные ряды.

Например:

```python id="r3v9km"
http_requests_total{
    method="GET",
    endpoint="/users",
    status="200"
}
```

Это позволяет отдельно анализировать:

```text id="n8q2px"
GET /users 200
GET /users 500
POST /users 201
```

---

# ⚠️ Cardinality

Количество уникальных комбинаций labels называется **cardinality**.

Опасный пример:

```python id="k5m8qz"
http_requests_total{
    user_id="123456"
}
```

Если пользователей миллионы, получится огромное количество временных рядов.

Поэтому в labels осторожно используют:

* `user_id`;
* `request_id`;
* UUID;
* произвольные URL;
* другие значения с очень большим количеством вариантов.

Лучше:

```python id="v2x7mp"
method
endpoint
status
```

при разумной нормализации значений.

---

# 🔢 Основные типы метрик

В Prometheus наиболее важны:

* Counter;
* Gauge;
* Histogram;
* Summary.

---

# 1. Counter

**Counter** — счётчик, который обычно только увеличивается.

Например:

```text id="f8m3qx"
http_requests_total
```

```text id="p7k2nz"
100
101
102
103
...
```

Используется для:

* количества запросов;
* количества ошибок;
* количества обработанных сообщений;
* количества событий.

Counter может сброситься, например, после рестарта процесса.

---

# 2. Gauge

**Gauge** — текущее значение, которое может увеличиваться и уменьшаться.

Например:

```text id="z4m8qp"
active_connections = 42
```

Потом:

```text id="s6x2kv"
active_connections = 37
```

Используется для:

* CPU;
* RAM;
* температуры;
* количества активных соединений;
* размера очереди;
* количества работающих задач.

---

# 🆚 Counter vs Gauge

| Counter                     | Gauge                           |
| --------------------------- | ------------------------------- |
| Счётчик                     | Текущее значение                |
| Обычно только растёт        | Растёт и уменьшается            |
| Requests total              | Active connections              |
| Errors total                | Memory usage                    |
| `rate()` часто используется | Обычно смотрят текущее значение |

Пример:

```text id="w3q8mx"
requests_total → Counter

active_users → Gauge
```

---

# 3. Histogram

**Histogram** показывает распределение значений.

Особенно полезен для **latency**.

Например:

```text id="k9m4pz"
HTTP request duration:

0–0.1 sec → 500 requests
0.1–0.5 sec → 1200 requests
0.5–1 sec → 300 requests
1–5 sec → 20 requests
```

Можно анализировать:

* P50;
* P90;
* P95;
* P99.

Например:

```text id="d7x2mq"
P95 latency = 450 ms
```

То есть примерно 95% наблюдений не превышают этот порог при соответствующей интерпретации histogram/quantile.

---

# 4. Summary

**Summary** также предназначен для анализа распределения значений и может рассчитывать quantiles.

Например:

```text id="p4m7xz"
P50
P90
P99
```

Но у Summary есть ограничения при агрегации quantiles между несколькими экземплярами приложения.

Поэтому для многих distributed-сценариев **Histogram часто удобнее**.

---

# 🌐 Как Prometheus получает метрики

Классическая модель Prometheus — **pull**.

Prometheus сам обращается к endpoint:

```text id="y6q3mw"
Prometheus
     │
     │ GET /metrics
     ↓
Application
```

Например:

```text id="m8x4qp"
/metrics
```

Приложение отдаёт:

```text id="c2k7vz"
http_requests_total 152340
```

Prometheus забирает эти данные и сохраняет их.

---

# 🔄 Pull vs Push

### Prometheus

```text id="n5q8mx"
Prometheus
    ↓
 GET /metrics
    ↓
Application
```

То есть **pull**.

### Push

При push-модели приложение само отправляет метрики:

```text id="x3m7kp"
Application
    ↓
 Metrics Server
```

Prometheus в стандартной модели предпочитает pull, хотя для отдельных сценариев существует **Pushgateway**.

---

# 📤 Exporter

**Exporter** — компонент, который предоставляет метрики в формате, понятном Prometheus.

Например:

```text id="v7q2mz"
PostgreSQL
    ↓
postgres_exporter
    ↓
/metrics
    ↓
Prometheus
```

Для Linux-сервера:

```text id="m4x8qn"
Node
 ↓
node_exporter
 ↓
/metrics
 ↓
Prometheus
```

Exporter нужен, когда сама система не предоставляет Prometheus-метрики напрямую.

---

# 🐍 Prometheus в Python

Python-приложение может использовать библиотеку:

```python id="h2k6qp"
prometheus-client
```

Например:

```python id="r8m3vz"
from prometheus_client import Counter

requests_total = Counter(
    "http_requests_total",
    "Total HTTP requests",
)

requests_total.inc()
```

Полученные метрики можно предоставить через `/metrics`.

---

# 🚀 Prometheus + FastAPI

Упрощённая архитектура:

```text id="c7m4px"
FastAPI
  │
  ├── /users
  ├── /orders
  └── /metrics
          ↑
          │
      Prometheus
```

Prometheus периодически делает:

```text id="y8q2mw"
GET /metrics
```

и получает метрики приложения.

---

# 🔍 PromQL

**PromQL (Prometheus Query Language)** — язык запросов Prometheus.

Например:

```python id="u5m9kx"
http_requests_total
```

Получить скорость роста Counter:

```python id="q3x7mp"
rate(http_requests_total[5m])
```

То есть:

> средняя скорость увеличения счётчика за последние 5 минут.

---

# 📊 Пример запроса ошибок

Допустим:

```text id="a8k2vz"
http_requests_total{
    status="500"
}
```

Можно получить скорость ошибок:

```python id="m6q4xp"
rate(http_requests_total{status="500"}[5m])
```

---

# 📈 Агрегация

PromQL позволяет группировать данные.

Например:

```python id="v9k3mq"
sum by (status) (
    rate(http_requests_total[5m])
)
```

Можно получить:

```text id="w4p8nz"
200 → 150 req/s
404 → 3 req/s
500 → 0.4 req/s
```

---

# ⏱️ Monitoring latency

Для Histogram можно получить, например, P95:

```python id="k7m2qx"
histogram_quantile(
    0.95,
    sum by (le) (
        rate(http_request_duration_seconds_bucket[5m])
    )
)
```

Это типичный пример использования Histogram + PromQL.

---

# 🚨 Alerting

Prometheus может использоваться вместе с **Alertmanager**.

Схема:

```text id="f3q8mx"
Prometheus
    ↓
Alert Rule
    ↓
Alertmanager
    ↓
Notification
```

Например:

```text id="n6m2pz"
HTTP 5xx > threshold
        ↓
      Alert
        ↓
   Alertmanager
        ↓
 Telegram / Email / Slack
```

Важно:

> **Prometheus собирает и хранит метрики, а Alertmanager занимается маршрутизацией и доставкой уведомлений.**

---

# 📊 Prometheus + Grafana

Очень распространённая связка:

```text id="s8k3mq"
Application
    ↓
Prometheus
    ↓
PromQL
    ↓
Grafana
```

### Prometheus

Собирает и хранит метрики.

### Grafana

Строит:

* dashboards;
* графики;
* таблицы;
* визуализации.

То есть:

> **Prometheus — данные, Grafana — визуализация.**

---

# ☸️ Prometheus в Kubernetes

В Kubernetes Prometheus может собирать метрики:

* Pods;
* Nodes;
* Kubernetes API;
* applications;
* Services;
* ingress;
* databases;
* exporters.

Упрощённо:

```text id="c6x9mp"
Kubernetes Cluster
│
├── Node
│   └── node metrics
│
├── Pod
│   └── application metrics
│
└── Exporters
        ↓
    Prometheus
        ↓
      Grafana
```

---

# 📦 Service Discovery

В Kubernetes не нужно вручную прописывать каждый Pod.

Prometheus может использовать **service discovery**, чтобы обнаруживать targets.

Например:

```text id="h5m2qx"
Kubernetes API
      ↓
Service Discovery
      ↓
Prometheus
      ↓
Targets
```

Это особенно важно в динамической среде, где Pods постоянно создаются и удаляются.

---

# 🎯 RED для Backend

Для HTTP-сервисов полезен подход **RED**:

```text id="m8q4vp"
R — Rate
E — Errors
D — Duration
```

То есть:

### Rate

Сколько запросов:

```text id="r7x3mk"
requests/sec
```

### Errors

Сколько ошибок:

```text id="p4m8qz"
5xx/sec
```

### Duration

Сколько занимает запрос:

```text id="v2k6nx"
P95 latency
P99 latency
```

---

# 🖥️ USE для инфраструктуры

Для ресурсов системы:

```text id="z5q8mp"
U — Utilization
S — Saturation
E — Errors
```

Например:

```text id="k3m7vx"
CPU utilization
Memory saturation
Disk errors
```

---

# 🆚 Prometheus vs Logs

| Prometheus                | Logs                            |
| ------------------------- | ------------------------------- |
| Числовые показатели       | Текстовые события               |
| Метрики                   | Детали событий                  |
| "500 errors/sec"          | "Database connection failed..." |
| Хорош для графиков/alerts | Хорош для расследования         |
| PromQL                    | Поиск/фильтрация логов          |

Они дополняют друг друга.

---

# 🆚 Prometheus vs Tracing

### Metrics

Отвечают:

> **Что происходит с системой в целом?**

```text id="g7m2qx"
P95 = 800 ms
```

### Tracing

Отвечает:

> **Где именно внутри запроса возникла задержка?**

```text id="c4n8vp"
Request
 ↓
API
 ↓ 50ms
Redis
 ↓ 700ms
PostgreSQL
```

---

# ⚠️ Высокая Cardinality

Одна из важных проблем Prometheus:

```python id="r6m2xz"
request_id="8f92..."
```

Если каждый запрос имеет уникальный `request_id`, количество series может стать огромным.

Поэтому:

```text id="q3v7mp"
❌ user_id
❌ request_id
❌ UUID

✅ method
✅ status
✅ normalized endpoint
```

Labels должны иметь контролируемое количество комбинаций.

---

# 💾 Prometheus Storage

Prometheus хранит метрики как временные ряды.

Например:

```text id="w8m3qx"
http_requests_total
 ├── method="GET"
 ├── endpoint="/users"
 └── status="200"
```

с различными значениями во времени:

```text id="n5q7vz"
10:00 → 1000
10:01 → 1050
10:02 → 1120
10:03 → 1200
```

Это позволяет анализировать изменение состояния системы во времени.

---

# 🔥 Что важно понимать про Prometheus

Prometheus особенно хорошо подходит для:

* infrastructure monitoring;
* Kubernetes;
* HTTP metrics;
* alerting;
* time-series data;
* service monitoring.

Но он не предназначен для хранения:

```text id="f8m2qx"
полных логов
```

или:

```text id="k6q4mz"
полных distributed traces
```

Для этого используются специализированные системы.

---

# 🏗️ Типичная архитектура Observability

```text id="x7m3qp"
                 Application
                /     |      \
               /      |       \
          Metrics    Logs    Traces
             ↓        ↓        ↓
        Prometheus   Loki   Tempo/Jaeger
             ↓
          Grafana
```

Получается:

```text id="m4q8vx"
Metrics → Prometheus
Logs   → Loki / Elasticsearch
Traces → Tempo / Jaeger
                 ↓
              Grafana
```

---

# 🎤 Как рассказать на собеседовании

> **Prometheus — система мониторинга и хранения временных рядов, предназначенная для сбора метрик. Обычно Prometheus работает по pull-модели и периодически забирает данные с `/metrics` у приложения или exporter'а. Метрики могут иметь labels и представлены типами Counter, Gauge, Histogram и Summary. Для запросов используется PromQL, а для визуализации часто используется Grafana. В Kubernetes Prometheus может использовать service discovery для динамического обнаружения Pod'ов и других targets.**

---

# ❓ Частые вопросы

### Что такое Prometheus?

Система мониторинга и хранения time-series метрик.

### Как Prometheus получает метрики?

В стандартной модели **pull** — сам обращается к `/metrics`.

### Что такое Exporter?

Компонент, который преобразует/предоставляет метрики системы в формате Prometheus.

### Что такое PromQL?

Язык запросов Prometheus.

### Prometheus и Grafana — одно и то же?

Нет.

```text id="n7m3qx"
Prometheus → собирает/хранит/запрашивает
Grafana    → визуализирует
```

### Counter vs Gauge?

**Counter** — счётчик событий, обычно растёт.

**Gauge** — текущее значение, может расти и уменьшаться.

### Зачем Histogram?

Для анализа распределения значений, например latency и расчёта перцентилей.

### Что такое Labels?

Дополнительные измерения метрики:

```python id="x2k8mp"
method="GET"
status="200"
```

### Что такое Cardinality?

Количество уникальных временных рядов/комбинаций label'ов. Слишком высокая cardinality может сильно увеличить нагрузку и объём хранения.

### Что такое Alertmanager?

Компонент экосистемы Prometheus, который обрабатывает alerts и маршрутизирует уведомления.

---

## 🔑 Главное

```python id="p8m4qx"
Application
    ↓
/metrics
    ↓
Prometheus
    ↓
PromQL
    ↓
Grafana
```

Запомнить четыре типа:

```python id="c6x2mz"
Counter   → количество событий
Gauge     → текущее значение
Histogram → распределение
Summary   → quantiles / распределение
```

И главную архитектурную идею:

> **Prometheus отвечает за метрики, Grafana — за визуализацию, Alertmanager — за доставку алертов.**
