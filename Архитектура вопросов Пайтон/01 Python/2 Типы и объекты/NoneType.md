# `NoneType` — тип значения `None`

## 🎯 Ответ на собеседовании

> **`NoneType` — тип, единственный экземпляр которого в Python — `None`. `None` используется для обозначения отсутствия значения или результата. Функция, которая явно не возвращает значение через `return`, фактически возвращает `None`. Для проверки на `None` обычно используют оператор `is`: `value is None`.**

### Главное

```python id="j7b2e6"
type(None)
# <class 'NoneType'>
```

В Python существует только один объект этого типа:

```python id="j1j8q9"
None
```

То есть:

```python id="8hks4k"
type(None) is NoneType
```

концептуально верно, но напрямую имя `NoneType` обычно не используется.

Правильная проверка:

```python id="8t0h8f"
value is None
```

---

# Коротко

`None` означает:

> **значение отсутствует / значения нет**

Например:

```python id="n7k9h1"
user = None
```

Это не:

```text
0
""
False
[]
```

а отдельное специальное значение Python.

---

# Что такое `NoneType`?

```python id="6h9j7s"
type(None)
```

Результат:

```text id="k1v2g3"
<class 'NoneType'>
```

То есть:

```text id="1x7r4m"
None
 ↓
объект
 ↓
тип NoneType
```

И в Python существует **единственный объект `None`** этого типа.

---

# `None` — не строка

```python id="7q0n2s"
value = None
```

Это не:

```python id="f7x1q2"
"None"
```

Сравнение:

```python id="m8v3c1"
None == "None"
# False
```

---

# `None` — не `False`

Это тоже важно:

```python id="x2k7p9"
None == False
# False
```

Но:

```python id="r5v8m2"
bool(None)
# False
```

То есть `None` является **Falsy**, но сам `None` и `False` — разные значения.

---

# `None` — не `0`

```python id="h3j9k1"
None == 0
# False
```

И:

```python id="p8m4s6"
None == ""
# False
```

И:

```python id="v6q2r8"
None == []
# False
```

`None` — самостоятельное значение, обозначающее отсутствие значения.

---

# Где используется `None`?

## 1. Значение отсутствует

```python id="b7k2m4"
middle_name = None
```

Например, у пользователя нет отчества.

---

## 2. Функция ничего явно не возвращает

Очень важный момент.

```python id="c4n8p1"
def hello():
    print("Hello")
```

Если вызвать:

```python id="q9s3k5"
result = hello()

print(result)
```

Получим:

```text id="f2m6v8"
Hello
None
```

Почему?

Потому что функция без `return` автоматически возвращает `None`.

---

# `return` без значения

```python id="e6r1t4"
def check():
    return
```

Результат:

```python id="w8y2u6"
result = check()

print(result)
# None
```

То же самое:

```python id="a3f7j9"
def check():
    return None
```

---

# `None` как результат операции

Некоторые методы изменяют объект и ничего не возвращают.

Например:

```python id="k5m8q2"
numbers = [1, 2, 3]

result = numbers.append(4)

print(result)
# None
```

При этом список изменился:

```python id="u1p6s4"
print(numbers)
# [1, 2, 3, 4]
```

Это важная особенность:

```text id="b2v7n9"
append()
    ↓
изменяет list
    ↓
возвращает None
```

---

# Проверка на `None`

Правильный вариант:

```python id="c8m3x5"
if value is None:
    print("Значение отсутствует")
```

Для отрицания:

```python id="q1w6e8"
if value is not None:
    print("Значение существует")
```

---

# Почему `is`, а не `==`?

Обычно `None` проверяют через:

```python id="z4k9p2"
value is None
```

а не:

```python id="n6r1t7"
value == None
```

Причина:

* `==` проверяет **равенство**;
* `is` проверяет **идентичность объектов**.

`None` — singleton, то есть в Python существует один объект `None`.

Поэтому:

```python id="s3h7k1"
value is None
```

является идиоматическим способом проверки.

---

# `None` как default-параметр

