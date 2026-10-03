# 🌐 WSGI — Web Server Gateway Interface

## 🎯 Ответ на собеседовании

**WSGI (Web Server Gateway Interface)** — стандартный интерфейс взаимодействия между **Python-веб-приложением и веб-сервером**.

Он определяет, как веб-сервер должен передавать HTTP-запрос Python-приложению и как приложение должно вернуть HTTP-ответ.

WSGI используется в основном для **синхронных Python-веб-приложений**, например Django и Flask.

Типичная схема:

```text
Клиент
   ↓
Web Server
   ↓
WSGI Server
   ↓
Django / Flask
   ↓
HTTP Response
```

Популярные WSGI-серверы:

* **Gunicorn**
* **uWSGI**
* **mod_wsgi**

---

## 🎤 Суперкоротко

```text
WSGI = стандарт связи веб-сервера с Python-приложением

WSGI → синхронный Python Web
ASGI → синхронный + асинхронный Python Web
```

Главное:

> **WSGI — это интерфейс/стандарт, а Gunicorn — сервер, который этот интерфейс реализует.**

---

# 🔌 Зачем нужен WSGI

Веб-сервер не должен знать внутреннее устройство Django или Flask.

Например:

```text
Nginx
  ↓
Gunicorn
  ↓
Django
```

Nginx принимает HTTP-запрос.

Gunicorn принимает запрос и передаёт его Python-приложению через WSGI.

Приложение формирует ответ.

Ответ возвращается обратно:

```text
Django
  ↓
Gunicorn
  ↓
Nginx
  ↓
Клиент
```

WSGI задаёт правила взаимодействия между Gunicorn и Python-приложением.

---

# 🧩 Как выглядит WSGI-приложение

Минимальное WSGI-приложение:

```python
def application(environ, start_response):
    status = "200 OK"
    headers = [
        ("Content-Type", "text/plain")
    ]

    start_response(status, headers)

    return [b"Hello, World!"]
```

WSGI-приложение — это вызываемый объект, который получает:

```text
environ
start_response
```

---

# 📦 `environ`

`environ` — словарь с информацией о HTTP-запросе и окружении.

Например:

```python
def application(environ, start_response):
    method = environ["REQUEST_METHOD"]
    path = environ["PATH_INFO"]

    ...
```

Там можно получить:

```text
REQUEST_METHOD
PATH_INFO
QUERY_STRING
CONTENT_TYPE
CONTENT_LENGTH
SERVER_NAME
SERVER_PORT
```

и другие параметры.

---

# 📤 `start_response`

`start_response` — функция, которую WSGI-сервер передаёт приложению.

Через неё приложение сообщает:

* HTTP-статус;
* HTTP-заголовки.

Например:

```python
start_response(
    "200 OK",
    [
        ("Content-Type", "text/plain")
    ]
)
```

После этого приложение возвращает тело ответа.

---

# 🔄 Полный цикл

Допустим, клиент отправил:

```text
GET /users
```

Происходит примерно следующее:

```text
1. Клиент
      ↓
2. Nginx
      ↓
3. Gunicorn
      ↓
4. WSGI
      ↓
5. Django
      ↓
6. Response
      ↓
7. Gunicorn
      ↓
8. Nginx
      ↓
9. Клиент
```

WSGI обеспечивает стандартный способ взаимодействия между сервером и Python-приложением.

---

# 🏗️ WSGI в Django

Django предоставляет WSGI-приложение.

Обычно оно находится в:

```text
project/
├── manage.py
└── project/
    ├── settings.py
    ├── urls.py
    ├── wsgi.py
    └── asgi.py
```

В `wsgi.py`:

```python
import os

from django.core.wsgi import get_wsgi_application

os.environ.setdefault(
    "DJANGO_SETTINGS_MODULE",
    "project.settings",
)

application = get_wsgi_application()
```

Gunicorn может запустить это приложение:

```text
project.wsgi:application
```

---

# 🚀 WSGI + Gunicorn

Типичный production-стек:

```text
Internet
   ↓
Nginx
   ↓
Gunicorn
   ↓
Django
```

Gunicorn запускает несколько worker-процессов:

```text
Gunicorn
 ├── Worker 1 → Django
 ├── Worker 2 → Django
 ├── Worker 3 → Django
 └── Worker 4 → Django
```

Это позволяет обрабатывать несколько запросов одновременно за счёт нескольких процессов.

---

# ⚠️ WSGI и асинхронность

Классический WSGI ориентирован на **синхронную модель выполнения**.

Например:

```python
def view(request):
    result = slow_operation()
    return response
```

Пока выполняется блокирующая операция, worker занят.

Поэтому для современных Python-приложений с активным использованием:

```text
asyncio
async/await
WebSocket
долгих соединений
```

используют **ASGI**.

---

# 🆚 WSGI vs ASGI

| WSGI                                   | ASGI                                   |
| -------------------------------------- | -------------------------------------- |
| Web Server Gateway Interface           | Asynchronous Server Gateway Interface  |
| В основном синхронный                  | Синхронный + асинхронный               |
| HTTP                                   | HTTP + WebSocket и другие протоколы    |
| Классическая модель request → response | Поддерживает долгоживущие соединения   |
| Django, Flask                          | FastAPI, Starlette, современный Django |
| Gunicorn, uWSGI                        | Uvicorn, Hypercorn, Daphne             |

Схематично:

```text
WSGI:

Request
   ↓
Python application
   ↓
Response
```

```text
ASGI:

Request / Event
      ↓
Async application
      ↓
Response / Event
```

---

# 🔥 WSGI vs Gunicorn

Это частый вопрос.

**WSGI** — стандарт интерфейса.

**Gunicorn** — WSGI-сервер.

То есть:

```text
WSGI
↓
определяет интерфейс


Gunicorn
↓
реализует серверную часть этого взаимодействия
```

Пример:

```text
Nginx
  ↓
Gunicorn
  ↓
WSGI
  ↓
Django
```

---

# 🧠 Аналогия

Представь розетку.

```text
WSGI = стандарт розетки
Gunicorn = устройство, которое подключается к розетке
Django = устройство, которое получает питание
```

WSGI определяет **правила взаимодействия**, но сам по себе не является веб-сервером.

---

# 🎯 Главное

```text
WSGI
↓
стандарт взаимодействия
↓
Web Server / WSGI Server
↓
Python Web Application
```

Запомнить:

```text
WSGI → классический синхронный Python Web
ASGI → современный async Python Web

WSGI ≠ сервер
Gunicorn ≠ WSGI

Gunicorn — WSGI-сервер
```

### Формула для собеседования

> **WSGI — стандарт интерфейса между веб-сервером и Python-веб-приложением, предназначенный преимущественно для синхронных приложений.**
