# Generics / TypeVar

## 🎯 Ответ на собеседовании

**Generics (обобщённые типы)** — механизм типизации, который позволяет писать функции и классы, работающие с **разными типами**, сохраняя при этом информацию о типах.

**`TypeVar`** — специальный объект из `typing`, который используется для создания **параметра типа**.

Простой пример:

```python
from typing import TypeVar

T = TypeVar("T")


def first(items: list[T]) -> T:
    return items[0]
```

Здесь `T` — неизвестный заранее тип.

```python
first([1, 2, 3])        # int
first(["a", "b", "c"])  # str
```

Одна функция работает с разными типами, но тип результата соответствует типу элементов списка.

---

## 📌 Что такое `TypeVar`?

```python
from typing import TypeVar

T = TypeVar("T")
```

`T` — **переменная типа**, а не обычная переменная.

Например:

```python
def get_first(items: list[T]) -> T:
    return items[0]
```

Можно представить:

```text
list[int]  → T = int  → результат int
list[str]  → T = str  → результат str
```

При этом функция одна.

---

## 🆚 `Any` vs `TypeVar`

Это важное различие.

### `Any`

```python
from typing import Any

def first(items: list[Any]) -> Any:
    return items[0]
```

Мы фактически говорим:

> «Тип мне не важен».

Связь между входом и результатом теряется.

---

### `TypeVar`

```python
T = TypeVar("T")

def first(items: list[T]) -> T:
    return items[0]
```

Здесь мы говорим:

> «Тип может быть любым, но входной и возвращаемый тип должны быть связаны».

Например:

```text
list[int] → int
list[str] → str
```

Поэтому `TypeVar` обычно предпочтительнее `Any`, когда связь типов важна.

---

## 📦 Generics для классов

Можно параметризовать целый класс:

```python
class Box[T]:
    def __init__(self, value: T):
        self.value = value

    def get(self) -> T:
        return self.value
```

Использование:

```python
int_box = Box(10)
str_box = Box("hello")
```

Статический анализатор понимает:

```text
int_box.get() → int
str_box.get() → str
```

Такой синтаксис параметров типов появился в **Python 3.12** (PEP 695).

Для Python 3.13 он доступен.

---

## 🔧 Старый синтаксис

До Python 3.12 обычно писали:

```python
from typing import Generic, TypeVar

T = TypeVar("T")


class Box(Generic[T]):
    def __init__(self, value: T):
        self.value = value

    def get(self) -> T:
        return self.value
```

Современный вариант:

```python
class Box[T]:
    ...
```

Для нового кода на Python 3.13 предпочтительнее современный синтаксис, если проект не ограничен старой версией Python.

---

## 🎯 Ограничение `TypeVar`

`TypeVar` можно ограничить определёнными типами.

```python
T = TypeVar("T", int, float)


def double(value: T) -> T:
    return value * 2
```

Теперь `T` может быть только:

```text
int
float
```

---

## 🔒 Bound

Можно задать не список допустимых типов, а **верхнюю границу**:

```python
from typing import TypeVar

T = TypeVar("T", bound="Animal")
```

Это означает:

> `T` должен быть `Animal` или его наследником.

Например:

```python
class Animal:
    def speak(self):
        ...


class Dog(Animal):
    pass
```

Тогда:

```text
T = Animal
T = Dog
```

допустимы.

---

## 🧩 Generics ≠ разные реализации

Важно понимать:

```python
def first[T](items: list[T]) -> T:
    return items[0]
```

не создаёт отдельную функцию для `int`, `str`, `float` и т. д.

Это **один и тот же Python-код**.

Generics в данном контексте в первую очередь помогают **статическому анализу типов** и IDE.

---

## 💼 Backend-пример

Generics особенно полезны для универсальных компонентов.

Например, репозиторий:

```python
class Repository[T]:
    def get(self, id: int) -> T:
        ...
    
    def save(self, obj: T) -> T:
        ...
```

Можно специализировать:

```python
user_repository: Repository[User]
order_repository: Repository[Order]
```

Тогда статический анализатор понимает:

```text
user_repository.get(...)  → User
order_repository.get(...) → Order
```

При этом реализация репозитория остаётся общей.

---

## 🆚 TypeVar vs Generic

|                  | `TypeVar`                   | `Generic` / generic syntax          |
| ---------------- | --------------------------- | ----------------------------------- |
| Что это          | Параметр типа               | Обобщённая функция/структура        |
| Пример           | `T = TypeVar("T")`          | `class Box[T]`                      |
| Задача           | Представить неизвестный тип | Параметризовать класс/функцию типом |
| Где используется | Функции, классы, типы       | В основном generic-классы и функции |

Фактически `TypeVar` — один из инструментов, с помощью которого раньше строили Generics.

---

## 🧠 Главное

```text
Generics
   ↓
один код → работает с разными типами

TypeVar
   ↓
параметр типа T

list[T]
   ↓
список элементов типа T

→ T
   ↓
результат того же типа
```

Не путать:

* `Any` → «тип может быть каким угодно, связь не важна»;
* `TypeVar` → «тип может быть разным, но сохраняем связь между типами»;
* `Generic` → создаём обобщённую структуру.

## 🎤 Суперкоротко

> **Generics** позволяют писать типобезопасный код, работающий с разными типами. **`TypeVar`** представляет параметр типа. Например, `def first[T](items: list[T]) -> T` означает: функция принимает список элементов типа `T` и возвращает элемент того же типа.
