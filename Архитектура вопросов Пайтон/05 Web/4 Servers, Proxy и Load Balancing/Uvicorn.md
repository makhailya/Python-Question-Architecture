# ⚡ Uvicorn

## 🎯 Ответ на собеседовании

**Uvicorn** — это высокопроизводительный **ASGI-сервер для Python**, который принимает сетевые соединения и передаёт запросы ASGI-приложению, например FastAPI.

Он используется для запуска асинхронных Python-веб-приложений и поддерживает `async/await`, WebSocket и другие возможности ASGI.

Типичная связка:

```text
Клиент
   ↓
Nginx
   ↓
Uvicorn
   ↓
ASGI
   ↓
FastAPI
```

Главное:

> **Uvicorn — это ASGI-сервер, FastAPI — ASGI-приложение, а ASGI — стандарт интерфейса между ними.**

---

## 🎤 Суперкоротко

```text
Uvicorn = ASGI-сервер

Uvicorn
   ↓
ASGI
   ↓
FastAPI
```

Запомнить:

```text
WSGI → Gunicorn
ASGI → Uvicorn
```

---

# 🔌 Зачем нужен Uvicorn

FastAPI само по себе не является сервером, который непосредственно слушает TCP-порт и принимает сетевые соединения.

Uvicorn выполняет эту роль:

```text
Client
  ↓
Uvicorn
  ↓
FastAPI
```

Он:

* принимает соединения;
* обрабатывает HTTP;
* поддерживает WebSocket;
* взаимодействует с ASGI-приложением;
* запускает event loop для асинхронной обработки.

---

# 🧩 Uvicorn и ASGI

Важно различать:

### ASGI

Это **стандарт интерфейса**:

```text
ASGI
↓
правила взаимодействия
↓
Server ↔ Application
```

### Uvicorn

Это **конкретная реализация ASGI-сервера**:

```text
Uvicorn
↓
реализует ASGI server interface
↓
FastAPI
```

Поэтому:

```text
ASGI ≠ Uvicorn
```

Так же как:

```text
WSGI ≠ Gunicorn
```

---

# 🚀 Запуск FastAPI

Допустим, есть `main.py`:

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/")
async def root():
    return {"message": "Hello"}
```

Запуск через Uvicorn:

```python
uvicorn main:app
```

Здесь:

```text
main
 ↓
main.py

app
 ↓
объект FastAPI
```

Можно указать host и port:

```python
uvicorn main:app --host 0.0.0.0 --port 8000
```

---

# 🔄 Как обрабатывается запрос

Клиент отправляет:

```text
GET /users
```

Поток:

```text
Client
   ↓
Uvicorn
   ↓
ASGI
   ↓
FastAPI
   ↓
Endpoint
   ↓
Response
   ↓
Uvicorn
   ↓
Client
```

Uvicorn отвечает за серверную часть, а FastAPI — за обработку логики приложения.

---

# ⚡ Uvicorn и `async/await`

Uvicorn предназначен для ASGI и хорошо подходит для асинхронных приложений:

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/users")
async def get_users():
    users = await get_users_from_database()
    return users
```

Во время `await` event loop может переключаться на другие задачи.

```text
Event Loop
    │
    ├── Request 1 → await DB
    │
    ├── Request 2 → await API
    │
    └── Request 3 → обработка
```

Важно:

> **Uvicorn не делает блокирующий код асинхронным автоматически.**

Например:

```python
import time


@app.get("/")
async def root():
    time.sleep(5)
    return {"status": "ok"}
```

`time.sleep()` блокирует поток выполнения.

---

# 🔌 Uvicorn и WebSocket

Uvicorn поддерживает WebSocket через ASGI.

Схема:

```text
Client
  ↕
WebSocket
  ↕
Uvicorn
  ↕
ASGI
  ↕
FastAPI
```

Например:

```python
from fastapi import FastAPI, WebSocket

app = FastAPI()


@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()

    while True:
        message = await websocket.receive_text()
        await websocket.send_text(message)
```

---

# 🌐 Uvicorn + Nginx

В production часто используется:

```text
Internet
    ↓
  Nginx
    ↓
 Uvicorn
    ↓
 FastAPI
    ↓
PostgreSQL
```

### Nginx

Может выполнять:

* reverse proxy;
* TLS termination;
* раздачу статических файлов;
* работу с внешними соединениями.

### Uvicorn

Отвечает за:

* ASGI;
* HTTP;
* WebSocket;
* запуск Python ASGI-приложения.

### FastAPI

Отвечает за:

* routing;
* validation;
* business logic;
* dependencies;
* формирование API-ответов.

---

# 🆚 Uvicorn и Gunicorn

Это частый вопрос на собеседовании.

| Uvicorn                | Gunicorn                         |
| ---------------------- | -------------------------------- |
| ASGI-сервер            | Классически WSGI-сервер          |
| FastAPI / Starlette    | Django / Flask                   |
| Async-first            | Классическая sync WSGI-модель    |
| Поддерживает WebSocket | Классический WSGI напрямую — нет |
| ASGI                   | WSGI                             |

Упрощённо:

```text
Django
   ↓
Gunicorn
   ↓
WSGI
```

```text
FastAPI
   ↓
Uvicorn
   ↓
ASGI
```

Но Gunicorn также может использовать ASGI worker, например Uvicorn worker:

```text
Nginx
  ↓
Gunicorn
  ↓
Uvicorn Worker
  ↓
FastAPI
```

Поэтому правильнее говорить не просто «Gunicorn только для Django», а:

> **Gunicorn традиционно используется как WSGI-сервер, но может работать с ASGI-приложениями через соответствующий worker.**

---

# 📦 Uvicorn и Workers

Uvicorn может запускать несколько worker-процессов:

```python
uvicorn main:app --workers 4
```

Схематично:

```text
Uvicorn
 ├── Worker 1
 ├── Worker 2
 ├── Worker 3
 └── Worker 4
```

Это позволяет использовать несколько процессов приложения.

Важно отличать:

```text
asyncio
↓
конкурентность внутри event loop


workers
↓
несколько процессов
```

---

# 🧠 Uvicorn и Event Loop

Uvicorn использует асинхронную модель Python.

Упрощённо:

```text
Uvicorn
   ↓
Event Loop
   ↓
ASGI Application
   ↓
FastAPI
```

Event loop управляет выполнением асинхронных задач.

Когда корутина выполняет:

```python
await some_io_operation()
```

она может уступить управление event loop, пока ожидается I/O.

---

# 🆚 `uvicorn` и `uvicorn[standard]`

Можно установить базовый пакет:

```python
pip install uvicorn
```

или дополнительные зависимости:

```python
pip install "uvicorn[standard]"
```

Второй вариант устанавливает дополнительные компоненты, которые могут улучшать производительность и удобство работы.

Для учебных проектов часто достаточно обычного:

```python
uvicorn main:app
```

---

# ⚠️ Uvicorn — не FastAPI

Это три разных уровня:

```text
FastAPI
↓
Web Framework


ASGI
↓
Interface / Standard


Uvicorn
↓
ASGI Server
```

Их нельзя считать одним и тем же.

Например:

```text
Uvicorn
   ↓
может запускать
   ↓
FastAPI
```

---

# 🎯 Главное

```text
Client
   ↓
Nginx
   ↓
Uvicorn
   ↓
ASGI
   ↓
FastAPI
```

Запомнить:

```text
ASGI   → стандарт интерфейса
Uvicorn → ASGI-сервер
FastAPI → ASGI-приложение
```

```text
WSGI
 ↓
Gunicorn

ASGI
 ↓
Uvicorn
```

### Формула для собеседования

> **Uvicorn — это ASGI-сервер для Python, который принимает HTTP/WebSocket-соединения и передаёт их ASGI-приложению, например FastAPI. Он поддерживает асинхронную модель `async/await` и используется для запуска FastAPI в production вместе с reverse proxy или непосредственно.**

```text
Uvicorn
= ASGI Server
+ HTTP
+ WebSocket
+ async/await
+ Event Loop
```
