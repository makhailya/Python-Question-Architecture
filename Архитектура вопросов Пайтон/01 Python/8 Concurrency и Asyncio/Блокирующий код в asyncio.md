# 🚫 Блокирующий код в `asyncio`

## 🎯 Ответ на собеседовании

**Блокирующий код в `asyncio`** — это синхронная операция, которая надолго останавливает текущий поток и, следовательно, **блокирует Event Loop**.

Если Event Loop заблокирован, другие асинхронные задачи не могут нормально выполняться.

> `async def` сам по себе **не делает код асинхронным и неблокирующим**.

---

## 📌 Пример

❌ Плохо:

```python
import asyncio
import time


async def task():
    print("start")
    time.sleep(5)
    print("end")
```

Проблема:

```python
time.sleep(5)
```

`time.sleep()` блокирует поток на 5 секунд.

Так как Event Loop работает в этом же потоке, на эти 5 секунд блокируются и другие `asyncio`-задачи.

---

## ✅ Неблокирующий вариант

Используем асинхронную операцию:

```python
async def task():
    print("start")
    await asyncio.sleep(5)
    print("end")
```

Теперь:

```text
task 1 → await sleep → Event Loop → task 2
                              ↓
                         task 3
                              ↓
                       task 1 готова
```

Пока `task 1` ждёт, Event Loop может выполнять другие задачи.

---

# ⚠️ Типичные блокирующие операции

В `asyncio` опасны обычные синхронные операции, если они могут выполняться долго:

### 🌐 Синхронный HTTP

```python
requests.get(url)
```

Вместо него используют асинхронный HTTP-клиент, например `aiohttp` или `httpx` в async-режиме.

### 🗄️ Синхронная работа с БД

Обычный синхронный драйвер:

```python
cursor.execute(...)
```

может блокировать Event Loop.

Используют асинхронный драйвер/клиент, если стек это поддерживает.

### 📁 Файловые операции

```python
with open("large_file.txt") as f:
    data = f.read()
```

Большие или медленные операции с файловой системой могут блокировать поток.

### ⏱️ `time.sleep()`

```python
time.sleep(5)
```

Всегда блокирует текущий поток.

---

# 🔄 Что делать с блокирующей функцией?

Если синхронную функцию невозможно заменить асинхронной, её можно вынести в отдельный поток:

```python
result = await asyncio.to_thread(
    blocking_function
)
```

Например:

```python
def read_file():
    with open("large_file.txt") as f:
        return f.read()


async def main():
    data = await asyncio.to_thread(read_file)
```

Теперь блокирующая функция выполняется не в Event Loop thread.

---

# 🧮 А если функция CPU-bound?

`asyncio.to_thread()` **не является универсальным решением для CPU-bound Python-кода**.

Например:

```python
def calculate():
    for i in range(100_000_000):
        ...
```

Если это тяжёлый Python-код, поток не даст полноценного CPU-параллелизма в обычном GIL-enabled CPython.

Для CPU-bound задач чаще используют:

```text
ProcessPoolExecutor
multiprocessing
```

---

# 📊 Что использовать?

| Операция                       | Что использовать                        |
| ------------------------------ | --------------------------------------- |
| Асинхронный HTTP               | `async` HTTP-клиент                     |
| Async БД                       | async-драйвер                           |
| Задержка                       | `await asyncio.sleep()`                 |
| Синхронная блокирующая функция | `asyncio.to_thread()`                   |
| CPU-bound Python-код           | `ProcessPoolExecutor` / multiprocessing |
| `time.sleep()` внутри async    | ❌ нельзя без необходимости              |

---

# 🧠 Главное

```text
async def
   ↓
не означает автоматически
   ↓
неблокирующий код
```

Важно **как выполняются операции внутри корутины**.

Если корутина выполняет:

```python
time.sleep()
requests.get()
heavy_calculation()
```

она может заблокировать Event Loop.

Если же она делает:

```python
await asyncio.sleep()
await async_http_request()
await async_db_query()
```

Event Loop может переключаться на другие задачи.

---

## 🎤 Суперкоротко

> **Блокирующий код в asyncio — это код, который блокирует поток Event Loop и не позволяет другим корутинам выполняться. `async def` сам по себе не устраняет блокировки. Например, `time.sleep()` и синхронный `requests.get()` блокируют Event Loop. Для I/O используют async-библиотеки, для неизбежного блокирующего кода — `asyncio.to_thread()`, а для CPU-bound задач обычно используют процессы.**
