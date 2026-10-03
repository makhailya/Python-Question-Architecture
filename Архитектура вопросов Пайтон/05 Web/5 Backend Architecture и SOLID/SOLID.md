# 🧱 SOLID — принципы ООП

## 🎯 Ответ на собеседовании

**SOLID** — это пять принципов проектирования программного обеспечения, которые помогают создавать код с низкой связанностью, высокой связностью, хорошей расширяемостью и удобной тестируемостью.

SOLID расшифровывается:

```text
S — Single Responsibility Principle
O — Open/Closed Principle
L — Liskov Substitution Principle
I — Interface Segregation Principle
D — Dependency Inversion Principle
```

Коротко:

```text
S → один класс — одна ответственность
O → открыт для расширения, закрыт для изменения
L → наследник должен корректно заменять родителя
I → лучше несколько маленьких интерфейсов, чем один большой
D → зависеть от абстракций, а не от конкретных реализаций
```

---

## 🎤 Суперкоротко

```text
S → одна ответственность
O → расширяй, не переписывай
L → наследник заменяет родителя
I → маленькие интерфейсы
D → зависимость от абстракций
```

---

# 🧩 Зачем нужен SOLID

Без принципов SOLID код постепенно превращается в:

```text
Один огромный класс
        ↓
много ответственности
        ↓
сильная связанность
        ↓
сложно тестировать
        ↓
сложно изменять
        ↓
любое изменение ломает другое
```

SOLID помогает разделять ответственность и уменьшать связанность компонентов.

---

# 🅢 S — Single Responsibility Principle

**Принцип единственной ответственности.**

> Класс должен иметь одну ответственность и одну причину для изменения.

Важно правильно понимать:

**SRP не означает, что в классе должен быть ровно один метод.**

Речь идёт именно об ответственности.

---

## ❌ Плохой пример

Один класс:

* работает с пользователем;
* сохраняет его в БД;
* отправляет email;
* генерирует отчёт.

```python
class User:
    def create_user(self):
        pass

    def save_to_database(self):
        pass

    def send_email(self):
        pass

    def generate_report(self):
        pass
```

У класса несколько причин для изменения:

```text
Изменилась БД
    ↓
меняем User

Изменился email
    ↓
меняем User

Изменился формат отчёта
    ↓
меняем User
```

---

## ✅ Хороший пример

Разделяем ответственности:

```python
class UserService:
    def create_user(self):
        pass


class UserRepository:
    def save(self, user):
        pass


class EmailService:
    def send(self, email):
        pass


class ReportGenerator:
    def generate(self, user):
        pass
```

Теперь каждый компонент отвечает за свою область.

```text
UserService
    ↓
бизнес-логика

UserRepository
    ↓
работа с БД

EmailService
    ↓
отправка email

ReportGenerator
    ↓
отчёты
```

---

## 🎯 Как сказать на собеседовании

**SRP означает, что класс должен иметь одну ответственность и одну причину для изменения. Это позволяет уменьшить связанность и сделать код проще для изменения и тестирования.**

---

# 🅞 O — Open/Closed Principle

**Принцип открытости/закрытости.**

> Программные сущности должны быть открыты для расширения, но закрыты для изменения.

То есть желательно добавлять новое поведение, не переписывая существующий код.

---

## ❌ Плохой пример

```python
class Payment:
    def pay(self, payment_type):
        if payment_type == "card":
            print("Pay by card")
        elif payment_type == "cash":
            print("Pay by cash")
        elif payment_type == "crypto":
            print("Pay by crypto")
```

Добавили новый способ оплаты:

```text
bank_transfer
```

Нужно изменять существующий класс.

---

## ✅ Лучше

Используем абстракцию и отдельные реализации:

```python
from abc import ABC, abstractmethod


class PaymentMethod(ABC):
    @abstractmethod
    def pay(self, amount):
        pass


class CardPayment(PaymentMethod):
    def pay(self, amount):
        print(f"Card: {amount}")


class CashPayment(PaymentMethod):
    def pay(self, amount):
        print(f"Cash: {amount}")


class CryptoPayment(PaymentMethod):
    def pay(self, amount):
        print(f"Crypto: {amount}")
```

Добавление нового способа:

```python
class BankTransferPayment(PaymentMethod):
    def pay(self, amount):
        print(f"Bank transfer: {amount}")
```

