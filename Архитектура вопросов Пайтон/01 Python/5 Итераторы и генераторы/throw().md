# ⚠️ `throw()`

🎯 **Ответ на собеседовании:**

`throw()` — метод генератора, который позволяет **передать исключение внутрь приостановленного генератора**.

Исключение возникает **в том месте, где генератор остановился на `yield`**.

```python id="5f2k8d"
generator.throw(ValueError)
```

---

## Простой пример

```python id="g2x8v1"
def generator():
    try:
        yield 1
    except ValueError:
        print("Ошибка обработана")

gen = generator()

next(gen)           # 1
gen.throw(ValueError)
```

Результат:

```text id="v9c4s2"
Ошибка обработана
```

### Что произошло

```text
next(gen)
    ↓
yield 1
    ↓
генератор приостановлен
    ↓
throw(ValueError)
    ↓
ValueError возникает внутри генератора
    ↓
except ValueError
    ↓
обработка
```

---

# Где именно возникает исключение?

Это важный момент.

```python id="9f7d3a"
def generator():
    try:
        value = yield 10
        yield value
    except ValueError:
        yield "error"
```

После:

```python id="8k2m1p"
gen = generator()

next(gen)
```

генератор остановился на:

```python id="8n4q6w"
value = yield 10
```

Теперь:

```python id="3p5x7c"
gen.throw(ValueError)
```

`ValueError` фактически возникает **в точке `yield`**, и выполнение переходит в `except`.

---

# Если исключение не обработано

Если внутри генератора нет подходящего `except`:

```python id="6v8k2q"
def generator():
    yield 1

gen = generator()

next(gen)
gen.throw(ValueError)
```

`ValueError` выйдет наружу:

```text id="h3s9d1"
ValueError
```

А генератор после этого завершается.

---

# `throw()` может передать аргументы

Можно передать экземпляр исключения:

```python id="u7n2m5"
gen.throw(ValueError("Некорректное значение"))
```

Или тип и аргументы:

```python id="z4c8w1"
gen.throw(ValueError, "Некорректное значение")
```

В генераторе:

```python id="r6p3k9"
try:
    yield
except ValueError as e:
    print(e)
```

---

# `throw()` vs `send()`

|                           | `send()`  | `throw()`                     |
| ------------------------- | --------- | ----------------------------- |
| Передаёт                  | Значение  | Исключение                    |
| Куда                      | В `yield` | В `yield`                     |
| Продолжает генератор      | ✅         | ✅, если исключение обработано |
| Может завершить генератор | Да        | Да                            |

Пример:

```text id="q8w3n6"
send(42)
   ↓
yield получает 42
```

А:

```text id="m5v9r2"
throw(ValueError)
   ↓
yield получает исключение
```

---

# `throw()` и `close()`

`close()` фактически используется для завершения генератора и связан с `GeneratorExit`.

```python id="c7k2p4"
gen.close()
```

Внутри генератора можно обработать очистку:

```python id="d9x4m1"
def generator():
    try:
        yield 1
    finally:
        print("cleanup")
```

Поэтому:

```text id="b3n8q5"
send()  → передать данные
throw() → передать исключение
close() → завершить генератор
```

---

## 🧠 Коротко для собеседования

> **`throw()` — метод генератора, который выбрасывает указанное исключение внутри приостановленного генератора, в точке последнего `yield`. Если генератор обрабатывает это исключение через `try/except`, он может продолжить работу; иначе исключение выходит наружу, а генератор завершается.**

### Запомнить

```text
yield
  ↓
генератор приостановлен
  ↓
throw(ValueError)
  ↓
ValueError возникает внутри генератора
  ↓
try/except?
 ├─ Да → обработка → генератор может продолжить
 └─ Нет → исключение наружу → генератор завершён
```
