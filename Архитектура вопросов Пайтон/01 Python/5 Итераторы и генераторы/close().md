# 🛑 `close()`

🎯 **Ответ на собеседовании:**

`close()` — метод генератора, который **принудительно завершает генератор**.

При вызове:

```python
gen.close()
```

в генератор в точке текущего `yield` выбрасывается специальное исключение `GeneratorExit`.

---

## Пример

```python
def generator():
    yield 1
    yield 2
    yield 3

gen = generator()

next(gen)  # 1
gen.close()

next(gen)  # StopIteration
```

После `close()` генератор больше не выдаёт значения.

```text
next()
  ↓
yield 1
  ↓
генератор приостановлен
  ↓
close()
  ↓
GeneratorExit
  ↓
генератор завершён
```

---

# `close()` и `finally`

Главное практическое применение — **гарантированно выполнить очистку ресурсов**.

```python
def generator():
    try:
        yield 1
    finally:
        print("Очистка ресурсов")

gen = generator()

next(gen)
gen.close()
```

Результат:

```text
Очистка ресурсов
```

Поэтому `finally` выполнится даже при закрытии генератора.

---

# Что такое `GeneratorExit`

`GeneratorExit` — специальное исключение, которое используется для завершения генератора.

При:

```python
gen.close()
```

Python фактически инициирует:

```python
GeneratorExit
```

внутри генератора.

Обычно его специально не перехватывают:

```python
try:
    yield value
except GeneratorExit:
    ...
```

Чаще для очистки используют:

```python
try:
    yield value
finally:
    cleanup()
```

---

# Что будет, если генератор обработает `GeneratorExit`?

Например:

```python
def generator():
    try:
        yield 1
    except GeneratorExit:
        print("Закрываемся")
```

```python
gen = generator()

next(gen)
gen.close()
```

Получим:

```text
Закрываемся
```

Генератор завершится.

⚠️ Важный нюанс: нельзя после получения `GeneratorExit` продолжать выдавать значения через `yield`. Это приводит к `RuntimeError`.

---

# `close()` после завершения

Если генератор уже завершён:

```python
gen.close()
gen.close()
```

повторный `close()` безопасен и ничего существенного не делает.

---

# `close()` vs `throw()`

Оба механизма связаны с передачей исключения внутрь генератора, но назначение разное:

| Метод         | Назначение                                |
| ------------- | ----------------------------------------- |
| `send(value)` | Передать значение                         |
| `throw(exc)`  | Передать произвольное исключение          |
| `close()`     | Завершить генератор через `GeneratorExit` |

```text
send()
  → значение

throw()
  → исключение

close()
  → GeneratorExit
  → завершение генератора
```

---

# Практический пример

Генератор может управлять ресурсом:

```python
def read_data():
    resource = open("data.txt")

    try:
        for line in resource:
            yield line
    finally:
        resource.close()
```

Если генератор перестали использовать и его закрыли:

```python
gen = read_data()

next(gen)
gen.close()
```

`finally` закроет файл.

---

## 🧠 Коротко для собеседования

> **`close()` принудительно завершает генератор. При его вызове внутри генератора возникает `GeneratorExit` в месте последнего `yield`. Это позволяет выполнить очистку ресурсов через `finally`. После `close()` генератор больше не выдаёт значения.**

### Запомнить

```text
close()
   ↓
GeneratorExit
   ↓
finally
   ↓
очистка
   ↓
генератор завершён
```

**Связка методов генератора:**

```text
__next__() → получить значение
send()     → передать значение
throw()    → передать исключение
close()    → завершить генератор
```
