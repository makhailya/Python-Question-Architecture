# 🧩 Task в asyncio

## 🎤 Короткий ответ

**Task** — это объект `asyncio`, который представляет **запланированное выполнение coroutine** в Event Loop.

Когда мы делаем:

```python
task = asyncio.create_task(coro())
```

coroutine передаётся Event Loop для выполнения как отдельная задача.

Task позволяет:

* запустить coroutine конкурентно с текущей;
* получить её результат позже;
* дождаться её через `await`;
* отменить её;
* проверить её состояние.

Главная формула:

```text
Coroutine
    ↓
asyncio.create_task()
    ↓
Task
    ↓
Event Loop
    ↓
выполнение
```

---

## 🗣️ Ответ на собеседовании

`Task` в `asyncio` — это объект, который оборачивает coroutine и планирует её выполнение в Event Loop.

Например:

```python
task = asyncio.create_task(fetch_data())
```

После этого coroutine становится отдельной задачей, которую Event Loop может выполнять конкурентно с другими задачами.

Мы можем продолжить выполнение текущей coroutine, а позже получить результат:

```python
task = asyncio.create_task(fetch_data())

# здесь выполняется другой код

result = await task
```

Task является awaitable, поэтому её можно передать в `await`.

Кроме результата Task хранит состояние выполнения: например, завершена ли она, была ли отменена и возникло ли исключение.

Важно отличать `Task` от `coroutine`: coroutine — это объект, описывающий асинхронное выполнение, а Task — уже запланированная для выполнения coroutine.

---

## 🧭 Где я нахожусь

```text
Python
│
└── 08 Concurrency и Asyncio
    │
    └── Asyncio
        │
        ├── Event Loop
        │
        ├── Coroutine
        │
        ├── Awaitable
        │   ├── Coroutine
        │   ├── Task ← Я здесь
        │   └── Future
        │
        ├── await
        ├── Selector
        └── Non-blocking I/O
```

---

# 📚 Разбор поглубже

## 1. Что такое Task

`Task` — это объект, который позволяет **запланировать coroutine на выполнение**.

Например:

```python
import asyncio


async def fetch_data():
    await asyncio.sleep(2)
    return "Данные"


async def main():
    task = asyncio.create_task(fetch_data())

    result = await task

    print(result)


asyncio.run(main())
```

Здесь:

```python
fetch_data()
```

создаёт coroutine.

А:

```python
asyncio.create_task(fetch_data())
```

создаёт `Task` и планирует эту coroutine.

---

# 2. Coroutine vs Task

Это одно из главных различий.

### Coroutine

```python
coro = fetch_data()
```

Получаем объект coroutine.

Он описывает асинхронное выполнение функции, но сам по себе ещё не является запланированной `Task`.

```text
async def
    ↓
вызов
    ↓
Coroutine
```

### Task

```python
task = asyncio.create_task(fetch_data())
```

Теперь:

```text
Coroutine
    ↓
create_task()
    ↓
Task
    ↓
Event Loop
```

То есть:

> **Coroutine — что выполнить.**

> **Task — запланированное выполнение этого coroutine.**

---

# 3. Зачем вообще нужна Task

Представим:

```python
async def main():
    result = await fetch_data()
    print("Продолжаем")
```

Мы сразу ждём `fetch_data()`.

А если операции независимы:

```python
async def main():
    task_a = asyncio.create_task(fetch_a())
    task_b = asyncio.create_task(fetch_b())

    # Здесь обе задачи уже могут выполняться конкурентно

    result_a = await task_a
    result_b = await task_b
```

Получается:

```text
Время →

Task A  ████──────████
Task B  ███────████
             ↑
        Event Loop
        переключается
```

Пока одна задача ждёт I/O, другая может продолжать выполняться.

---

# 4. `create_task()`

Наиболее распространённый способ создания Task:

```python
task = asyncio.create_task(coro())
```

Например:

```python
import asyncio


async def worker(name):
    await asyncio.sleep(1)
    return name


async def main():
    task = asyncio.create_task(worker("A"))

    result = await task

    print(result)


asyncio.run(main())
```

`create_task()` требует coroutine и планирует её выполнение в текущем Event Loop.

---

# 5. Task начинает выполняться не обязательно сразу

Например:

