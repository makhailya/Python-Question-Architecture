# 🐍 Django

## 🎯 Ответ на собеседовании

**Django** — это высокоуровневый Python-фреймворк для разработки веб-приложений.

Он предоставляет готовую инфраструктуру для:

* обработки HTTP-запросов;
* маршрутизации URL;
* работы с базой данных через ORM;
* аутентификации и авторизации;
* работы с формами;
* административной панели;
* middleware;
* безопасности;
* шаблонов.

Django использует архитектурный паттерн, который обычно называют **MVT (Model–View–Template)**.

Упрощённо:

```text
Client
   ↓
URL
   ↓
View
   ↓
Model / ORM
   ↓
Database
   ↓
View
   ↓
Template
   ↓
Response
```

Django подходит как для монолитных веб-приложений, так и для создания API с использованием **Django REST Framework (DRF)**.

---

## 🎤 Суперкоротко

```text
Django = Python Web Framework

Основные компоненты:

URL → View → Model → Database
             ↓
          Template
```

Главное:

> **Django — batteries-included фреймворк: он предоставляет большую часть инфраструктуры веб-приложения из коробки.**

---

# 🏗️ Архитектура Django

Чаще всего Django описывают через **MVT**:

```text
Model
  ↓
работа с данными

View
  ↓
бизнес-логика / обработка запроса

Template
  ↓
представление данных
```

### Model

Отвечает за структуру и работу с данными.

```python
from django.db import models


class User(models.Model):
    name = models.CharField(max_length=100)
    email = models.EmailField()
```

Django ORM сопоставляет модель с таблицей базы данных.

---

### View

Обрабатывает запрос:

```python
from django.http import JsonResponse


def users(request):
    return JsonResponse({"users": []})
```

View получает HTTP-запрос и формирует HTTP-ответ.

---

### Template

Используется для формирования HTML:

```html
<h1>Hello, {{ name }}</h1>
```

---

# 🔄 Жизненный цикл запроса

Допустим, пользователь открывает:

```text
/users/
```

Происходит примерно следующее:

```text
Client
  ↓
HTTP Request
  ↓
Django
  ↓
URL Dispatcher
  ↓
View
  ↓
Model / ORM
  ↓
Database
  ↓
View
  ↓
Template
  ↓
HTTP Response
  ↓
Client
```

---

# 🔗 URL Routing

Маршруты определяются в `urls.py`.

```python
from django.urls import path

from .views import users


urlpatterns = [
    path("users/", users),
]
```

Когда приходит:

```text
GET /users/
```

Django находит соответствующий `View`.

```text
/users/
   ↓
urls.py
   ↓
users()
```

---

# 👁️ View

View — компонент, который обрабатывает HTTP-запрос.

Простейший пример:

```python
from django.http import HttpResponse


def hello(request):
    return HttpResponse("Hello!")
```

Маршрут:

```python
from django.urls import path

from .views import hello


urlpatterns = [
    path("hello/", hello),
]
```

Запрос:

```text
GET /hello/
```

попадёт в:

```python
hello(request)
```

---

# 🗄️ Django ORM

**ORM (Object-Relational Mapping)** позволяет работать с базой данных через Python-объекты вместо написания SQL вручную.

Модель:

```python
from django.db import models


class Product(models.Model):
    name = models.CharField(max_length=100)
    price = models.DecimalField(max_digits=10, decimal_places=2)
```

Получение объектов:

```python
products = Product.objects.all()
```

Фильтрация:

```python
products = Product.objects.filter(price__gte=1000)
```

Создание:

```python
Product.objects.create(
    name="Laptop",
    price=100000,
)
```

Django ORM сам формирует SQL-запросы.

---

# 🧱 Model → Database

Например:

```python
class Product(models.Model):
    name = models.CharField(max_length=100)
```

Django может создать таблицу примерно такого вида:

```text
Product
----------------
id
name
```

Модель становится Python-представлением данных базы.

---

# 🔄 Migrations

**Миграции** позволяют синхронизировать структуру моделей Django со структурой базы данных.

После изменения модели:

```python
class Product(models.Model):
    name = models.CharField(max_length=100)
    price = models.IntegerField()
```

создаются миграции:

```text
models.py
   ↓
makemigrations
   ↓
migration files
   ↓
migrate
   ↓
Database
```

Основные команды:

```python
python manage.py makemigrations
python manage.py migrate
```

`makemigrations` создаёт файлы миграций.

`migrate` применяет их к базе данных.

---

# 📦 Project и App

Это важное различие.

### Project

Весь Django-проект.

Например:

```text
shop/
```

### App

Отдельная функциональная часть проекта.

