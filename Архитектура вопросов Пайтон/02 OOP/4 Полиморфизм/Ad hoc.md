# Ad hoc полиморфизм

> 🎯 **Ответ на собеседовании**
>
> **Ad hoc полиморфизм** — это возможность использовать одну операцию с разными типами данных, предоставляя для каждого типа свою реализацию.
>
> В классических языках он часто реализуется через **перегрузку функций или методов**. В Python классической перегрузки функций по сигнатуре нет, но похожее поведение можно реализовать, например, с помощью `functools.singledispatch`.
>
> `singledispatch` выбирает нужную реализацию **во время выполнения (runtime)** по типу **первого аргумента**.

---

## Что такое ad hoc полиморфизм

**Ad hoc полиморфизм** — одна операция → разные реализации для конкретных типов.

Например, логически у нас есть одна операция:

```python
process(value)
```

Но для разных типов она должна работать по-разному:

```text
int  → обработать как число
str  → обработать как строку
list → обработать как список
```

В классических языках это часто называют **перегрузкой (overloading)**.

---

## Перегрузка функций в Python

Классической перегрузки по сигнатуре в Python нет.

Так делать нельзя:

```python
def process(value: int):
    return value * 2


def process(value: str):
    return value.upper()
```

Вторая функция просто **перезапишет первую**.

```python
process(10)
```

будет вызвана версия для `str`, которая ожидает строку.

> ⚠️ Аннотации типов (`int`, `str`) сами по себе не создают перегрузку.

---

# `functools.singledispatch`

Python предоставляет механизм `singledispatch`:

```python
from functools import singledispatch
```

Он позволяет зарегистрировать разные реализации одной функции для разных типов.

```python
from functools import singledispatch


@singledispatch
def process(value):
    return f"Unknown: {value}"


@process.register
def _(value: int):
    return f"Integer: {value}"


@process.register
def _(value: str):
    return f"String: {value}"
```

Теперь:

```python
process(10)
# Integer: 10

process("hello")
# String: hello
```

---

## Как работает `singledispatch`

```text
process(10)
     ↓
смотрим тип первого аргумента
     ↓
int
     ↓
выбираем реализацию для int
```

А для строки:

```text
process("hello")
     ↓
смотрим тип первого аргумента
     ↓
str
     ↓
выбираем реализацию для str
```

То есть выбор происходит **runtime**.

---

## Почему `single`?

`singledispatch` означает **single dispatch** — диспетчеризация выполняется только по **одному аргументу**.

Причём именно по **первому аргументу**:

```python
process(value, other)
```

Тип `value` участвует в выборе реализации, а тип `other` — нет.

---

## Регистрация реализации

Можно явно указать тип:

```python
@process.register(int)
def _(value):
    return value * 2
```

Или использовать аннотацию:

```python
@process.register
def _(value: int):
    return value * 2
```

Оба варианта регистрируют реализацию для `int`.

---

## Наследование типов

`singledispatch` учитывает **иерархию типов**.

Например:

```python
from functools import singledispatch


@singledispatch
def process(value):
    return "default"


@process.register
def _(value: int):
    return "integer"
```

Для подкласса `int`, если отдельной реализации нет, будет использована наиболее подходящая зарегистрированная реализация с учётом MRO.

---

# `singledispatch` vs `typing.overload`

Эти механизмы часто путают.

|                            | `singledispatch`        | `typing.overload`     |
| -------------------------- | ----------------------- | --------------------- |
| Выполняется                | **Runtime**             | Статический анализ    |
| Выбирает реализацию        | ✅ Да                    | ❌ Нет                 |
| Меняет поведение программы | ✅ Да                    | ❌ Нет                 |
| Назначение                 | Runtime-диспетчеризация | Типизация             |
| Основной критерий          | Тип первого аргумента   | Типы для type checker |

### `typing.overload`

```python
from typing import overload


@overload
def process(value: int) -> int: ...


@overload
def process(value: str) -> str:


def process(value):
    if isinstance(value, int):
        return value * 2
    return value.upper()
```

`overload` помогает **mypy / Pyright / IDE** понять возможные варианты типов.

Но runtime-выбор всё равно реализуется самостоятельно:

```python
if isinstance(value, int):
    ...
```

---

# Основные виды полиморфизма

Для собеседования полезно различать:

| Вид            | Идея                                        | Python                                          |
| -------------- | ------------------------------------------- | ----------------------------------------------- |
| **Ad hoc**     | Разные реализации для конкретных типов      | `singledispatch`, runtime-проверки              |
| **Subtype**    | Общий интерфейс для разных подтипов         | Наследование, Protocol, duck typing             |
| **Parametric** | Один алгоритм работает с разными типами     | `Generic`, `TypeVar`                            |
| **Static**     | Выбор варианта на этапе анализа/компиляции  | `typing.overload` помогает статическому анализу |
| **Dynamic**    | Реализация определяется во время выполнения | Duck typing, overriding                         |

---

## 🧠 Что запомнить

```text
Ad hoc
   ↓
одна операция
   ↓
разные конкретные типы
   ↓
разные реализации
```

```text
singledispatch
   ↓
runtime
   ↓
тип первого аргумента
   ↓
выбор зарегистрированной реализации
```

### Короткая формула

> **Ad hoc = одна операция → разные реализации для конкретных типов.**

> **`singledispatch` = runtime-реализация ad hoc полиморфизма по типу первого аргумента.**

> **`typing.overload` = описание вариантов для статического анализатора, а не runtime-перегрузка.**