```python
task = asyncio.create_task(worker())
print("Создали Task")
```

После `create_task()` задача запланирована, но текущая coroutine продолжает выполняться.

Event Loop получит возможность запустить Task, когда управление будет возвращено ему.

Упрощённо:

```text
main()
 │
 ├── create_task(worker())
 │
 ├── print(...)
 │
 ├── await ...
 │
 ▼
Event Loop
 │
 └── worker()
```

Поэтому полезно различать:

```text
создать Task
    ≠
мгновенно выполнить её до конца
```

---

# 6. Task является Awaitable

Task можно передать в `await`:

```python
result = await task
```

Это связано с предыдущей темой:

```text
Awaitable
│
├── Coroutine
├── Task
└── Future
```

Поэтому:

```python
await task
```

означает:

> дождаться завершения Task и получить её результат.

---

# 7. Task и результат

Если coroutine возвращает значение:

```python
async def calculate():
    return 42
```

создаём Task:

```python
task = asyncio.create_task(calculate())
```

Получаем результат:

```python
result = await task
```

Получим:

```python
42
```

Схема:

```text
Coroutine
    ↓
Task
    ↓
выполнение
    ↓
return 42
    ↓
Task завершена
    ↓
await task
    ↓
42
```

---

# 8. Task и исключения

Если coroutine завершилась исключением:

```python
async def worker():
    raise ValueError("Ошибка")
```

то при ожидании Task исключение будет получено:

```python
task = asyncio.create_task(worker())

try:
    await task
except ValueError:
    print("Ошибка обработана")
```

Важно понимать:

```text
Task
 │
 ├── успешно завершилась → result
 │
 └── завершилась ошибкой → exception
```

---

# 9. Состояния Task

У Task есть состояние выполнения.

Упрощённо:

```text
Task
 │
 ├── создана / запланирована
 │
 ├── выполняется
 │
 ├── ожидает
 │
 └── завершена
       ├── результат
       └── исключение
```

Можно проверить:

```python
task.done()
```

Завершена ли задача.

И:

```python
task.cancelled()
```

Была ли она отменена.

Также можно получить результат:

```python
task.result()
```

Но `result()` следует использовать только после завершения Task. Если задача ещё выполняется, результат ещё не готов.

---

# 10. Отмена Task

Task можно отменить:

```python
task.cancel()
```

Например:

```python
task = asyncio.create_task(worker())

task.cancel()
```

При отмене в coroutine обычно возникает `asyncio.CancelledError`.

Например:

```python
async def worker():
    try:
        await asyncio.sleep(10)
    except asyncio.CancelledError:
        print("Задача отменена")
        raise
```

Отмена особенно важна для серверных приложений, таймаутов и graceful shutdown.

---

# 11. Task и Event Loop

Task не выполняется сама по себе.

Она связана с Event Loop:

```text
                    Event Loop
                        │
          ┌─────────────┼─────────────┐
          ▼             ▼             ▼
       Task A         Task B        Task C
          │             │             │
        await          await          CPU
          │             │
          ▼             ▼
         I/O           I/O
```

Event Loop определяет, какая готовая Task может продолжить выполнение.

Поэтому:

```text
Task
 ↓
Event Loop
 ↓
Coroutine
 ↓
await
 ↓
ожидание I/O
 ↓
Event Loop
 ↓
другая Task
```

---

# 12. Task и конкурентность

Task является одним из основных инструментов для организации **конкурентного выполнения** в `asyncio`.

Например:

```python
async def main():
    task_a = asyncio.create_task(fetch_a())
    task_b = asyncio.create_task(fetch_b())

    await task_a
    await task_b
```

Здесь обе задачи были запланированы до того, как мы начали их ждать.

Поэтому они могут выполняться конкурентно.

Сравним:

### Последовательно

```python
a = await fetch_a()
b = await fetch_b()
```

```text
A ███████
        B ███████
```

### Конкурентно

```python
task_a = asyncio.create_task(fetch_a())
task_b = asyncio.create_task(fetch_b())

a = await task_a
b = await task_b
```

```text
A █████────████
B ███────██████
```

---

# 13. Task и `asyncio.gather()`

Когда нужно одновременно запланировать несколько операций, можно использовать:

```python
results = await asyncio.gather(
    fetch_a(),
    fetch_b(),
    fetch_c(),
)
```