Существующие классы менять не пришлось.

---

## 🎯 Как сказать на собеседовании

**OCP означает, что систему следует проектировать так, чтобы новое поведение можно было добавлять через расширение, не изменяя уже работающий код.**

---

# 🅛 L — Liskov Substitution Principle

**Принцип подстановки Лисков.**

> Объекты подкласса должны быть способны заменить объекты базового класса без нарушения корректности программы.

Проще:

**Если `B` наследуется от `A`, то `B` должен нормально работать там, где ожидается `A`.**

---

## ❌ Классический пример

Есть базовый класс:

```python
class Bird:
    def fly(self):
        print("Flying")
```

Создаём:

```python
class Sparrow(Bird):
    def fly(self):
        print("Sparrow flies")
```

Нормально.

Но:

```python
class Penguin(Bird):
    def fly(self):
        raise NotImplementedError
```

Теперь:

```python
def make_bird_fly(bird: Bird):
    bird.fly()
```

Для `Sparrow` работает:

```python
make_bird_fly(Sparrow())
```

Для `Penguin`:

```python
make_bird_fly(Penguin())
```

получаем исключение.

Значит, модель наследования спроектирована неправильно.

---

## 🧠 Суть LSP

Нельзя использовать наследование только потому, что объекты похожи по смыслу.

Нужно, чтобы наследник сохранял контракт базового класса.

```text
Base
 ↓
Subclass
 ↓
можно использовать вместо Base
```

Если это невозможно — вероятно, нарушен LSP.

---

## 🎯 Как сказать на собеседовании

**LSP означает, что экземпляр дочернего класса должен корректно заменять экземпляр родительского класса, не нарушая ожидаемого поведения программы.**

---

# 🅘 I — Interface Segregation Principle

**Принцип разделения интерфейсов.**

> Клиенты не должны зависеть от методов, которые они не используют.

Лучше иметь несколько маленьких специализированных интерфейсов, чем один большой.

---

## ❌ Плохой пример

Представим огромный интерфейс:

```python
from abc import ABC, abstractmethod


class Worker(ABC):
    @abstractmethod
    def work(self):
        pass

    @abstractmethod
    def eat(self):
        pass

    @abstractmethod
    def sleep(self):
        pass
```

Теперь робот тоже должен реализовывать:

```python
class Robot(Worker):
    def work(self):
        pass

    def eat(self):
        pass

    def sleep(self):
        pass
```

Но робот не ест и не спит.

Интерфейс слишком большой.

---

## ✅ Лучше

Разделяем интерфейсы:

```python
from abc import ABC, abstractmethod


class Workable(ABC):
    @abstractmethod
    def work(self):
        pass


class Eatable(ABC):
    @abstractmethod
    def eat(self):
        pass


class Sleepable(ABC):
    @abstractmethod
    def sleep(self):
        pass
```

Теперь робот реализует только нужный интерфейс:

```python
class Robot(Workable):
    def work(self):
        pass
```

---

## 🎯 Как сказать на собеседовании

**ISP означает, что класс не должен быть вынужден зависеть от методов, которые ему не нужны. Поэтому большие интерфейсы лучше разделять на небольшие специализированные интерфейсы.**

---

# 🅓 D — Dependency Inversion Principle

**Принцип инверсии зависимостей.**

> Модули высокого уровня не должны зависеть от модулей низкого уровня. Оба должны зависеть от абстракций.

И:

> Абстракции не должны зависеть от деталей. Детали должны зависеть от абстракций.

Это самый сложный принцип SOLID.

---

# ❌ Плохой пример

```python
class PostgreSQLDatabase:
    def save(self, user):
        pass


class UserService:
    def __init__(self):
        self.database = PostgreSQLDatabase()

    def save_user(self, user):
        self.database.save(user)
```

`UserService` напрямую зависит от конкретной БД:

```text
UserService
     ↓
PostgreSQLDatabase
```

Если понадобится:

```text
MongoDB
SQLite
MockDatabase
```

придётся менять `UserService`.

---

# ✅ Dependency Inversion

Создаём абстракцию:

```python
from abc import ABC, abstractmethod


class UserRepository(ABC):
    @abstractmethod
    def save(self, user):
        pass
```

Реализация:

```python
class PostgreSQLRepository(UserRepository):
    def save(self, user):
        print("Save to PostgreSQL")
```

Сервис зависит от абстракции:

```python
class UserService:
    def __init__(self, repository: UserRepository):
        self.repository = repository

    def save_user(self, user):
        self.repository.save(user)
```

Теперь можно передать любую реализацию:

```python
repository = PostgreSQLRepository()
service = UserService(repository)
```

---

# 💉 Dependency Injection

На практике DIP часто реализуется через **Dependency Injection (DI)**.

Зависимость передаётся объекту извне:

```python
class UserService:
    def __init__(self, repository):
        self.repository = repository
```

Вместо:

```python
class UserService:
    def __init__(self):
        self.repository = PostgreSQLRepository()
```

Схема:

```text
❌ Было:

UserService
     ↓
создаёт PostgreSQLRepository


✅ Стало:

PostgreSQLRepository
        ↓
    UserService
```

Зависимость приходит извне.

---

# 🆚 DIP и DI

Их часто путают.

**DIP** — принцип проектирования.

**DI** — способ реализации, при котором зависимости передаются извне.

```text
DIP
 ↓
Зависеть от абстракций

DI
 ↓
Передавать зависимости извне
```

---

# 🧩 SOLID целиком

```text
S — Single Responsibility
    ↓
Одна ответственность

O — Open/Closed
    ↓
Расширяй, не изменяй

L — Liskov Substitution
    ↓
Наследник заменяет родителя

I — Interface Segregation
    ↓
Маленькие специализированные интерфейсы

D — Dependency Inversion
    ↓
Зависимость от абстракций
```

---

# 🔗 Как принципы связаны между собой

SOLID — не пять полностью независимых правил.

Например:

```text
SRP
 ↓
разделяем ответственности
 ↓
получаем отдельные компоненты
 ↓
DIP
 ↓
связываем их через абстракции
 ↓
OCP
 ↓
добавляем новые реализации
```

А LSP и ISP помогают правильно проектировать эти абстракции.

---

# 🐍 SOLID в Python

В Python SOLID часто реализуется через:

```text
ABC
abstractmethod
Protocol
composition
dependency injection
polymorphism
duck typing
```

Например, для DIP вместо обязательного наследования от `ABC` можно использовать `Protocol`:

```python
from typing import Protocol


class Repository(Protocol):
    def save(self, user):
        ...


class UserService:
    def __init__(self, repository: Repository):
        self.repository = repository
```

Python позволяет использовать **структурную типизацию**, поэтому конкретный класс может подходить под `Protocol`, если имеет необходимый интерфейс.

---

# ⚠️ SOLID ≠ закон

SOLID — это **принципы проектирования**, а не строгие правила, которые нужно применять всегда.

Не стоит создавать:

```text
1 класс
↓
1 интерфейс
↓
1 абстракция
↓
1 фабрика
↓
1 стратегия
```

для каждой маленькой функции.

Избыточное применение SOLID может привести к:

```text
слишком много абстракций
        ↓
сложный код
        ↓
сложно понимать
```

Поэтому принципы применяют там, где они действительно уменьшают сложность и связанность.

---

# 🎯 Что могут попросить на собеседовании

Самые важные вопросы:

**Что такое SOLID?**

→ Пять принципов проектирования ПО.

**Что означает SRP?**

→ Одна ответственность и одна причина для изменения.

**Что означает OCP?**

→ Открыт для расширения, закрыт для изменения.

**Что означает LSP?**

→ Наследник должен корректно заменять родителя.

**Что означает ISP?**

→ Не заставлять класс зависеть от ненужных методов.

**Что означает DIP?**

→ Зависеть от абстракций, а не от конкретных реализаций.

**Чем DIP отличается от DI?**

→ DIP — принцип, DI — механизм/паттерн передачи зависимостей.

---

# 🧠 Финальная шпаргалка

```text
S → Single Responsibility
    «Одна ответственность»

O → Open/Closed
    «Расширяй, не изменяй»

L → Liskov Substitution
    «Наследник заменяет родителя»

I → Interface Segregation
    «Не заставляй реализовывать лишнее»

D → Dependency Inversion
    «Зависеть от абстракций»
```

**SOLID → меньше связанности → проще изменять → проще тестировать → проще расширять.**
