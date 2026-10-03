# 🧩 Паттерны GoF

## 🎯 Ответ на собеседовании

**GoF (Gang of Four)** — классические шаблоны проектирования, описанные в книге *Design Patterns: Elements of Reusable Object-Oriented Software*.

Всего выделяют **23 паттерна**, разделённых на 3 группы:

```text id="q7m2kx"
GoF — 23 паттерна
│
├── Порождающие — 5
│
├── Структурные — 7
│
└── Поведенческие — 11
```

Они описывают типовые решения проблем проектирования объектов и их взаимодействия.

> **Паттерн — не готовый кусок кода, а типовое решение архитектурной проблемы.**

---

## 🎤 Суперкоротко

> **GoF — 23 классических паттерна проектирования: 5 порождающих, 7 структурных и 11 поведенческих.**

```text id="w5v9rc"
Порождающие
→ создание объектов

Структурные
→ композиция объектов и классов

Поведенческие
→ взаимодействие и распределение ответственности
```

---

# 🏗️ 1. [[Порождающие паттерны GoF]]

Отвечают на вопрос:

> **Как создавать объекты?**

Всего **5**.

| Паттерн          | Идея                                 |
| ---------------- | ------------------------------------ |
| Singleton        | Один экземпляр                       |
| Factory Method   | Создание через фабричный метод       |
| Abstract Factory | Семейство связанных объектов         |
| Builder          | Пошаговое создание сложного объекта  |
| Prototype        | Создание копии существующего объекта |

---

## 🔹 Singleton

Гарантирует наличие одного экземпляра класса и предоставляет к нему общий доступ.

```python
class Singleton:
    _instance = None

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance
```

```text id="7s3mqp"
Singleton()
Singleton()
   ↓
один объект
```

⚠️ В Python Singleton часто не нужен: модуль сам по себе кэшируется и фактически выступает как singleton-like механизм.

---

## 🔹 Factory Method

Позволяет создавать объекты через отдельный метод, не привязываясь к конкретному классу объекта.

```python
class NotificationFactory:
    def create(self, notification_type):
        if notification_type == "email":
            return EmailNotification()

        if notification_type == "sms":
            return SMSNotification()
```

```text id="k3v8mx"
Factory
   ↓
┌──────────────┐
│ Email        │
│ SMS          │
│ Push         │
└──────────────┘
```

Полезен, когда конкретный тип объекта определяется во время выполнения.

---

## 🔹 Abstract Factory

Создаёт **семейства связанных объектов**.

Например:

```text id="r6n2wp"
WindowsFactory
 ├── WindowsButton
 └── WindowsCheckbox

LinuxFactory
 ├── LinuxButton
 └── LinuxCheckbox
```

В отличие от Factory Method, здесь фабрика обычно отвечает за создание нескольких связанных продуктов.

---

## 🔹 Builder

Используется для пошагового создания сложного объекта.

```python
user = (
    UserBuilder()
    .set_name("Ilya")
    .set_email("test@example.com")
    .set_age(31)
    .build()
)
```

```text id="j4k7vq"
Builder
  ↓
name
  ↓
email
  ↓
age
  ↓
build()
  ↓
Object
```

Особенно полезен, когда объект имеет много параметров или вариантов конфигурации.

---

## 🔹 Prototype

Создание нового объекта на основе копирования существующего.

```python
from copy import deepcopy

new_object = deepcopy(existing_object)
```

```text id="q8x2mv"
Existing Object
      ↓
    copy
      ↓
New Object
```

Полезен, когда создание объекта с нуля дорого или сложно.

---

# 🧱 2. [[Структурные паттерны GoF]]

Отвечают на вопрос:

> **Как объединять классы и объекты?**

Всего **7**.

| Паттерн   | Идея                                |
| --------- | ----------------------------------- |
| Adapter   | Совместить несовместимые интерфейсы |
| Bridge    | Разделить абстракцию и реализацию   |
| Composite | Представить дерево объектов         |
| Decorator | Динамически добавить поведение      |
| Facade    | Упростить сложную подсистему        |
| Flyweight | Разделять общие данные              |
| Proxy     | Объект-заместитель                  |

---

## 🔹 Adapter

Позволяет объектам с несовместимыми интерфейсами работать вместе.

```text id="s5k3md"
Client
  ↓
Adapter
  ↓
Legacy Service
```

Например, старый сервис:

```python
legacy.send_message(text)
```

а новый код ожидает:

```python
client.send(text)
```

Adapter преобразует один интерфейс в другой.

---

## 🔹 Bridge

Разделяет **абстракцию** и **реализацию**, чтобы их можно было изменять независимо.

```text id="x7n4pq"
Abstraction
     ↓
Implementation
```

