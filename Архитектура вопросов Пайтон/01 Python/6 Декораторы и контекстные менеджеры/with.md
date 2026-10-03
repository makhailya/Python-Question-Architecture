# 🔐 `with`

🎯 **Ответ на собеседовании:**

`with` — конструкция Python для работы с **контекстными менеджерами**. Она гарантирует выполнение действий при входе и выходе из блока, например освобождение ресурса даже при возникновении исключения.

```python
with open("data.txt") as file:
    data = file.read()
```

После выхода из блока файл будет закрыт.

---

# Как работает `with`

Контекстный менеджер должен реализовать:

```python
__enter__()
__exit__()
```

Упрощённо:

```python
with manager() as value:
    do_something()
```

работает примерно так:

```python
manager = manager()

value = manager.__enter__()

try:
    do_something()
except BaseException as e:
    if not manager.__exit__(type(e), e, e.__traceback__):
        raise
else:
    manager.__exit__(None, None, None)
```

То есть:

```text
with
 ↓
__enter__()
 ↓
код внутри блока
 ↓
__exit__()
 ↓
выход / обработка исключения
```

---

# `as`

Переменная после `as` получает результат `__enter__()`:

```python
with manager() as value:
    ...
```

примерно:

```python
value = manager.__enter__()
```

Например:

```python
class Connection:
    def __enter__(self):
        self.connection = connect()
        return self.connection

    def __exit__(self, exc_type, exc_value, traceback):
        self.connection.close()


with Connection() as connection:
    connection.execute(...)
```

Здесь `connection` — результат `__enter__()`.

---

# Что происходит при исключении

```python
with manager():
    raise ValueError("Ошибка")
```

Python вызывает:

```python
manager.__exit__(
    ValueError,
    exception,
    traceback
)
```

Если `__exit__` возвращает:

```python
True
```

исключение **подавляется**.

Если:

```python
False
```

или `None`, исключение продолжает распространяться.

---

# Почему `with` полезен?

Главная задача — **гарантированное управление ресурсами**.

Без `with`:

```python
file = open("data.txt")

try:
    data = file.read()
finally:
    file.close()
```

С `with`:

```python
with open("data.txt") as file:
    data = file.read()
```

`with` делает код:

* короче;
* безопаснее;
* понятнее;
* менее подверженным утечкам ресурсов.

---

# Что можно использовать с `with`

Контекстные менеджеры применяются не только к файлам.

### Файлы

```python
with open("data.txt") as file:
    ...
```

### Блокировки

```python
with lock:
    shared_data.append(value)
```

После блока блокировка освобождается.

### Соединения с БД

```python
with connection:
    ...
```

### Транзакции

```python
with transaction:
    ...
```

### Временное изменение состояния

```python
with temporary_settings():
    ...
```

---

# `with` и несколько контекстов

Можно открыть несколько ресурсов:

```python
with open("input.txt") as src, open("output.txt", "w") as dst:
    data = src.read()
    dst.write(data)
```

Это эквивалентно вложенным `with`:

```python
with open("input.txt") as src:
    with open("output.txt", "w") as dst:
        ...
```

Контексты закрываются **в обратном порядке**:

```text
src.__enter__()
dst.__enter__()

код

dst.__exit__()
src.__exit__()
```

---

# `with` vs `try/finally`

`with` удобно использовать, когда нужно гарантировать cleanup:

```python
resource = acquire()

try:
    use(resource)
finally:
    release(resource)
```

Вместо этого можно инкапсулировать управление ресурсом:

```python
with resource_manager() as resource:
    use(resource)
```

Именно контекстный менеджер отвечает за:

```text
__enter__ → acquire / setup
__exit__  → release / cleanup
```

---

# `with` не только для ресурсов

Контекстный менеджер может временно изменять состояние программы:

```python
with decimal.localcontext() as ctx:
    ctx.prec = 50
```

После выхода состояние возвращается к предыдущему контексту.

То есть более общее определение:

> `with` используется для безопасного управления **ресурсом или временным состоянием**.

---

# Асинхронный `with`

Для асинхронного кода существует:

```python
async with
```

Он работает через:

```python
__aenter__()
__aexit__()
```

Пример:

```python
async with session.get(url) as response:
    data = await response.text()
```

Связка:

```text
with       → __enter__ / __exit__
async with → __aenter__ / __aexit__
```

---

## 🧠 Коротко для собеседования

> **`with` — конструкция Python для работы с контекстными менеджерами. При входе вызывается `__enter__`, его результат попадает в `as`, после выполнения блока вызывается `__exit__`. `__exit__` вызывается также при исключении и может подавить его, если вернёт `True`. Основное применение — безопасное управление ресурсами и гарантированный cleanup.**

### Запомнить

```text
with manager() as value:
        ↓
__enter__()
        ↓
value
        ↓
код
        ↓
__exit__()
```

И для async:

```text
with       → __enter__ / __exit__
async with → __aenter__ / __aexit__
```
