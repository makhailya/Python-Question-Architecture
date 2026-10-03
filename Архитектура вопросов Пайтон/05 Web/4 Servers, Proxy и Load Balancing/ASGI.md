# ⚡ ASGI — Asynchronous Server Gateway Interface

## 🎯 Ответ на собеседовании

**ASGI (Asynchronous Server Gateway Interface)** — стандарт интерфейса взаимодействия между веб-сервером и Python-приложением, который поддерживает **асинхронную обработку запросов**, а также долгоживущие соединения и протоколы вроде [[WebSocket]].

ASGI является развитием идеи WSGI и позволяет Python-приложениям работать с `async/await`.

Типичная схема:

```text
Клиент
   ↓
Web Server / ASGI Server
   ↓
FastAPI / Starlette / Django
   ↓
HTTP Response
```

Популярные ASGI-серверы:

* **Uvicorn**
* **Hypercorn**
* **Daphne**

---

## 🎤 Суперкоротко

```text
ASGI = стандарт взаимодействия сервера с Python-приложением

WSGI → в основном синхронный
ASGI → синхронный + асинхронный

ASGI → HTTP + WebSocket + другие долгоживущие соединения
```

Главное:

> **ASGI позволяет Python-приложению использовать асинхронную модель `async/await`.**

---

# 🔌 Зачем нужен ASGI

Без ASGI классическое WSGI-приложение ориентировано на синхронную модель:

```text
Request
   ↓
Application
   ↓
Response
```

ASGI позволяет работать с событиями асинхронно:

```text
Request
   ↓
Async Application
   ↓
Response
```

При этом приложение может не блокировать event loop во время ожидания I/O.

Например:

```python
async def get_user():
    user = await database.fetch_one()
    return user
```

Пока приложение ждёт ответ базы данных, event loop может выполнять другие корутины.

---

# ⚙️ ASGI и `async/await`

Одна из главных причин появления ASGI — поддержка асинхронной модели Python.

Например, FastAPI:

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/users")
async def get_users():
    return {"users": []}
```

Здесь endpoint является корутиной.

ASGI-сервер умеет взаимодействовать с таким приложением в асинхронной модели.

---

# 🚀 ASGI + Uvicorn

Типичный стек FastAPI:

```text
Internet
   ↓
Nginx
   ↓
Uvicorn
   ↓
FastAPI
```

**Uvicorn** — ASGI-сервер.

Он принимает сетевые соединения и передаёт события ASGI-приложению.

Например:

```text
Uvicorn
   ↓
FastAPI application
```

---

# 🧩 ASGI-приложение

На низком уровне ASGI-приложение взаимодействует с сервером через интерфейс:

```python
async def application(scope, receive, send):
    ...
```

Здесь:

* `scope` — информация о соединении/запросе;
* `receive` — получение событий;
* `send` — отправка событий.

Упрощённо:

```text
scope
 ↓
информация о запросе


receive
 ↓
получение событий


send
 ↓
отправка ответа
```

---

# 📦 `scope`

`scope` содержит информацию о текущем соединении.

Например:

```python
async def application(scope, receive, send):
    print(scope["type"])
```

Для HTTP это может быть:

```text
http
```

Для WebSocket:

```text
websocket
```

Таким образом, ASGI может работать не только с HTTP-запросами.

---

# 🔄 `receive` и `send`

ASGI использует событийную модель.

Приложение получает события через `receive`:

```python
event = await receive()
```

И отправляет события через `send`:

```python
await send(event)
```

Упрощённая схема:

```text
Server
  │
  │ receive
  ↓
Application
  │
  │ send
  ↓
Server
```

---

# 🌐 WebSocket

Одно из важных преимуществ ASGI — поддержка WebSocket.

Например, приложение может поддерживать постоянное соединение:

```text
Клиент
  ↕
WebSocket
  ↕
ASGI
  ↕
Python application
```

В отличие от обычной модели:

```text
Request → Response
```

WebSocket предполагает длительное двустороннее соединение.

---

# 🆚 WSGI vs ASGI

| WSGI                                 | ASGI                                   |
| ------------------------------------ | -------------------------------------- |
| Web Server Gateway Interface         | Asynchronous Server Gateway Interface  |
| В основном синхронная модель         | Поддерживает async                     |
| Request → Response                   | Событийная модель                      |
| HTTP                                 | HTTP, WebSocket и другие протоколы     |
| Классические Django/Flask-приложения | FastAPI, Starlette, современный Django |
| Gunicorn, uWSGI                      | Uvicorn, Hypercorn, Daphne             |

Главное различие:

```text
WSGI
↓
синхронная модель


ASGI
↓
асинхронная модель
↓
async/await
↓
долгие соединения
```

---

# ⚠️ Важный нюанс

**ASGI не делает любой код автоматически асинхронным.**

Например:

```python
import time


async def endpoint():
    time.sleep(5)
    return {"status": "ok"}
```

`time.sleep()` — блокирующая операция.

Она заблокирует event loop.

Правильнее использовать асинхронный API:

```python
import asyncio


async def endpoint():
    await asyncio.sleep(5)
    return {"status": "ok"}
```

Поэтому:

```text
ASGI
≠
весь код автоматически async
```

ASGI предоставляет асинхронную модель, но код приложения тоже должен корректно её использовать.

---

# 🏗️ ASGI в FastAPI

FastAPI построен поверх **Starlette**, которая предоставляет ASGI-совместимую инфраструктуру.

Типичный production-стек:

```text
                    ┌──────────────┐
                    │    Client    │
                    └──────┬───────┘
                           ↓
                    ┌──────────────┐
                    │    Nginx     │
                    └──────┬───────┘
                           ↓
                    ┌──────────────┐
                    │   Uvicorn    │
                    │    ASGI      │
                    └──────┬───────┘
                           ↓
                    ┌──────────────┐
                    │   FastAPI    │
                    └──────────────┘
```

---

# 🧠 ASGI и Event Loop

В асинхронном приложении:

```text
                Event Loop
                    │
        ┌───────────┼───────────┐
        ↓           ↓           ↓
   Coroutine 1  Coroutine 2  Coroutine 3
        │           │           │
        ↓           ↓           ↓
      await       await       await
```

Когда одна корутина ожидает I/O через `await`, event loop может переключиться на другую готовую задачу.

Например:

```python
async def handler():
    user = await get_user()
    orders = await get_orders()
    return user, orders
```

Это особенно полезно для I/O-bound задач.

---

# 🎯 Главное

```text
ASGI
↓
стандарт взаимодействия
↓
ASGI-сервер
↓
Python Web Application
```

Запомнить:

```text
WSGI → классический синхронный Python Web
ASGI → async Python Web + WebSocket
```

```text
Uvicorn → ASGI-сервер
FastAPI → ASGI-приложение
```

И самое важное:

> **ASGI — это стандарт интерфейса, а не конкретный сервер.**

### Формула для собеседования

```text
WSGI → sync
ASGI → async

ASGI
↓
async/await
↓
event loop
↓
конкурентная обработка I/O
↓
WebSocket
```