Например:

```text id="v8m2zc"
Notification
   ├── Email
   └── SMS

Sender
   ├── SMTP
   └── API
```

---

## 🔹 Composite

Позволяет работать с отдельными объектами и группами объектов одинаковым образом.

Классический пример — дерево файлов:

```text id="f3q8nv"
Folder
├── File
├── File
└── Folder
    ├── File
    └── File
```

И `File`, и `Folder` реализуют общий интерфейс.

---

## 🔹 Decorator

Добавляет объекту поведение без изменения его исходного класса.

```python
@cache
@log
def get_user():
    ...
```

```text id="n5w7xr"
Function
   ↓
Logging
   ↓
Caching
   ↓
Function
```

Очень важный паттерн для Python, поскольку язык поддерживает декораторы напрямую.

---

## 🔹 Facade

Предоставляет простой интерфейс к сложной подсистеме.

Например:

```python
order_service.create_order()
```

внутри:

```text id="c4m9vz"
create_order()
   ↓
validate_user()
   ↓
reserve_inventory()
   ↓
create_payment()
   ↓
send_notification()
```

Клиенту не нужно знать детали всех подсистем.

---

## 🔹 Flyweight

Позволяет экономить память, разделяя общие объекты или данные.

```text id="m7p2xq"
Object A ──┐
Object B ──┼──→ Shared Data
Object C ──┘
```

Полезен при огромном количестве похожих объектов.

---

## 🔹 Proxy

Объект-заместитель, который контролирует доступ к другому объекту.

```text id="r9k4wm"
Client
  ↓
Proxy
  ↓
Real Object
```

Proxy может добавить:

* авторизацию;
* кеширование;
* lazy loading;
* логирование;
* контроль доступа.

---

# 🔄 3. [[Поведенческие паттерны GoF]]

Отвечают на вопрос:

> **Как объекты взаимодействуют между собой?**

Всего **11**.

| Паттерн                 | Идея                             |
| ----------------------- | -------------------------------- |
| Chain of Responsibility | Цепочка обработчиков             |
| Command                 | Представить действие как объект  |
| Interpreter             | Интерпретация языка/грамматики   |
| Iterator                | Последовательный обход коллекции |
| Mediator                | Центральный посредник            |
| Memento                 | Сохранение состояния             |
| Observer                | Уведомление подписчиков          |
| State                   | Поведение зависит от состояния   |
| Strategy                | Взаимозаменяемые алгоритмы       |
| Template Method         | Скелет алгоритма                 |
| Visitor                 | Операции над структурой объектов |

---

## 🔹 Chain of Responsibility

Запрос проходит через цепочку обработчиков:

```text id="e6v3kp"
Request
  ↓
Handler A
  ↓
Handler B
  ↓
Handler C
```

Каждый обработчик либо обрабатывает запрос, либо передаёт дальше.

Пример:

```text id="t8m4qn"
Authentication
      ↓
Authorization
      ↓
Validation
      ↓
Business Logic
```

---

## 🔹 Command

Представляет действие как отдельный объект.

```text id="q2v7mx"
Command
  ↓
execute()
  ↓
Action
```

Например:

```python
class CreateUserCommand:
    def execute(self):
        ...
```

Позволяет:

* ставить команды в очередь;
* логировать;
* повторять;
* откладывать выполнение;
* реализовывать undo.

---

## 🔹 Interpreter

Определяет представление грамматики и интерпретирует выражения этого языка.

Применяется для простых DSL и языков выражений.

Например:

```text id="w9c3kr"
age > 18 AND active = true
```

---

## 🔹 Iterator

Позволяет последовательно обходить коллекцию, не раскрывая её внутреннее устройство.

В Python это встроено в сам язык:

```python
for user in users:
    print(user)
```

Под капотом используются:

```python
iter()
next()
```

---

## 🔹 Mediator

Объекты не взаимодействуют напрямую, а общаются через посредника.

```text id="k6p2vz"
Service A ──┐
Service B ──┼──→ Mediator
Service C ──┘
```

Цель — уменьшить количество прямых зависимостей между объектами.

---

## 🔹 Memento

Позволяет сохранить состояние объекта и позже восстановить его.

```text id="d5r8mx"
Object
  ↓
Memento
  ↓
State snapshot
```

Классический пример:

```text id="p7q3nv"
Редактор
  ↓
изменение
  ↓
snapshot
  ↓
изменение
  ↓
Undo
```

---

## 🔹 Observer

Один объект уведомляет множество подписчиков об изменении состояния.

```text id="a8m4qx"
Subject
  │
  ├──→ Observer A
  ├──→ Observer B
  └──→ Observer C
```

