# Virtual Host (vhost) в RabbitMQ 🏠

## 🎯 Ответ на собеседовании

**Virtual Host (vhost) — логически изолированное пространство внутри RabbitMQ, в котором находятся Exchanges, Queues, Bindings и другие ресурсы.**

Vhost позволяет разделить один RabbitMQ между несколькими приложениями или окружениями.

Например:

```text id="4c5v8q"
RabbitMQ
│
├── /production
│    ├── Exchanges
│    ├── Queues
│    └── Bindings
│
├── /staging
│    ├── Exchanges
│    └── Queues
│
└── /dev
     ├── Exchanges
     └── Queues
```

Пользователь получает права доступа **в рамках конкретного vhost**.

Главная идея:

> **vhost — это логическая граница изоляции ресурсов и прав доступа внутри RabbitMQ.**

---

## 🎤 Суперкоротко

```text id="jv0qj3"
RabbitMQ
   ↓
vhost
   ↓
Exchange + Queue + Binding
```

Vhost нужен для:

* изоляции приложений;
* разделения `dev / staging / production`;
* разграничения прав;
* использования одного RabbitMQ несколькими командами/сервисами.

---

# Что такое vhost

`vhost` расшифровывается как **Virtual Host**.

Это логическое пространство внутри RabbitMQ.

Например:

```text id="b5zz9v"
RabbitMQ
    │
    ├── /orders
    ├── /payments
    └── /notifications
```

Каждый vhost содержит собственные RabbitMQ-ресурсы.

---

# Что находится внутри vhost

В vhost могут находиться:

```text id="5q3zkl"
Virtual Host
│
├── Exchanges
├── Queues
├── Bindings
└── permissions
```

Например:

```text id="g5d9s4"
/production
│
├── events_exchange
├── orders_queue
├── payments_queue
└── bindings
```

---

# Зачем нужны vhost

Представим один RabbitMQ для трёх окружений:

```text id="czp9e7"
RabbitMQ
│
├── /dev
├── /staging
└── /production
```

Теперь ресурсы разных окружений логически разделены.

Например:

```text id="r7sgv1"
/dev/orders_queue

/staging/orders_queue

/production/orders_queue
```

Даже одинаковое имя Queue может существовать в разных vhost.

---

# Одинаковые имена Queue

Это важный момент.

Можно иметь:

```text id="89zqv7"
/dev/orders_queue
/production/orders_queue
```

Это разные Queue, потому что они находятся в разных vhost.

То же относится к Exchanges.

Например:

```text id="e0o9mt"
/dev/events
/production/events
```

---

# Vhost как namespace

Очень удобно воспринимать vhost как **namespace**.

Например:

```text id="9q8b0p"
vhost = /production
```

внутри него:

```text id="m2q2y8"
orders
payments
notifications
```

А в:

```text id="a2x7z4"
vhost = /dev
```

могут находиться Queue с теми же именами:

```text id="w9d6pi"
orders
payments
notifications
```

Но это разные ресурсы.

---

# Vhost и пользователи

Пользователь RabbitMQ не получает автоматически доступ ко всем vhost.

Права задаются отдельно.

Например:

```text id="v46k7c"
User: orders_service

/production → разрешён
/staging    → запрещён
/dev        → запрещён
```

Другой пользователь:

```text id="d6y1vo"
User: developer

/dev        → разрешён
/staging    → разрешён
/production → запрещён
```

Это позволяет реализовать изоляцию.

---

# Permissions

RabbitMQ использует permissions для контроля доступа к ресурсам vhost.

Основные права:

```text id="3g7k0s"
configure
write
read
```

Упрощённо:

### `configure`

Можно создавать/изменять определённые ресурсы.

### `write`

Можно публиковать сообщения.

### `read`

Можно читать сообщения.

---

# Пример

Есть:

```text id="w9o4h8"
/production
```

И пользователь:

```text id="2y4p2k"
orders_service
```

