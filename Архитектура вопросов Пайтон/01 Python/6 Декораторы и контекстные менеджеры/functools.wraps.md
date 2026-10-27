# 🧩 `functools.wraps`

## 🎤 Короткий ответ

`functools.wraps` — это декоратор, который используют внутри **декораторов**, чтобы сохранить метаданные исходной функции после её оборачивания.

Без `wraps` после декорирования функция фактически представлена объектом `wrapper`, поэтому теряются или меняются важные атрибуты, например:

- `__name__`
- `__doc__`
- `__module__`
- `__annotations__`
- `__wrapped__`

Пример:

```python
from functools import wraps

def log_calls(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print("Вызов функции")
        return func(*args, **kwargs)

    return wrapper
```

Главная идея:

> **`wraps` делает так, чтобы обёрнутая функция сохраняла метаданные оригинальной функции.**

---

## 🗣️ Ответ на собеседовании

`functools.wraps` используется при создании собственных декораторов.

Когда мы пишем декоратор, исходная функция заменяется функцией-обёрткой `wrapper`. Без `functools.wraps` у этой функции будут метаданные `wrapper`: например, имя будет `wrapper`, а docstring исходной функции потеряется.

`wraps` копирует основные метаданные исходной функции в обёртку и также устанавливает `__wrapped__`, через который можно получить исходную функцию.

Обычно декоратор выглядит так:

```python
from functools import wraps

def decorator(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)

    return wrapper
```

Это особенно важно для отладки, документации, introspection и инструментов, которые анализируют сигнатуры и функции.

---

## 🧭 Где я нахожусь

```text
Python
└── Функции
    └── Декораторы
        ├── Декоратор
        ├── *args и **kwargs
        ├── functools.wraps ← Я здесь
        ├── functools.partial
        └── Интроспекция функций
```

---

## 📚 Разбор поглубже

### 1. Зачем вообще нужен `wraps`

Рассмотрим простой декоратор:

```python
def log_calls(func):
    def wrapper(*args, **kwargs):
        print("Вызов")
        return func(*args, **kwargs)

    return wrapper
```

Используем:

```python
@log_calls
def calculate(a, b):
    """Складывает два числа."""
    return a + b
```

Python фактически делает:

```python
calculate = log_calls(calculate)
```

То есть имя `calculate` теперь ссылается на `wrapper`.

Проверим:

```python
print(calculate.__name__)
```

Получим:

```text
wrapper
```

И:

```python
print(calculate.__doc__)
```

Получим:

```text
None
```

Хотя исходная функция называлась `calculate` и имела docstring.

---

## 2. Что делает `@wraps`

Исправляем декоратор:

```python
from functools import wraps

def log_calls(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print("Вызов")
        return func(*args, **kwargs)

    return wrapper
```

Теперь:

```python
@log_calls
def calculate(a, b):
    """Складывает два числа."""
    return a + b
```

Проверяем:

```python
print(calculate.__name__)
```

Получаем:

```text
calculate
```

И:

```python
print(calculate.__doc__)
```

Получаем:

```text
Складывает два числа.
```

---

## 3. Что происходит под капотом

`wraps` — это удобная форма использования `functools.update_wrapper`.

Упрощённо:

```python
@wraps(func)
def wrapper(...):
    ...
```

означает:

```python
wrapper = wraps(func)(wrapper)
```

А `wraps(func)` возвращает функцию, которая настраивает `wrapper`.

Внутри используется `update_wrapper()`.

Упрощённо он переносит метаданные:

```text
Оригинальная функция
        │
        ├── __name__
        ├── __doc__
        ├── __module__
        ├── __annotations__
        └── ...
        │
        ↓
     wrapper
```

И дополнительно устанавливает:

```python
wrapper.__wrapped__ = func
```

---

## 4. `__wrapped__`

Это особенно важная часть.

После:

```python
from functools import wraps

def decorator(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)

    return wrapper
```

можно получить исходную функцию:

```python
original = decorated.__wrapped__
```

То есть:

```text
decorated
    ↓
wrapper
    ↓
__wrapped__
    ↓
original function
```

Это используется инструментами Python для introspection и иногда позволяет обходить слой декоратора.

---

## 5. `wraps` не «копирует функцию»

Это важный нюанс.

`wraps` **не превращает `wrapper` в исходную функцию**.

После декорирования всё ещё вызывается `wrapper`:

```python
@decorator
def hello():
    ...
```

Фактически:

```text
hello → wrapper → original hello
```

`wraps` в основном сохраняет метаданные и информацию о связи wrapper → original.

---

## 6. `wraps` и `*args`, `**kwargs`

Очень часто они используются вместе:

```python
from functools import wraps

def timer(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print("start")

        result = func(*args, **kwargs)

        print("finish")
        return result

    return wrapper
```

Здесь:

```text
*args
    ↓
позиционные аргументы

**kwargs
    ↓
именованные аргументы

@wraps(func)
    ↓
сохраняет метаданные func
```

Поэтому это очень типичный шаблон Python-декораторов.

---

## 7. Что именно сохраняет `wraps`

`wraps` использует `update_wrapper()` и по умолчанию переносит определённые атрибуты.

В частности:

```text
__module__
__name__
__qualname__
__doc__
__annotations__
```

Также обновляет `__dict__` wrapper-а атрибутами оригинальной функции и устанавливает:

```python
__wrapped__
```

Поэтому корректнее говорить:

> `wraps` сохраняет важные метаданные исходной функции.

А не:

> `wraps` копирует абсолютно все атрибуты функции.

---

## 8. Почему это важно для FastAPI и других инструментов

Декораторы используются не только для логирования.

Они могут встречаться в:

- FastAPI;
- Flask;
- pytest;
- кэшировании;
- логировании;
- авторизации;
- retry-механизмах;
- мониторинге;
- middleware-подобных обёртках.

Если декоратор неправильно сохраняет информацию о функции, инструменты introspection могут видеть не исходную функцию, а `wrapper`.

Например, имя:

```text
wrapper
```

вместо:

```text
get_user
```

может ухудшить:

- отладку;
- traceback;
- документацию;
- introspection;
- работу инструментов, анализирующих функцию.

---

## 9. `wraps` и сигнатура функции

Есть важный нюанс.

Допустим:

```python
def decorator(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)

    return wrapper
```

Реальная Python-сигнатура `wrapper`:

```python
(*args, **kwargs)
```

Но благодаря `__wrapped__` инструменты introspection могут восстановить сигнатуру исходной функции.

Например:

```python
import inspect

print(inspect.signature(decorated))
```

может показать сигнатуру исходной функции благодаря цепочке `__wrapped__`.

Это одна из причин, почему `wraps` важен не только для `__name__` и `__doc__`.

---

## 10. Что будет без `wraps`

### Без `wraps`

```python
def decorator(func):
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)

    return wrapper
```

Проблема:

```text
__name__        → wrapper
__doc__         → wrapper's docstring
__wrapped__     → отсутствует
```

### С `wraps`

```python
def decorator(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)

    return wrapper
```

Получаем:

```text
__name__        → оригинальное имя
__doc__         → оригинальный docstring
__wrapped__     → оригинальная функция
```

---

## 11. `wraps` vs `update_wrapper`

Они связаны:

```python
from functools import wraps, update_wrapper
```

`update_wrapper()` — основная функция, которая обновляет wrapper.

`wraps()` — удобная фабрика декораторов для использования именно в синтаксисе:

```python
@wraps(func)
def wrapper(...):
    ...
```

То есть:

```text
update_wrapper()
    ↓
низкоуровневая функция

wraps()
    ↓
удобный вариант для декораторов
```

---

## 12. Типичный production-шаблон

```python
from functools import wraps


def require_auth(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        check_auth()
        return func(*args, **kwargs)

    return wrapper
```

Использование:

```python
@require_auth
def get_profile(user_id):
    """Получить профиль пользователя."""
    ...
```

Теперь декоратор добавляет поведение:

```text
get_profile()
      ↓
wrapper()
      ↓
check_auth()
      ↓
original get_profile()
```

Но при этом метаданные `get_profile` сохраняются.

---

## 🎤 Вопросы на собеседовании

### Зачем нужен `functools.wraps`?

Чтобы при создании декоратора сохранить метаданные исходной функции и информацию о том, какую функцию оборачивает `wrapper`.

### Что будет без `wraps`?

После декорирования имя функции и другие метаданные могут указывать на `wrapper`, а `__wrapped__` не будет установлен.

### Что делает `@wraps(func)`?

Это удобный способ применить `functools.update_wrapper()` к функции-обёртке.

### Что такое `__wrapped__`?

Атрибут, который указывает на исходную функцию, обёрнутую декоратором.

### `wraps` сохраняет исходную функцию?

Нет. Исходная функция всё ещё вызывается через `wrapper`. `wraps` сохраняет метаданные и связь с оригиналом.

### Почему `wraps` часто используется вместе с `*args` и `**kwargs`?

`*args` и `**kwargs` позволяют wrapper принимать произвольную сигнатуру аргументов, а `wraps` сохраняет метаданные исходной функции.

### Нужно ли всегда использовать `wraps` в собственных декораторах?

Практически всегда, когда декоратор оборачивает обычную функцию и не требует намеренно скрывать её исходные метаданные. Это стандартный и рекомендуемый паттерн.