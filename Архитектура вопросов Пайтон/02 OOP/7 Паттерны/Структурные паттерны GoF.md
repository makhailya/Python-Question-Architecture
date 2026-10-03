# Структурные паттерны GoF 🧩

## 🎯 Ответ на собеседовании

**Структурные паттерны GoF** описывают, как организовать классы и объекты в более крупные структуры.

Их задача — сделать систему:

* менее связанной;
* гибкой;
* расширяемой;
* удобной для изменения;
* проще для интеграции разных компонентов.

В GoF есть **7 структурных паттернов**:

1. **Adapter** — адаптирует один интерфейс к другому.
2. **Bridge** — разделяет абстракцию и реализацию.
3. **Composite** — позволяет работать с объектами и группами объектов одинаково.
4. **Decorator** — динамически добавляет объекту поведение.
5. **Facade** — предоставляет простой интерфейс к сложной подсистеме.
6. **Flyweight** — экономит память за счёт повторного использования объектов.
7. **Proxy** — предоставляет заместителя для другого объекта.

---

## 🎤 Суперкоротко

| Паттерн       | Главная идея                                          |
| ------------- | ----------------------------------------------------- |
| **Adapter**   | Сделать несовместимые интерфейсы совместимыми         |
| **Bridge**    | Разделить абстракцию и реализацию                     |
| **Composite** | Одинаково работать с объектом и группой объектов      |
| **Decorator** | Добавить поведение без изменения класса               |
| **Facade**    | Спрятать сложную подсистему за простым API            |
| **Flyweight** | Переиспользовать общие объекты и экономить память     |
| **Proxy**     | Поставить объект-заместитель перед настоящим объектом |

Главная формула:

```text
Структурные паттерны
        ↓
Как соединить объекты и классы?
        ↓
Гибкая структура системы
```

---

# 1. Adapter 🔌

**Adapter** позволяет использовать объект с несовместимым интерфейсом.

Представим, что наш код ожидает:

```python
class Payment:
    def pay(self, amount):
        pass
```

Но сторонняя библиотека предоставляет:

```python
class StripeClient:
    def make_payment(self, value):
        print(f"Payment: {value}")
```

Интерфейсы разные.

Создаём Adapter:

```python
class StripeAdapter:
    def __init__(self, stripe_client):
        self.stripe_client = stripe_client

    def pay(self, amount):
        return self.stripe_client.make_payment(amount)
```

Теперь:

```python
stripe = StripeClient()
payment = StripeAdapter(stripe)

payment.pay(1000)
```

Получаем:

```text
Наш код
   ↓
Payment interface
   ↓
Adapter
   ↓
StripeClient
```

### Где применяется

* интеграция сторонних API;
* работа с legacy-кодом;
* совместимость разных библиотек;
* переход между версиями API.

### Запомнить

> **Adapter меняет интерфейс существующего объекта, не меняя сам объект.**

---

# 2. Bridge 🌉

**Bridge** разделяет абстракцию и её реализацию, чтобы они могли изменяться независимо.

Например:

```text
Уведомление
├── Email
├── SMS
└── Push

Способ отправки
├── API
└── Queue
```

Без Bridge количество комбинаций может быстро расти.

С Bridge:

```text
Notification
      │
      ↓
Sender interface
   ↙     ↓     ↘
 Email   SMS   Push
```

Пример:

```python
class Sender:
    def send(self, message):
        raise NotImplementedError


class EmailSender(Sender):
    def send(self, message):
        print(f"Email: {message}")


class SMSender(Sender):
    def send(self, message):
        print(f"SMS: {message}")


class Notification:
    def __init__(self, sender):
        self.sender = sender

    def notify(self, message):
        self.sender.send(message)
```

Использование:

```python
notification = Notification(EmailSender())
notification.notify("Hello")
```

Теперь `Notification` и `Sender` можно развивать независимо.

### Bridge vs Adapter

Это частый вопрос.

**Adapter:**

> Нужно сделать уже существующие несовместимые интерфейсы совместимыми.