`gather()` работает с awaitable и организует конкурентное выполнение переданных операций.

Можно также передать уже созданные Task:

```python
task_a = asyncio.create_task(fetch_a())
task_b = asyncio.create_task(fetch_b())

results = await asyncio.gather(task_a, task_b)
```

На практике часто достаточно:

```python
results = await asyncio.gather(
    fetch_a(),
    fetch_b(),
)
```

---

# 14. Task ≠ Thread

Это принципиальное различие.

```text
Task
 ↓
asyncio
 ↓
Event Loop
```

А:

```text
Thread
 ↓
операционная система
 ↓
поток выполнения
```

Упрощённо:

```text
Один процесс
│
└── Thread
    │
    └── Event Loop
        ├── Task A
        ├── Task B
        └── Task C
```

Можно иметь много Task в одном потоке.

---

# 15. Task ≠ Process

Task не является отдельным процессом.

Она не получает собственное адресное пространство и не запускается автоматически на отдельном CPU-ядре.

```text
Task
→ конкурентность

Process
→ отдельный процесс
→ собственная память
→ возможность CPU-параллелизма
```

---

# 16. Task ≠ параллелизм

Это тоже частая ошибка на собеседованиях.

Если у нас:

```text
Task A
Task B
Task C
```

это не означает:

```text
CPU 1 → A
CPU 2 → B
CPU 3 → C
```

Обычно `asyncio` работает как конкурентная модель, где Event Loop управляет задачами.

Главная цель:

```text
не ждать I/O впустую
```

а не распределять CPU-вычисления по ядрам.

---

# 17. Полная схема

Теперь можно связать предыдущие темы:

```text
async def
    │
    ▼
Coroutine
    │
    │ create_task()
    ▼
Task
    │
    ▼
Event Loop
    │
    ├── выполняет Task
    │
    ├── Task встречает await
    │
    ▼
Task приостанавливается
    │
    ▼
Event Loop выполняет другую Task
    │
    ▼
I/O завершилось
    │
    ▼
Task снова готова
    │
    ▼
Coroutine продолжает выполнение
```

Это и есть основа конкурентного выполнения в `asyncio`.

---

## 🎤 Вопросы на собеседовании

### Что такое Task?

`Task` — объект `asyncio`, представляющий запланированное выполнение coroutine в Event Loop.

### Как создать Task?

```python
task = asyncio.create_task(coro())
```

### Зачем нужна Task?

Чтобы запланировать coroutine как отдельную задачу и выполнять её конкурентно с другими задачами.

### Является ли Task awaitable?

Да.

```python
result = await task
```

### Чем Task отличается от Coroutine?

Coroutine — объект асинхронного выполнения, полученный при вызове `async def`.

Task — запланированная Event Loop coroutine.

```text
Coroutine
    ↓
create_task()
    ↓
Task
```

### Чем Task отличается от Thread?

Task — абстракция `asyncio`, которой управляет Event Loop.

Thread — поток выполнения, управляемый операционной системой.

### Создаёт ли `asyncio.create_task()` новый поток?

Нет.

### Создаёт ли Task отдельный процесс?

Нет.

### Выполняются ли Task параллельно?

Не обязательно. `asyncio` предоставляет конкурентность. Само наличие нескольких Task не означает физический параллелизм на разных CPU-ядрах.

### Что будет, если Task завершилась исключением?

При ожидании Task через `await` соответствующее исключение будет проброшено в ожидающую coroutine.

### Как проверить, завершилась ли Task?

```python
task.done()
```

### Как отменить Task?

```python
task.cancel()
```

После отмены coroutine обычно получает `asyncio.CancelledError`.

### Что происходит с Task во время `await` I/O?

Текущая coroutine приостанавливается, а Event Loop может выполнять другие готовые Task.

---

## 🧠 Формула

```text
Coroutine
    ↓
create_task()
    ↓
Task
    ↓
Event Loop
    ↓
выполнение
    ↓
await I/O
    ↓
Task приостанавливается
    ↓
Event Loop → другая Task
    ↓
I/O завершилось
    ↓
Task продолжает выполнение
    ↓
результат / исключение
```

**Главная мысль:**

> **Coroutine описывает асинхронное выполнение, Task планирует это выполнение, а Event Loop управляет Task.**
