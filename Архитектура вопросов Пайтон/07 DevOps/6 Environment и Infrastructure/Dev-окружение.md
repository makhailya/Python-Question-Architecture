# 🛠️ Dev-окружение

## 🎤 Короткий ответ

**Dev-окружение (Development Environment)** — это среда, в которой разработчик **пишет, запускает и отлаживает код** до передачи изменений на тестирование и production.

Обычно включает:

* исходный код;
* нужную версию Python;
* зависимости проекта;
* базу данных;
* Redis и другие сервисы;
* переменные окружения;
* Docker/Compose;
* инструменты разработки и тестирования.

## 🎯 Формула для собеседования

> **Dev-окружение = код + зависимости + конфигурация + необходимые сервисы → локальная разработка и отладка.**

Типичный вариант для Python Backend:

```text id="d3v8m2"
Developer
    ↓
Git branch
    ↓
Python + dependencies
    ↓
Docker Compose
    ├── FastAPI
    ├── PostgreSQL
    └── Redis
```

---

# 🔹 Зачем нужно Dev-окружение

Разработчику нужно запускать проект **в условиях, максимально приближенных к реальным**, но без воздействия на production.

Например:

```text id="e7m4qx"
Developer
   ↓
Dev
   ↓
Tests
   ↓
Staging
   ↓
Production
```

В Dev можно:

* изменять код;
* экспериментировать;
* ломать и восстанавливать окружение;
* запускать debugger;
* тестировать API;
* работать с тестовой БД.

---

# 🔹 Из чего состоит Dev-окружение

Например Python Backend:

```text id="p6q3mx"
Dev Environment
│
├── Source Code
├── Python
├── Dependencies
├── PostgreSQL
├── Redis
├── Environment Variables
├── Docker
├── Tests
└── IDE
```

---

# 🔹 Python и виртуальное окружение

Для Python важно изолировать зависимости разных проектов.

Например:

```text id="k8m4vx"
Project A
 └── Django 5

Project B
 └── Django 4
```

Если устанавливать всё глобально, версии могут конфликтовать.

Поэтому используют:

```text id="x5q7mc"
venv
Poetry
uv
```

Например стандартный `venv`:

```python id="v7m3qx"
python -m venv .venv
```

А затем активируют окружение и устанавливают зависимости.

---

# 🔹 Зависимости проекта

Проект должен явно описывать свои зависимости.

Например через:

```text id="j4q8mx"
pyproject.toml
```

или другие файлы управления зависимостями.

В твоём случае, например, можно использовать **Poetry**:

```text id="m8q3vx"
pyproject.toml
poetry.lock
```

`pyproject.toml` описывает зависимости и настройки проекта.

`poetry.lock` фиксирует конкретные версии зависимостей.

Это позволяет разработчикам получать максимально одинаковое окружение.

---

# 🔹 Environment Variables

Конфигурацию обычно не хардкодят в коде.

Например:

```text id="a6m4qx"
DATABASE_HOST
DATABASE_PORT
DATABASE_NAME
DATABASE_USER
DATABASE_PASSWORD
REDIS_URL
SECRET_KEY
```

Приложение получает их из окружения.

Например:

```python id="r8q2mc"
import os

database_url = os.getenv("DATABASE_URL")
```

---

# 🔹 `.env`

Для локальной разработки часто используют `.env`:

```python id="f3m7vx"
DATABASE_URL=postgresql://user:password@localhost:5432/app
REDIS_URL=redis://localhost:6379
DEBUG=true
```

Но `.env` с настоящими секретами обычно **не коммитят в Git**.

Например:

```text id="z5q8mx"
.env
```

добавляют в:

```text id="b7m3qx"
.gitignore
```

В репозитории можно хранить:

```text id="p4q6vx"
.env.example
```

с примером необходимых переменных без настоящих секретов.

---

# 🔹 Dev Database

В Dev обычно используется отдельная база:

```text id="c8m4qx"
Development
    ↓
PostgreSQL Dev
```

а не production database.

Например:

```text id="v3q7mc"
dev_db
staging_db
production_db
```

Это принципиально важно:

> Разработка не должна случайно изменять production-данные.

---

# 🔹 Docker в Dev-окружении

Docker позволяет стандартизировать окружение.

Например:

```text id="u6m8qx"
docker compose up
       ↓
┌─────────────────────┐
│ FastAPI             │
│ PostgreSQL          │
│ Redis               │
└─────────────────────┘
```

