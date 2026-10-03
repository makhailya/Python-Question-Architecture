# Циклы — for / while

## Коротко

Циклы позволяют многократно выполнять один и тот же блок кода.

В Python основные циклы:

- `for` — перебирает элементы итерируемого объекта;
- `while` — выполняется, пока условие истинно.

Пример `for`:

```python
for number in [1, 2, 3]:
    print(number)
```

Пример `while`:

```python
x = 0

while x < 3:
    print(x)
    x += 1
```

## На собеседовании достаточно сказать

> В Python есть два основных цикла: `for` и `while`. `for` используется для последовательного перебора элементов итерируемого объекта, а `while` выполняется до тех пор, пока его условие истинно. Управлять выполнением цикла можно с помощью `break`, `continue` и `pass`.

## Что важно помнить

### `for`

`for` перебирает элементы итерируемого объекта:

```python
numbers = [10, 20, 30]

for number in numbers:
    print(number)
```

Результат:

```text
10
20
30
```

На каждой итерации переменная `number` получает очередной элемент:

```text
number = 10
number = 20
number = 30
```

---

## `for` со строкой

Строка тоже является итерируемым объектом:

```python
for letter in "Python":
    print(letter)
```

Результат:

```text
P
y
t
h
o
n
```

---

## `range()`

Для перебора последовательности чисел часто используется `range()`:

```python
for i in range(5):
    print(i)
```

Результат:

```text
0
1
2
3
4
```

Важно:

```python
range(5)
```

не включает `5`.

Можно задать начало и конец:

```python
for i in range(2, 5):
    print(i)
```

Результат:

```text
2
3
4
```

Можно задать шаг:

```python
for i in range(0, 10, 2):
    print(i)
```

Результат:

```text
0
2
4
6
8
```

---

## `while`

`while` выполняет код, пока условие Truthy:

```python
x = 0

while x < 3:
    print(x)
    x += 1
```

Результат:

```text
0
1
2
```

После каждой итерации проверяется условие:

```text
x < 3
```

Когда оно становится `False`, цикл завершается.

---

## Бесконечный цикл

Если условие `while` никогда не становится `False`, цикл будет выполняться бесконечно:

```python
while True:
    print("Hello")
```

Такой цикл обычно завершают с помощью `break`:

```python
while True:
    command = input()

    if command == "exit":
        break
```

---

## `for` и `while`: разница

### `for`

Используется, когда нужно перебрать элементы:

```python
for item in items:
    print(item)
```

### `while`

Используется, когда выполнение зависит от условия:

```python
while condition:
    ...
```

Упрощённо:

```text
for
→ перебираем элементы

while
→ выполняем, пока условие истинно
```

---

## Вложенные циклы

Один цикл можно поместить внутрь другого:

```python
for i in range(3):
    for j in range(2):
        print(i, j)
```

Результат:

```text
0 0
0 1
1 0
1 1
2 0
2 1
```

Внутренний цикл полностью выполняется для каждой итерации внешнего.

---

## `break`

`break` полностью прекращает выполнение текущего цикла:

```python
for number in range(10):

    if number == 5:
        break

    print(number)
```

Результат:

```text
0
1
2
3
4
```

Когда `number` становится `5`, цикл завершается.

Подробнее:

[[break, continue и pass]]

---

## `continue`

`continue` пропускает текущую итерацию и переходит к следующей:

```python
for number in range(5):

    if number == 2:
        continue

    print(number)
```

Результат:

```text
0
1
3
4
```

Подробнее:

[[break, continue и pass]]

---

## `else` у цикла

У циклов Python есть необязательный `else`.

Он выполняется, если цикл завершился **обычным способом**, без `break`.

Например:

```python
for number in range(3):
    print(number)
else:
    print("Цикл завершён")
```

Результат:

```text
0
1
2
Цикл завершён
```

Но если был `break`:

```python
for number in range(3):

    if number == 1:
        break

    print(number)
else:
    print("Цикл завершён")
```

Результат:

```text
0
```

`else` не выполняется.

Это часто спрашивают на собеседованиях.

---

## Простой пример

Найдём первое число, которое делится на 7:

```python
numbers = [3, 8, 12, 14, 20]

for number in numbers:
    if number % 7 == 0:
        print(number)
        break
```

Результат:

```text
14
```

Цикл перебирает числа и останавливается, когда находит подходящее.

---

## Главное

`for`:

```python
for item in iterable:
    ...
```

→ перебирает элементы итерируемого объекта.

`while`:

```python
while condition:
    ...
```

→ выполняется, пока условие истинно.

`break`:

```text
полностью завершает цикл
```

`continue`:

```text
пропускает текущую итерацию
```

`else` у цикла:

```text
выполняется, если цикл завершился без break
```

Главная формула:

> **`for` → перебор**  
> **`while` → условие**  
> **`break` → выйти**  
> **`continue` → следующая итерация**

## Связи

- [[Условия — if  elif  else]]
- [[Truthy и Falsy — Truthy and Falsy]]
- [[break, continue и pass]]
- [[Распаковка — Unpacking]]
- [[Индексация и срезы — Indexing and Slicing]]