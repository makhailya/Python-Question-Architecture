# `bool` — логический тип

## 🎯 Ответ на собеседовании

> **`bool` — встроенный логический тип Python, который имеет всего два значения: `True` и `False`. В Python `bool` является подклассом `int`: `True` фактически соответствует `1`, а `False` — `0`. Тип `bool` immutable.**

### Если спросят про наследование

```python
isinstance(True, int)
# True
```

> **`bool` наследуется от `int`, поэтому `True` и `False` участвуют в некоторых числовых операциях как `1` и `0`.**

### Если спросят про `is` и `==`

```python
True == 1
# True

True is 1
# False
```

`==` сравнивает **значения**, а `is` — **идентичность объектов**.

---

# Коротко

`bool` используется для представления логического состояния:

```python
is_active = True
is_admin = False
```

Всего два значения:

```python
True
False
```

Важно: `True` и `False` пишутся с **заглавной первой буквы**.

```python
true   # NameError
false  # NameError
```

Правильно:

```python
True
False
```

---

# Тип `bool`

```python
type(True)
# <class 'bool'>

type(False)
# <class 'bool'>
```

Проверка:

```python
isinstance(True, bool)
# True
```

---

# `bool` является подклассом `int`

Это одна из особенностей Python:

```python
issubclass(bool, int)
# True
```

Поэтому:

```python
isinstance(True, int)
# True
```

И:

```python
isinstance(False, int)
# True
```

Можно воспринимать:

```text
True  → 1
False → 0
```

---

# `bool` в математических операциях

Из-за наследования от `int`:

```python
True + True
# 2
```

```python
True + False
# 1
```

```python
False * 10
# 0
```

Но в обычном коде использовать `bool` как число стоит только тогда, когда это действительно делает логику понятнее.

---

# `bool()` — преобразование в Boolean

Функция `bool()` преобразует значение в `True` или `False`.

```python
bool(1)
# True

bool(0)
# False
```

```python
bool("hello")
# True

bool("")
# False
```

---

# Truthy и Falsy

В Python объекты могут иметь **логическое значение**, даже если они сами не являются `bool`.

Например:

```python
bool(10)
# True

bool(-5)
# True

bool(0)
# False
```

Для строк:

```python
bool("Python")
# True

bool("")
# False
```

Для списков:

```python
bool([1, 2, 3])
# True

bool([])
# False
```

### Основные Falsy-значения

К `False` при приведении к `bool` относятся:

```text
False
None
0
0.0
0j
""
[]
()
{}
set()
```

Остальные значения в обычных случаях являются **Truthy**.

---

# `bool` в `if`

Это основное применение:

```python
is_logged_in = True

if is_logged_in:
    print("Пользователь авторизован")
```

Необязательно писать:

```python
if is_logged_in == True:
```

Обычно правильнее:

```python
if is_logged_in:
```

Для отрицания:

```python
if not is_logged_in:
    print("Не авторизован")
```

---

# Логические операторы

## `and`

Возвращает результат логического И:

```python
True and True
# True

True and False
# False
```

## `or`

Логическое ИЛИ:

```python
True or False
# True

False or False
# False
```

## `not`

Отрицание:

```python
not True
# False

not False
# True
```

---

# ⚠️ Важная особенность `and` и `or`

На самом деле `and` и `or` **не обязательно возвращают `bool`**.

Они возвращают один из операндов.

Например:

```python
0 or 10
# 10
```

```python
10 or 20
# 10
```

```python
10 and 20
# 20
```

```python
0 and 20
# 0
```

Это часто спрашивают на собеседованиях.

### Как это работает

`or` ищет **первое Truthy-значение**:

```python
"" or "Python"
# "Python"
```

`and` ищет **первое Falsy-значение**, а если его нет — возвращает последний операнд:

```python
10 and 20
# 20
```

---

# Сравнения возвращают `bool`

Например:

```python
10 > 5
# True

10 == 5
# False
```

```python
"abc" == "abc"
# True
```

Результат операторов сравнения — `bool`.

---

# `bool` и `None`

`None` не является `False`:

```python
None == False
# False
```

Но:

```python
bool(None)
# False
```

То есть `None` является **Falsy**, но это другой объект и другой тип.

---

# `bool` и `is`

Для проверки именно `True` или `False` можно использовать:

```python
if value is True:
    ...
```

Но чаще достаточно:

```python
if value:
    ...
```

Для проверки `None` рекомендуется именно:

```python
if value is None:
    ...
```

а не:

```python
if value == None:
```

---

# `bool` — immutable

`bool` является неизменяемым типом.

Значения:

```python
True
False
```

нельзя изменить.

Переменная может начать ссылаться на другое значение:

```python
is_active = True

is_active = False
```

Но объект `True` при этом не изменяется.

---

# Где используется?

`bool` используется практически во всех программах:

* условия;
* проверки;
* флаги;
* состояния;
* разрешения;
* результаты сравнений;
* логика программы.

Например:

```python
is_authenticated = True
has_access = False
is_active = True
```

---

# 🧠 Шпаргалка

```text
bool
│
├── True / False
│
├── immutable
│
├── bool — подкласс int
│     │
│     ├── True  → 1
│     └── False → 0
│
├── bool() → преобразование в Boolean
│
├── Truthy / Falsy
│
├── and → логическое И
├── or  → логическое ИЛИ
└── not → отрицание
```

### ⭐ Что обязательно помнить

```python
True == 1
# True

True is 1
# False

isinstance(True, int)
# True

bool(0)
# False

bool(1)
# True

bool("")
# False

bool("0")
# True
```

**Главная мысль:**

> `bool` — логический тип с двумя значениями `True` и `False`, причём в Python он является подклассом `int`.