Вместо ручной установки PostgreSQL и Redis каждый разработчик запускает одинаковый набор контейнеров.

---

# 🔹 Docker Compose

Например:

```python id="n4q8vx"
services:
  app:
    build: .
    ports:
      - "8000:8000"

  db:
    image: postgres:15

  redis:
    image: redis:7
```

Запуск:

```python id="s7m3qx"
docker compose up
```

Теперь Dev-окружение поднимается одной командой.

---

# 🔹 Почему одинаковое окружение важно

Проблема:

```text id="e8q4mc"
"У меня работает"
        ↓
"У тебя другая версия Python"
        ↓
"У тебя другой PostgreSQL"
        ↓
"У тебя другая версия библиотеки"
```

Чем сильнее различаются окружения, тем больше неожиданных проблем.

Поэтому используют:

* фиксированные версии;
* lock-файлы;
* Docker;
* `.env.example`;
* автоматическую установку зависимостей.

---

# 🔹 Dev vs Staging vs Production

| Окружение      | Назначение                |
| -------------- | ------------------------- |
| **Dev**        | разработка                |
| **Staging**    | проверка перед production |
| **Production** | реальная эксплуатация     |

### Dev

```text id="d5m8qx"
Разработчик
   ↓
Код
   ↓
Dev
```

Можно активно экспериментировать.

### Staging

```text id="s4q7mx"
Feature
   ↓
CI
   ↓
Staging
   ↓
Проверка
```

Обычно стараются сделать окружение максимально похожим на production.

### Production

```text id="p8m3vx"
Users
  ↓
Production
```

Здесь работают реальные пользователи и данные.

---

# 🔹 Dev ≠ Production

Dev может отличаться от Production:

```text id="q6m4vx"
Dev:
1 FastAPI container
1 PostgreSQL
1 Redis

Production:
10 FastAPI Pods
3 PostgreSQL nodes
Redis cluster
Load Balancer
Monitoring
```

Но при этом желательно, чтобы:

* версии ПО были совместимы;
* конфигурация была предсказуемой;
* основные зависимости имели одинаковое поведение;
* процесс запуска был максимально похож.

---

# 🔹 Local Development

Dev-окружение может быть полностью локальным:

```text id="x8q3mv"
MacBook
 │
 ├── Python
 ├── IDE
 ├── Docker
 │    ├── FastAPI
 │    ├── PostgreSQL
 │    └── Redis
 └── Git
```

Это называется **local development environment**.

---

# 🔹 Remote Development

Dev-окружение может находиться на удалённом сервере:

```text id="r5m8qx"
Developer
    ↓
Remote Dev Environment
    ↓
Docker / VM / Kubernetes
```

Это может быть удобно, если проект требует много ресурсов или сложной инфраструктуры.

---

# 🔹 Debugging

Dev-окружение должно позволять легко отлаживать приложение.

Например в Python IDE можно использовать:

```text id="b7m4qx"
Breakpoint
   ↓
Start Debugger
   ↓
Inspect variables
   ↓
Step over / Step into
```

Это одно из основных отличий Dev от Production.

---

# 🔹 Hot Reload

Во время разработки удобно автоматически перезапускать приложение после изменения кода.

Например FastAPI/Uvicorn:

```python id="w3m7qx"
uvicorn app:app --reload
```

Изменили:

```text id="e4m8vx"
app.py
```

и сервер автоматически перезапустился.

Для production `--reload` обычно не используют.

---

# 🔹 Dev Configuration

В Dev могут использоваться отдельные настройки:

```text id="m6q2vx"
DEBUG=true
LOG_LEVEL=DEBUG
DATABASE_URL=dev_db
```

В Production:

```text id="q8m4xc"
DEBUG=false
LOG_LEVEL=INFO
DATABASE_URL=production_db
```

Конфигурация должна приходить извне приложения, а не быть жёстко зашитой в код.

---

# 🔹 Dev Secrets

Секреты Dev тоже должны быть защищены.

Не стоит делать:

```python id="j5m8qx"
SECRET_KEY = "my-secret"
```

в исходном коде.

Используются:

```text id="r7q3mv"
.env
Secret Manager
CI/CD Secrets
Kubernetes Secrets
```

При этом `.env` с секретами не должен попадать в публичный Git-репозиторий.

