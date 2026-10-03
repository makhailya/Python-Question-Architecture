# 🧵 `run_in_executor` / `asyncio.to_thread`

## 🎯 Ответ на собеседовании

`asyncio.to_thread()` и `loop.run_in_executor()` позволяют выполнить **блокирующую синхронную функцию в отдельном потоке**, чтобы она не блокировала Event Loop.

Используются, когда у нас есть синхронная функция, которую нельзя или нецелесообразно переписать на async API.

---

# 📌 Проблема

Допустим, есть блокирующая функция:

```python
import time


def blocking_task():
    time.sleep(5)
    return "done"
```

Если вызвать её напрямую внутри async-кода:

```python
async def main():
    result = blocking_task()
```

Event Loop будет заблокирован на 5 секунд.

```text
Event Loop
    │
    └── blocking_task()
            ↓
          ⏳ 5 сек
            ↓
       Event Loop стоит
```

---

# ✅ `asyncio.to_thread()`

Современный и простой способ:

```python
import asyncio


async def main():
    result = await asyncio.to_thread(
        blocking_task
    )
```

Схема:

```text
Event Loop
    │
    ├── main()
    │
    └── await to_thread(...)
              ↓
        Thread Pool
              ↓
        blocking_task()
```

Пока функция выполняется в другом потоке, Event Loop может заниматься другими async-задачами.

---

# 📌 Передача аргументов

```python
def blocking_task(name, delay):
    time.sleep(delay)
    return name


async def main():
    result = await asyncio.to_thread(
        blocking_task,
        "Ilya",
        5,
    )
```

---

# ⚙️ `run_in_executor()`

Более низкоуровневый способ:

```python
import asyncio


async def main():
    loop = asyncio.get_running_loop()

    result = await loop.run_in_executor(
        None,
        blocking_task,
    )
```

`None` означает использование **default executor**.

Можно передать собственный executor:

```python
from concurrent.futures import ThreadPoolExecutor


async def main():
    loop = asyncio.get_running_loop()

    with ThreadPoolExecutor(max_workers=5) as executor:
        result = await loop.run_in_executor(
            executor,
            blocking_task,
        )
```

---

# 🆚 `to_thread()` vs `run_in_executor()`

|                   | `asyncio.to_thread()`          | `run_in_executor()`          |
| ----------------- | ------------------------------ | ---------------------------- |
| Уровень           | Высокоуровневый API            | Более низкоуровневый         |
| Основной сценарий | Выполнить функцию в потоке     | Выполнить функцию в executor |
| ThreadPool        | Использует default thread pool | Можно указать executor       |
| Процесс           | ❌                              | Можно `ProcessPoolExecutor`  |
| Простота          | 🟢                             | 🟡                           |

Главное отличие:

**`to_thread()` ориентирован именно на отдельный поток**, а `run_in_executor()` позволяет выбрать executor, включая `ProcessPoolExecutor`.

---

# 🧮 А что с CPU-bound?

Вот это важно.

### `to_thread()`

Для тяжёлого CPU-bound Python-кода в обычном **GIL-enabled CPython** обычно не является способом получить параллельное выполнение на нескольких ядрах.

```python
await asyncio.to_thread(heavy_calculation)
```

Если функция выполняет Python-код и упирается в GIL, несколько потоков не дадут полноценного CPU-параллелизма.

---

### `ProcessPoolExecutor`

Для CPU-bound задачи можно использовать процесс:

```python
from concurrent.futures import ProcessPoolExecutor


async def main():
    loop = asyncio.get_running_loop()

    with ProcessPoolExecutor() as executor:
        result = await loop.run_in_executor(
            executor,
            heavy_calculation,
        )
```

Получаем:

```text
Event Loop
     │
     ↓
ProcessPoolExecutor
     │
     ├── Process 1 → CPU
     ├── Process 2 → CPU
     └── Process 3 → CPU
```

---

# ⚠️ Не путать с `await`

`await` сам по себе **не переносит функцию в другой поток**.

❌ Это не значит:

```python
await blocking_task()
```

что `blocking_task()` автоматически станет неблокирующей.

Если `blocking_task()` — обычная синхронная функция, такой код вообще некорректен для обычного `await`.

Нужно:

```python
await asyncio.to_thread(blocking_task)
```

---

# 🌐 Типичный Backend-пример

Предположим, у нас FastAPI и старая синхронная библиотека:

```python
def legacy_api_call():
    return requests.get(
        "https://example.com"
    )
```

Можно:

```python
async def endpoint():
    response = await asyncio.to_thread(
        legacy_api_call
    )

    return response.json()
```

Синхронный `requests` выполняется в отдельном потоке, а Event Loop не блокируется ожиданием сети.

Но если библиотека имеет полноценный async API, обычно лучше использовать его напрямую.

---

# 🧠 Главное

```text
Есть blocking sync function
          ↓
Нужно вызвать её из asyncio
          ↓
     ┌────┴────┐
     ↓         ↓
to_thread   run_in_executor
     ↓         ↓
  Thread    Executor
```

### Выбор

```text
I/O-bound + sync функция
        ↓
asyncio.to_thread()
```

```text
Нужен конкретный executor
        ↓
run_in_executor()
```

```text
CPU-bound
        ↓
ProcessPoolExecutor
```

---

## 🎤 Суперкоротко

> **`asyncio.to_thread()` и `run_in_executor()` позволяют вынести блокирующую синхронную функцию из Event Loop в executor. `to_thread()` — простой высокоуровневый вариант для запуска в thread pool. `run_in_executor()` более гибкий и позволяет использовать как `ThreadPoolExecutor`, так и `ProcessPoolExecutor`. Для CPU-bound Python-кода в обычном CPython обычно используют процессный executor.**
