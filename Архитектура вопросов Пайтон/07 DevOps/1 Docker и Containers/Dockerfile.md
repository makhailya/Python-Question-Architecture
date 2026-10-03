# 📄 Dockerfile

## 🎤 Короткий ответ

**Dockerfile** — это текстовый файл с инструкциями, по которым Docker **собирает Docker Image**.

В нём описываются базовый образ, рабочая директория, зависимости, файлы приложения, переменные окружения и команда запуска.

## 🎯 Формула для собеседования

> **Dockerfile → `docker build` → Docker Image → `docker run` → Container.**

---

# 🔹 Что такое Dockerfile

Dockerfile — обычный текстовый файл с именем:

```text
Dockerfile
```

Например:

```python id="q7m2kd"
FROM python:3.13-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["python", "main.py"]
```

Docker читает инструкции сверху вниз и формирует из них **Image**.

---

# 🔹 Основные инструкции

| Инструкция   | Назначение                                         |
| ------------ | -------------------------------------------------- |
| `FROM`       | базовый Image                                      |
| `WORKDIR`    | рабочая директория                                 |
| `COPY`       | копирование файлов                                 |
| `ADD`        | копирование файлов с дополнительными возможностями |
| `RUN`        | выполнить команду при сборке                       |
| `CMD`        | команда по умолчанию при запуске                   |
| `ENTRYPOINT` | основной исполняемый процесс                       |
| `ENV`        | переменная окружения                               |
| `ARG`        | аргумент только для этапа сборки                   |
| `EXPOSE`     | документирует порт приложения                      |
| `USER`       | пользователь, от которого работает процесс         |
| `VOLUME`     | объявляет точку монтирования                       |
| `LABEL`      | метаданные Image                                   |

---

# 🔹 `FROM`

Определяет базовый Image.

```python id="m4n9qp"
FROM python:3.13-slim
```

То есть:

```text
python:3.13-slim
       ↓
базовый слой
       ↓
на него добавляется наше приложение
```

`FROM` практически всегда является первой основной инструкцией Dockerfile.

Можно использовать:

```python id="f5a2n8"
FROM python:3.13-slim
```

или:

```python id="k3x7vb"
FROM ubuntu:24.04
```

---

# 🔹 `WORKDIR`

Устанавливает рабочую директорию:

```python id="r8d1mc"
WORKDIR /app
```

После этого:

```python id="v2n6qa"
COPY . .
```

означает копирование файлов в:

```text
/app
```

А:

```python id="z6c4wp"
RUN python main.py
```

будет выполняться из `/app`.

**Преимущество:** не нужно постоянно писать `cd /app`.

---

# 🔹 `COPY`

Копирует файлы из build context в Image.

```python id="e5x9rt"
COPY requirements.txt .
COPY . .
```

Например:

```text
Проект:
├── Dockerfile
├── requirements.txt
└── app/
    └── main.py
```

После:

```python id="j2m7vk"
WORKDIR /app
COPY . .
```

получим:

```text
/app
├── Dockerfile
├── requirements.txt
└── app
    └── main.py
```

На практике обычно используют `.dockerignore`, чтобы не отправлять в build context ненужные файлы.

---

# 🔹 `.dockerignore`

Работает аналогично `.gitignore`.

Например:

```text
.git
.venv
__pycache__
.pytest_cache
.env
.idea
*.pyc
```

Это позволяет не включать ненужные файлы в build context.

Особенно важно не отправлять секреты вроде `.env`, если они не должны попадать в контекст сборки.

---

# 🔹 `RUN`

Выполняет команду **во время сборки Image**.

Например:

```python id="c8p4ys"
RUN pip install --no-cache-dir -r requirements.txt
```

Или:

```python id="w6t3bn"
RUN apt-get update && apt-get install -y curl
```

Главное:

> **`RUN` выполняется при `docker build`, а не при каждом `docker run`.**

Результат команды становится частью Image layer.

---

# 🔹 `CMD`

Определяет команду **по умолчанию при запуске контейнера**.

```python id="a9q5hf"
CMD ["python", "main.py"]
```

При:

```python id="x4r8mp"
docker run myapp
```

Docker запустит:

```text
python main.py
```