Пример:

```text id="x3k7mz"
Order created
     ↓
 ┌───┴────┐
 ↓        ↓
Email   Analytics
```

---

## 🔹 State

Поведение объекта изменяется в зависимости от его состояния.

Например, заказ:

```text id="m5v9qc"
Created
  ↓
Paid
  ↓
Shipped
  ↓
Delivered
```

В разных состояниях допустимы разные операции.

---

## 🔹 Strategy

Позволяет менять алгоритм независимо от клиента.

```text id="r4k8xp"
PaymentService
      ↓
 ┌────┼────┐
 ↓    ↓    ↓
Card Cash Crypto
```

Например:

```python
class PaymentService:
    def __init__(self, strategy):
        self.strategy = strategy

    def pay(self, amount):
        return self.strategy.pay(amount)
```

Strategy особенно хорошо сочетается с **Dependency Injection**.

---

## 🔹 Template Method

Определяет общий алгоритм, оставляя отдельные шаги подклассам.

```text id="q6m3vz"
process()
  ↓
validate()
  ↓
execute()
  ↓
save()
```

Общий алгоритм фиксирован, конкретные шаги могут переопределяться.

---

## 🔹 Visitor

Позволяет добавлять новые операции над объектами структуры, не изменяя сами классы элементов.

```text id="u7p2mx"
Structure
 ├── Element A
 ├── Element B
 └── Element C
        ↑
     Visitor
```

Часто встречается в задачах с AST, компиляторами и сложными структурами объектов.

---

# 📊 Все 23 паттерна

## 🏗️ Порождающие — 5

```text id="j8v4qx"
1. Singleton
2. Factory Method
3. Abstract Factory
4. Builder
5. Prototype
```

## 🧱 Структурные — 7

```text id="n3m7kc"
6. Adapter
7. Bridge
8. Composite
9. Decorator
10. Facade
11. Flyweight
12. Proxy
```

## 🔄 Поведенческие — 11

```text id="r5x9vm"
13. Chain of Responsibility
14. Command
15. Interpreter
16. Iterator
17. Mediator
18. Memento
19. Observer
20. State
21. Strategy
22. Template Method
23. Visitor
```

---

# 🐍 Какие особенно важны для Python Backend

Для Junior Python разработчика я бы особенно хорошо понимал:

| Паттерн                     | Почему важен                      |
| --------------------------- | --------------------------------- |
| **Factory**                 | Создание объектов                 |
| **Builder**                 | Сложная конфигурация объектов     |
| **Adapter**                 | Интеграция разных API             |
| **Decorator**               | Очень распространён в Python      |
| **Facade**                  | Service Layer / упрощение API     |
| **Proxy**                   | Кэширование, lazy loading, доступ |
| **Observer**                | Events / callbacks                |
| **Strategy**                | Замена алгоритмов                 |
| **Chain of Responsibility** | Middleware                        |
| **Command**                 | Очереди и команды                 |
| **State**                   | Статусы сущностей                 |

---

# ⚠️ Паттерн ≠ обязательная архитектура

Не нужно применять GoF-паттерны просто ради использования паттернов.

Плохо:

```text id="c7m2vz"
Простая задача
     ↓
5 классов
     ↓
3 фабрики
     ↓
2 интерфейса
     ↓
сложность
```

Хороший принцип:

> **Сначала проблема → потом подходящее решение → если нужно, паттерн.**

---

# 🔗 GoF и SOLID

Паттерны часто помогают реализовывать принципы **SOLID**.

Например:

```text id="m4q8xn"
Strategy
   ↓
Open/Closed
Dependency Inversion
```

```text id="v7k3pz"
Decorator
   ↓
Open/Closed
Single Responsibility
```

```text id="x5n9qm"
Factory
   ↓
Dependency Inversion
Single Responsibility
```

Но паттерн сам по себе **не гарантирует соблюдение SOLID**.

---

# 🧠 Главное

```text id="p8m4vx"
GoF
│
├── Creational — 5
│   ├── Singleton
│   ├── Factory Method
│   ├── Abstract Factory
│   ├── Builder
│   └── Prototype
│
├── Structural — 7
│   ├── Adapter
│   ├── Bridge
│   ├── Composite
│   ├── Decorator
│   ├── Facade
│   ├── Flyweight
│   └── Proxy
│
└── Behavioral — 11
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

### Формула для собеседования

> **GoF — это 23 классических паттерна проектирования, разделённых на порождающие, структурные и поведенческие. Порождающие отвечают за создание объектов, структурные — за их композицию, поведенческие — за взаимодействие и распределение ответственности. Паттерн является типовым решением проблемы проектирования, а не обязательной конструкцией кода.**
