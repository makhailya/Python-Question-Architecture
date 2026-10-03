# ⚡ FastAPI

## 🎯 Ответ на собеседовании

**FastAPI** — современный Python-фреймворк для создания веб-приложений и особенно **REST API**.

Он построен на **Starlette** для web-части и использует **Pydantic** для валидации и сериализации данных.

FastAPI работает поверх **ASGI**, поэтому поддерживает асинхронные обработчики и `async/await`.

Основные возможности:

* маршрутизация;
* обработка HTTP-запросов;
* валидация данных через Pydantic;
* Dependency Injection через `Depends`; [[Dependency Injection]]
* автоматическая генерация OpenAPI-схемы;
* Swagger UI и ReDoc;
* поддержка WebSocket;
* синхронные и асинхронные endpoint'ы.

Типичная схема:

```text
Клиент
   ↓
Uvicorn
   ↓
ASGI
   ↓
FastAPI
   ├── Routing
   ├── Depends
   ├── Pydantic
   └── OpenAPI
        ↓
     Database / Services
```

---

## 🎤 Суперкоротко

```text
FastAPI
↓
Python Web Framework
↓
ASGI
↓
async/await
↓
Pydantic + Depends + OpenAPI
```

Главное:

> **FastAPI — ASGI-фреймворк для создания API, использующий Pydantic для валидации данных и `Depends` для Dependency Injection.**

---

# 🏗️ На чём построен FastAPI

FastAPI использует несколько ключевых компонентов:

```text
FastAPI
   │
   ├── Starlette
   │      ↓
   │   Web / ASGI
   │
   └── Pydantic
          ↓
       Validation
       Serialization
```

### Starlette

Отвечает за web-функциональность:

* HTTP;
* routing;
* middleware;
* WebSocket;
* ASGI.

### Pydantic

Отвечает за:

* валидацию данных;
* преобразование данных;
* схемы запросов и ответов.

---

# 🔌 Простейший FastAPI-пример

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/")
def root():
    return {"message": "Hello, World!"}
```

Запрос:

```text
GET /
```

Ответ:

```python
{
    "message": "Hello, World!"
}
```

---

# 🔗 Routing

Маршрут связывает HTTP-метод и URL с Python-функцией.

```python
@app.get("/users")
def get_users():
    return {"users": []}
```

Здесь:

```text
GET /users
     ↓
get_users()
```

FastAPI поддерживает основные HTTP-методы:

```python
@app.get("/users")
def get_users():
    ...


@app.post("/users")
def create_user():
    ...


@app.put("/users/{user_id}")
def update_user(user_id: int):
    ...


@app.delete("/users/{user_id}")
def delete_user(user_id: int):
    ...
```

---

# 📦 Pydantic и валидация

Одна из ключевых возможностей FastAPI — автоматическая валидация входных данных.

```python
from pydantic import BaseModel


class UserCreate(BaseModel):
    name: str
    age: int
```

Используем модель:

```python
@app.post("/users")
def create_user(user: UserCreate):
    return user
```

Если клиент отправит:

```python
{
    "name": "Ilya",
    "age": 31
}
```

FastAPI передаст в функцию валидированный объект `UserCreate`.

---

# ❌ Ошибка валидации

Если клиент отправит неправильные данные:

```python
{
    "name": "Ilya",
    "age": "abc"
}
```

Pydantic обнаружит ошибку.

FastAPI вернёт HTTP-ошибку валидации вместо того, чтобы передать некорректные данные в endpoint.

---

# 🔢 Path Parameters

Параметры можно получать непосредственно из URL:

```python
@app.get("/users/{user_id}")
def get_user(user_id: int):
    return {"user_id": user_id}
```

Запрос:

```text
GET /users/10
```

FastAPI:

```text
"10"
 ↓
int
 ↓
user_id = 10
```

Типизация используется FastAPI для определения ожидаемого типа и валидации.

---

# 🔎 Query Parameters

Например:

```python
@app.get("/users")
def get_users(
    limit: int = 10,
    offset: int = 0,
):
    return {
        "limit": limit,
        "offset": offset,
    }
```

Запрос:

```text
GET /users?limit=20&offset=10
```

Получим:

```text
limit = 20
offset = 10
```

---

# 💉 Dependency Injection

FastAPI имеет встроенный механизм DI через `Depends`.

```python
from fastapi import Depends


def get_current_user():
    return {"id": 1}


@app.get("/profile")
def profile(user=Depends(get_current_user)):
    return user
```

FastAPI автоматически:

```text
Depends(get_current_user)
        ↓
вызывает dependency
        ↓
получает результат
        ↓
передаёт его в endpoint
```

Это позволяет выносить общую логику:

* авторизацию;
* получение DB session;
* проверки;
* repositories;
* сервисы.

---

# ⚡ `async` и `await`

FastAPI поддерживает асинхронные endpoint'ы:

```python
@app.get("/users")
async def get_users():
    users = await get_users_from_db()
    return users
```

Во время `await` event loop может выполнять другие задачи.

Это особенно полезно для **I/O-bound операций**:

```text
HTTP
↓
Database
↓
External API
↓
Files
```

Но важно:

> **FastAPI не делает обычный блокирующий код автоматически асинхронным.**

Например:

```python
import time


@app.get("/")
async def root():
    time.sleep(5)
    return {"status": "ok"}
