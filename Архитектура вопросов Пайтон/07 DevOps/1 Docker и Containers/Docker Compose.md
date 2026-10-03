# 🐳 Docker Compose

## 🎤 Короткий ответ

**Docker Compose** — это инструмент для определения и запуска **нескольких связанных Docker-контейнеров** как единого приложения.

Конфигурация обычно описывается в файле `compose.yaml` или `docker-compose.yml`: какие нужны сервисы, из каких Image их запускать или как собирать, какие порты, volumes, networks и переменные окружения использовать.

## 🎯 Формула для собеседования

> **Docker Compose = описание нескольких сервисов → одна конфигурация → `docker compose up` → всё приложение запускается вместе.**

Например:

```text id="3x8kq1"
Docker Compose
      │
      ├── FastAPI
      ├── PostgreSQL
      ├── Redis
      └── Celery
```

---

# 🔹 Зачем нужен Docker Compose

Представим Python Backend:

```text id="7m2qpd"
FastAPI
   │
   ├── PostgreSQL
   ├── Redis
   └── Celery
```

Без Compose пришлось бы отдельно запускать контейнеры:

```python
docker run ...
docker run ...
docker run ...
docker run ...
```

И вручную настраивать:

* сети;
* порты;
* volumes;
* environment variables;
* зависимости между сервисами.

Compose позволяет описать всё в одном файле:

```python id="q6r9wv"
services:
  app:
    ...
  db:
    ...
  redis:
    ...
  worker:
    ...
```

И запустить:

```python id="n4x7kc"
docker compose up -d
```

---

# 🔹 `compose.yaml`

Простейший пример:

```python id="m8f2qa"
services:
  app:
    image: myapp:latest

  db:
    image: postgres:15

  redis:
    image: redis:7
```

Здесь определены три сервиса:

```text id="h3k7pd"
app
db
redis
```

Compose создаёт и запускает соответствующие контейнеры.

---

# 🔹 Service

**Service** — это описание контейнера в Compose.

Например:

```python id="x5n8vr"
services:
  db:
    image: postgres:15
```

`db` — имя сервиса.

`postgres:15` — Image.

Другой сервис может обратиться к нему по имени:

```text id="1p7dqa"
db:5432
```

То есть внутри Compose-сети:

```text id="s6m2kx"
FastAPI → db:5432 → PostgreSQL
```

Не нужно искать IP-адрес контейнера.

---

# 🔹 `image`

Указывает готовый Docker Image:

```python id="a4q8nc"
services:
  redis:
    image: redis:7
```

Compose создаст контейнер из:

```text id="2v6fpk"
redis:7
```

---

# 🔹 `build`

Вместо готового Image можно указать, что Image нужно собрать из Dockerfile:

```python id="k9m3wx"
services:
  app:
    build: .
```

Compose найдёт Dockerfile в указанной директории и выполнит сборку.

Можно указать конкретный Dockerfile:

```python id="v7c2mz"
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
```

Схема:

```text id="j5r8nd"
Dockerfile
     ↓
  build
     ↓
  Image
     ↓
 Container
```

---

# 🔹 `ports`

Публикует порт контейнера на хост:

```python id="q2f6vb"
services:
  app:
    ports:
      - "8000:8000"
```

Получаем:

```text id="6m8qsr"
Host
localhost:8000
      ↓
Container:8000
```

Формат:

```text
HOST:CONTAINER
```

---

# 🔹 `environment`

Передаёт переменные окружения:

```python id="w4k9pc"
services:
  db:
    image: postgres:15
    environment:
      POSTGRES_DB: app
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
```

Приложение может получить настройки базы через environment.

На практике секреты лучше не хранить непосредственно в Compose-файле в открытом виде.

Можно использовать `.env`:

```python id="b8n5rx"
services:
  db:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
```

---

# 🔹 `env_file`

Можно передать контейнеру переменные из отдельного файла:

```python id="f3m7qa"
services:
  app:
    env_file:
      - .env
```

Например `.env`:

```text id="j6v2kd"
DATABASE_URL=postgresql://user:password@db:5432/app
REDIS_URL=redis://redis:6379
```

⚠️ `.env` с секретами обычно добавляют в `.gitignore`.

---

# 🔹 `volumes`

Используются для постоянного хранения данных:

