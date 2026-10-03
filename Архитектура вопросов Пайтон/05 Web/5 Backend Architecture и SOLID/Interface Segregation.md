# 🅘 Interface Segregation Principle — принцип разделения интерфейсов

## 🎯 Ответ на собеседовании

**Interface Segregation Principle (ISP)** — принцип разделения интерфейсов.

Он говорит:

> **Клиенты не должны зависеть от методов, которые они не используют.**

Вместо одного большого интерфейса лучше создать несколько маленьких и специализированных интерфейсов.

В Python это особенно удобно реализовывать через **абстрактные классы (`ABC`)** или **`Protocol`**.

---

## 🎤 Суперкоротко

```text id="8qv1hk"
ISP
 ↓
Не заставляй класс
реализовывать ненужные методы
```

Плохо:

```text id="3vks9c"
Один огромный интерфейс
```

Хорошо:

```text id="x0k4wz"
Несколько маленьких интерфейсов
```

---

# 🧩 Что такое интерфейс

**Интерфейс** — это контракт, который определяет, какие операции должен предоставлять объект.

Например:

```python id="q1d2f4"
class Printer:

    def print_document(self):
        pass
```

Контракт говорит:

```text id="z8h7e3"
Printer
   ↓
print_document()
```

Клиент может работать с объектом через этот контракт.

---

# ❌ Нарушение ISP

Представим многофункциональное устройство:

```python id="v3x8k9"
from abc import ABC, abstractmethod


class Machine(ABC):

    @abstractmethod
    def print_document(self):
        pass

    @abstractmethod
    def scan_document(self):
        pass

    @abstractmethod
    def fax_document(self):
        pass
```

Обычный принтер умеет только печатать:

```python id="r7m2qa"
class SimplePrinter(Machine):

    def print_document(self):
        print("Printing")

    def scan_document(self):
        raise NotImplementedError

    def fax_document(self):
        raise NotImplementedError
```

Проблема:

```text id="f3v0md"
SimplePrinter
      ↓
обязан реализовать:
      ↓
print ✅
scan  ❌
fax   ❌
```

Принтер не должен зависеть от методов сканирования и факса.

Это нарушение ISP.

---

# ✅ Соблюдение ISP

Разделяем большой интерфейс на несколько маленьких:

```python id="k8s4aa"
from abc import ABC, abstractmethod


class Printable(ABC):

    @abstractmethod
    def print_document(self):
        pass


class Scannable(ABC):

    @abstractmethod
    def scan_document(self):
        pass


class Faxable(ABC):

    @abstractmethod
    def fax_document(self):
        pass
```

Теперь простой принтер реализует только нужный интерфейс:

```python id="3b9v2x"
class SimplePrinter(Printable):

    def print_document(self):
        print("Printing")
```

МФУ может реализовать несколько:

```python id="f2j7qp"
class MultiFunctionPrinter(Printable, Scannable, Faxable):

    def print_document(self):
        print("Printing")

    def scan_document(self):
        print("Scanning")

    def fax_document(self):
        print("Faxing")
```

Получаем:

```text id="c5m4dr"
Printable
    ↑
SimplePrinter

Printable
Scannable
Faxable
    ↑
MultiFunctionPrinter
```

---

# 🧠 Главная идея ISP

Не нужно создавать интерфейс:

```text id="3h5q2x"
Machine
 ├── print()
 ├── scan()
 ├── fax()
 ├── copy()
 ├── staple()
 └── ...
```

если разные классы используют только часть этих методов.

Лучше:

```text id="k9j4z7"
Printable
Scannable
Faxable
Copyable
```

Каждый класс выбирает только нужные возможности.

---

# 🐍 ISP в Python через Protocol

В Python интерфейс часто удобно описывать через `Protocol`.

```python id="w4s8cm"
from typing import Protocol


class Printable(Protocol):

    def print_document(self) -> None:
        ...
```

Теперь функции может быть достаточно этого контракта:

```python id="x8q2mz"
def print_document(printer: Printable) -> None:
    printer.print_document()
```

Любой объект с подходящим методом может использоваться:

```python id="p7f3qa"
class SimplePrinter:

    def print_document(self) -> None:
        print("Printing")
```

Это соответствует подходу Python с **duck typing** и структурной типизацией.

---

# 🔗 ISP и композиция

ISP хорошо сочетается с композицией.

Например:

```python id="q4k9nb"
class Printer:

    def print_document(self):
        pass


class Scanner:

    def scan_document(self):
        pass


class MFP:

    def __init__(self, printer, scanner):
        self.printer = printer
        self.scanner = scanner
```

MFP использует отдельные компоненты вместо огромного интерфейса.

