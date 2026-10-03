# Duck Typing

## 🎯 Ответ на собеседовании

**Duck typing** — принцип динамической типизации, при котором важен **не конкретный тип объекта**, а наличие у него необходимых методов и атрибутов.

Название происходит от идеи:

> «Если объект выглядит как утка и ведёт себя как утка — это утка».

В Python поэтому часто не проверяют класс объекта, а просто используют нужный интерфейс.

---

## 📌 Пример

```python id="h7k2mp"
class Dog:
    def speak(self):
        print("Гав")


class Cat:
    def speak(self):
        print("Мяу")


def make_sound(animal):
    animal.speak()
```

Функции не важно, `Dog` это или `Cat`:

```python id="q3v8nd"
make_sound(Dog())
make_sound(Cat())
```

Результат:

```text id="m9x4cz"
Гав
Мяу
```

`make_sound()` просто ожидает, что объект предоставляет метод `speak()`.

---

## 🆚 Проверка типа vs Duck Typing

### Явная проверка типа

```python id="a5k9rx"
def make_sound(animal):
    if isinstance(animal, Dog):
        animal.speak()
```

Функция жёстко привязана к `Dog`.

### Duck typing

```python id="w2f6pz"
def make_sound(animal):
    animal.speak()
```

Функция работает с **любым объектом**, у которого есть `speak()`.

---

## 🧩 Наследование не требуется

Это важная особенность.

```python id="k8q1vb"
class Dog:
    def speak(self):
        print("Гав")


class Robot:
    def speak(self):
        print("Beep")
```

`Dog` и `Robot` никак не связаны:

```text id="j6m4xa"
Dog       Robot
 │          │
 └── speak ─┘
```

Но:

```python id="p9v3kc"
def make_sound(obj):
    obj.speak()
```

работает с обоими.

---

## ⚠️ Duck typing может привести к ошибке во время выполнения

Если передать объект без нужного метода:

```python id="e4k8mz"
class Car:
    pass


make_sound(Car())
```

получим:

```text id="d2r7qx"
AttributeError
```

Python обнаруживает проблему **во время выполнения**.

---

## 🔗 Duck Typing и полиморфизм

Duck typing является одним из способов реализации **динамического полиморфизма** в Python.

```text id="c7p3wa"
Динамический полиморфизм
          ↓
     Duck typing
          ↓
Один интерфейс → разные объекты
```

Например:

```python id="u5n8kd"
def save(repository):
    repository.save()
```

`repository` может быть:

```text id="z1q6mv"
PostgresRepository
RedisRepository
InMemoryRepository
```

Если у объекта есть `save()`, функция может его использовать.

---

## 🏗️ Duck Typing + Protocol

`Protocol` позволяет **формализовать duck typing для статической типизации**.

Без `Protocol`:

```python id="n8x2qa"
def save(repository):
    repository.save()
```

С `Protocol`:

```python id="f4m7vc"
from typing import Protocol


class Repository(Protocol):
    def save(self) -> None:
        ...


def process(repository: Repository):
    repository.save()
```

Теперь статический анализатор может проверить, соответствует ли объект нужному интерфейсу.

```text id="q9w5zb"
Duck typing
     ↓
«есть нужный метод — используй»

Protocol
     ↓
«есть нужный метод — и статический анализатор это проверит»
```

---

## 🆚 Duck Typing vs ABC

### ABC

Использует **явное наследование**:

```python id="t3m8xa"
class Repository(ABC):
    ...
    

class PostgresRepository(Repository):
    ...
```

### Duck typing

Наследование не требуется:

```python id="b6q2pn"
class PostgresRepository:
    def save(self):
        ...
```

Если есть `save()` — объект подходит.

---

## 💼 Пример из Backend

Допустим, сервис отправляет уведомление:

```python id="v8k3qd"
def send_notification(sender, message):
    sender.send(message)
```

Можно передать:

```python id="p2m6xa"
class EmailSender:
    def send(self, message):
        ...


class TelegramSender:
    def send(self, message):
        ...
```

Сервис не зависит от конкретного класса:

```python id="r7w4nk"
send_notification(EmailSender(), "Привет")
send_notification(TelegramSender(), "Привет")
```

Это уменьшает связанность и упрощает замену реализации.

---

## 🧠 Главное

* Duck typing → **важно поведение объекта, а не его класс**.
* Не требует наследования.
* Основан на динамической типизации Python.
* Ошибки отсутствующего интерфейса обычно обнаруживаются в runtime.
* Часто используется для динамического полиморфизма.
* `Protocol` позволяет описывать такой интерфейс для статической типизации.
* Принцип: **«если объект поддерживает нужный интерфейс — его можно использовать»**.

## 🎤 Суперкоротко

> **Duck typing** — это подход Python, при котором мы не проверяем конкретный тип объекта, а просто используем необходимые методы и атрибуты. Если объект поддерживает нужный интерфейс — он подходит. Это один из основных механизмов динамического полиморфизма в Python.
