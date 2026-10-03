# type() и isinstance() — type and isinstance()

## Коротко

`type()` возвращает **точный тип объекта**.

`isinstance()` проверяет, является ли объект экземпляром указанного типа **или его наследника**.

Пример:

```python
x = 10

type(x) is int
# True

isinstance(x, int)
# True
```

Для проверки типа объекта обычно используют `isinstance()`.

## На собеседовании достаточно сказать

> `type()` возвращает точный тип объекта, а `isinstance()` проверяет, является ли объект экземпляром указанного класса или его наследника. Поэтому для проверки типа обычно используют `isinstance()`, особенно если нужно учитывать наследование.

## Что важно помнить

### `type()`

`type()` возвращает точный тип объекта:

```python
x = 10

type(x)
# <class 'int'>
```

Можно проверить точное соответствие типа:

```python
type(x) is int
# True
```

Важно понимать, что `type()` позволяет проверить именно **конкретный тип объекта**.

---

## `isinstance()`

`isinstance()` проверяет, является ли объект экземпляром указанного класса:

```python
x = 10

isinstance(x, int)
# True
```

Можно передать несколько типов:

```python
x = 10

isinstance(x, (int, float))
# True
```

В этом случае проверяется:

> Является ли `x` экземпляром `int` или `float`?

---

## Главное отличие — наследование

Рассмотрим пример:

```python
class Animal:
    pass


class Dog(Animal):
    pass


dog = Dog()
```

Проверим через `type()`:

```python
type(dog) is Dog
# True

type(dog) is Animal
# False
```

`dog` имеет точный тип `Dog`, поэтому проверка на `Animal` возвращает `False`.

Теперь проверим через `isinstance()`:

```python
isinstance(dog, Dog)
# True

isinstance(dog, Animal)
# True
```

Почему?

Потому что `Dog` является наследником `Animal`.

`isinstance()` учитывает наследование.

`type() is` проверяет только точное соответствие типа.

---

## Сравнение

```python
type(x) is SomeClass
```

означает:

> Объект имеет точно этот класс?

А:

```python
isinstance(x, SomeClass)
```

означает:

> Объект является экземпляром этого класса или его наследника?

---

## Проверка нескольких типов

`isinstance()` позволяет передать кортеж типов:

```python
value = 10

isinstance(value, (int, float))
# True
```

Можно использовать это, например, когда функция принимает несколько типов данных:

```python
def process(value):

    if isinstance(value, (int, float)):
        return value * 2

    return None
```

Теперь:

```python
process(10)
# 20

process(2.5)
# 5.0

process("10")
# None
```

---

## `bool` и `int` — важный нюанс

В Python `bool` является подклассом `int`.

Поэтому:

```python
isinstance(True, int)
# True
```

Но:

```python
type(True) is int
# False
```

Потому что точный тип объекта — `bool`:

```python
type(True) is bool
# True
```

Это хороший пример различия между `type()` и `isinstance()`.

---

## Когда использовать `type()`

`type()` полезен, когда нужен **точный тип объекта**:

```python
type(value) is int
```

Например:

```python
value = 10

if type(value) is int:
    print("Это именно int")
```

Но для обычной проверки типа чаще используют `isinstance()`.

---

## Когда использовать `isinstance()`

`isinstance()` подходит, когда нужно проверить принадлежность объекта к классу с учётом наследования:

```python
if isinstance(value, Animal):
    ...
```

Также `isinstance()` удобно использовать для проверки нескольких допустимых типов:

```python
if isinstance(value, (int, float)):
    ...
```

---

## Простой пример

Напишем функцию, которая определяет, является ли объект строкой:

```python
def check_value(value):

    if isinstance(value, str):
        return "Это строка"

    return "Это не строка"
```

Использование:

```python
check_value("Python")
# "Это строка"

check_value(123)
# "Это не строка"
```

---

## Главное

| Проверка                      | Что делает                        |
| ----------------------------- | --------------------------------- |
| `type(x)`                     | Возвращает тип объекта            |
| `type(x) is int`              | Проверяет точный тип `int`        |
| `isinstance(x, int)`          | Проверяет `int` и его наследников |
| `isinstance(x, (int, float))` | Проверяет несколько типов         |

Главное различие:

```python
type(x) is SomeClass
```

→ **точно этот класс**

```python
isinstance(x, SomeClass)
```

→ **этот класс или его наследник**

Для обычной проверки типа:

```python
isinstance(x, SomeClass)
```

обычно предпочтительнее.

## Связи

- [[1. Типы Данных в Пайтон — Python Data Types]]
- [[Объект и ссылка — Object and Reference]]
- [[Равенство и идентичность — Equality and Identity]]
- [[ООП — Object-Oriented Programming]]
