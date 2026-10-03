# 🐳 Docker

## 🎤 Короткий ответ

**Docker** — это платформа для запуска приложений в изолированных **контейнерах**. Контейнер содержит приложение, его зависимости и настройки окружения, благодаря чему приложение работает предсказуемо на разных машинах.

Docker использует **образы (images)** как шаблоны для создания контейнеров.

## 🎯 Формула для собеседования

> **Docker = Image → Container → изолированное окружение для приложения.**

---

# 🔹 Зачем нужен Docker

Без Docker приложение может зависеть от окружения:

```text
Python 3.13
PostgreSQL 15
Redis
зависимости
переменные окружения
```

На компьютере разработчика всё работает, а на сервере:

```text
Python другой версии
другая библиотека
нет Redis
другая конфигурация
→ приложение не запускается
```

Docker позволяет описать окружение приложения и запускать его одинаково:

```text
Docker
  ↓
Container
  ├── Application
  ├── Dependencies
  ├── Runtime
  └── Configuration
```

---

# 🔹 Container

**Container (контейнер)** — запущенный экземпляр Docker Image.

Например:

```text
Image: python:3.13
        ↓
   Container
        ↓
   Python application
```

Контейнер:

* изолирован от других контейнеров;
* имеет собственную файловую систему;
* имеет собственное сетевое окружение;
* использует ядро хостовой ОС;
* может быть остановлен, удалён и создан заново.

Контейнер — это **не полноценная виртуальная машина**.

---

# 🔹 Image

**Docker Image** — неизменяемый шаблон, из которого создаются контейнеры.

Например:

```text
python:3.13
postgres:15
redis:7
nginx:latest
```

Один Image может использоваться для создания множества контейнеров:

```text
             Docker Image
                  │
        ┌─────────┼─────────┐
        ↓         ↓         ↓
   Container  Container  Container
```

Image состоит из слоёв.

При изменении Dockerfile Docker старается переиспользовать уже собранные слои через cache.

---

# 🔹 Dockerfile

**Dockerfile** — инструкция, по которой Docker собирает Image.

Пример Python-приложения:

```python
FROM python:3.13-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["python", "main.py"]
```

Основные инструкции:

| Инструкция   | Назначение                     |
| ------------ | ------------------------------ |
| `FROM`       | базовый Image                  |
| `WORKDIR`    | рабочая директория             |
| `COPY`       | копирование файлов             |
| `RUN`        | команда при сборке Image       |
| `CMD`        | команда запуска контейнера     |
| `ENTRYPOINT` | основной исполняемый процесс   |
| `ENV`        | переменная окружения           |
| `EXPOSE`     | декларация используемого порта |

---

# 🔹 Build

Чтобы получить Image из Dockerfile:

```python
docker build -t myapp:1.0 .
```

Здесь:

```text
docker build
    ↓
Dockerfile
    ↓
Docker Image
```

`-t` задаёт имя и тег Image.

---

# 🔹 Run

Запуск контейнера:

```python
docker run -d -p 8000:8000 myapp:1.0
```

Например:

```text
-p 8000:8000
```

означает:

```text
Host : Container
8000 : 8000
```

То есть запрос:

```text
localhost:8000
```

на хосте перенаправляется на порт `8000` внутри контейнера.

---

# 🔹 Port Mapping

Контейнеры имеют собственное сетевое пространство.

Например:

```text
Host
localhost:8000
      │
      ↓
Docker
      │
      ↓
Container:8000
      │
      ↓
FastAPI
```

Важно:

**`EXPOSE` не публикует порт наружу.**

```python
EXPOSE 8000
```

только документирует, какой порт приложение использует.

Публикация выполняется через:

```python
docker run -p 8000:8000 myapp
```

---

# 🔹 Volume

Контейнер обычно считается **эфемерным**: его можно удалить и создать заново.

Если данные должны переживать удаление контейнера, используют **Volume**.

```text
Container
    │
    ↓
 Volume
    │
    ↓
Persistent data
```

Например PostgreSQL:

```text
PostgreSQL Container
        │
        ↓
   postgres_data
        │
        ↓
   Database files
```

Пример:

```python
docker run -v postgres_data:/var/lib/postgresql/data postgres:15
```

Volume позволяет сохранить данные независимо от жизненного цикла контейнера.

---

# 🔹 Bind Mount

Можно смонтировать конкретную директорию хоста:

```python
docker run -v ./src:/app/src myapp
```

Получается:

```text
Host ./src
     │
     ↓
Container /app/src
```

### Volume vs Bind Mount