**Bridge:**

> Нужно изначально разделить две независимые иерархии, чтобы они развивались отдельно.

---

# 3. Composite 🌳

**Composite** позволяет одинаково работать с отдельным объектом и группой объектов.

Классический пример — файловая система:

```text
Directory
├── File
├── File
└── Directory
    ├── File
    └── File
```

И файл, и директория могут иметь метод:

```python
size()
```

Пример:

```python
class File:
    def __init__(self, size):
        self.size = size

    def get_size(self):
        return self.size


class Directory:
    def __init__(self, children):
        self.children = children

    def get_size(self):
        return sum(
            child.get_size()
            for child in self.children
        )
```

Теперь клиенту не важно:

```python
file.get_size()
```

или:

```python
directory.get_size()
```

Он работает с единым интерфейсом.

### Где применяется

* файловые системы;
* деревья;
* меню;
* UI-компоненты;
* группы объектов;
* AST.

### Главное

> **Composite превращает дерево объектов в единый интерфейс работы с листьями и контейнерами.**

---

# 4. Decorator 🎁

**Decorator** позволяет динамически добавлять объекту поведение, не изменяя его исходный класс.

Например:

```python
class Service:
    def execute(self):
        print("Execute")
```

Добавим логирование:

```python
class LoggingDecorator:
    def __init__(self, service):
        self.service = service

    def execute(self):
        print("Start")
        self.service.execute()
        print("Finish")
```

Использование:

```python
service = Service()
service = LoggingDecorator(service)

service.execute()
```

Получаем:

```text
LoggingDecorator
        ↓
     Service
```

### Backend-примеры

Decorator особенно часто встречается в:

* middleware;
* логировании;
* кэшировании;
* авторизации;
* retry;
* метриках.

Например:

```python
def log_execution(func):
    def wrapper(*args, **kwargs):
        print("Start")
        result = func(*args, **kwargs)
        print("Finish")
        return result

    return wrapper
```

Использование:

```python
@log_execution
def process():
    print("Processing")
```

### Важно

Python-декоратор функции — практическая реализация идеи **Decorator**, хотя паттерн GoF первоначально описывает объектную структуру.

### Главное

> **Decorator добавляет поведение объекту через обёртку, не изменяя его исходный класс.**

---

# 5. Facade 🏢

**Facade** предоставляет простой интерфейс к сложной подсистеме.

Допустим, оформление заказа требует:

```text
OrderService
PaymentService
InventoryService
NotificationService
ShippingService
```

Клиенту не хочется управлять всеми сервисами самостоятельно.

Создаём Facade:

```python
class OrderFacade:
    def __init__(
        self,
        payment,
        inventory,
        notification,
    ):
        self.payment = payment
        self.inventory = inventory
        self.notification = notification

    def create_order(self, order):
        self.payment.pay(order.total)
        self.inventory.reserve(order.items)
        self.notification.send(order.user)
```

Теперь клиент вызывает:

```python
facade.create_order(order)
```

вместо нескольких вызовов.

```text
Client
  ↓
Facade
  ├── PaymentService
  ├── InventoryService
  └── NotificationService
```

### Где применяется

Очень распространённая идея в backend:

```text
Controller / Endpoint
        ↓
    Service
        ↓
несколько внутренних компонентов
```

### Главное

> **Facade скрывает сложность подсистемы и предоставляет клиенту простой интерфейс.**

---

# 6. Flyweight 🪶

**Flyweight** используется для экономии памяти за счёт совместного использования объектов с одинаковым внутренним состоянием.

Представим миллион объектов:

```text
User
User
User
User
...
```

У них может быть общая информация:

```text
role = "user"
permissions = (...)
```

Вместо хранения одинаковых данных миллион раз можно использовать один общий объект.

Пример:

```python
class Role:
    def __init__(self, name):
        self.name = name


class RoleFactory:
    _roles = {}

    @classmethod
    def get_role(cls, name):
        if name not in cls._roles:
            cls._roles[name] = Role(name)

        return cls._roles[name]
```

