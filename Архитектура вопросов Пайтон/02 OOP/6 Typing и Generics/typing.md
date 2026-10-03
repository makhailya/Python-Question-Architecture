# `typing`

## 🎯 Ответ на собеседовании

**`typing`** — стандартный модуль Python, предоставляющий инструменты для **аннотаций типов** и построения типизированного кода.

Он помогает IDE и статическим анализаторам (`mypy`, `Pyright`) проверять код, но **обычно не заставляет Python проверять типы во время выполнения**.

```python id="w7j3kx"
from typing import Protocol, TypeVar, Literal
```

---

## 📌 Зачем нужен `typing`?

Python динамически типизирован, поэтому такой код допустим:

```python id="2q8m1v"
def add(a, b):
    return a + b
```

С помощью аннотаций можно явно описать ожидаемые типы:

```python id="9s4f2k"
def add(a: int, b: int) -> int:
    return a + b
```

Это позволяет:

* улучшить читаемость;
* получить подсказки IDE;
* находить ошибки статическим анализом;
* описывать сложные типы;
* строить Generics и Protocol.

---

## 🔹 Основные инструменты `typing`

### `Optional`

Означает, что значение может быть указанного типа или `None`.

```python id="5k9r1c"
from typing import Optional

def find_user(user_id: int) -> Optional[User]:
    ...
```

То есть:

```text id="7f3n2m"
User | None
```

В современном Python можно писать проще:

```python id="8d1v6p"
def find_user(user_id: int) -> User | None:
    ...
```

---

## 🔹 `Union`

Описывает несколько возможных типов:

```python id="q2m7xc"
from typing import Union

value: Union[int, str]
```

Современный синтаксис:

```python id="p8k4za"
value: int | str
```

Для Python 3.13 предпочтительнее использовать `|`.

---

## 🔹 `Any`

Означает:

> Значение может иметь любой тип.

```python id="m5c8vq"
from typing import Any

value: Any
```

`Any` отключает значительную часть статической проверки для этого значения.

Поэтому злоупотреблять им не стоит.

---

## 🔹 `TypeVar`

Используется для создания **параметров типов**.

```python id="j6w3qa"
from typing import TypeVar

T = TypeVar("T")


def first(items: list[T]) -> T:
    return items[0]
```

Здесь:

```text id="u2k7mx"
list[int] → int
list[str] → str
```

Подробнее → [[Generics / TypeVar]].

---

## 🔹 `Generic`

Позволяет создавать обобщённые классы.

Старый синтаксис:

```python id="a9f4kd"
from typing import Generic, TypeVar

T = TypeVar("T")


class Box(Generic[T]):
    def __init__(self, value: T):
        self.value = value
```

В Python 3.12+ появился более простой синтаксис:

```python id="z3m8wp"
class Box[T]:
    def __init__(self, value: T):
        self.value = value
```

Для Python 3.13 рекомендуется современный синтаксис.

---

## 🔹 `Protocol`

Используется для **структурной типизации**.

```python id="c7v2na"
from typing import Protocol


class Speaker(Protocol):
    def speak(self) -> None:
        ...
```

Класс не обязан наследоваться от `Speaker`.

Если у него есть `speak()`, он соответствует этому Protocol с точки зрения статического анализатора.

Подробнее → [[Protocol]].

---

## 🔹 `Literal`

Позволяет указать **конкретные допустимые значения**:

```python id="n4q8yc"
from typing import Literal

def set_status(
    status: Literal["active", "inactive"]
):
    ...
```

Допустимо:

```python id="k2m6vb"
set_status("active")
set_status("inactive")
```

А:

```python id="r8x3pz"
set_status("deleted")
```

статический анализатор может определить как ошибку.

---

## 🔹 `Callable`

Описывает вызываемый объект — например, функцию.

```python id="f6k1md"
from typing import Callable

def process(
    callback: Callable[[int], str]
) -> str:
    return callback(10)
```

Это означает:

```text id="b3q7wa"
Callable[[int], str]
       ↓
принимает int
возвращает str
```

---

## 🔹 `TypedDict`

Позволяет описывать структуру словаря:

```python id="v5m9xc"
from typing import TypedDict


class UserData(TypedDict):
    name: str
    age: int
```

Теперь:

```python id="h8k2pz"
user: UserData = {
    "name": "Ilya",
    "age": 31,
}
```

Статический анализатор может проверить наличие и типы ключей.

При этом это всё ещё обычный `dict` во время выполнения.

---

## 🔹 `Final`

Используется для обозначения значения, которое не должно быть переопределено:

```python id="q9w4kc"
from typing import Final

MAX_CONNECTIONS: Final = 100
```

Это указание для статического анализатора, а не полноценный runtime-запрет.

---

## ⚠️ `typing` не превращает Python в статически типизированный язык

Аннотация:

```python id="x2v7ma"
def add(a: int, b: int) -> int:
    return a + b
```

не означает, что Python автоматически выбросит ошибку при:

```python id="m6k1rz"
add("hello", "world")
```

Python обычно не выполняет полноценную проверку этих аннотаций сам.

Для этого используются инструменты вроде:

```text id="d4p8qs"
mypy
Pyright
Pylance
```

---

## 🧩 Что относится к `typing`

Упрощённо:

```text id="c9x3mv"
typing
├── Any
├── Optional
├── Union
├── Literal
├── Callable
├── TypeVar
├── Generic
├── Protocol
├── TypedDict
└── Final
```

При этом в современном Python многие конструкции теперь можно записывать **обычным синтаксисом языка**:

```python id="s7k2na"
list[int]
dict[str, int]
int | None
class Box[T]:
    ...
```

Поэтому `typing` сегодня особенно важен для инструментов, которые не выражаются полностью встроенным синтаксисом.

---

## 🧠 Главное

* `typing` → стандартный модуль для **типовых аннотаций и статического анализа**.
* Не выполняет полноценную проверку типов во время выполнения.
* `TypeVar` → параметры типов.
* `Generic` → обобщённые классы/структуры.
* `Protocol` → структурная типизация.
* `TypedDict` → типизированная структура `dict`.
* `Literal` → конкретные допустимые значения.
* `Callable` → тип функции/вызываемого объекта.
* `Any` → любой тип, но снижает пользу статической проверки.
* В Python 3.13 часть старых конструкций `typing` имеет более современный синтаксис.

## 🎤 Суперкоротко

> **`typing`** — стандартный модуль Python для аннотаций типов и поддержки статического анализа. Он предоставляет `TypeVar`, `Generic`, `Protocol`, `TypedDict`, `Literal`, `Callable` и другие инструменты. Сам по себе `typing` обычно не проверяет типы во время выполнения.
