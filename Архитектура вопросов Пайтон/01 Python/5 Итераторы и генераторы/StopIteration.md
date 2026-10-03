# 🛑 `StopIteration`

🎯 **Ответ на собеседовании:**

`StopIteration` — специальное исключение Python, которое **сообщает, что итератор больше не содержит элементов**.

Обычно оно выбрасывается методом `__next__()`:

```python
def __next__(self):
    if закончились_элементы:
        raise StopIteration
```

---

## Пример

```python
numbers = iter([10, 20])

print(next(numbers))  # 10
print(next(numbers))  # 20
print(next(numbers))  # StopIteration
```

После `20` итератор сообщает, что элементов больше нет.

---

# Как `for` использует `StopIteration`

Когда мы пишем:

```python
for number in numbers:
    print(number)
```

Python концептуально делает:

```python
iterator = iter(numbers)

while True:
    try:
        number = next(iterator)
    except StopIteration:
        break

    print(number)
```

То есть `for` **перехватывает `StopIteration` и завершает цикл**.

Поэтому обычно мы не видим это исключение при использовании `for`.

---

# `StopIteration` ≠ ошибка

Хотя технически это исключение (`Exception`), в контексте итераторов оно является **нормальным механизмом завершения итерации**.

```text
next()
  ↓
есть элемент?
  ├── Да → вернуть элемент
  │
  └── Нет → StopIteration
                    ↓
                  for → break
```

---

# В собственном итераторе

Например:

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

```python
counter = Counter(3)

for value in counter:
    print(value)
```

Результат:

```text
1
2
3
```

После `3` вызывается `__next__()`, который выбрасывает `StopIteration`, и `for` заканчивается.

---

# Важный момент: `StopIteration` не возвращается

Неправильно говорить:

> "`__next__` возвращает `StopIteration`."

Правильно:

> **`__next__` возвращает следующий элемент или выбрасывает `StopIteration`, когда элементы закончились.**

То есть:

```python
return value
```

или:

```python
raise StopIteration
```

---

# Генераторы

У генераторов `StopIteration` возникает автоматически после завершения функции:

```python
def numbers():
    yield 1
    yield 2
```

```python
it = numbers()

next(it)  # 1
next(it)  # 2
next(it)  # StopIteration
```

При обычном использовании `for` это также скрыто:

```python
for number in numbers():
    print(number)
```

---

## 🧠 Коротко для собеседования

> **`StopIteration` — специальное исключение, сигнализирующее об окончании итерации. `__next__()` выбрасывает его, когда элементов больше нет, а `for` перехватывает `StopIteration` и завершает цикл.**

### Связка

```text
__iter__()       → получить итератор
      ↓
__next__()       → получить следующий элемент
      ↓
элементы закончились
      ↓
StopIteration
      ↓
for завершает цикл
```
