# @abstractmethod

`@abstractmethod` — декоратор из модуля `abc`, который объявляет **абстрактный метод**: метод, который дочерний класс должен реализовать.

```python
from abc import ABC, abstractmethod


class Animal(ABC):

    @abstractmethod
    def make_sound(self):
        pass


class Dog(Animal):

    def make_sound(self):
        return "Гав"
```

## 🎯 Ответ на собеседовании

> `@abstractmethod` помечает метод как абстрактный. Если в классе остаются нереализованные абстрактные методы, такой класс нельзя создать напрямую. Дочерний класс должен их реализовать.

---

## Как работает

Если класс наследуется от `ABC` и содержит `@abstractmethod`:

```python
from abc import ABC, abstractmethod


class Animal(ABC):

    @abstractmethod
    def make_sound(self):
        pass
```

Создать объект напрямую нельзя:

```python
animal = Animal()
# TypeError: Can't instantiate abstract class Animal
```

Но дочерний класс может реализовать метод:

```python
class Dog(Animal):

    def make_sound(self):
        return "Гав"


dog = Dog()
```

---

## Важно: `@abstractmethod` не обязательно означает `pass`

Абстрактный метод может иметь реализацию:

```python
class Animal(ABC):

    @abstractmethod
    def make_sound(self):
        print("Какой-то звук")
```

Дочерний класс всё равно должен переопределить его, чтобы считаться конкретным классом.

При этом реализацию родителя можно вызвать:

```python
class Dog(Animal):

    def make_sound(self):
        super().make_sound()
        print("Гав")
```

---

## Несколько абстрактных методов

```python
class Payment(ABC):

    @abstractmethod
    def pay(self, amount):
        pass

    @abstractmethod
    def refund(self, amount):
        pass
```

Класс должен реализовать **все** абстрактные методы:

```python
class CardPayment(Payment):

    def pay(self, amount):
        print(f"Оплата: {amount}")

    def refund(self, amount):
        print(f"Возврат: {amount}")
```

Если реализовать только один:

```python
class CardPayment(Payment):

    def pay(self, amount):
        print(f"Оплата: {amount}")
```

то `CardPayment` тоже останется абстрактным и создать его нельзя.

---

## Где используется

Основной сценарий — определить **контракт** для дочерних классов:

```text
Payment
├── pay()
└── refund()

       ↓

CardPayment
BankTransfer
CryptoPayment
```

Каждый конкретный класс обязан предоставить свою реализацию.

---

## `ABC` + `@abstractmethod`

Обычно используются вместе:

```python
from abc import ABC, abstractmethod


class Repository(ABC):

    @abstractmethod
    def save(self, obj):
        pass

    @abstractmethod
    def get(self, id):
        pass
```

* `ABC` — делает класс абстрактным базовым классом.
* `@abstractmethod` — обозначает обязательные для реализации методы.

### ⚠️ Важный нюанс

`@abstractmethod` сам по себе не делает класс абстрактным в обычном смысле:

```python
class A:

    @abstractmethod
    def foo(self):
        pass


A()  # формально создать можно
```

Механизм абстрактных классов работает через `ABCMeta`, обычно посредством наследования от `ABC`.

---

## Связь с `Protocol`

| `ABC + @abstractmethod`                   | `Protocol`                    |
| ----------------------------------------- | ----------------------------- |
| Номинальное наследование                  | Структурная типизация         |
| Нужно наследоваться от ABC                | Наследоваться необязательно   |
| Контракт проверяется при создании объекта | В основном статический анализ |
| Может содержать состояние и реализацию    | Обычно описывает интерфейс    |

### Запомнить

```text
ABC
 ↓
абстрактный класс
 ↓
@abstractmethod
 ↓
обязательный метод
```

**Ключевая идея:** `@abstractmethod` задаёт контракт: «конкретный класс обязан предоставить эту реализацию».
