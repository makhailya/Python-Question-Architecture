# Поведенческие паттерны GoF 🧠

## 🎯 Ответ на собеседовании

**Поведенческие паттерны GoF** описывают взаимодействие объектов, передачу ответственности между ними и организацию поведения системы.

В GoF есть **11 поведенческих паттернов**:

1. **Chain of Responsibility** — передача запроса по цепочке обработчиков.
2. **Command** — представление действия в виде объекта.
3. **Interpreter** — интерпретация языка или выражений.
4. **Iterator** — последовательный обход коллекции.
5. **Mediator** — централизованное взаимодействие объектов.
6. **Memento** — сохранение и восстановление состояния объекта.
7. **Observer** — уведомление зависимых объектов об изменениях.
8. **State** — изменение поведения объекта в зависимости от его состояния.
9. **Strategy** — взаимозаменяемые алгоритмы.
10. **Template Method** — общий алгоритм с переопределяемыми шагами.
11. **Visitor** — добавление операций над объектами без изменения их классов.

Главная идея:

```text
Поведенческие паттерны
        ↓
Как объекты взаимодействуют?
        ↓
Кто отвечает за действие?
        ↓
Как изменить поведение без сильной связанности?
```

---

## 🎤 Суперкоротко

| Паттерн                     | Идея                                        |
| --------------------------- | ------------------------------------------- |
| **Chain of Responsibility** | Передать запрос следующему обработчику      |
| **Command**                 | Инкапсулировать действие в объект           |
| **Interpreter**             | Интерпретировать выражения                  |
| **Iterator**                | Последовательно обходить коллекцию          |
| **Mediator**                | Общаться через посредника                   |
| **Memento**                 | Сохранить состояние и восстановить его      |
| **Observer**                | Уведомлять подписчиков об изменениях        |
| **State**                   | Менять поведение в зависимости от состояния |
| **Strategy**                | Подменять алгоритм                          |
| **Template Method**         | Задать скелет алгоритма                     |
| **Visitor**                 | Добавлять операции к структуре объектов     |

---

# 1. Chain of Responsibility ⛓️

**Chain of Responsibility** передаёт запрос по цепочке обработчиков, пока один из них его не обработает.

```text
Request
   ↓
Handler 1
   ↓
Handler 2
   ↓
Handler 3
   ↓
Result
```

Пример:

```python
class Handler:
    def __init__(self, next_handler=None):
        self.next_handler = next_handler

    def handle(self, request):
        if self.next_handler:
            return self.next_handler.handle(request)

        return None


class AuthHandler(Handler):
    def handle(self, request):
        if not request.get("user"):
            return "Unauthorized"

        return super().handle(request)


class ValidationHandler(Handler):
    def handle(self, request):
        if not request.get("data"):
            return "Invalid data"

        return super().handle(request)
```

Цепочка:

```python
handler = AuthHandler(
    ValidationHandler()
)

result = handler.handle(request)
```

### Backend-примеры

* middleware;
* обработка HTTP-запросов;
* цепочки валидаторов;
* обработчики ошибок;
* pipeline обработки данных.

### Главное

> **Chain of Responsibility позволяет передавать запрос между последовательностью обработчиков, не связывая отправителя с конкретным обработчиком.**

---

# 2. Command 🎯

**Command** превращает действие в отдельный объект.

Вместо:

```python
service.create_user(user)
```

можно представить действие как объект:

```python
class CreateUserCommand:
    def __init__(self, service, user):
        self.service = service
        self.user = user

    def execute(self):
        return self.service.create_user(self.user)
```

Теперь:

```python
command = CreateUserCommand(service, user)
command.execute()
```

Схема:

```text
Invoker
   ↓
Command
   ↓
Receiver
```

### Зачем

Команду можно:

* сохранить;
* поставить в очередь;
* повторить;
* логировать;
* отменить;
* выполнить позже.

### Backend-примеры

```text
HTTP request
    ↓
Command
    ↓
Service
```

или:

```text
API
 ↓
Command
 ↓
Celery
 ↓
Worker
```

### Главное

> **Command инкапсулирует запрос или действие как объект.**

---

# 3. Interpreter 🗣️

