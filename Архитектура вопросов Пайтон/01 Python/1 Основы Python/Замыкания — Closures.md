# Замыкания — Closures

## Коротко

**Замыкание (closure)** — это функция, которая запоминает значения из внешней области видимости даже после завершения выполнения внешней функции.

Пример:

```python
def outer():
    message = "Hello"

    def inner():
        print(message)

    return inner
```

Получаем функцию:

```python
func = outer()
```

Хотя `outer()` уже завершилась, `func` всё ещё помнит `message`:

```python
func()
```

Результат:

```text
Hello
```

## На собеседовании достаточно сказать

> Замыкание — это функция, которая захватывает и сохраняет переменные из окружающей области видимости. Благодаря этому внутренняя функция может обращаться к переменным внешней функции даже после завершения её выполнения.

## Что важно помнить

### Базовый пример

```python
def outer():
    message = "Hello"

    def inner():
        print(message)

    return inner


func = outer()

func()
```

Результат:

```text
Hello
```

После:

```python
func = outer()
```

функция `outer()` завершилась.

Но `inner()` сохранила доступ к:

```python
message
```

---

## Как это связано с LEGB

Когда `inner()` выполняет:

```python
print(message)
```

Python ищет `message` по LEGB:

```text
L → Local
E → Enclosing  ← здесь находится message
G → Global
B → Built-in
```

То есть `message` находится в **Enclosing scope**.

Подробнее:

[[Область видимости и LEGB — Scope and LEGB]]

---

## Почему это называется замыканием

Можно представить:

```text
outer()
│
├── message = "Hello"
│
└── inner()
       │
       └── запоминает message
```

После завершения `outer()`:

```text
outer() завершилась
       ↓
inner() продолжает существовать
       ↓
inner() помнит message
```

Именно это и называется замыканием.

---

## Создание независимых замыканий

Можно создать несколько функций с разными сохранёнными значениями:

```python
def make_greeting(message):

    def greet():
        print(message)

    return greet


hello = make_greeting("Hello")
bye = make_greeting("Goodbye")

hello()
bye()
```

Результат:

```text
Hello
Goodbye
```

Каждое замыкание хранит своё значение:

```text
hello → message = "Hello"

bye   → message = "Goodbye"
```

---

## Замыкание может изменять переменную внешней функции

По умолчанию внутренняя функция может читать переменную внешней функции:

```python
def counter():
    count = 0

    def increment():
        print(count)

    return increment
```

Но если нужно изменить `count`, используется `nonlocal`:

```python
def counter():
    count = 0

    def increment():
        nonlocal count
        count += 1
        return count

    return increment
```

Теперь:

```python
counter_func = counter()

print(counter_func())
print(counter_func())
print(counter_func())
```

Результат:

```text
1
2
3
```

Переменная `count` сохраняется между вызовами.

Подробнее:

[[Область видимости и LEGB — Scope and LEGB]]

---

## Где применяются замыкания

Замыкания используются, например, для:

- создания функций с сохранённым состоянием;
- фабрик функций;
- декораторов;
- настройки поведения функции.

Особенно важны они при изучении декораторов.

Подробнее:

[[Декораторы — Decorators]]

---

## Простой пример

Создадим функцию, которая создаёт умножитель:

```python
def make_multiplier(n):

    def multiply(x):
        return x * n

    return multiply
```

Создадим функцию:

```python
double = make_multiplier(2)
```

Теперь:

```python
print(double(5))
print(double(10))
```

Результат:

```text
10
20
```

`double()` помнит значение:

```python
n = 2
```

Хотя функция `make_multiplier()` уже завершила выполнение.

Можно создать другую:

```python
triple = make_multiplier(3)

print(triple(5))
```

Результат:

```text
15
```

Теперь:

```text
double → n = 2
triple → n = 3
```

---

## Главное

Для замыкания нужны три вещи:

```text
1. Внешняя функция
        ↓
2. Внутренняя функция использует переменную внешней
        ↓
3. Внутренняя функция возвращается наружу
```

Например:

```python
def outer(value):

    def inner():
        return value

    return inner
```

После:

```python
func = outer(10)
```

`func` сохраняет доступ к `value`.

Главная формула:

> **Замыкание = функция + сохранённая ссылка на переменные окружающей области.**

И важно не путать:

```text
Scope
→ где имя доступно

Closure
→ функция сохраняет доступ к переменной внешней области
```

## Связи

- [[Функции — Functions]]
- [[Область видимости и LEGB — Scope and LEGB]]
- [[Переменные и присваивание — Variables and Assignment]]
- [[nonlocal]]
- [[Декораторы — Decorators]]