Очень распространённый паттерн:

```python id="v9m2c6"
def get_user(user_id=None):
    if user_id is None:
        print("ID не передан")
```

Почему используют `None`?

Потому что он позволяет отличить:

```text id="p4x8r1"
аргумент не передали
```

от конкретного значения.

---

# ⚠️ Почему иногда нельзя использовать `False` или `0`

Представим функцию:

```python id="j6q1w8"
def process(value=None):
    if value is None:
        print("Значение не передали")
```

Теперь:

```python id="x3r7m5"
process(0)
```

`0` — это конкретное значение.

А:

```python id="e8k2v4"
process()
```

означает отсутствие значения.

Если бы мы проверяли:

```python id="t5n9c3"
if not value:
```

то `0`, `False`, `""`, `[]` и `None` могли бы попасть в одну ветку.

Поэтому для проверки именно отсутствия значения:

```python id="d1f6h9"
if value is None:
```

---

# `None` и Truthy/Falsy

```python id="k7m2p5"
bool(None)
# False
```

Поэтому:

```python id="w4c8n1"
if None:
    print("...")
```

не выполнится.

Но важно:

```python id="q9s5v3"
None
0
False
""
[]
{}
```

все являются Falsy, **но это разные типы и значения**.

---

# `None` в коллекциях

`None` можно хранить в любых коллекциях:

```python id="h2j7m4"
data = [1, None, 3]
```

```python id="p5s9k1"
data = {
    "name": "Илья",
    "age": None
}
```

```python id="v8c3r6"
data = (1, None, "Python")
```

---

# `None` и аннотации типов

В современном Python можно указать, что функция может вернуть значение или `None`:

```python id="n4x7q2"
def find_user(user_id: int) -> str | None:
    ...
```

Это означает:

```text id="m8p1z5"
результат может быть:

str
или
None
```

Например:

```python id="s6k2w9"
def find_user(user_id: int) -> str | None:
    if user_id == 1:
        return "Илья"

    return None
```

Это особенно часто встречается в backend-разработке.

---

# `None` в базах данных

В Python:

```python id="r3v8m1"
None
```

часто соответствует `NULL` в SQL.

Например:

```text id="c7h2q9"
Python       SQL

None    →    NULL
```

Но это **не одно и то же понятие на уровне реализации** — просто они часто соответствуют друг другу при работе с БД.

Например:

```python id="u5m9s2"
user.email = None
```

может означать `NULL` в соответствующем поле базы данных.

---

# `NoneType` vs `None`

Важно не путать:

```text id="f4k7p2"
None
   ↓
значение / объект

NoneType
   ↓
тип этого объекта
```

Аналогично:

```text id="z8n3c6"
10
 ↓
int

"Hello"
 ↓
str

None
 ↓
NoneType
```

---

# Проверка типа

Можно:

```python id="a2v6k9"
type(value) is type(None)
```

Но в реальном коде так почти никогда не делают.

Обычно:

```python id="s7p1m4"
value is None
```

Если нужно узнать тип:

```python id="q5r8x2"
type(value)
# <class 'NoneType'>
```

---

# 🧠 Шпаргалка

```text id="k3w7n1"
NoneType
│
├── тип объекта None
│
├── существует только один None
│
├── обозначает отсутствие значения
│
├── bool(None) → False
│
├── НЕ равно:
│     ├── 0
│     ├── False
│     ├── ""
│     └── []
│
├── функция без return → None
├── return без значения → None
│
├── None ↔ часто соответствует SQL NULL
│
└── проверка:
      value is None
      value is not None
```

## ⭐ Самое главное

```python id="j5q9s3"
def get_name():
    pass

result = get_name()

print(result)
# None

print(type(result))
# <class 'NoneType'>
```

Запомни формулу:

**`None` — специальное значение, `NoneType` — его тип.**

И на собеседовании особенно хорошо звучит:

> **«Для проверки именно на отсутствие значения я использую `is None`, а не `== None`, потому что `is` проверяет идентичность с singleton-объектом `None`.»**
