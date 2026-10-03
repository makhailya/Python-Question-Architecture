# ABC — Abstract Base Class

## 🎯 Ответ на собеседовании

**ABC (Abstract Base Class)** — это абстрактный базовый класс, который задаёт **общий интерфейс для дочерних классов** и может содержать абстрактные методы, реализацию которых обязаны предоставить наследники.

В Python ABC реализуется через модуль `abc`:

```python id="3g8q2w"
from abc import ABC, abstractmethod
```

---

## 📌 Простой пример

```python id="n8v4la"
from abc import ABC, abstractmethod


class Animal(ABC):

    @abstractmethod
    def speak(self):
        pass


class Dog(Animal):
    def speak(self):
        print("Гав")
```

Теперь:

```python id="c6j1px"
dog = Dog()
dog.speak()
```

работает:

```text id="4j6z8a"
Гав
```

А создать `Animal` напрямую нельзя:

```python id="w5f7mx"
animal = Animal()
```

Будет:

```text id="g0z3qk"
TypeError
```

потому что `Animal` содержит нереализованный абстрактный метод.

---

## 🔑 `@abstractmethod`

Декоратор:

```python id="7v5j2c"
@abstractmethod
```

помечает метод как **абстрактный**.

```python id="x8k1nd"
class Payment(ABC):

    @abstractmethod
    def pay(self, amount):
        pass
```

Он говорит:

> «Каждый конкретный класс-наследник должен предоставить реализацию `pay()`».

Например:

```python id="q2p6rt"
class CardPayment(Payment):
    def pay(self, amount):
        print(f"Карта: {amount}")


class CashPayment(Payment):
    def pay(self, amount):
        print(f"Наличные: {amount}")
```

---

## 🚫 Что будет, если не реализовать метод?

```python id="6m4f8x"
class CardPayment(Payment):
    pass
```

Такой класс останется **абстрактным**.

```python id="8q7n2m"
CardPayment()
```

вызовет:

```text id="0a6v3k"
TypeError
```

---

## 🧩 ABC может содержать обычные методы

Абстрактный класс не обязан состоять только из абстрактных методов.

```python id="s3d9kf"
class Animal(ABC):

    @abstractmethod
    def speak(self):
        pass

    def sleep(self):
        print("Спит")
```

Наследник обязан реализовать `speak()`, но автоматически получает `sleep()`.

---

## 💡 Абстрактный метод может иметь реализацию

Это важный нюанс Python.

```python id="q7m2az"
class Animal(ABC):

    @abstractmethod
    def speak(self):
        print("Издаёт звук")
```

Метод одновременно является **абстрактным** и содержит реализацию.

Наследник всё равно должен его переопределить, чтобы стать конкретным классом.

---

## 🏗️ Зачем использовать ABC?

Основные задачи:

* определить общий интерфейс;
* заставить наследников реализовать определённые методы;
* предотвратить создание неполностью реализованных объектов;
* сделать архитектуру кода более явной;
* использовать общий контракт для разных реализаций.

---

## 💼 Backend-пример

Например, интерфейс хранилища:

```python id="5y2h8c"
from abc import ABC, abstractmethod


class UserRepository(ABC):

    @abstractmethod
    def get_by_id(self, user_id: int):
        pass

    @abstractmethod
    def save(self, user):
        pass
```

Можно создать разные реализации:

```python id="q4n8wd"
class PostgresUserRepository(UserRepository):

    def get_by_id(self, user_id):
        ...

    def save(self, user):
        ...


class InMemoryUserRepository(UserRepository):

    def get_by_id(self, user_id):
        ...

    def save(self, user):
        ...
```

Бизнес-логика работает с интерфейсом:

```text id="e7f4za"
UserRepository
      ↑
 ┌────┴─────┐
 │          │
Postgres   InMemory
```

Это помогает соблюдать **Dependency Inversion Principle** и уменьшает связанность компонентов.

---

## 🆚 ABC vs обычный класс

| Обычный класс                            | ABC                                                        |
| ---------------------------------------- | ---------------------------------------------------------- |
| Можно создать экземпляр                  | Нельзя создать, если есть нереализованные abstract methods |
| Методы необязательны для переопределения | Абстрактные методы обязательны                             |
| Контракт не enforced                     | Контракт проверяется Python                                |
| Может использоваться напрямую            | Обычно является базой для конкретных реализаций            |

---

## 🆚 ABC vs Protocol

Оба механизма позволяют описывать интерфейс, но подходят для разных подходов.

### ABC

Использует **явное наследование**:

```python id="z3w8yp"
class Dog(Animal):
    ...
```

### Protocol

Использует **структурную типизацию**:

```python id="s8q2kd"
from typing import Protocol


class Speaker(Protocol):
    def speak(self) -> None:
        ...
```

Класс не обязан явно наследоваться от `Speaker` — достаточно соответствовать интерфейсу.

```text id="x0q5na"
ABC      → nominal typing
Protocol → structural typing
```

---

## 🧠 Главное

* `ABC` → **Abstract Base Class**.
* Используется для создания абстрактных базовых классов.
* Импортируется из `abc`.
* `@abstractmethod` объявляет обязательный метод.
* ABC может содержать как абстрактные, так и обычные методы.
* Нельзя создать экземпляр класса с нереализованными абстрактными методами.
* Наследник обязан реализовать все необходимые abstract methods.
* Часто используется для определения **контрактов и интерфейсов**.

## 🎤 Суперкоротко

> **ABC** — абстрактный базовый класс, который задаёт контракт для наследников. С помощью `@abstractmethod` можно объявить методы, которые конкретные классы обязаны реализовать. Класс с нереализованными абстрактными методами нельзя инстанцировать.
