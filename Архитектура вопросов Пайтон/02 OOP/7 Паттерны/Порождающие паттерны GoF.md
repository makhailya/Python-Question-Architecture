# Порождающие паттерны GoF 🏗️

## 🎯 Ответ на собеседовании

**Порождающие паттерны GoF** отвечают за создание объектов.

Их задача — **скрыть или структурировать логику создания объектов**, чтобы код не был жёстко связан с конкретными классами.

В GoF есть **5 порождающих паттернов**:

1. **Singleton** — один экземпляр класса.
2. **Factory Method** — создание объекта через фабричный метод.
3. **Abstract Factory** — создание семейства связанных объектов.
4. **Builder** — пошаговое создание сложного объекта.
5. **Prototype** — создание нового объекта через копирование существующего.

Главная идея:

```text
Без паттерна:

код → напрямую создаёт конкретный класс

С паттерном:

код → абстракция создания → конкретный объект
```

---

## 🎤 Суперкоротко

> Порождающие паттерны управляют созданием объектов и уменьшают зависимость кода от конкретных классов.

| Паттерн          | Идея                              | Пример                        |
| ---------------- | --------------------------------- | ----------------------------- |
| Singleton        | Один экземпляр                    | конфигурация приложения       |
| Factory Method   | Выбрать, какой объект создать     | разные типы уведомлений       |
| Abstract Factory | Создать семейство объектов        | PostgreSQL/MySQL компоненты   |
| Builder          | Создавать сложный объект по шагам | сложный HTTP-запрос           |
| Prototype        | Клонировать существующий объект   | создание похожих конфигураций |

---

# 1. Singleton 🔒

**Singleton** гарантирует, что у класса существует только **один экземпляр**, и предоставляет глобальную точку доступа к нему.

Пример:

```python
class Singleton:
    _instance = None

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)

        return cls._instance
```

Использование:

```python
a = Singleton()
b = Singleton()

print(a is b)
# True
```

### Где может применяться

* конфигурация;
* registry;
* кэш;
* некоторые менеджеры ресурсов.

### Важный нюанс Python

В Python Singleton часто **не нужен**.

Например, вместо него можно использовать модуль:

```python
# config.py

DATABASE_URL = "postgresql://..."
```

Импортируемый модуль фактически используется как единый объект.

### На собеседовании

> Singleton гарантирует существование одного экземпляра класса. Но в Python его часто избегают из-за глобального состояния и используют dependency injection или модульный уровень.

---

# 2. Factory Method 🏭

**Factory Method** переносит создание объекта в отдельный метод.

Вместо:

```python
if notification_type == "email":
    notification = EmailNotification()
elif notification_type == "sms":
    notification = SMSNotification()
```

создание можно вынести в фабрику:

```python
class NotificationFactory:
    @staticmethod
    def create(notification_type):
        if notification_type == "email":
            return EmailNotification()

        if notification_type == "sms":
            return SMSNotification()

        raise ValueError("Unknown notification type")
```

Использование:

```python
notification = NotificationFactory.create("email")
```

### Зачем

Клиентскому коду не обязательно знать конкретный класс:

```text
Client
   ↓
Factory
   ↓
EmailNotification
SMSNotification
PushNotification
```

### Backend-пример

Например, сервис отправки сообщений:

```python
class EmailSender:
    def send(self, message):
        print("Email:", message)


class SmsSender:
    def send(self, message):
        print("SMS:", message)


class SenderFactory:
    @staticmethod
    def create(sender_type):
        if sender_type == "email":
            return EmailSender()

        if sender_type == "sms":
            return SmsSender()

        raise ValueError("Unknown sender")
```

---

# 3. Abstract Factory 🏭🏭

**Abstract Factory** предоставляет интерфейс для создания **семейства связанных объектов**.

Например, приложение поддерживает PostgreSQL и MySQL.

Для PostgreSQL нужны:

```text
PostgreSQLConnection
PostgreSQLQuery
PostgreSQLTransaction
```

Для MySQL:

```text
MySQLConnection
MySQLQuery
MySQLTransaction
```

Фабрика выбирает целое семейство:

```text
        DatabaseFactory
          /          \
         /            \
PostgreSQLFactory   MySQLFactory
      ↓                 ↓
Connection           Connection
Query                Query
Transaction          Transaction
```

Пример:

```python
class PostgreSQLFactory:
    def create_connection(self):
        return PostgreSQLConnection()

    def create_query(self):
        return PostgreSQLQuery()
```

Клиент работает с фабрикой:

```python
factory = PostgreSQLFactory()

connection = factory.create_connection()
query = factory.create_query()
```

### Factory Method vs Abstract Factory

**Factory Method:**

> Создать один определённый объект.

**Abstract Factory:**

> Создать семейство связанных объектов.

|             | Factory Method    | Abstract Factory                        |
| ----------- | ----------------- | --------------------------------------- |
| Что создаёт | Один тип продукта | Семейство продуктов                     |
| Сложность   | Проще             | Сложнее                                 |
| Пример      | `create_sender()` | `create_connection()`, `create_query()` |

---

# 4. Builder 🧱

**Builder** используется для пошагового создания сложного объекта.

Особенно полезен, когда объект имеет много параметров.

Без Builder:

