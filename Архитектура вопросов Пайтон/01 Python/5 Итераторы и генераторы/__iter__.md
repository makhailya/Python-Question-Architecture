# 🔄 `__iter__`

🎯 **Ответ на собеседовании:**

`__iter__` — специальный метод, который реализует **протокол итерации**. Он вызывается функцией `iter()` и должен вернуть **итератор**.

```python
iterator = iter(obj)
```

Фактически Python вызывает:

```python
iterator = obj.__iter__()
```

---

## Простой пример

```python
class Numbers:
    def __iter__(self):
        return iter([1, 2, 3])


numbers = Numbers()

iterator = iter(numbers)

print(next(iterator))  # 1
```

Здесь `Numbers` — **итерируемый объект**, а `iter([1, 2, 3])` возвращает итератор.

---

# `__iter__` у самого итератора

Если объект является **итератором**, его `__iter__` обычно возвращает `self`:

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

Теперь:

```python
counter = Counter(3)

iter(counter) is counter
# True
```

То есть итератор сам является итерируемым объектом.

---

# Как `for` использует `__iter__`

Когда пишем:

```python
for x in obj:
    print(x)
```

Python концептуально делает:

```python
iterator = iter(obj)

while True:
    try:
        x = next(iterator)
        print(x)
    except StopIteration:
        break
```

Поэтому `__iter__` нужен, чтобы получить объект, из которого `for` будет получать элементы.

---

# `__iter__` vs `__next__`

| Метод        | Назначение                 |
| ------------ | -------------------------- |
| `__iter__()` | Получить итератор          |
| `__next__()` | Получить следующий элемент |

Например:

```python
it = iter([10, 20, 30])

next(it)  # 10
next(it)  # 20
```

Здесь:

```text
iter(list)
    ↓
__iter__()
    ↓
iterator
    ↓
__next__()
    ↓
10 → 20 → 30 → StopIteration
```

---

# Iterable и Iterator

### Iterable

Имеет `__iter__()` и позволяет получить новый итератор.

```python
numbers = [1, 2, 3]

iter(numbers)
```

Список — iterable, но не iterator.

### Iterator

Имеет:

```python
__iter__()
__next__()
```

И обычно:

```python
iterator.__iter__() is iterator
```

---

# Почему `__iter__` должен возвращать итератор?

Потому что `iter(obj)` должен предоставить объект, поддерживающий `__next__()`.

Например:

```python
class Numbers:
    def __iter__(self):
        return iter([1, 2, 3])
```

Здесь всё правильно.

А так — неправильно:

```python
class Numbers:
    def __iter__(self):
        return [1, 2, 3]
```

Потому что список — iterable, но **не iterator**.

---

# Можно вернуть генератор

Очень распространённый вариант:

```python
class Numbers:
    def __iter__(self):
        for i in range(3):
            yield i
```

Генератор является итератором, поэтому это корректная реализация.

```python
numbers = Numbers()

for number in numbers:
    print(number)
```

---

## 🧠 Главное запомнить

```text
iter(obj)
   ↓
obj.__iter__()
   ↓
iterator
   ↓
next(iterator)
   ↓
iterator.__next__()
```

### Короткий ответ

> **`__iter__` — метод протокола итерации, который возвращает итератор. Функция `iter()` вызывает `__iter__()`. У самого итератора `__iter__()` обычно возвращает `self`, а `__next__()` выдаёт следующий элемент.**

**Связка, которую стоит знать наизусть:**

`__iter__` → **получить итератор**
`__next__` → **получить следующий элемент**
`StopIteration` → **элементы закончились**
