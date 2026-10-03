# 🔀 `functools.singledispatch`

🎯 **Ответ на собеседовании:**

`functools.singledispatch` — декоратор, который позволяет реализовать **перегрузку функции по типу первого аргумента**.

В зависимости от типа первого аргумента Python во время выполнения выбирает подходящую реализацию.

```python id="d7q2ka"
from functools import singledispatch


@singledispatch
def process(value):
    raise TypeError("Unsupported type")


@process.register
def _(value: int):
    return value * 2


@process.register
def _(value: str):
    return value.upper()
```

Использование:

```python id="m9v4cp"
process(10)        # 20
process("hello")   # HELLO
```

---

# Как это работает

Основная функция:

```python id="x8k3qz"
@singledispatch
def process(value):
    ...
```

становится **generic function**.

Для отдельных типов регистрируются реализации:

```python id="j5n1wb"
@process.register
def _(value: int):
    ...
```

```python id="q6t8vs"
@process.register
def _(value: str):
    ...
```

При вызове:

```python id="3p7w2m"
process(10)
```

`singledispatch` смотрит на тип **первого аргумента**:

```text id="7f4c9n"
10
 ↓
int
 ↓
зарегистрирован обработчик int
 ↓
process(int)
```

---

# Важный момент: только первый аргумент

`singledispatch` называется **single** dispatch именно потому, что диспетчеризация выполняется только по **первому аргументу**.

```python id="k4m8s2"
@singledispatch
def process(value, option):
    ...
```

Выбор реализации зависит от:

```python id="6z1q5r"
type(value)
```

а `option` не участвует в выборе реализации.

---

# Регистрация типов

Можно регистрировать несколько типов:

```python id="h8v3x6"
@process.register
def _(value: float):
    return round(value)
```

```python id="r2m7k4"
@process.register
def _(value: list):
    return len(value)
```

Теперь:

```python id="n5c9p1"
process(3.14)
process([1, 2, 3])
```

используют соответствующие реализации.

---

# Наследование типов

`singledispatch` учитывает **MRO**.

Например:

```python id="w7d3q8"
@process.register
def _(value: object):
    return "object"


@process.register
def _(value: int):
    return "int"
```

Для:

```python id="z4k1m6"
process(10)
```

будет выбрана реализация `int`, поскольку `int` находится ближе к объекту в иерархии типов.

Если точной реализации нет, Python ищет подходящую реализацию через иерархию типов.

---

# Регистрация без аннотации

Тип можно указать явно:

```python id="v9q2x5"
@process.register(int)
def _(value):
    return value * 2
```

Это полезно, когда тип нельзя или неудобно указать через аннотацию.

---

# Получение зарегистрированных реализаций

У generic-функции есть:

```python id="c6m8r3"
process.registry
```

Можно посмотреть зарегистрированные типы:

```python id="a1f7k9"
print(process.registry)
```

Также можно проверить, какая реализация будет выбрана:

```python id="p4x2n8"
process.dispatch(int)
```

---

# `singledispatch` vs `overload`

Это часто спрашивают.

| `singledispatch`                     | `typing.overload`                  |
| ------------------------------------ | ---------------------------------- |
| Работает в runtime                   | Работает для статической типизации |
| Реально выбирает реализацию          | Не выбирает реализацию             |
| Диспетчеризация по первому аргументу | Описывает разные сигнатуры         |
| `functools`                          | `typing`                           |

То есть:

```python id="y8r4m2"
@process.register
```

→ **реальное runtime-поведение**

А:

```python id="u6k1p9"
@overload
```

→ **подсказка статическому анализатору**

---

# Почему это Ad hoc полиморфизм?

Потому что для разных конкретных типов можно определить **разные реализации одной операции**:

```text id="q3n7v5"
process()
   │
   ├── int    → реализация №1
   ├── str    → реализация №2
   ├── float  → реализация №3
   └── list   → реализация №4
```

Это хороший пример **ad hoc полиморфизма в Python**.

---

## 🧠 Коротко для собеседования

> **`functools.singledispatch` — механизм runtime-диспетчеризации, позволяющий зарегистрировать разные реализации одной функции для разных типов первого аргумента. Базовая функция создаётся через `@singledispatch`, а реализации регистрируются через `@func.register`. При вызове выбирается наиболее подходящая реализация с учётом иерархии типов.**

### Главное запомнить

```text id="8r5m2k"
@singledispatch
       ↓
generic function
       ↓
@func.register(int)
@func.register(str)
@func.register(float)
       ↓
runtime dispatch
       ↓
тип первого аргумента
```

**Ключевое слово:** `single` → диспетчеризация **по одному аргументу — первому**.