**Interpreter** используется для интерпретации выражений некоторого языка.

Например, есть простой язык:

```text
"age > 18"
```

Можно представить выражение объектами:

```python
class GreaterThan:
    def __init__(self, field, value):
        self.field = field
        self.value = value

    def interpret(self, context):
        return context[self.field] > self.value
```

Использование:

```python
expression = GreaterThan("age", 18)

result = expression.interpret({"age": 25})
```

Результат:

```text
True
```

### Где применяется

* DSL;
* парсеры;
* правила;
* выражения фильтрации;
* простые языки запросов.

### Важно

В реальном backend чаще используют готовые парсеры, AST и специализированные библиотеки.

### Главное

> **Interpreter представляет грамматику языка объектами и выполняет их интерпретацию.**

---

# 4. Iterator 🔄

**Iterator** предоставляет последовательный доступ к элементам коллекции, не раскрывая её внутреннюю структуру.

В Python паттерн Iterator встроен в сам язык.

Например:

```python
numbers = [1, 2, 3]

for number in numbers:
    print(number)
```

Под капотом используется протокол:

```python
__iter__()
__next__()
```

Пример собственного итератора:

```python
class Counter:
    def __init__(self, limit):
        self.current = 0
        self.limit = limit

    def __iter__(self):
        return self

    def __next__(self):
        if self.current >= self.limit:
            raise StopIteration

        self.current += 1
        return self.current
```

Использование:

```python
for number in Counter(3):
    print(number)
```

Результат:

```text
1
2
3
```

### Главное

> **Iterator позволяет обходить коллекцию единообразным способом, скрывая детали её внутреннего устройства.**

В Python особенно важно знать:

```text
iter()
 ↓
__iter__()
 ↓
next()
 ↓
__next__()
```

---

# 5. Mediator 🤝

**Mediator** переносит логику взаимодействия объектов в отдельный объект-посредник.

Без Mediator:

```text
A ↔ B
A ↔ C
A ↔ D
B ↔ C
B ↔ D
C ↔ D
```

Получается много связей.

С Mediator:

```text
      A
      ↓
B → Mediator ← C
      ↑
      D
```

Объекты взаимодействуют через посредника.

Пример:

```python
class ChatMediator:
    def send(self, message, sender):
        for user in self.users:
            if user != sender:
                user.receive(message)

    def add_user(self, user):
        self.users.append(user)
```

### Backend-примеры

* event bus;
* message broker;
* orchestration layer;
* координация нескольких компонентов.

### Главное

> **Mediator уменьшает количество прямых связей между объектами, централизуя их взаимодействие.**

---

# 6. Memento 💾

**Memento** позволяет сохранить состояние объекта и позже восстановить его.

Классический пример:

```text
Редактор
   ↓
Memento
   ↓
Предыдущее состояние
```

Например:

```python
class Editor:
    def __init__(self):
        self.text = ""

    def save(self):
        return self.text

    def restore(self, state):
        self.text = state
```

Использование:

```python
editor = Editor()

editor.text = "Hello"

state = editor.save()

editor.text = "Hello World"

editor.restore(state)

print(editor.text)
# Hello
```

### Где применяется

* undo/redo;
* сохранение состояния;
* snapshots;
* откат состояния.

### Главное

> **Memento сохраняет состояние объекта, чтобы его можно было восстановить позже, не раскрывая детали внутреннего состояния.**

---

# 7. Observer 👀

**Observer** создаёт зависимость «один ко многим».

Когда объект изменяется, он уведомляет своих подписчиков.

```text
             Subject
            /   |   \
           ↓    ↓    ↓
       Observer Observer Observer
```

Пример:

```python
class Subject:
    def __init__(self):
        self.observers = []

    def subscribe(self, observer):
        self.observers.append(observer)

    def notify(self, event):
        for observer in self.observers:
            observer.update(event)
```

Observer:

```python
class EmailObserver:
    def update(self, event):
        print(f"Email: {event}")
```

Использование:

```python
subject = Subject()

subject.subscribe(EmailObserver())

subject.notify("Order created")
```

### Backend-примеры

