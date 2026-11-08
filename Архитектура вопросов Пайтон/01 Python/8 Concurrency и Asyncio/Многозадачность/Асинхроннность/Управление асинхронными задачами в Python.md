# ⚡ Управление асинхронными задачами в Python

## 🎤 Короткий ответ

**Управление асинхронными задачами** — это создание, запуск, ожидание, отмена и обработка результатов нескольких асинхронных операций с помощью `asyncio`.

В Python для этого используются `asyncio.create_task()`, `asyncio.gather()`, `asyncio.wait()`, `asyncio.as_completed()` и `asyncio.TaskGroup`.

Эти инструменты позволяют выполнять независимые I/O-bound операции конкурентно в одном Event Loop, контролировать их жизненный цикл и обрабатывать ошибки.

## 🗣️ Ответ на собеседовании

В `asyncio` асинхронные операции представлены корутинами, которые могут быть запланированы на выполнение как объекты `Task`.

Для создания отдельной задачи используется `asyncio.create_task()`. Она регистрирует корутину в Event Loop, после чего та может выполняться конкурентно с другими задачами.

Если нужно запустить несколько независимых операций и собрать их результаты, можно использовать `asyncio.gather()`. Для более гибкого ожидания подходит `asyncio.wait()`, а `asyncio.as_completed()` позволяет обрабатывать результаты по мере завершения операций.

Начиная с Python 3.11 появился `asyncio.TaskGroup`, который реализует структурированную конкурентность. Он управляет группой связанных задач, дожидается их завершения и при обычной ошибке одной задачи отменяет остальные.

Также важно правильно обрабатывать отмену через `CancelledError`, использовать таймауты и ограничивать количество конкурентных операций через `Semaphore`.

В Backend это применяется при работе с внешними API, асинхронными запросами к базам данных и другими I/O-bound операциями.

## 🧭 Где я нахожусь

```text
01 Python
└── 08 Concurrency и Asyncio
    ├── Конкурентность и параллелизм
    ├── Event Loop
    ├── Coroutine
    ├── await
    ├── Awaitable
    ├── Task
    ├── Future
    └── Управление асинхронными задачами ← Я здесь
        ├── asyncio.create_task()
        ├── asyncio.gather()
        ├── asyncio.wait()
        ├── asyncio.as_completed()
        ├── asyncio.TaskGroup
        ├── Отмена задач
        ├── Таймауты
        └── Ограничение конкурентности
```

## 📚 Разбор поглубже

### 1. Что такое асинхронная задача

В Python важно различать **Coroutine** и **Task**.

**Coroutine** — объект, представляющий выполнение асинхронной функции.

**Task** — объект, который позволяет Event Loop управлять выполнением корутины.

```python
import asyncio


async def worker():
    await asyncio.sleep(1)
    return "Готово"


async def main():
    coroutine = worker()

    task = asyncio.create_task(coroutine)

    result = await task
    print(result)


asyncio.run(main())
```

Здесь:

1. `worker()` создаёт объект корутины.
2. `create_task()` планирует её выполнение.
3. Event Loop выполняет задачу.
4. `await task` ожидает её завершения.
5. `result` получает возвращённое значение.

Само создание объекта корутины через `worker()` не запускает выполнение её тела.

### 2. Жизненный цикл Task

Упрощённо:

```text
Создание корутины
       |
       v
asyncio.create_task()
       |
       v
Задача запланирована
       |
       v
Выполнение в Event Loop
       |
       ├── await → ожидание
       |              |
       |              v
       |         Возобновление
       |
       ├── Успех → результат
       |
       ├── Ошибка → исключение
       |
       └── Отмена → CancelledError
```

Задача может завершиться успешно, завершиться с исключением или быть отменена.

### 3. asyncio.create_task()

Используется для создания и планирования отдельной задачи.

```python
import asyncio


async def worker(name):
    await asyncio.sleep(1)
    return f"{name}: OK"


async def main():
    task1 = asyncio.create_task(worker("A"))
    task2 = asyncio.create_task(worker("B"))

    result1 = await task1
    result2 = await task2

    print(result1, result2)


asyncio.run(main())
```

Обе задачи запланированы до первого `await task1`, поэтому их ожидания могут перекрываться.

**Важный нюанс:** сохраняй ссылки на созданные задачи и обеспечивай их корректное завершение. Не стоит бесконтрольно создавать фоновые задачи и забывать о них.

### 4. asyncio.gather()

Позволяет конкурентно выполнять несколько awaitable-объектов и собрать результаты.

```python
import asyncio


async def worker(name, delay):
    await asyncio.sleep(delay)
    return name


async def main():
    results = await asyncio.gather(
        worker("A", 3),
        worker("B", 1),
        worker("C", 2),
    )

    print(results)


asyncio.run(main())
```