```python
request = Request(
    url,
    method,
    headers,
    params,
    timeout,
    auth,
    retries,
)
```

Можно получить длинный и плохо читаемый конструктор.

С Builder:

```python
request = (
    RequestBuilder()
    .url("https://example.com")
    .method("GET")
    .timeout(10)
    .retries(3)
    .build()
)
```

### Идея

```text
Builder
   ↓
url()
   ↓
method()
   ↓
headers()
   ↓
timeout()
   ↓
build()
   ↓
Request
```

### Где встречается

* сложные конфигурации;
* HTTP-запросы;
* SQL-запросы;
* Docker-конфигурации;
* объекты с большим количеством опциональных параметров.

### Python и Builder

В Python необходимость в Builder часто уменьшается благодаря:

* keyword arguments;
* `dataclass`;
* default values;
* `**kwargs`.

Например:

```python
from dataclasses import dataclass


@dataclass
class Request:
    url: str
    method: str = "GET"
    timeout: int = 10
    retries: int = 3
```

Поэтому Builder стоит применять, когда объект действительно сложный.

---

# 5. Prototype 🧬

**Prototype** создаёт новый объект путём **копирования существующего объекта**.

В Python для этого можно использовать `copy`.

```python
import copy


original = {
    "name": "Ilya",
    "role": "developer",
}

clone = copy.deepcopy(original)
```

Теперь:

```python
clone["name"] = "Alex"

print(original["name"])
# Ilya

print(clone["name"])
# Alex
```

### Shallow Copy vs Deep Copy

```python
copy.copy()
```

создаёт поверхностную копию.

```python
copy.deepcopy()
```

создаёт глубокую копию вложенных объектов.

```text
Shallow Copy

original ───────→ nested object
clone    ───────→ nested object


Deep Copy

original ───────→ nested object A

clone    ───────→ nested object B
```

### Где применять

Prototype полезен, когда:

* объект дорого создавать заново;
* нужно создавать много похожих объектов;
* объект уже содержит необходимую конфигурацию.

---

# Factory Method vs Builder vs Prototype

Это часто путают.

### Factory Method

Отвечает:

> **Какой объект создать?**

```python
user = UserFactory.create("admin")
```

---

### Builder

Отвечает:

> **Как пошагово собрать сложный объект?**

```python
request = (
    RequestBuilder()
    .url(url)
    .timeout(10)
    .build()
)
```

---

### Prototype

Отвечает:

> **Как получить новый объект на основе существующего?**

```python
new_config = copy.deepcopy(old_config)
```

---

# Как связаны порождающие паттерны

```text
                 Создание объектов
                       │
        ┌──────────────┼──────────────┐
        │              │              │
     Factory         Builder       Prototype
        │              │              │
   какой объект?   как собрать?   копировать?
        │
   ┌────┴─────┐
   │          │
Factory    Abstract
Method     Factory
```

Singleton решает другую задачу:

```text
Singleton
    ↓
Сколько экземпляров?
    ↓
Один
```

---

# Порождающие паттерны и SOLID

Порождающие паттерны часто помогают соблюдать **Dependency Inversion Principle**.

Плохо:

```python
class OrderService:
    def __init__(self):
        self.payment = StripePayment()
```

`OrderService` напрямую зависит от конкретного класса.

Лучше:

```python
class OrderService:
    def __init__(self, payment):
        self.payment = payment
```

Теперь конкретную реализацию можно создать через фабрику:

```python
payment = PaymentFactory.create("stripe")

service = OrderService(payment)
```

Получается:

```text
Factory
   ↓
Concrete implementation
   ↓
Dependency Injection
   ↓
Service
```

---

# Порождающие паттерны в Python Backend

На практике наиболее полезно понимать:

### Factory

Например:

```text
PaymentFactory
├── StripePayment
├── YooKassaPayment
└── PayPalPayment
```

### Builder

Например:

```text
QueryBuilder
├── select()
├── where()
├── order_by()
└── build()
```

### Singleton

Например:

```text
ApplicationConfig
```

Но часто вместо Singleton используют **DI**.

### Prototype

Например:

```text
Шаблон конфигурации
        ↓
     deepcopy()
        ↓
Новая конфигурация
```

### Abstract Factory

Например:

```text
DatabaseFactory
├── PostgreSQLFactory
└── MySQLFactory
```

---

# ⚠️ Что важно на собеседовании

Не стоит говорить:

> «Порождающие паттерны нужны для того, чтобы создавать объекты».

Это слишком поверхностно.

Лучше:

> **Порождающие паттерны инкапсулируют и структурируют создание объектов, уменьшая связанность клиентского кода с конкретными реализациями.**

И дополнительно:

> **В Python многие классические паттерны реализуются проще благодаря динамической типизации, функциям первого класса, keyword arguments, dataclass и dependency injection. Поэтому GoF-паттерн не нужно применять только ради самого паттерна.**

---

# 🧠 Формула

```text
Порождающие GoF = 5 паттернов

Singleton
    → один объект

Factory Method
    → создать нужный объект

Abstract Factory
    → создать семейство объектов

Builder
    → собрать сложный объект по шагам

Prototype
    → создать объект копированием
```

**Главная идея:**

> **Не просто создавать объекты, а отделять код, который использует объект, от кода, который знает, как этот объект создать.**
