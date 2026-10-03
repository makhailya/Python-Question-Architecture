# 🔁 `__next__`

🎯 **Ответ на собеседовании:**

`__next__` — специальный метод протокола итерации, который **возвращает следующий элемент итератора**.

Если элементы закончились, он должен выбросить исключение [[StopIteration]].

```python
next(iterator)
```

Фактически:

```python
iterator.__next__()
```

---

## Пример

```python
class Counter:
    def __init__(self, limit):
        self.current = 0
        self.limit = limit

    def __iter__(self):
        return self

    def __next__(self):
        if self.current >= self.limit:
            raise StopIteration

        self.current += 1
        return self.current
```

Использование:

```python
counter = Counter(3)

next(counter)  # 1
next(counter)  # 2
next(counter)  # 3
next(counter)  # StopIteration
```

---

# Как работает `__next__`

У итератора есть **состояние**, которое определяет, какой элемент вернуть следующим.

В нашем примере:

```python
self.current
```

хранит текущую позицию.

```text
current = 0
    ↓
next() → 1
    ↓
current = 1
    ↓
next() → 2
    ↓
current = 2
    ↓
next() → 3
    ↓
current = 3
    ↓
next() → StopIteration
```

---

# `__next__` и `next()`

`next()` — встроенная функция Python:

```python
next(iterator)
```

Она вызывает:

```python
iterator.__next__()
```

То есть:

```python
next(iterator)
```

и

```python
iterator.__next__()
```

по смыслу делают одно и то же.

---

# `StopIteration`

Когда элементов больше нет, `__next__` должен выбросить:

```python
raise StopIteration
```

Например:

```python
class Numbers:
    def __init__(self):
        self.numbers = [10, 20]
        self.index = 0

    def __iter__(self):
        return self

    def __next__(self):
        if self.index >= len(self.numbers):
            raise StopIteration

        value = self.numbers[self.index]
        self.index += 1
        return value
```

```python
it = Numbers()

next(it)  # 10
next(it)  # 20
next(it)  # StopIteration
```

---

# Как `for` использует `__next__`

Когда выполняется:

```python
for x in iterator:
    print(x)
```

Python концептуально делает:

```python
it = iter(iterator)

while True:
    try:
        x = next(it)
        print(x)
    except StopIteration:
        break
```

То есть `for` постоянно вызывает `__next__()`, пока тот не сообщит:

```text
StopIteration
```

---

# `__iter__` vs `__next__`

| Метод           | Что делает                         |
| --------------- | ---------------------------------- |
| `__iter__()`    | Возвращает итератор                |
| `__next__()`    | Возвращает следующий элемент       |
| `StopIteration` | Сообщает, что элементы закончились |

Связка:

```text
iter(obj)
   ↓
__iter__()
   ↓
iterator
   ↓
next(iterator)
   ↓
__next__()
   ↓
следующий элемент
   ↓
...
   ↓
StopIteration
```

---

# Важный момент

`__next__` должен быть у **итератора**, а не обязательно у любого iterable.

Например, список:

```python
numbers = [1, 2, 3]
```

является iterable:

```python
iter(numbers)
```

но сам список не является iterator:

```python
next(numbers)  # TypeError
```

Полученный итератор уже имеет `__next__`:

```python
it = iter(numbers)

next(it)  # 1
```

---

## 🧠 Коротко для собеседования

> **`__next__` — метод итератора, который возвращает следующий элемент. Когда элементы заканчиваются, он выбрасывает `StopIteration`. Встроенная функция `next()` вызывает этот метод, а цикл `for` использует его до получения `StopIteration`.**

**Запомнить:**

```text
__iter__  → КАК ПОЛУЧИТЬ ИТЕРАТОР
__next__  → КАК ПОЛУЧИТЬ СЛЕДУЮЩИЙ ЭЛЕМЕНТ
StopIteration → ЭЛЕМЕНТЫ ЗАКОНЧИЛИСЬ
```