Но `CMD` можно переопределить:

```python id="s3d7kc"
docker run myapp python other.py
```

---

# 🔹 `ENTRYPOINT`

Определяет основной исполняемый процесс контейнера.

```python id="b6w2qn"
ENTRYPOINT ["python"]
```

Вместе с:

```python id="n8k4rz"
CMD ["main.py"]
```

получаем:

```text
python main.py
```

Упрощённо:

```text
ENTRYPOINT → основной executable
CMD        → аргументы/значения по умолчанию
```

---

# 🔹 `CMD` vs `ENTRYPOINT`

| `CMD`                             | `ENTRYPOINT`                      |
| --------------------------------- | --------------------------------- |
| Команда/аргументы по умолчанию    | Основной процесс                  |
| Легко переопределяется            | Обычно сохраняется                |
| Можно использовать самостоятельно | Часто используется вместе с `CMD` |

Пример:

```python id="t5q9wl"
ENTRYPOINT ["python"]
CMD ["main.py"]
```

Запуск:

```python id="v8c3mx"
docker run myapp
```

→ `python main.py`

А:

```python id="h7n2pd"
docker run myapp other.py
```

→ `python other.py`

---

# 🔹 `ENV`

Задаёт переменную окружения внутри контейнера:

```python id="d3f7ka"
ENV APP_ENV=production
```

В приложении:

```python id="m8v2qx"
import os

environment = os.getenv("APP_ENV")
```

Можно задать несколько:

```python id="c6r9bt"
ENV PYTHONUNBUFFERED=1
ENV APP_ENV=production
```

⚠️ **Секреты не следует хранить через `ENV` в Dockerfile**, потому что они могут попасть в Image и историю его сборки.

---

# 🔹 `ARG`

`ARG` — переменная, доступная **во время сборки**.

```python id="w4k6ps"
ARG PYTHON_VERSION=3.13

FROM python:${PYTHON_VERSION}-slim
```

Передать значение:

```python id="z9h3vn"
docker build --build-arg PYTHON_VERSION=3.12 .
```

Главное отличие:

```text
ARG → build time
ENV → runtime
```

---

# 🔹 `EXPOSE`

Документирует порт, который использует приложение:

```python id="r5m7xc"
EXPOSE 8000
```

Важно:

> `EXPOSE` **не публикует порт** на хосте.

Для публикации нужен `docker run -p`:

```python id="f2k8qd"
docker run -p 8000:8000 myapp
```

---

# 🔹 `USER`

Позволяет указать пользователя, от которого будет работать приложение:

```python id="v7c4nx"
USER appuser
```

Запуск приложения не от `root` обычно является более безопасной практикой.

Например:

```python id="j8p2mz"
RUN useradd -m appuser
USER appuser

CMD ["python", "main.py"]
```

---

# 🔹 `LABEL`

Добавляет метаданные к Image:

```python id="q3f6kb"
LABEL maintainer="backend-team"
LABEL version="1.0"
```

Это может использоваться для документации и автоматизации.

---

# 🔹 `VOLUME`

Объявляет директорию как точку для хранения данных:

```python id="n4w8ry"
VOLUME /data
```

На практике способ хранения данных обычно задаётся при запуске контейнера или через Compose.

Например:

```python id="e6m3pt"
docker run -v mydata:/data myapp
```

---

# 🔹 Build Context

При:

```python id="x2v7hc"
docker build -t myapp .
```

`.` — это **build context**.

Docker получает доступ к файлам внутри этого контекста, например:

```text
.
├── Dockerfile
├── requirements.txt
├── app/
└── .dockerignore
```

Поэтому:

```python id="k5r9qd"
COPY . .
```

копирует содержимое build context.

---

# 🔹 Dockerfile и слои

Большинство инструкций Dockerfile создают слои или влияют на конфигурацию Image.

Например:

```python id="u3n8fs"
FROM python:3.13-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["python", "main.py"]
```

Условно:

```text
┌─────────────────────┐
│ Application code    │
├─────────────────────┤
│ Python dependencies │
├─────────────────────┤
│ Python base image   │
└─────────────────────┘
```

Docker может использовать cache уже созданных слоёв.

---

# 🔹 Почему порядок инструкций важен