```

`time.sleep()` блокирует выполнение.

---

# 🔄 Sync и Async Endpoint

FastAPI позволяет использовать обычные функции:

```python
@app.get("/sync")
def sync_endpoint():
    return {"status": "ok"}
```

И асинхронные:

```python
@app.get("/async")
async def async_endpoint():
    return {"status": "ok"}
```

То есть:

```text
FastAPI
├── def
└── async def
```

---

# 📤 Response Model

Можно определить структуру ответа:

```python
from pydantic import BaseModel


class UserResponse(BaseModel):
    id: int
    name: str


@app.get("/users/{user_id}", response_model=UserResponse)
def get_user(user_id: int):
    return {
        "id": user_id,
        "name": "Ilya",
    }
```

FastAPI использует модель для:

* валидации ответа;
* сериализации;
* формирования OpenAPI-схемы.

---

# 📚 Автоматическая документация

FastAPI автоматически генерирует OpenAPI-схему.

Из неё строятся интерактивные документации.

Обычно доступны:

```text
/docs
```

Swagger UI

и:

```text
/redoc
```

ReDoc.

Схема:

```text
Python type hints
       ↓
FastAPI
       ↓
OpenAPI
       ↓
Swagger UI / ReDoc
```

Это одно из заметных преимуществ FastAPI.

---

# 🌐 FastAPI и ASGI

FastAPI — **ASGI-приложение**.

Обычно его запускают через Uvicorn:

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
FastAPI application
```

Архитектура:

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

---

# 🔌 FastAPI и WebSocket

FastAPI поддерживает WebSocket:

```python
from fastapi import WebSocket


@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()

    while True:
        message = await websocket.receive_text()
        await websocket.send_text(message)
```

Схема:

```text
Client
  ↕
WebSocket
  ↕
Uvicorn
  ↕
FastAPI
```

---

# 🧱 Middleware

FastAPI поддерживает middleware.

Middleware выполняется вокруг обработки запроса:

```text
Request
   ↓
Middleware
   ↓
Endpoint
   ↓
Middleware
   ↓
Response
```

Middleware можно использовать для:

* логирования;
* CORS;
* обработки заголовков;
* измерения времени;
* других cross-cutting concerns.

---

# 🔐 Authentication и Authorization

FastAPI не заставляет использовать один конкретный способ аутентификации.

Можно реализовать:

* JWT;
* OAuth2;
* API keys;
* HTTP Basic;
* собственную схему.

Часто используются `Depends` и security utilities FastAPI.

Например, общая идея:

```text
Request
   ↓
Authentication
   ↓
Current User
   ↓
Endpoint
```

---

# 🆚 FastAPI vs Django

| FastAPI                    | Django                            |
| -------------------------- | --------------------------------- |
| API-first                  | Full-stack                        |
| ASGI                       | WSGI + ASGI                       |
| Pydantic                   | Django Forms / Models             |
| `Depends`                  | Middleware / другие механизмы DI  |
| Нет встроенной ORM         | Встроенный ORM                    |
| Нет встроенной Admin       | Есть Admin                        |
| Swagger/OpenAPI из коробки | Обычно дополнительные инструменты |
| Очень удобен для async API | Большая готовая экосистема        |

Упрощённо:

```text
FastAPI
↓
API / async / microservices

Django
↓
full-stack / monolith / готовая инфраструктура
```

---

# 🆚 FastAPI и Flask

| FastAPI                         | Flask                             |
| ------------------------------- | --------------------------------- |
| ASGI                            | WSGI                              |
| Async-first                     | Изначально sync                   |
| Pydantic                        | Нет встроенной Pydantic-валидации |
| OpenAPI автоматически           | Обычно дополнительные библиотеки  |
| Type hints активно используются | Type hints не являются основой    |
| `Depends`                       | Нет встроенного аналога           |

---

# 🧠 Типичная архитектура FastAPI-проекта

В реальном backend-проекте часто разделяют:

```text
app/
├── main.py
├── api/
│   └── routes/
├── schemas/
├── models/
├── services/
├── repositories/
├── dependencies/
└── core/
```

И поток запроса:

```text
HTTP Request
     ↓
Router
     ↓
Dependencies
     ↓
Service
     ↓
Repository
     ↓
Database
     ↓
Service
     ↓
Response Schema
     ↓
HTTP Response
```

Это позволяет разделить ответственность между слоями.

---

# 🎯 Главное

Запомнить основные элементы:

```text
FastAPI
│
├── ASGI
├── Starlette
├── Pydantic
├── Depends
├── OpenAPI
├── Swagger
├── ReDoc
├── WebSocket
└── async/await
```

Связка для Backend:

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
   ├── Router
   ├── Depends
   ├── Pydantic
   └── Service
        ↓
    Repository
        ↓
    PostgreSQL
```

### Формула для собеседования

> **FastAPI — современный ASGI-фреймворк для создания API на Python. Он использует Starlette для web-функциональности, Pydantic для валидации и сериализации данных, а `Depends` для Dependency Injection. Благодаря ASGI поддерживает асинхронные endpoint'ы и WebSocket, а OpenAPI-документация генерируется автоматически.**

```text
FastAPI
= ASGI
+ Starlette
+ Pydantic
+ Depends
+ OpenAPI
+ async/await
+ WebSocket
```
