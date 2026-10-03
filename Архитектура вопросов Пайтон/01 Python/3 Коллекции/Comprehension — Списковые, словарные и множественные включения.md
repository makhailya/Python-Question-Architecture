# Comprehension — Списковые, словарные и множественные включения

## Коротко

**Comprehension** — это компактный синтаксис Python для создания коллекций на основе другой последовательности или итерируемого объекта.

Основные виды:

```text
List Comprehension → []
Dict Comprehension → {}
Set Comprehension → {}
```

Например, вместо:

```python
numbers = []

for i in range(5):
    numbers.append(i * 2)
```

можно написать:

```python
numbers = [i * 2 for i in range(5)]
```

Результат одинаковый:

```python
[0, 2, 4, 6, 8]
```

## На собеседовании достаточно сказать

> Comprehension — это компактный синтаксис Python для создания коллекций на основе итерируемого объекта. С помощью comprehension можно создавать списки, множества и словари, а также использовать условие для фильтрации элементов.

## Что важно помнить

### List Comprehension

Базовый синтаксис:

```python
[выражение for элемент in iterable]
```

Например:

```python
numbers = [1, 2, 3, 4]

squares = [number ** 2 for number in numbers]

print(squares)
```

Результат:

```text
[1, 4, 9, 16]
```

Обычный вариант:

```python
squares = []

for number in numbers:
    squares.append(number ** 2)
```

Comprehension делает то же самое компактнее:

```python
squares = [number ** 2 for number in numbers]
```

---

## List Comprehension с условием

Можно добавить `if`:

```python
numbers = [1, 2, 3, 4, 5, 6]

even_numbers = [
    number
    for number in numbers
    if number % 2 == 0
]
```

Результат:

```text
[2, 4, 6]
```

Здесь:

```python
if number % 2 == 0
```

фильтрует элементы.

Обычный цикл:

```python
even_numbers = []

for number in numbers:
    if number % 2 == 0:
        even_numbers.append(number)
```

---

## `if` в выражении

У comprehension есть ещё один вариант, когда `if/else` находится перед `for`.

Например:

```python
numbers = [1, 2, 3, 4]

result = [
    "even" if number % 2 == 0 else "odd"
    for number in numbers
]
```

Результат:

```text
["odd", "even", "odd", "even"]
```

Здесь:

```python
"even" if number % 2 == 0 else "odd"
```

определяет, **какое значение добавить**.

Это отличается от:

```python
for number in numbers
if number % 2 == 0
```

где `if` используется для **фильтрации**.

---

## Фильтрация vs условное выражение

Фильтрация:

```python
[
    number
    for number in numbers
    if number > 3
]
```

Получаем только подходящие элементы.

Условное выражение:

```python
[
    "big" if number > 3 else "small"
    for number in numbers
]
```

Добавляем элемент для каждой итерации, но выбираем его значение.

Важно различать:

```text
... for ... if ...
→ фильтрация

value1 if condition else value2
→ выбор значения
```

---

## Dict Comprehension

Можно создавать словари:

```python
numbers = [1, 2, 3, 4]

squares = {
    number: number ** 2
    for number in numbers
}
```

Результат:

```python
{
    1: 1,
    2: 4,
    3: 9,
    4: 16
}
```

Базовый синтаксис:

```python
{ключ: значение for элемент in iterable}
```

---

## Set Comprehension

Можно создавать множества:

```python
numbers = [1, 2, 2, 3, 3, 4]

unique = {
    number
    for number in numbers
}
```

Результат:

```text
{1, 2, 3, 4}
```

Повторяющиеся значения автоматически удаляются, потому что `set` хранит только уникальные элементы.

---

## Вложенные циклы

Comprehension может содержать несколько `for`.

Например:

```python
pairs = [
    (x, y)
    for x in [1, 2]
    for y in [10, 20]
]
```

Результат:

```text
[
    (1, 10),
    (1, 20),
    (2, 10),
    (2, 20)
]
```

Это эквивалентно:

```python
pairs = []

for x in [1, 2]:
    for y in [10, 20]:
        pairs.append((x, y))
```

Но вложенные comprehension быстро становятся сложными для чтения.

---

## Comprehension и вложенные структуры

Например, можно «расплющить» список списков:

```python
matrix = [
    [1, 2],
    [3, 4],
    [5, 6],
]

numbers = [
    number
    for row in matrix
    for number in row
]
```

Получим:

```text
[1, 2, 3, 4, 5, 6]
```

Порядок `for` соответствует вложенным циклам:

```python
for row in matrix:
    for number in row:
        ...
```

---

## Важный нюанс: comprehension создаёт новую коллекцию

Например:

```python
numbers = [1, 2, 3]

squares = [number ** 2 for number in numbers]
```

`numbers` и `squares` — разные списки:

```python
numbers is squares  # False
```

Comprehension создаёт новую коллекцию.

---

## Comprehension vs обычный цикл

Comprehension хорошо подходит, когда операция простая и понятная:

```python
squares = [x ** 2 for x in numbers]
```

Но если логика становится сложной, обычный цикл часто читается лучше:

```python
result = []

for item in items:
    if condition:
        ...
    else:
        ...
```

Не стоит использовать comprehension только ради того, чтобы сделать код короче.

Главный критерий:

> **Comprehension должен оставаться читаемым.**

---

## Простой пример

Получим квадраты только чётных чисел:

```python
numbers = [1, 2, 3, 4, 5, 6]

squares = [
    number ** 2
    for number in numbers
    if number % 2 == 0
]

print(squares)
```

Результат:

```text
[4, 16, 36]
```

Логика:

```text
берём числа
    ↓
проверяем, чётное ли число
    ↓
если да
    ↓
возводим в квадрат
    ↓
добавляем в новый список
```

---

## Главное

Основной List Comprehension:

```python
[expression for item in iterable]
```

С фильтрацией:

```python
[expression for item in iterable if condition]
```

Dict Comprehension:

```python
{key: value for item in iterable}
```

Set Comprehension:

```python
{expression for item in iterable}
```

Условное выражение:

```python
[value1 if condition else value2 for item in iterable]
```

Главная формула:

```text
Comprehension
├── list → [...]
├── dict → {...}
└── set  → {...}
```

Используй comprehension, когда он делает код **короче и понятнее**, а не просто короче.

## Связи

- [[Циклы — for и while]]
- [[Условия — if  elif  else]]
- [[Truthy и Falsy — Truthy and Falsy]]
- [[Распаковка — Unpacking]]
- [[list]]
- [[dict]]
- [[set]]