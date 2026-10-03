# 🔓 `__enter__`

🎯 **Ответ на собеседовании:**

`__enter__` — специальный метод **протокола контекстного менеджера**, который автоматически вызывается при входе в блок `with`.

```python
with manager() as value:
    ...
```

При входе Python вызывает:

```python
value = manager.__enter__()
```

---

## Пример

```python
class Connection:
    def __enter__(self):
        print("Соединение открыто")
        return self

    def __exit__(self, exc_type, exc_value, traceback):
        print("Соединение закрыто")
```

Использование:

```python
with Connection() as connection:
    print("Работаем")
```

Результат:

```text
Соединение открыто
Работаем
Соединение закрыто
```

---

# Что возвращает `__enter__`?

Значение, возвращённое `__enter__`, попадает в переменную после `as`:

```python
with manager() as value:
    ...
```

примерно:

```python
value = manager.__enter__()
```

Поэтому можно вернуть:

```python
return self
```

или какой-либо другой объект:

```python
class FileManager:
    def __enter__(self):
        self.file = open("data.txt")
        return self.file
```

Теперь:

```python
with FileManager() as file:
    data = file.read()
```

Здесь `file` — это именно результат `__enter__()`.

---

# `__enter__` и `__exit__`

Они работают вместе:

```text
with manager() as value:
        ↓
__enter__()
        ↓
value получает результат
        ↓
код внутри with
        ↓
__exit__()
        ↓
очистка
```

Минимальная реализация:

```python
class Manager:
    def __enter__(self):
        print("Вход")
        return self

    def __exit__(self, exc_type, exc_value, traceback):
        print("Выход")
```

---

# Где выполняется подготовка ресурса?

Обычно в `__enter__`:

```python
class Database:
    def __enter__(self):
        self.connection = connect()
        return self.connection

    def __exit__(self, exc_type, exc_value, traceback):
        self.connection.close()
```

То есть:

```text
__enter__ → acquire / setup
__exit__  → release / cleanup
```

---

# Что если `__enter__` выбросит исключение?

Если ошибка произошла внутри `__enter__`, тело `with` **не выполняется**.

```python
class Manager:
    def __enter__(self):
        raise RuntimeError("Не удалось открыть ресурс")

    def __exit__(self, exc_type, exc_value, traceback):
        print("exit")
```

```python
with Manager():
    print("Работа")
```

`"Работа"` не будет напечатано.

Также важно: если `__enter__` не завершился успешно, `__exit__` для этого `with` обычно не вызывается.

---

# `__enter__` vs `__init__`

Не стоит путать:

### `__init__`

Инициализирует **объект при его создании**:

```python
manager = Manager()
```

### `__enter__`

Подготавливает объект **при входе в `with`**:

```python
with manager:
    ...
```

```text
Manager()
   ↓
__init__()
   ↓
объект создан

with manager:
   ↓
__enter__()
   ↓
работа
   ↓
__exit__()
```

---

## 🧠 Коротко для собеседования

> **`__enter__` — метод протокола контекстного менеджера, который вызывается при входе в `with`. Обычно он выполняет подготовку или захват ресурса и возвращает объект, который становится значением после `as`.**

### Главное запомнить:

```python
with manager() as value:
```

≈

```python
manager = Manager()
value = manager.__enter__()

try:
    ...
finally:
    manager.__exit__(...)
```

**Формула:**

```text
__enter__ → войти / получить ресурс
__exit__  → выйти / освободить ресурс
```