Например:

```text
users
products
orders
payments
```

Структура:

```text
shop/
├── manage.py
├── shop/
│   ├── settings.py
│   ├── urls.py
│   ├── asgi.py
│   └── wsgi.py
│
├── users/
├── products/
└── orders/
```

Один Django Project может содержать несколько Apps.

---

# ⚙️ `manage.py`

`manage.py` — командный интерфейс Django-проекта.

Например:

```python
python manage.py runserver
```

Запускает development server.

Другие команды:

```python
python manage.py makemigrations
python manage.py migrate
python manage.py createsuperuser
python manage.py shell
```

---

# 🛡️ Middleware

**Middleware** — компоненты, которые обрабатывают запросы и ответы до или после View.

Схематично:

```text
Request
   ↓
Middleware
   ↓
Middleware
   ↓
View
   ↓
Middleware
   ↓
Response
```

Middleware используется, например, для:

* аутентификации;
* работы с сессиями;
* логирования;
* обработки безопасности;
* добавления заголовков.

---

# 👤 Authentication

Django предоставляет встроенную систему пользователей.

Она включает механизмы для:

* пользователей;
* паролей;
* групп;
* permissions;
* authentication.

Например:

```python
request.user
```

позволяет получить текущего пользователя.

---

# 🔐 CSRF

Django имеет встроенную защиту от **CSRF (Cross-Site Request Forgery)**.

Для HTML-форм используется CSRF-токен:

```html
<form method="post">
    {% csrf_token %}
</form>
```

Это защищает приложение от определённого класса атак, связанных с подделкой запросов от имени пользователя.

---

# 🖥️ Django Admin

Django предоставляет готовую административную панель.

Можно зарегистрировать модель:

```python
from django.contrib import admin

from .models import Product


admin.site.register(Product)
```

После этого модель будет доступна через Django Admin.

Это одно из преимуществ Django как **batteries-included framework**.

---

# ⚡ Django и REST API

Сам Django не является исключительно API-фреймворком.

Для создания REST API часто используется:

**Django REST Framework (DRF).**

Схема:

```text
Client
   ↓
HTTP Request
   ↓
Django
   ↓
DRF
   ↓
Serializer
   ↓
View / ViewSet
   ↓
ORM
   ↓
Database
```

Например, Django + DRF позволяют создавать:

```text
GET    /api/users/
POST   /api/users/
GET    /api/users/1/
PATCH  /api/users/1/
DELETE /api/users/1/
```

---

# ⚡ Django и ASGI / WSGI

Django поддерживает оба интерфейса.

В проекте обычно есть:

```text
wsgi.py
asgi.py
```

```text
WSGI
 ↓
синхронное приложение


ASGI
 ↓
асинхронная инфраструктура
```

Для production могут использоваться соответствующие серверы:

```text
WSGI → Gunicorn / uWSGI

ASGI → Uvicorn / Hypercorn / Daphne
```

---

# 🆚 Django vs FastAPI

| Django                                    | FastAPI                                  |
| ----------------------------------------- | ---------------------------------------- |
| Full-stack framework                      | API/Web framework                        |
| Большое количество встроенных компонентов | Минималистичнее                          |
| ORM                                       | Нет встроенного ORM                      |
| Admin                                     | Нет встроенной админки                   |
| Templates                                 | Нет основной встроенной системы шаблонов |
| Authentication                            | Много готовых возможностей               |
| REST API через DRF                        | REST API — основной сценарий             |
| WSGI + ASGI                               | ASGI                                     |

Упрощённо:

```text
Django
↓
большая готовая экосистема


FastAPI
↓
гибкий современный API-first подход
```

---

# 🧠 Django в двух словах

```text
Django
│
├── URL routing
├── Views
├── Models
├── ORM
├── Migrations
├── Templates
├── Middleware
├── Authentication
├── Security
├── Admin
└── Management commands
```

---

# 🎯 Главное

Запомнить архитектуру:

```text
             Django
                │
       ┌────────┼────────┐
       ↓        ↓        ↓
      URL      View     Model
                │         │
                ↓         ↓
            Template   Database
```

И основной путь запроса:

```text
HTTP Request
     ↓
URL
     ↓
View
     ↓
ORM
     ↓
Database
     ↓
View
     ↓
HTTP Response
```

### Формула для собеседования

> **Django — высокоуровневый Python-фреймворк для разработки веб-приложений с большим количеством встроенных возможностей: ORM, routing, middleware, authentication, admin, migrations и другими компонентами.**

```text
Django = Web Framework
        + ORM
        + Admin
        + Auth
        + Middleware
        + Security
        + Templates
        + Migrations
```