```python id="n9q4xm"
services:
  db:
    image: postgres:15
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

Схема:

```text id="r5k8wc"
PostgreSQL Container
        │
        ↓
 postgres_data
        │
        ↓
 Persistent storage
```

Если контейнер удалить, данные Volume сохраняются.

---

# 🔹 Bind Mount

Compose также может монтировать директорию проекта:

```python id="z7p3hs"
services:
  app:
    volumes:
      - ./app:/app
```

Получаем:

```text id="2q9mxa"
Host ./app
     ↓
Container /app
```

Это особенно удобно при разработке.

---

# 🔹 `networks`

Compose может создавать отдельные Docker-сети.

Например:

```python id="c5v8ny"
services:
  app:
    networks:
      - backend

  db:
    networks:
      - backend

networks:
  backend:
```

Теперь:

```text id="q8m2kr"
app ───────→ db
      backend
```

Контейнеры в одной сети могут обращаться друг к другу по имени сервиса.

---

# 🔹 Почему `localhost` часто является ошибкой

Например, FastAPI находится в контейнере `app`, PostgreSQL — в контейнере `db`.

Неправильно:

```text id="x4p7qa"
DATABASE_HOST=localhost
```

Потому что внутри контейнера:

```text id="7n3mcs"
localhost
   ↓
сам контейнер app
```

Правильно:

```text id="m6v8rz"
DATABASE_HOST=db
```

Потому что:

```text id="1k5qwp"
app → db → PostgreSQL
```

---

# 🔹 `depends_on`

Позволяет указать зависимость сервисов:

```python id="u9r4mx"
services:
  app:
    depends_on:
      - db
      - redis

  db:
    image: postgres:15

  redis:
    image: redis:7
```

Compose будет учитывать порядок запуска.

Но важный нюанс:

> **`depends_on` не означает, что PostgreSQL уже готов принимать подключения.**

Контейнер PostgreSQL может уже запуститься, но сама БД ещё инициализируется.

Для этого используют `healthcheck` и соответствующую логику зависимости/ожидания готовности.

---

# 🔹 `healthcheck`

Позволяет определить, считается ли контейнер здоровым.

Например:

```python id="e8k3vp"
services:
  db:
    image: postgres:15
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user -d app"]
      interval: 5s
      timeout: 5s
      retries: 5
```

Теперь Docker может проверять:

```text id="v6q9wm"
Container
    ↓
healthcheck
    ↓
healthy / unhealthy
```

---

# 🔹 `restart`

Можно определить политику перезапуска:

```python id="k3m7rx"
services:
  app:
    restart: unless-stopped
```

Распространённые варианты:

```text id="s8q2nd"
no
always
on-failure
unless-stopped
```

Например:

```python id="a5v9kc"
restart: unless-stopped
```

означает, что контейнер будет автоматически перезапускаться после остановки/сбоя, кроме случая, когда пользователь явно остановил его.

---

# 🔹 Полный пример Python Backend

```python id="t7m3qx"
services:
  app:
    build: .
    ports:
      - "8000:8000"
    environment:
      DATABASE_URL: postgresql://user:password@db:5432/app
      REDIS_URL: redis://redis:6379
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_started

  db:
    image: postgres:15
    environment:
      POSTGRES_DB: app
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user -d app"]
      interval: 5s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7

volumes:
  postgres_data:
```

Архитектура:

```text id="n6c4wp"
                  Docker Compose
                       │
        ┌──────────────┼──────────────┐
        ↓              ↓              ↓
      FastAPI       PostgreSQL       Redis
        │              │
        │              ↓
        │         postgres_data
        │
        └────────→ db:5432
        └────────→ redis:6379