Результат:

```text
['A', 'B', 'C']
```

Результаты возвращаются в порядке аргументов, даже если задачи завершаются в другом порядке.

При обычной ошибке одной задачи `gather()` по умолчанию передаёт исключение вызывающему коду, но не отменяет автоматически остальные задачи.

### 5. asyncio.wait()

Используется, когда необходимо гибко управлять ожиданием набора задач.

```python
import asyncio


async def worker(delay):
    await asyncio.sleep(delay)
    return delay


async def main():
    tasks = {
        asyncio.create_task(worker(1)),
        asyncio.create_task(worker(3)),
    }

    done, pending = await asyncio.wait(
        tasks,
        return_when=asyncio.FIRST_COMPLETED,
    )

    for task in done:
        print(task.result())

    for task in pending:
        task.cancel()

    await asyncio.gather(
        *pending,
        return_exceptions=True,
    )


asyncio.run(main())
```

`asyncio.wait()` возвращает два множества:

- `done` — завершённые задачи;
- `pending` — задачи, которые ещё не завершились.

Режимы ожидания:

| Режим | Поведение |
|---|---|
| `ALL_COMPLETED` | Ждать завершения всех |
| `FIRST_COMPLETED` | Ждать первого завершения |
| `FIRST_EXCEPTION` | Ждать первого исключения или завершения всех, если ошибок нет |

В современных версиях Python `asyncio.wait()` ожидает объекты Task/Future, а не обычные объекты корутин.

Важно: при таймауте или раннем возврате `wait()` не отменяет оставшиеся задачи автоматически.

### 6. asyncio.as_completed()

Позволяет получать результаты по мере завершения операций.

```python
import asyncio


async def worker(name, delay):
    await asyncio.sleep(delay)
    return name


async def main():
    tasks = [
        worker("A", 3),
        worker("B", 1),
        worker("C", 2),
    ]

    for completed in asyncio.as_completed(tasks):
        result = await completed
        print(result)


asyncio.run(main())
```

Результат:

```text
B
C
A
```

Это удобно, когда не нужно ждать самую медленную операцию, чтобы начать обрабатывать уже готовые результаты.

### 7. asyncio.TaskGroup

`TaskGroup` появился в Python 3.11.

Он позволяет управлять связанными задачами внутри одного контекстного менеджера.

```python
import asyncio


async def worker(name):
    await asyncio.sleep(1)
    return f"{name}: OK"


async def main():
    async with asyncio.TaskGroup() as group:
        task1 = group.create_task(worker("A"))
        task2 = group.create_task(worker("B"))

    print(task1.result())
    print(task2.result())


asyncio.run(main())
```

Когда выполнение выходит из блока `async with`, все задачи группы уже завершены.

Если одна задача завершается обычным необработанным исключением, `TaskGroup` отменяет остальные незавершённые задачи, ожидает их завершения и передаёт ошибки через `ExceptionGroup` или `BaseExceptionGroup`.

Это и есть один из принципов **Structured Concurrency** — жизненный цикл дочерних задач ограничен областью, в которой они созданы.

### 8. Сравнение инструментов

| Инструмент | Основная задача |
|---|---|
| `create_task()` | Создать и запланировать задачу |
| `gather()` | Собрать результаты нескольких операций |
| `wait()` | Дождаться заданного условия для набора задач |
| `as_completed()` | Обрабатывать результаты по мере готовности |
| `TaskGroup` | Управлять связанными задачами как группой |

Выбор зависит от того, требуется ли фиксированный порядок результатов, ранняя обработка, частичное ожидание или согласованное управление ошибками.

### 9. Отмена задач

Задачу можно отменить методом `cancel()`.

```python
import asyncio


async def worker():
    try:
        while True:
            print("Работаю...")
            await asyncio.sleep(1)

    except asyncio.CancelledError:
        print("Задача отменена")
        raise


async def main():
    task = asyncio.create_task(worker())

    await asyncio.sleep(2)

    task.cancel()

    try:
        await task
    except asyncio.CancelledError:
        print("Отмена подтверждена")


asyncio.run(main())
```

`cancel()` не останавливает задачу мгновенно. Он запрашивает отмену, которая обычно проявляется как `CancelledError` в точке ожидания.

Если корутина долго выполняет синхронный код без `await`, она не сможет своевременно обработать отмену.

### 10. Таймауты

Чтобы ограничить время выполнения, можно использовать `asyncio.wait_for()`:

```python
import asyncio


async def slow_operation():
    await asyncio.sleep(10)
    return "OK"


async def main():
    try:
        result = await asyncio.wait_for(
            slow_operation(),
            timeout=2,
        )
        print(result)

    except TimeoutError:
        print("Время ожидания истекло")


asyncio.run(main())
```