```text id="x7m2kd"
MFP
 ├── Printer
 └── Scanner
```

---

# 🆚 ISP и SRP

Они похожи, но говорят о разных вещах.

### SRP

> **У класса должна быть одна ответственность.**

```text id="8d5y2f"
SRP
 ↓
Класс
 ↓
Одна ответственность
```

### ISP

> **Клиент не должен зависеть от ненужных методов интерфейса.**

```text id="m4p8sc"
ISP
 ↓
Интерфейс
 ↓
Только необходимые методы
```

Можно запомнить:

```text id="n7k3qa"
SRP → разделяем ответственность

ISP → разделяем интерфейс
```

---

# 🆚 ISP и LSP

Эти принципы тоже связаны.

### LSP

Наследник должен корректно заменять родителя:

```text id="j2r6pm"
Parent
  ↑
Child
  ↓
не ломает контракт
```

### ISP

Не заставляем класс реализовывать методы, которые ему не нужны:

```text id="w9c5vx"
Большой интерфейс
      ↓
разделяем
      ↓
маленькие интерфейсы
```

Часто нарушение ISP приводит к проблемам с LSP.

Например:

```python id="z4x8nc"
class Bird:

    def fly(self):
        pass


class Penguin(Bird):

    def fly(self):
        raise NotImplementedError
```

Вместо этого можно разделить контракт:

```python id="r5k7md"
class Bird:
    def eat(self):
        pass


class FlyingBird(Bird):

    def fly(self):
        pass
```

Теперь пингвин не обязан реализовывать `fly()`.

---

# 🎯 Пример из backend

Допустим, есть интерфейс репозитория:

```python id="v8c3qa"
class UserRepository:

    def create(self, user):
        pass

    def get(self, user_id):
        pass

    def update(self, user):
        pass

    def delete(self, user_id):
        pass
```

Если какой-то компонент работает только с чтением:

```python id="n6m4pk"
class UserReport:

    def __init__(self, repository):
        self.repository = repository

    def generate(self):
        return self.repository.get(1)
```

ему не нужны:

```text id="j9q2wx"
create()
update()
delete()
```

Можно выделить интерфейс чтения:

```python id="p3x7vb"
from typing import Protocol


class UserReader(Protocol):

    def get(self, user_id: int):
        ...
```

Теперь `UserReport` зависит только от необходимого ему контракта:

```python id="c6m8zn"
class UserReport:

    def __init__(self, repository: UserReader):
        self.repository = repository

    def generate(self):
        return self.repository.get(1)
```

---

# 🧠 Почему ISP полезен

Разделение интерфейсов даёт:

```text id="a4p7yd"
меньше зависимостей
      ↓
меньше связности
      ↓
проще тестировать
      ↓
проще изменять
      ↓
меньше побочных эффектов
```

Например, изменение метода `delete()` не должно заставлять компонент, который умеет только `get()`, пересобирать или менять свою зависимость.

---

# ⚠️ ISP не означает «делай интерфейс из одного метода»

Интерфейс может содержать несколько методов.

Главное:

```text id="q8m3zx"
Методы должны быть
логически связаны
```

Плохо:

```text id="v5k7aa"
UserInterface
 ├── save()
 ├── send_email()
 ├── generate_report()
 ├── connect_to_database()
 └── calculate_tax()
```

Хорошо:

```text id="e9p4wx"
UserRepository
 ├── save()
 ├── get()
 └── delete()

EmailSender
 └── send()

ReportGenerator
 └── generate()
```

---

# 🎯 Как ответить, если попросят пример

> **Например, если есть интерфейс `Machine` с методами `print`, `scan` и `fax`, простой принтер будет вынужден реализовать методы, которые он не поддерживает. Это нарушение ISP. Лучше разделить интерфейс на `Printable`, `Scannable` и `Faxable`, чтобы каждый класс зависел только от необходимых ему методов.**

---

# 🧠 Главное

```text id="2x8w7q"
ISP
 │
 ├── Большой интерфейс
 │       ↓
 │    ❌ плохо
 │
 └── Маленькие специализированные
     интерфейсы
         ↓
       ✅ хорошо
```

**Клиент должен зависеть только от тех методов, которые ему действительно нужны.**

В Python для реализации ISP часто используются:

```text id="7m4q2c"
ABC
Protocol
Duck Typing
Composition
```

---

# 🧠 Формула для запоминания

**ISP = не заставляй клиента зависеть от того, что ему не нужно.**

Или ещё короче:

```text id="r3v9kx"
Большой интерфейс
       ↓
разделить
       ↓
маленькие интерфейсы
```

**SRP разделяет ответственности, ISP разделяет интерфейсы.**

# Связанные темы:

[[SOLID|SOLID]]