| Volume                     | Bind Mount                |
| -------------------------- | ------------------------- |
| Управляется Docker         | Управляется пользователем |
| Обычно для persistent data | Часто для разработки      |
| Docker сам хранит данные   | Указывается путь хоста    |

---

# 🔹 Environment Variables

Конфигурацию приложения часто передают через переменные окружения:

```python
docker run \
  -e DATABASE_URL=postgresql://user:password@db/app \
  myapp
```

В приложении:

```python
import os

database_url = os.getenv("DATABASE_URL")
```

Секреты не рекомендуется зашивать в Dockerfile или Image.

---

# 🔹 Docker Network

Контейнеры могут находиться в одной Docker-сети и обращаться друг к другу по имени.

Например:

```text
        app
         │
         ↓
      postgres
         │
         ↓
       redis
```

Создаём сеть:

```python
docker network create backend
```

Подключаем контейнеры:

```python
docker run -d --network backend --name db postgres:15
docker run -d --network backend --name app myapp
```

Приложение может обращаться к PostgreSQL:

```text
postgresql://...@db:5432/...
```

Здесь `db` — имя контейнера/сетевой DNS-алиас, а не `localhost`.

---

# 🔹 Docker Compose

Когда приложение состоит из нескольких сервисов, вручную запускать контейнеры неудобно.

Например:

```text
FastAPI
PostgreSQL
Redis
Nginx
```

Для этого используют **Docker Compose**.

Пример:

```python
services:
  app:
    build: .
    ports:
      - "8000:8000"
    depends_on:
      - db
      - redis

  db:
    image: postgres:15
    environment:
      POSTGRES_DB: app
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password

  redis:
    image: redis:7
```

Запуск:

```python
docker compose up -d
```

Остановка:

```python
docker compose down
```

Логи:

```python
docker compose logs -f
```

Список контейнеров:

```python
docker compose ps
```

---

# 🔹 Docker Compose и сеть

Compose автоматически создаёт сеть для проекта.

Поэтому сервисы могут обращаться друг к другу по имени:

```text
app → db
app → redis
```

Например:

```text
DATABASE_HOST=db
REDIS_HOST=redis
```

Не:

```text
DATABASE_HOST=localhost
```

Потому что внутри контейнера `localhost` указывает **на этот же контейнер**.

---

# 🔹 Docker Registry

Image можно хранить в Registry.

Например:

```text
Developer
    ↓
docker build
    ↓
Image
    ↓
docker push
    ↓
Docker Registry
    ↓
docker pull
    ↓
Server
```

Популярный вариант — Docker Hub.

Registry нужен для хранения и распространения Image.

---

# 🔹 Container Registry ≠ Container

Важно различать:

```text
Image
  ↓
Registry
  ↓
docker pull
  ↓
Container
```

**Registry** хранит Images.

**Container** запускается из Image.

---

# 🔹 Docker vs Virtual Machine

| Docker Container           | Virtual Machine      |
| -------------------------- | -------------------- |
| Использует ядро хоста      | Своя гостевая ОС     |
| Обычно легче               | Обычно тяжелее       |
| Быстро запускается         | Запускается дольше   |
| Меньше потребляет ресурсов | Больше ресурсов      |
| Изоляция на уровне ОС      | Полная виртуализация |
| Удобен для микросервисов   | Нужна полноценная ОС |

Упрощённо:

```text
Virtual Machine

Hardware
   ↓
Hypervisor
   ↓
Guest OS
   ↓
Application


Docker

Hardware
   ↓
Host OS / Kernel
   ↓
Docker
   ↓
Container
   ↓
Application
```

---

# 🔹 Docker ≠ Virtual Machine

Это частый вопрос на собеседовании.

Docker-контейнер **не является виртуальной машиной**.

Контейнеры используют ядро операционной системы хоста, но изолируют процессы, файловую систему, сеть и ресурсы.

---

# 🔹 Docker Container Lifecycle

Типичный жизненный цикл:

```text
Dockerfile
    ↓
docker build
    ↓
Image
    ↓
docker run
    ↓
Container
    ↓
Running
    ↓
Stop
    ↓
Stopped
    ↓
Start / Remove
```

Основные команды:

```python
docker ps
docker ps -a
docker images
docker build
docker run
docker stop
docker start
docker restart
docker rm
docker rmi
docker logs
docker exec
```

---

# 🔹 `docker exec`

Позволяет выполнить команду внутри уже работающего контейнера:

```python
docker exec -it app bash
```

Например:

```python
docker exec -it db psql -U user -d app
```

Это удобно для диагностики.

---

# 🔹 `CMD` vs `ENTRYPOINT`

Оба определяют запуск контейнера, но используются немного по-разному.