* event-driven architecture;
* pub/sub;
* уведомления;
* события домена;
* Kafka/RabbitMQ;
* hooks/signals.

### Главное

> **Observer позволяет одному объекту уведомлять множество зависимых объектов об изменениях.**

---

# 8. State 🔀

**State** позволяет объекту изменять своё поведение в зависимости от текущего состояния.

Например, заказ:

```text
NEW
 ↓
PAID
 ↓
SHIPPED
 ↓
DELIVERED
```

Поведение заказа зависит от состояния.

```python
class Order:
    def __init__(self, state):
        self.state = state

    def pay(self):
        self.state.pay(self)
```

Состояния:

```python
class NewState:
    def pay(self, order):
        print("Payment accepted")
        order.state = PaidState()


class PaidState:
    def pay(self, order):
        print("Already paid")
```

Теперь поведение определяется текущим State.

### Где применяется

* workflow;
* state machine;
* статусы заказа;
* состояния платежа;
* состояния подключения.

### Главное

> **State позволяет менять поведение объекта при изменении его внутреннего состояния.**

---

# 9. Strategy 🎯

**Strategy** позволяет использовать разные алгоритмы через единый интерфейс.

Например, разные способы расчёта доставки:

```text
Delivery
   ├── CourierStrategy
   ├── PickupStrategy
   └── ExpressStrategy
```

Пример:

```python
class CourierDelivery:
    def calculate(self, order):
        return 500


class PickupDelivery:
    def calculate(self, order):
        return 0


class Order:
    def __init__(self, delivery_strategy):
        self.delivery_strategy = delivery_strategy

    def delivery_price(self):
        return self.delivery_strategy.calculate(self)
```

Использование:

```python
order = Order(CourierDelivery())

price = order.delivery_price()
```

Можно заменить алгоритм:

```python
order.delivery_strategy = PickupDelivery()
```

### Backend-примеры

* разные способы оплаты;
* алгоритмы расчёта цены;
* сортировка;
* авторизация;
* выбор способа доставки;
* разные алгоритмы обработки данных.

### Strategy vs State

Очень частый вопрос.

**Strategy:**

> Мы выбираем **какой алгоритм использовать**.

**State:**

> Объект находится в **каком-то состоянии**, которое определяет его поведение.

```text
Strategy → "Как делать?"
State    → "В каком состоянии?"
```

### Главное

> **Strategy инкапсулирует взаимозаменяемые алгоритмы и позволяет менять их независимо от клиента.**

---

# 10. Template Method 📋

**Template Method** определяет общий скелет алгоритма, оставляя отдельные шаги подклассам.

Например:

```python
class DataProcessor:
    def process(self):
        self.load()
        self.transform()
        self.save()

    def load(self):
        raise NotImplementedError

    def transform(self):
        raise NotImplementedError

    def save(self):
        raise NotImplementedError
```

Подкласс:

```python
class CSVProcessor(DataProcessor):
    def load(self):
        print("Load CSV")

    def transform(self):
        print("Transform CSV")

    def save(self):
        print("Save CSV")
```

Общий алгоритм:

```text
process()
   ↓
load()
   ↓
transform()
   ↓
save()
```

### Backend-пример

Очень хорошо подходит для ETL:

```text
Extract
  ↓
Transform
  ↓
Load
```

Сам алгоритм общий, а конкретные шаги могут различаться.

### Главное

> **Template Method задаёт скелет алгоритма в базовом классе, а отдельные шаги реализуются или переопределяются наследниками.**

---

# 11. Visitor 🚶

**Visitor** позволяет добавлять новые операции над объектами, не изменяя сами классы этих объектов.

Представим AST:

```text
Expression
├── Number
├── Add
└── Multiply
```

Нам нужно выполнять разные операции:

```text
AST
├── CalculateVisitor
├── PrintVisitor
└── ValidateVisitor
```

То есть структура объектов остаётся прежней, а новые операции добавляются через Visitor.

Пример идеи:

```python
class Number:
    def accept(self, visitor):
        return visitor.visit_number(self)


class PrintVisitor:
    def visit_number(self, number):
        return str(number.value)
```

### Где применяется