Ему можно разрешить:

```text id="0gqv3a"
configure → orders_.*
write     → orders_.*
read      → orders_.*
```

Но запретить доступ к:

```text id="r2jv4h"
payments_.*
```

Таким образом сервис получает только необходимые права.

---

# Vhost и Exchange

Exchange принадлежит конкретному vhost.

Например:

```text id="j8r4s1"
/production
    ↓
events_exchange
```

Другой vhost:

```text id="4g0lqs"
/dev
    ↓
events_exchange
```

Это два разных Exchange.

---

# Vhost и Queue

То же самое с Queue.

```text id="qk5m3a"
/production
    ↓
orders_queue
```

и:

```text id="qf6j9k"
/dev
    ↓
orders_queue
```

Это независимые Queue.

---

# Vhost и Binding

Binding также существует в контексте конкретного vhost.

Например:

```text id="x8tq6v"
/production
    Exchange
       ↓ binding
    Queue
```

Нельзя просто взять Binding из одного vhost и использовать его в другом.

---

# Vhost и Connection

Connection RabbitMQ подключается к конкретному vhost.

Упрощённо:

```text id="07k4mz"
Application
    ↓
Connection
    ↓
vhost
    ↓
Exchange / Queue
```

Например, приложение может подключиться к:

```text id="w0p6av"
/production
```

и работать с ресурсами этого vhost.

---

# Connection URL

Типичный URI подключения выглядит примерно так:

```python id="t0m3px"
amqp://user:password@rabbitmq:5672/production
```

Здесь:

```text id="8m0d1r"
user       → пользователь
password   → пароль
rabbitmq   → hostname
5672       → AMQP port
production → vhost
```

На практике vhost может требовать URL-encoding, если его имя содержит специальные символы.

---

# `/` — vhost по умолчанию

В RabbitMQ существует vhost:

```text id="h0x7v8"
/
```

Это default vhost.

Поэтому часто можно увидеть подключения вроде:

```text id="m7j6j4"
amqp://guest:guest@localhost:5672/
```

Но в production часто создают отдельные vhost для изоляции.

---

# Vhost и Docker

В Docker-среде часто бывает:

```text id="4myj38"
Docker Compose
     ↓
RabbitMQ
     ↓
vhost
     ↓
application
```

Например:

```text id="i6n5v4"
orders-service
     ↓
RabbitMQ
     ↓
/production
```

А локальная разработка:

```text id="9u1h6r"
orders-service
     ↓
RabbitMQ
     ↓
/dev
```

---

# Vhost и окружения

Очень распространённый вариант:

```text id="q8q0zi"
RabbitMQ
│
├── /dev
├── /staging
└── /production
```

Преимущество:

```text id="m4l5t0"
dev
   ↓
не взаимодействует напрямую
   ↓
production
```

При правильных permissions.

---

# Vhost ≠ отдельный RabbitMQ

Это важное различие.

Если создать:

```text id="q6c6pn"
/dev
/staging
/production
```

это **не три RabbitMQ-сервера**.

Это один RabbitMQ:

```text id="r8m9b4"
RabbitMQ
 ├── /dev
 ├── /staging
 └── /production
```

---

# Vhost ≠ Docker Container

Также не нужно путать:

```text id="v2f9sl"
Docker Container
```

и:

```text id="yn2n9z"
RabbitMQ vhost
```

Container изолирует процесс/окружение на уровне Docker.

Vhost изолирует RabbitMQ-ресурсы логически.

Можно иметь:

```text id="afq2z3"
1 RabbitMQ Container
        ↓
3 vhost
```

---

# Vhost и безопасность

Vhost помогает реализовать принцип:

> **Least Privilege — минимально необходимые права.**

Например:

```text id="q7n5kc"
payments_service
       ↓
/production
       ↓
только payments_*
```

Сервис не должен иметь полный доступ ко всем очередям RabbitMQ, если ему это не требуется.

---

