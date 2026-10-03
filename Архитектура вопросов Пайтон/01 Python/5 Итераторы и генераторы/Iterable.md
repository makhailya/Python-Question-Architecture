# 🔄 Iterable

🎯 **Ответ на собеседовании:**

**Iterable (итерируемый объект)** — это объект, из которого можно получить **итератор** с помощью функции `iter()`.

```python
iterator = iter(iterable)
```

Например:

```python
numbers = [1, 2, 3]

iterator = iter(numbers)
```

`list` — это `Iterable`, а полученный через `iter()` объект — `Iterator`.

---

## Как Python определяет Iterable

Обычно объект является итерируемым, если он предоставляет метод:

```python
__iter__()
```

который возвращает итератор.

```python
class Numbers:
    def __iter__(self):
        return iter([1, 2, 3])
```

Теперь:

```python
numbers = Numbers()

for number in numbers:
    print(number)
```

---

# Примеры Iterable

Многие стандартные контейнеры являются итерируемыми:

```python
list
tuple
str
dict
set
range
```

Например:

```python
for char in "Python":
    print(char)
```

или:

```python
for key in {"name": "Ilya", "age": 31}:
    print(key)
```

---

# Iterable vs Iterator

Это **ключевое различие**.

|                               | Iterable               | Iterator       |
| ----------------------------- | ---------------------- | -------------- |
| `__iter__()`                  | ✅                      | ✅              |
| `__next__()`                  | Не обязательно         | ✅              |
| Можно получить через `iter()` | ✅                      | ✅              |
| Получает следующий элемент    | Через итератор         | Через `next()` |
| Пример                        | `list`, `tuple`, `str` | `iter(list)`   |

Например:

```python
numbers = [1, 2, 3]
```

```python
iter(numbers)      # получаем iterator
```

А:

```python
iterator = iter(numbers)

next(iterator)    # 1
next(iterator)    # 2
```

---

# Почему список — Iterable, но не Iterator?

```python
numbers = [1, 2, 3]

iter(numbers)  # работает
```

Но:

```python
next(numbers)
```

❌

```text
TypeError: 'list' object is not an iterator
```

Потому что список умеет **создавать итератор**, но сам не обязан хранить состояние текущего обхода.

---

# `for` работает с Iterable

Когда Python видит:

```python
for item in iterable:
    ...
```

он сначала получает итератор:

```python
iterator = iter(iterable)
```

а затем вызывает:

```python
next(iterator)
```

до тех пор, пока не получит:

```python
StopIteration
```

Упрощённо:

```text
Iterable
   ↓
iter()
   ↓
Iterator
   ↓
next()
   ↓
элемент
   ↓
next()
   ↓
элемент
   ↓
StopIteration
```

---

# Iterable может создавать новый итератор

Это важное отличие от самого итератора.

```python
numbers = [1, 2, 3]

it1 = iter(numbers)
it2 = iter(numbers)
```

`it1` и `it2` — **разные итераторы**:

```python
it1 is it2
# False
```

И каждый имеет собственное состояние обхода.

```python
next(it1)  # 1
next(it1)  # 2

next(it2)  # 1
```

---

# Собственный Iterable

Можно сделать класс, который возвращает новый итератор:

```python
class Numbers:
    def __init__(self, numbers):
        self.numbers = numbers

    def __iter__(self):
        return iter(self.numbers)
```

```python
numbers = Numbers([10, 20, 30])

for number in numbers:
    print(number)
```

Здесь:

```text
Numbers → Iterable
list     → Iterable
iter(list) → Iterator
```

---

# Iterable и генераторы

Генератор является **Iterator**, а значит, одновременно является и `Iterable`.

```python
def numbers():
    yield 1
    yield 2
```

```python
gen = numbers()

iter(gen) is gen
# True
```

То есть:

> **Каждый Iterator является Iterable, но не каждый Iterable является Iterator.**

Это очень хорошая формулировка для собеседования.

---

## 🧠 Коротко для собеседования

> **Iterable — это объект, из которого можно получить итератор через `iter()`. Обычно он реализует `__iter__()`. Например, `list`, `tuple`, `str`, `dict` и `set` являются Iterable. Сам Iterable не обязан иметь `__next__()` — следующий элемент выдаёт уже Iterator.**

### Запомнить

```text
Iterable → __iter__() → Iterator
Iterator → __next__() → следующий элемент
                              ↓
                       StopIteration
```

**Главная формула:**

> **Iterable — можно итерировать. Iterator — умеет выдавать следующий элемент.**