Плохой вариант:

```python id="b8m4rx"
COPY . .

RUN pip install -r requirements.txt
```

Если изменится любой файл проекта, Docker может потерять cache для последующего `RUN`.

Лучше:

```python id="n2q7vw"
COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .
```

Теперь изменение только Python-кода не требует заново устанавливать зависимости, если соответствующий слой может быть взят из cache.

---

# 🔹 Multi-stage Build

Можно использовать несколько стадий сборки:

```python id="p6c9yd"
FROM python:3.13 AS builder

WORKDIR /app

COPY requirements.txt .
RUN pip install --prefix=/install -r requirements.txt


FROM python:3.13-slim

WORKDIR /app

COPY --from=builder /install /usr/local
COPY . .

CMD ["python", "main.py"]
```

Идея:

```text
Builder
   ↓
сборка / установка
   ↓
Runtime Image
   ↓
только необходимые файлы
```

Преимущества:

* меньший итоговый Image;
* меньше лишних инструментов;
* лучше разделены build и runtime окружения.

---

# 🔹 Пример Dockerfile для FastAPI

Для Python Backend:

```python id="r4k8sz"
FROM python:3.13-slim

WORKDIR /app

ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8000

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

Схема:

```text
Dockerfile
    ↓
docker build
    ↓
FastAPI Image
    ↓
docker run
    ↓
Container
    ↓
Uvicorn
    ↓
FastAPI
```

---

# 🔹 Dockerfile vs Docker Compose

Это разные вещи.

| Dockerfile                      | Docker Compose                                |
| ------------------------------- | --------------------------------------------- |
| Описывает **как собрать Image** | Описывает **как запустить набор сервисов**    |
| `FROM`, `COPY`, `RUN`, `CMD`    | `services`, `ports`, `volumes`, `environment` |
| Обычно один Image               | Может управлять несколькими контейнерами      |
| `docker build`                  | `docker compose up`                           |

Например:

```text
Dockerfile
    ↓
app Image
```

А Compose:

```text
docker-compose.yml
       ↓
 ┌─────┼─────────┐
 ↓     ↓         ↓
App  PostgreSQL  Redis
```

---

# 🔹 Частые вопросы на собеседовании

### Что такое Dockerfile?

> Текстовый файл с инструкциями для сборки Docker Image.

### Чем `RUN` отличается от `CMD`?

> `RUN` выполняется при сборке Image, а `CMD` задаёт команду по умолчанию при запуске контейнера.

### Чем `CMD` отличается от `ENTRYPOINT`?

> `ENTRYPOINT` задаёт основной исполняемый процесс, а `CMD` — команду или аргументы по умолчанию.

### Что делает `FROM`?

> Выбирает базовый Docker Image.

### Что делает `COPY`?

> Копирует файлы из build context в Image.

### Что делает `EXPOSE`?

> Документирует порт контейнера, но сам порт наружу не публикует.

### `ARG` vs `ENV`?

> `ARG` используется во время сборки, `ENV` задаёт переменные окружения для Image/контейнера.

### Зачем `.dockerignore`?

> Чтобы исключить ненужные файлы из build context и не отправлять их в Docker при сборке.

### Почему Dockerfile стараются оптимизировать по слоям?

> Чтобы эффективно использовать build cache и уменьшить время повторных сборок.

---

## 🔑 Главное

```text
Dockerfile
    │
    ├── FROM       → базовый Image
    ├── WORKDIR    → рабочая директория
    ├── COPY       → файлы
    ├── RUN        → команды при сборке
    ├── ENV        → переменные окружения
    ├── ARG        → аргументы сборки
    ├── EXPOSE     → документация порта
    ├── USER       → пользователь
    ├── ENTRYPOINT → основной процесс
    └── CMD        → команда/аргументы по умолчанию
            ↓
       docker build
            ↓
        Docker Image
            ↓
        docker run
            ↓
        Container
```

### 🎤 Финальная формула

> **Dockerfile описывает процесс сборки Docker Image. `FROM` задаёт базовый образ, `COPY` переносит файлы, `RUN` выполняет команды при сборке, `ENV` задаёт окружение, `ENTRYPOINT` и `CMD` определяют запуск контейнера.**