* AST;
* компиляторы;
* анализаторы кода;
* сложные структуры объектов.

### Главное

> **Visitor позволяет добавлять новые операции над существующей структурой объектов без изменения классов этой структуры.**

---

# Strategy vs Template Method

Оба паттерна связаны с алгоритмами, но механизм разный.

|          | Strategy                   | Template Method          |
| -------- | -------------------------- | ------------------------ |
| Механизм | Композиция                 | Наследование             |
| Алгоритм | Передаётся объект          | Задаётся базовым классом |
| Замена   | Обычно во время выполнения | Через подкласс           |
| Гибкость | Выше                       | Ниже                     |

```text
Strategy

Service
   ↓
Strategy
   ↓
Algorithm


Template Method

BaseClass
   ↓
Template Method
   ↓
Subclass steps
```

---

# Observer vs Mediator

### Observer

Объект сообщает подписчикам:

```text
Subject
 ↓ ↓ ↓
Observers
```

### Mediator

Объекты общаются через посредника:

```text
A ─┐
B ─┼→ Mediator
C ─┘
```

**Observer** — про уведомления.

**Mediator** — про централизованное взаимодействие.

---

# State vs Strategy

```text
Strategy
→ выбор алгоритма

State
→ изменение поведения из-за состояния
```

Пример:

```text
Strategy:
"Оплачивать через Stripe или YooKassa?"

State:
"Заказ NEW, PAID или SHIPPED?"
```

---

# Все 11 поведенческих паттернов

```text
                 Поведенческие GoF
                         │
 ┌───────────┬───────────┼───────────┬────────────┐
 │           │           │           │            │
Chain      Command    Interpreter  Iterator    Mediator
 │           │           │           │            │
цепочка    действие    язык        обход       посредник
 │
 ├───────────┬───────────┬───────────┬────────────┐
 │           │           │           │            │
Memento   Observer      State      Strategy   Template Method
 │           │           │           │            │
состояние   события    состояние   алгоритм     алгоритм
 │
 └───────────────────────────────┐
                                 ↓
                              Visitor
                              операции
```

---

# 🐍 Что особенно важно для Python Backend

Из 11 паттернов наиболее практичны:

### ⭐ Chain of Responsibility

```text
Middleware
Validators
Request processing
```

### ⭐ Command

```text
Task
Queue
Actions
CQRS
```

### ⭐ Iterator

```text
iter()
next()
generators
```

### ⭐ Observer

```text
Events
Signals
Pub/Sub
Kafka
RabbitMQ
```

### ⭐ Strategy

```text
Разные алгоритмы
Payment
Delivery
Pricing
Authorization
```

### ⭐ State

```text
Order workflow
Payment status
State machine
```

### ⭐ Template Method

```text
ETL
Общий pipeline
Базовые сервисы
```

---

# 🧠 Главная шпаргалка

```text
Chain of Responsibility
→ передать запрос по цепочке

Command
→ действие как объект

Interpreter
→ интерпретировать язык

Iterator
→ обходить коллекцию

Mediator
→ взаимодействовать через посредника

Memento
→ сохранить состояние

Observer
→ уведомить подписчиков

State
→ поведение зависит от состояния

Strategy
→ заменить алгоритм

Template Method
→ общий скелет алгоритма

Visitor
→ новая операция над существующей структурой
```

## Все GoF в одной картине

```text
GOF — 23 паттерна
│
├── Порождающие — 5
│   ├── Singleton
│   ├── Factory Method
│   ├── Abstract Factory
│   ├── Builder
│   └── Prototype
│
├── Структурные — 7
│   ├── Adapter
│   ├── Bridge
│   ├── Composite
│   ├── Decorator
│   ├── Facade
│   ├── Flyweight
│   └── Proxy
│
└── Поведенческие — 11
    ├── Chain of Responsibility
    ├── Command
    ├── Interpreter
    ├── Iterator
    ├── Mediator
    ├── Memento
    ├── Observer
    ├── State
    ├── Strategy
    ├── Template Method
    └── Visitor
```

> **Порождающие** → как создавать объекты.
> **Структурные** → как соединять объекты.
> **Поведенческие** → как организовать их взаимодействие и поведение.