Если операция не успела завершиться, `wait_for()` запрашивает её отмену и ожидает завершения процедуры отмены. Поэтому фактическое время может немного превышать указанный таймаут.

В Python 3.11+ также есть `asyncio.timeout()`:

```python
async with asyncio.timeout(2):
    await slow_operation()
```

### 11. Ограничение конкурентности

Если создать 1000 задач, это не означает, что внешняя система выдержит 1000 одновременных запросов.

Для ограничения конкурентности используется `asyncio.Semaphore`.

```python
import asyncio


async def worker(number, semaphore):
    async with semaphore:
        await asyncio.sleep(1)
        return number


async def main():
    semaphore = asyncio.Semaphore(5)

    results = await asyncio.gather(
        *(worker(i, semaphore) for i in range(20))
    )

    print(results)


asyncio.run(main())
```

Внутри защищённого участка одновременно могут находиться не более пяти задач.

При этом Semaphore не является полноценным rate limiter: он ограничивает количество одновременных операций, а не запросов в секунду.

### 12. Пример из Backend

Предположим, API должен получить данные из трёх независимых сервисов:

```python
async def get_dashboard(user_id):
    async with asyncio.TaskGroup() as group:
        user_task = group.create_task(
            get_user(user_id)
        )

        orders_task = group.create_task(
            get_orders(user_id)
        )

        payments_task = group.create_task(
            get_payments(user_id)
        )

    return {
        "user": user_task.result(),
        "orders": orders_task.result(),
        "payments": payments_task.result(),
    }
```

Здесь три независимых I/O-bound операции выполняются конкурентно.

Если одна из них завершается обычной необработанной ошибкой, `TaskGroup` отменяет остальные незавершённые операции.

Важно: нельзя автоматически считать любые операции независимыми. Например, конкурентное использование одного экземпляра SQLAlchemy `AsyncSession` несколькими задачами не поддерживается.

### 13. Типичные ошибки

**Ошибка 1. Считать, что create_task() создаёт новый поток.**

Нет. Задача выполняется в рамках Event Loop, если явно не используются другие механизмы выполнения.

**Ошибка 2. Считать, что gather() отменяет остальные задачи при ошибке одной.**

По умолчанию этого не происходит.

**Ошибка 3. Создавать задачи и не управлять их жизненным циклом.**

Это может привести к потерянным исключениям и незавершённым операциям.

**Ошибка 4. Использовать блокирующий код внутри async-функций.**

Например, `time.sleep()` блокирует Event Loop. Для асинхронного ожидания следует использовать `asyncio.sleep()`.

**Ошибка 5. Путать конкурентность и параллелизм.**

`asyncio` обычно обеспечивает конкурентность за счёт кооперативного переключения задач, а не параллельное выполнение Python-кода на нескольких ядрах.

**Ошибка 6. Игнорировать отмену и таймауты.**

Это может приводить к зависшим операциям, утечкам ресурсов и затруднять корректное завершение приложения.

## 🎤 Вопросы на собеседовании

**1. Что такое Task в asyncio?**

Объект, который управляет выполнением корутины в Event Loop и позволяет получить результат, исключение или отменить выполнение.

**2. Чем Coroutine отличается от Task?**

Coroutine представляет асинхронное вычисление, а Task планирует его выполнение и управляет жизненным циклом.

**3. Чем create_task() отличается от gather()?**

`create_task()` создаёт отдельную задачу, а `gather()` координирует несколько awaitable-объектов и собирает результаты.

**4. Когда использовать asyncio.wait()?**

Когда нужно дождаться определённого условия: завершения всех задач, первой завершённой задачи или первого исключения.

**5. Зачем нужен asyncio.as_completed()?**

Чтобы обрабатывать результаты по мере завершения операций, не дожидаясь самой медленной.

**6. Чем TaskGroup отличается от gather()?**

`TaskGroup` обеспечивает структурированную конкурентность и при обычной ошибке одной задачи отменяет остальные незавершённые задачи группы.

**7. Как отменить Task?**

Вызвать `task.cancel()` и корректно обработать `asyncio.CancelledError`.

**8. Как ограничить время выполнения задачи?**

Использовать `asyncio.wait_for()` или `asyncio.timeout()`.

**9. Как ограничить количество конкурентных запросов?**

Использовать `asyncio.Semaphore` или специализированный механизм управления лимитами.

**10. Когда управление асинхронными задачами полезно в Backend?**

При конкурентной работе с внешними API, сетевыми соединениями и другими независимыми I/O-bound операциями, когда важно контролировать ожидание, ошибки и отмену задач.