# 🔒 `__exit__`

🎯 **Ответ на собеседовании:**

`__exit__` — специальный метод **протокола контекстного менеджера**, который автоматически вызывается при выходе из блока `with`.

Он используется для **освобождения ресурсов и обработки исключений**.

```python id="h5v2x1"
with manager():
    ...
```

После завершения блока Python вызывает:

```python id="b8n4k7"
manager.__exit__(exc_type, exc_value, traceback)
```

---

# Параметры `__exit__`

Метод получает три аргумента, связанных с исключением:

```python id="q6m3p9"
def __exit__(self, exc_type, exc_value, traceback):
    ...
```

| Параметр    | Что содержит         |
| ----------- | -------------------- |
| `exc_type`  | Тип исключения       |
| `exc_value` | Экземпляр исключения |
| `traceback` | Traceback исключения |

Если внутри `with` исключения **не произошло**:

```python id="x4r7n2"
exc_type is None
exc_value is None
traceback is None
```

---

# Когда вызывается `__exit__`

Он вызывается при выходе из `with`:

### Обычное завершение

```python id="m9q2w5"
with Manager():
    print("Работа")
```

→ `__exit__()` вызывается без информации об исключении.

### Исключение

```python id="k3v8p1"
with Manager():
    raise ValueError("Ошибка")
```

→ `__exit__()` получает информацию о `ValueError`.

Поэтому `__exit__` позволяет гарантировать cleanup **даже при исключении**.

---

# Возвращаемое значение

Это один из самых важных моментов.

Если `__exit__` возвращает:

```python id="f7m1x9"
True
```

то исключение **подавляется**.

```python id="r2k6c4"
class Manager:
    def __enter__(self):
        return self

    def __exit__(self, exc_type, exc_value, traceback):
        return True


with Manager():
    raise ValueError("Ошибка")

print("Продолжаем")
```

Результат:

```text id="a8n3d6"
Продолжаем
```

Если вернуть:

```python id="w5q9s2"
False
```

или `None`, исключение продолжит распространяться.

```python id="j1k4m8"
def __exit__(self, exc_type, exc_value, traceback):
    return False
```

---

# Типичный `__exit__`

Обычно его используют для освобождения ресурса:

```python id="e6p2v9"
class Connection:
    def __enter__(self):
        self.connection = connect()
        return self.connection

    def __exit__(self, exc_type, exc_value, traceback):
        self.connection.close()
```

Схема:

```text id="z7x3c1"
__enter__()
    ↓
получить ресурс
    ↓
работа внутри with
    ↓
__exit__()
    ↓
освободить ресурс
```

---

# Аналогия с `try/finally`

Концептуально:

```python id="v4n8q6"
resource = acquire()

try:
    use(resource)
finally:
    release(resource)
```

можно выразить через:

```python id="p9c2x5"
with resource_manager() as resource:
    use(resource)
```

`__exit__` отвечает за действия при выходе из контекста.

---

# `__exit__` и `__enter__`

| Метод                          | Назначение                                    |
| ------------------------------ | --------------------------------------------- |
| `__enter__`                    | Вход в контекст, получение/подготовка ресурса |
| `__exit__`                     | Выход из контекста, очистка ресурса           |
| `__exit__` получает исключение | Да                                            |
| Может подавить исключение      | Да, если возвращает `True`                    |

---

# Пример с обработкой конкретного исключения

```python id="u8r5k2"
class Manager:
    def __enter__(self):
        return self

    def __exit__(self, exc_type, exc_value, traceback):
        if exc_type is ValueError:
            print("ValueError обработан")
            return True

        return False
```

Теперь:

```python id="x6m3q9"
with Manager():
    raise ValueError("Ошибка")
```

`ValueError` будет подавлен.

Но:

```python id="c4n7v1"
with Manager():
    raise TypeError("Ошибка")
```

`TypeError` продолжит распространяться.

---

# Важный нюанс

`__exit__` **не обязан всегда обрабатывать исключения**.

Чаще его задача — именно гарантированная очистка:

```python id="r3y7k5"
def __exit__(self, exc_type, exc_value, traceback):
    self.resource.close()
    return False
```

То есть ресурс освобождается, а исключение продолжает идти вверх по стеку.

---

## 🧠 Коротко для собеседования

> **`__exit__` — метод контекстного менеджера, который вызывается при выходе из блока `with`. Он используется для освобождения ресурсов и получает `exc_type`, `exc_value` и `traceback`, если внутри блока произошло исключение. Если `__exit__` возвращает `True`, исключение подавляется; если `False` или `None` — распространяется дальше.**

### Главное:

```text id="h2m8q4"
with
 ↓
__enter__()
 ↓
код
 ↓
__exit__(exc_type, exc_value, traceback)
 ↓
cleanup
```

**Ключевая формула:**

> `__enter__` → **подготовить / получить ресурс**
> `__exit__` → **освободить ресурс / обработать исключение**