### CMD

Задаёт команду/аргументы по умолчанию:

```python
CMD ["python", "main.py"]
```

Её можно переопределить при `docker run`.

### ENTRYPOINT

Определяет основной исполняемый процесс контейнера:

```python
ENTRYPOINT ["python"]
```

А CMD может задавать аргументы:

```python
CMD ["main.py"]
```

Получается:

```text
python main.py
```

Упрощённо:

> **ENTRYPOINT — основной executable, CMD — значения по умолчанию.**

---

# 🔹 Docker Image Layers

Dockerfile:

```python
FROM python:3.13-slim

COPY requirements.txt .

RUN pip install -r requirements.txt

COPY . .
```

Создаёт несколько слоёв.

Условно:

```text
┌─────────────────┐
│ Application     │
├─────────────────┤
│ Dependencies    │
├─────────────────┤
│ Python          │
├─────────────────┤
│ Base Image      │
└─────────────────┘
```

Docker использует cache, поэтому порядок инструкций в Dockerfile влияет на скорость сборки.

Например, зависимости лучше копировать отдельно:

```python
COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .
```

Если изменился только код приложения, слой установки зависимостей можно переиспользовать.

---

# 🔹 Docker и безопасность

Контейнер — это изоляция, но **не абсолютная граница безопасности**.

Практические принципы:

* не запускать приложение от `root`, если это не требуется;
* использовать минимальные base images;
* не хранить секреты в Image;
* ограничивать capabilities;
* обновлять образы;
* не публиковать ненужные порты;
* сканировать зависимости и образы.

---

# 🔹 Docker в Python Backend

Типичная архитектура:

```text
                 Nginx
                   │
                   ↓
              FastAPI
                   │
        ┌──────────┼──────────┐
        ↓          ↓          ↓
    PostgreSQL    Redis      Celery
```

Каждый компонент может быть отдельным контейнером:

```text
┌─────────────────────────────────────┐
│ Docker Compose                      │
│                                     │
│  ┌───────┐   ┌────────────┐         │
│  │ Nginx │ → │ FastAPI    │         │
│  └───────┘   └─────┬──────┘         │
│                     │                │
│              ┌──────┴──────┐         │
│              ↓             ↓         │
│         PostgreSQL       Redis       │
│                            ↑         │
│                         Celery       │
└─────────────────────────────────────┘
```

Это особенно удобно для локальной разработки и CI/CD.

---

# 🔹 Docker и CI/CD

Docker хорошо сочетается с CI/CD:

```text
Git Push
   ↓
CI
   ↓
Tests
   ↓
docker build
   ↓
Image
   ↓
Registry
   ↓
Deploy
   ↓
Server
```

Преимущество — в production можно запускать **тот же Image**, который был протестирован в CI.

---

# 🔹 Частые вопросы на собеседовании

### Что такое Docker?

> Платформа контейнеризации, позволяющая запускать приложения в изолированных окружениях.

### Чем Image отличается от Container?

> Image — шаблон, Container — запущенный экземпляр этого шаблона.

### Docker — это виртуальная машина?

> Нет. Контейнеры используют ядро хостовой ОС, тогда как VM запускает отдельную гостевую ОС.

### Зачем Dockerfile?

> Для декларативного описания процесса сборки Docker Image.

### Зачем Docker Compose?

> Для определения и запуска нескольких связанных контейнеров как единого приложения.

### Зачем Volume?

> Для хранения данных независимо от жизненного цикла контейнера.

### Что делает `-p 8000:8000`?

> Публикует порт контейнера `8000` на порт `8000` хоста.

### Что такое Registry?

> Хранилище Docker Images, откуда их можно `push` и `pull`.

### Почему внутри контейнера `localhost` — это не хост?

> Потому что `localhost` внутри контейнера указывает на сам контейнер.

### Зачем Docker Network?

> Для сетевого взаимодействия контейнеров, в том числе обращения к сервисам по их именам.

---

## 🔑 Главное

```text
Docker
  │
  ├── Image       → шаблон
  │
  ├── Container   → запущенный Image
  │
  ├── Dockerfile  → инструкция сборки Image
  │
  ├── Volume      → постоянные данные
  │
  ├── Network     → связь контейнеров
  │
  ├── Registry    → хранение Images
  │
  └── Compose     → управление несколькими сервисами
```

### 🎤 Финальная формула

> **Docker упаковывает приложение и его окружение в Image, из которого запускается изолированный Container. Dockerfile описывает сборку, Volume сохраняет данные, Network связывает контейнеры, Registry хранит Images, а Compose позволяет управлять несколькими сервисами.**
