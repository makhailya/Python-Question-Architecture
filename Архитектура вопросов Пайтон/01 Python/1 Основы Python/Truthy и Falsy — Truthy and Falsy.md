# Truthy и Falsy — Truthy and Falsy

## Коротко

В Python любое значение в условии может интерпретироваться как `True` или `False`.

**Truthy** — значение, которое рассматривается как `True`.

**Falsy** — значение, которое рассматривается как `False`.

Например:

```python
if "hello":
    print("Truthy")
```

Строка `"hello"` — Truthy.

А пустая строка:

```python
if "":
    print("Falsy")
```

— Falsy.

## На собеседовании достаточно сказать

> В Python объекты имеют логическое представление. Значения, которые интерпретируются как `True`, называют Truthy, а как `False` — Falsy. К основным Falsy относятся `None`, `False`, числовой ноль, пустые коллекции и пустая строка. Остальные значения обычно Truthy.

## Что важно помнить

### Основные Falsy-значения

В Python к Falsy относятся:

```python
None
False
0
0.0
0j
""
[]
()
{}
set()
```

Например:

```python
bool(None)   # False
bool(False)  # False
bool(0)      # False
bool("")     # False
bool([])     # False
bool({})     # False
```

---

## Основные Truthy-значения

Большинство остальных объектов — Truthy.

Например:

```python
bool(1)          # True
bool(-1)         # True
bool("hello")    # True
bool([1, 2, 3])  # True
bool((1, 2))     # True
bool({"a": 1})   # True
```

Важно:

> Отрицательное число тоже Truthy.

```python
bool(-100)  # True
```

И непустая коллекция тоже Truthy:

```python
bool([0])  # True
```

Здесь список содержит `0`, но сам список **не пустой**.

---

## `bool()`

Для явного преобразования значения в логический тип используется `bool()`:

```python
bool(value)
```

Например:

```python
print(bool(10))
print(bool(0))
print(bool("Python"))
print(bool(""))
```

Результат:

```text
True
False
True
False
```

---

## Использование в `if`

Python автоматически проверяет Truthy/Falsy значение в условии:

```python
name = "Ilya"

if name:
    print("Имя указано")
```

Поскольку строка `"Ilya"` непустая, она Truthy.

Если:

```python
name = ""

if name:
    print("Имя указано")
else:
    print("Имя не указано")
```

получим:

```text
Имя не указано
```

---

## Проверка списка

Это особенно часто используется с коллекциями.

Вместо:

```python
if len(items) > 0:
    print("Список не пуст")
```

можно написать:

```python
if items:
    print("Список не пуст")
```

А вместо:

```python
if len(items) == 0:
    print("Список пуст")
```

можно:

```python
if not items:
    print("Список пуст")
```

Например:

```python
items = []

if not items:
    print("Список пуст")
```

---

## `not`

Оператор `not` инвертирует логическое значение.

```python
bool(0)      # False
not 0        # True
```

```python
bool(10)     # True
not 10       # False
```

Для Falsy:

```python
not None     # True
not ""       # True
not []       # True
```

Для Truthy:

```python
not "hello"  # False
not [1, 2]   # False
```

---

## `and` и `or`

Truthy/Falsy также важны при работе с `and` и `or`.

### `and`

```python
a = 10
b = 20

result = a and b

print(result)
```

Результат:

```text
20
```

Если первое значение Falsy, Python возвращает его:

```python
a = 0
b = 20

result = a and b

print(result)
```

Результат:

```text
0
```

### `or`

Если первое значение Truthy:

```python
a = 10
b = 20

result = a or b

print(result)
```

Результат:

```text
10
```

Если первое значение Falsy:

```python
a = 0
b = 20

result = a or b

print(result)
```

Результат:

```text
20
```

Важно:

> `and` и `or` возвращают один из своих операндов, а не обязательно `True` или `False`.

---

## `None` и Truthy/Falsy

`None` является Falsy:

```python
bool(None)  # False
```

Но проверять именно `None` лучше так:

```python
if value is None:
    ...
```

а не:

```python
if not value:
    ...
```

Потому что `not value` сработает и для других Falsy-значений.

Например:

```python
value = 0

if not value:
    print("Сработало")
```

Здесь условие сработает, хотя `value` — это `0`, а не `None`.

Поэтому:

```python
if value is None:
```

означает:

> значение именно `None`

А:

```python
if not value:
```

означает:

> значение является Falsy

Подробнее:

[[NoneType]]

---

## Простой пример

```python
users = []

if users:
    print("Пользователи есть")
else:
    print("Пользователей нет")
```

Результат:

```text
Пользователей нет
```

Если добавить пользователя:

```python
users = ["Ilya"]

if users:
    print("Пользователи есть")
else:
    print("Пользователей нет")
```

Результат:

```text
Пользователи есть
```

Причина:

```text
[]          → Falsy
["Ilya"]    → Truthy
```

---

## Таблица основных значений

| Значение | bool() |
|---|---:|
| `None` | `False` |
| `False` | `False` |
| `0` | `False` |
| `0.0` | `False` |
| `""` | `False` |
| `[]` | `False` |
| `()` | `False` |
| `{}` | `False` |
| `set()` | `False` |
| `1` | `True` |
| `-1` | `True` |
| `"hello"` | `True` |
| `[1]` | `True` |
| `(1,)` | `True` |
| `{"a": 1}` | `True` |

## Главное

Запомни:

```text
Falsy
├── None
├── False
├── 0
├── ""
├── []
├── ()
├── {}
└── set()

Остальные значения обычно → Truthy
```

В условиях Python автоматически использует это поведение:

```python
if value:
    ...
```

Проверка на Falsy:

```python
if not value:
    ...
```

Но если нужно проверить именно `None`:

```python
if value is None:
    ...
```

Главная формула:

> **Truthy → воспринимается как True**  
> **Falsy → воспринимается как False**

## Связи

- [[NoneType]]
- [[Равенство и идентичность — Equality vs Identity]]
- [[1. Типы Данных в Пайтон — Python Data Types]]
- [[Объект и ссылка — Object and Reference]]
- [[1.1 Изменяемые и неизменяемые типы данных — Mutable vs Immutable]]