---

# 🔹 Dev-окружение и Git

Обычно разработчик работает в отдельной ветке:

```text id="x4m8vq"
main
 │
 └── feature/orders
          ↓
       Developer
          ↓
       Dev env
          ↓
       Tests
          ↓
         MR
```

То есть изменения сначала проверяются в Dev, затем отправляются на Code Review.

---

# 🔹 Dev-окружение и CI

Dev — это среда разработки.

CI — автоматическая проверка изменений.

```text id="c7m4qx"
Developer
    ↓
Dev
    ↓
Git push
    ↓
CI
    ├── Tests
    ├── Lint
    ├── Type check
    └── Build
```

CI не является Dev-окружением, хотя использует собственную среду выполнения для проверок.

---

# 🔹 Dev-окружение и контейнеризация

Без Docker:

```text id="n8q3vx"
Developer A
Python 3.12
PostgreSQL 15
Redis 7

Developer B
Python 3.13
PostgreSQL 16
Redis 7
```

Возможны различия.

С Docker:

```text id="k5m7qx"
compose.yaml
      ↓
 одинаковые версии
      ↓
Developer A
Developer B
```

Docker помогает сделать окружение более воспроизводимым.

---

# 🔹 Reproducible Environment

**Воспроизводимое окружение** — окружение, которое можно предсказуемо создать заново с теми же версиями и настройками.

Например:

```text id="p4m8vx"
git clone
     ↓
install dependencies
     ↓
docker compose up
     ↓
Dev Environment
```

Другой разработчик может получить практически такое же окружение.

Это особенно важно для командной разработки.

---

# 🔹 Типичная структура Python-проекта

Например:

```text id="f8m3qx"
project/
├── app/
├── tests/
├── Dockerfile
├── compose.yaml
├── pyproject.toml
├── poetry.lock
├── .env.example
├── .gitignore
└── README.md
```

README должен объяснять, например:

```text id="v6q4mx"
1. Clone repository
2. Install dependencies
3. Configure .env
4. Start Docker Compose
5. Run tests
```

---

# 🔹 Частые вопросы на собеседовании

### Что такое Dev-окружение?

> Среда, в которой разработчик создаёт, запускает, тестирует и отлаживает приложение.

### Из чего состоит Dev-окружение?

> Из исходного кода, runtime, зависимостей, конфигурации и необходимых внешних сервисов, например PostgreSQL и Redis.

### Зачем Docker в Dev?

> Чтобы стандартизировать и воспроизводимо запускать окружение с нужными версиями сервисов и зависимостей.

### Чем Dev отличается от Production?

> Dev предназначен для разработки и отладки, Production — для работы с реальными пользователями и данными.

### Что такое `.env`?

> Файл с переменными окружения, который часто используют для локальной конфигурации и Dev-секретов; реальные секреты обычно не коммитят в Git.

### Зачем `.env.example`?

> Чтобы показать необходимые переменные окружения без хранения настоящих секретов.

### Зачем `poetry.lock`?

> Для фиксации конкретных версий зависимостей и воспроизводимости окружения.

### Зачем отдельная Dev-база?

> Чтобы разработка и тестирование не затрагивали production-данные.

### Что такое reproducible environment?

> Окружение, которое можно предсказуемо создать заново с одинаковыми версиями и конфигурацией.

### Что такое hot reload?

> Автоматический перезапуск приложения после изменения исходного кода, используемый преимущественно в Dev.

---

# 🔑 Главное

```text id="y5m8qx"
                  Dev Environment
                        │
       ┌────────────────┼────────────────┐
       ↓                ↓                ↓
     Code          Dependencies      Configuration
       │                │                │
       └────────────────┼────────────────┘
                        ↓
                 Runtime / Services
                        │
              ┌─────────┼─────────┐
              ↓         ↓         ↓
           FastAPI   PostgreSQL  Redis
              │
              ↓
          Tests / Debug
              │
              ↓
             Git
              │
              ↓
          Merge Request
```

> **Dev** → место, где разрабатываем и отлаживаем.
> **Staging** → проверяем перед production.
> **Production** → обслуживаем реальных пользователей.
> **Docker** → помогает стандартизировать окружение.
> **`.env`** → конфигурация окружения.
> **`pyproject.toml` / lock-файл** → зависимости проекта.
> **Reproducibility** → возможность получить одинаковое окружение снова.