Использование:

```python
role1 = RoleFactory.get_role("admin")
role2 = RoleFactory.get_role("admin")

print(role1 is role2)
# True
```

### Идея

```text
1000 объектов
      ↓
общая часть
      ↓
один Flyweight
```

### Где применяется

* большие объёмы похожих объектов;
* кэширование;
* игровые объекты;
* текстовые редакторы;
* большие структуры данных.

### Главное

> **Flyweight уменьшает потребление памяти за счёт разделения общего состояния между объектами.**

---

# 7. Proxy 🕵️

**Proxy** — объект-заместитель, который контролирует доступ к другому объекту.

```text
Client
  ↓
 Proxy
  ↓
Real Object
```

Proxy может добавить:

* проверку доступа;
* lazy loading;
* кэширование;
* логирование;
* удалённый вызов.

Пример:

```python
class RealService:
    def get_data(self):
        return "data"


class ServiceProxy:
    def __init__(self, service):
        self.service = service

    def get_data(self, user):
        if not user.is_admin:
            raise PermissionError("Access denied")

        return self.service.get_data()
```

Теперь клиент работает с Proxy:

```python
proxy = ServiceProxy(RealService())

data = proxy.get_data(user)
```

### Backend-примеры

Proxy часто встречается в архитектуре:

```text
Client
  ↓
Nginx
  ↓
Gunicorn
  ↓
Application
```

Nginx может выступать как reverse proxy.

Также похожую идею используют:

* ORM lazy loading;
* API Gateway;
* caching proxy;
* authorization proxy;
* remote proxy.

### Главное

> **Proxy предоставляет тот же или совместимый интерфейс, контролируя доступ к реальному объекту.**

---

# Adapter vs Decorator vs Proxy

Очень важное сравнение.

| Паттерн       | Зачем оборачиваем объект? |
| ------------- | ------------------------- |
| **Adapter**   | Изменить интерфейс        |
| **Decorator** | Добавить поведение        |
| **Proxy**     | Контролировать доступ     |

```text
Adapter
  ↓
"Говори со мной на другом интерфейсе"


Decorator
  ↓
"Я добавлю тебе новую функциональность"


Proxy
  ↓
"Я решу, можно ли и как обращаться к тебе"
```

---

# Facade vs Proxy

Они тоже часто путаются.

### Facade

Упрощает **сложную подсистему**:

```text
Client
  ↓
Facade
  ↓
A + B + C + D
```

### Proxy

Представляет **конкретный объект**:

```text
Client
  ↓
Proxy
  ↓
Real Object
```

---

# Все 7 структурных паттернов

```text
              Структурные GoF
                    │
     ┌──────────────┼───────────────┐
     │              │               │
  Adapter         Bridge         Composite
     │              │               │
 интерфейс      разделение       дерево
                 абстракции
     │
 ┌───┴─────────────────────────────────┐
 │                                     │
Decorator                           Facade
 │                                     │
поведение                          упрощение
 │                                     │
 ├───────────────┐             ┌───────┘
 │               │             │
Flyweight       Proxy       подсистема
 │               │
память         доступ
```

---

# 🧠 Шпаргалка

```text
Adapter
→ несовместимые интерфейсы

Bridge
→ независимые абстракция и реализация

Composite
→ объект + группа объектов

Decorator
→ добавить поведение

Facade
→ упростить сложную подсистему

Flyweight
→ экономить память

Proxy
→ контролировать доступ
```

## Связь с Backend

```text
Adapter
→ интеграция внешнего API

Bridge
→ независимые уровни абстракций

Composite
→ деревья / структуры данных

Decorator
→ middleware / logging / caching

Facade
→ service layer / API facade

Flyweight
→ кэширование / экономия памяти

Proxy
→ Nginx / API Gateway / lazy loading
```

> **Ключевая мысль:** структурные паттерны отвечают не за то, **как создать объект**, а за то, **как организовать взаимодействие объектов и классов между собой**.