# Vhost и несколько команд

Представим компанию:

```text id="4m8q0p"
RabbitMQ
│
├── /team-orders
├── /team-payments
└── /team-notifications
```

Каждой команде можно предоставить отдельный namespace и permissions.

Но на практике выбор между vhost, отдельным RabbitMQ-кластером и другими способами изоляции зависит от требований к безопасности, производительности и эксплуатации.

---

# Vhost и отказоустойчивость

Очень важно:

**Vhost не является механизмом репликации.**

Если нужна отказоустойчивость Queue, используются механизмы вроде:

```text id="z0n8x4"
Quorum Queue
```

А vhost отвечает за:

```text id="b9c4d7"
изоляцию ресурсов
```

То есть:

```text id="w4m3pe"
Vhost
→ isolation

Quorum Queue
→ high availability
```

---

# Vhost и Cluster

Vhost существует внутри RabbitMQ deployment.

Например:

```text id="y2q3v8"
RabbitMQ Cluster
│
├── Node 1
├── Node 2
└── Node 3
      ↓
    vhost
      ↓
   Queues
```

Vhost не является отдельным сервером.

---

# Типичный backend-сценарий

Есть три окружения:

```text id="k8o6p4"
/dev
/staging
/production
```

Application config:

```python id="p7n6bc"
RABBITMQ_VHOST = "/production"
```

Подключение:

```python id="c6b2ea"
amqp://app_user:password@rabbitmq:5672/%2Fproduction
```

Приложение работает только внутри нужного vhost.

---

# Важное различие: vhost и namespace приложения

Можно использовать vhost для:

* окружений;
* команд;
* приложений;
* tenant isolation.

Но не всегда стоит создавать отдельный vhost для каждого микросервиса.

Слишком большое количество vhost усложняет эксплуатацию.

Архитектура должна учитывать:

* количество приложений;
* permissions;
* monitoring;
* backup;
* operational complexity;
* требования к изоляции.

---

# 🎯 Частые вопросы на собеседовании

### Что такое vhost?

> Логически изолированное пространство RabbitMQ, содержащее Exchanges, Queues и Bindings и имеющее собственную область прав доступа.

### Зачем нужен vhost?

> Для логической изоляции ресурсов и разграничения доступа между приложениями или окружениями.

### Может ли одна Queue существовать в двух vhost?

Да, если у них одинаковое имя, это всё равно будут разные Queue:

```text
/dev/orders
/production/orders
```

### Vhost — это отдельный RabbitMQ?

Нет. Это логическая область внутри одного RabbitMQ deployment.

### Vhost обеспечивает отказоустойчивость?

Нет. Для отказоустойчивости используются механизмы вроде Quorum Queue и кластеризации.

### Какие основные permissions есть?

```text
configure
write
read
```

### Может ли пользователь иметь доступ только к одному vhost?

Да.

### Можно ли использовать один RabbitMQ для dev и production?

Технически да, используя разные vhost, но для production обычно отдельно оценивают требования к безопасности, ресурсам и отказоустойчивости. В критичных системах часто предпочитают отдельную инфраструктуру.

---

# 🧠 Итоговая схема

```text id="j9m3x2"
                    RabbitMQ
                       │
          ┌────────────┼────────────┐
          ↓            ↓            ↓
        /dev        /staging    /production
          │            │            │
          ↓            ↓            ↓
      Exchanges     Exchanges    Exchanges
      Queues        Queues       Queues
      Bindings      Bindings     Bindings
```

Права:

```text id="7k8p2m"
User
  ↓
Permissions
  ↓
Vhost
  ↓
RabbitMQ resources
```

### Формула для собеседования

> **Vhost в RabbitMQ — это логически изолированное пространство ресурсов. Внутри него находятся Exchanges, Queues и Bindings, а права пользователей задаются в контексте vhost. Vhost позволяет разделять приложения и окружения, но не является механизмом отказоустойчивости или репликации.**