```

---

# 🔹 Основные команды

### Запустить

```python id="f8m4kv"
docker compose up
```

В фоне:

```python id="w2q7nc"
docker compose up -d
```

---

### Собрать Images и запустить

```python id="p5x8mr"
docker compose up --build
```

---

### Остановить

```python id="k9v3qs"
docker compose stop
```

Контейнеры останавливаются, но не удаляются.

---

### Остановить и удалить контейнеры

```python id="r4m7wx"
docker compose down
```

---

### Удалить ещё и Volume

```python id="c8q2nd"
docker compose down -v
```

⚠️ Это может удалить данные PostgreSQL из Compose Volume.

---

### Посмотреть контейнеры

```python id="j6v9ka"
docker compose ps
```

---

### Посмотреть логи

```python id="m3r8qx"
docker compose logs
```

Конкретного сервиса:

```python id="v7k2pc"
docker compose logs app
```

В реальном времени:

```python id="a4n9wf"
docker compose logs -f app
```

---

### Выполнить команду внутри контейнера

```python id="x8q5mr"
docker compose exec app bash
```

Например:

```python id="p2m7kc"
docker compose exec db psql -U user -d app
```

---

# 🔹 `docker compose` vs `docker-compose`

Старый вариант:

```python id="h5r8zn"
docker-compose up
```

Современный вариант:

```python id="q3m6vx"
docker compose up
```

Сейчас Compose является частью современного Docker CLI и обычно используется команда:

```text id="6c8r4p"
docker compose
```

---

# 🔹 Docker Compose и Dockerfile

Их часто используют вместе:

```text id="u4q9mx"
             Dockerfile
                 ↓
              Image
                 ↓
        ┌────────┴────────┐
        ↓                 ↓
     Compose            Registry
        ↓
    Container
```

**Dockerfile отвечает:**

> Как собрать Image?

**Compose отвечает:**

> Как запустить приложение из нескольких сервисов?

Например:

```text id="m7x2qa"
Dockerfile
   ↓
FastAPI Image
   ↓
docker compose
   ↓
FastAPI + PostgreSQL + Redis
```

---

# 🔹 Docker Compose и Kubernetes

Это тоже важно для собеседования.

**Docker Compose** обычно используют для:

* локальной разработки;
* тестовых окружений;
* небольших deployment-сценариев;
* запуска нескольких связанных контейнеров.

**Kubernetes** предназначен для более сложной оркестрации контейнеров:

* кластер;
* автоматическое масштабирование;
* service discovery;
* rolling updates;
* self-healing;
* распределённый deployment.

Упрощённо:

```text id="y5k8qp"
Docker
  ↓
отдельные контейнеры

Docker Compose
  ↓
несколько связанных контейнеров

Kubernetes
  ↓
оркестрация контейнеров в кластере
```

---

# 🔹 Частые вопросы на собеседовании

### Что такое Docker Compose?

> Инструмент для определения и запуска многоконтейнерных приложений из единой конфигурации.

### Что такое `service`?

> Описание контейнера/сервиса в Compose-файле.

### Чем `image` отличается от `build`?

> `image` использует готовый Image, а `build` указывает, как собрать Image из Dockerfile.

### Что делает `ports`?

> Публикует порт контейнера на порт хоста.

### Что делает `volumes`?

> Подключает постоянное хранилище или директории к контейнеру.

### Зачем `depends_on`?

> Задаёт зависимости и порядок запуска сервисов, но сам по себе не гарантирует готовность приложения.

### Почему внутри Compose нельзя обращаться к PostgreSQL через `localhost`?

> Потому что `localhost` внутри контейнера указывает на этот же контейнер. Нужно обращаться к сервису по имени, например `db:5432`.

### Что делает `docker compose down`?

> Останавливает и удаляет контейнеры и связанные с ними ресурсы проекта; named volumes по умолчанию не удаляются.

### Что делает `docker compose down -v`?

> Дополнительно удаляет Compose-managed volumes, поэтому данные БД могут быть потеряны.

### Нужен ли Dockerfile для Compose?

> Нет. Compose может запускать готовые Images через `image`. Dockerfile нужен, если Image нужно собирать через `build`.

---

## 🔑 Главное

```text id="r8m3vc"
Docker Compose
      │
      ├── services      → какие сервисы запускаем
      ├── image         → готовый Image
      ├── build         → собрать Image
      ├── ports         → публикация портов
      ├── environment   → переменные окружения
      ├── env_file      → переменные из файла
      ├── volumes       → постоянные данные
      ├── networks      → связь контейнеров
      ├── depends_on    → зависимости запуска
      ├── healthcheck   → проверка состояния
      └── restart       → политика перезапуска
               ↓
       docker compose up
               ↓
       несколько контейнеров
               ↓
        единое приложение
```

### 🎤 Финальная формула

> **Dockerfile описывает, как собрать Image, а Docker Compose — как запустить и связать несколько сервисов. В Compose через `services` описываются контейнеры, `ports` — публикация портов, `volumes` — постоянные данные, `networks` — взаимодействие, `environment` — конфигурация, а `depends_on` и `healthcheck` помогают управлять запуском сервисов.**
