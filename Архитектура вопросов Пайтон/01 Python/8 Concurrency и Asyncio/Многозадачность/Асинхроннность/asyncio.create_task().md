# ⚡ asyncio.create_task() в Python

## 🎤 Короткий ответ

**`asyncio.create_task()`** — функция, которая создаёт объект `Task` из корутины и планирует её выполнение в текущем Event Loop.

Она позволяет запустить асинхронную операцию конкурентно с другими задачами, не ожидая её немедленного завершения.

`create_task()` не создаёт новый поток или процесс. Задачи выполняются в рамках Event Loop и переключаются преимущественно в точках `await`.

## 🗣️ Ответ на собеседовании

`asyncio.create_task()` используется для создания и планирования асинхронной задачи.

Когда мы вызываем `async def`-функцию, получаем объект корутины, но её тело ещё не выполняется. Чтобы запланировать выполнение независимо от немедленного ожидания результата, можно передать корутину в `asyncio.create_task()`.

Функция возвращает объект `asyncio.Task`, с помощью которого можно дождаться результата через `await`, проверить состояние, получить исключение или запросить отмену.

Например, если Backend-сервису нужно выполнить два независимых HTTP-запроса, можно создать две задачи и затем дождаться обеих. Пока одна ожидает ответа по сети, Event Loop может выполнять другую.

Важно контролировать жизненный цикл созданных задач, сохранять на них ссылки и обрабатывать исключения. Для групп связанных задач в Python 3.11+ часто удобнее использовать `asyncio.TaskGroup`.

## 🧭 Где я нахожусь

```text
01 Python
└── 08 Concurrency и Asyncio
    ├── Event Loop
    ├── Coroutine
    ├── await
    ├── Awaitable
    ├── Task
    ├── Future
    └── Управление асинхронными задачами
        ├── asyncio.create_task() ← Я здесь
        ├── asyncio.gather()
        ├── asyncio.wait()
        ├── asyncio.as_completed()
        ├── asyncio.TaskGroup
        ├── Отмена задач
        └── Таймауты
```

## 📚 Разбор поглубже

### 1. Зачем нужен create_task()

Представим, что у нас есть две асинхронные операции, каждая из которых ожидает одну секунду.

**Последовательное выполнение:**

```python
import asyncio


async def worker(name):
    await asyncio.sleep(1)
    return f"{name}: готово"


async def main():
    result1 = await worker("A")
    result2 = await worker("B")

    print(result1, result2)


asyncio.run(main())
```

Общее время — примерно 2 секунды.

Почему? Сначала `main()` дожидается завершения `worker("A")`, а только затем запускает `worker("B")`.

**Конкурентное выполнение через create_task():**

```python
import asyncio


async def worker(name):
    await asyncio.sleep(1)
    return f"{name}: готово"


async def main():
    task1 = asyncio.create_task(worker("A"))
    task2 = asyncio.create_task(worker("B"))

    result1 = await task1
    result2 = await task2

    print(result1, result2)


asyncio.run(main())
```

Общее время — примерно 1 секунда.

Почему?

Обе задачи запланированы до первого ожидания. Когда `main()` доходит до `await task1`, Event Loop может выполнять обе задачи и переключаться между ними.

**Главная мысль:** `create_task()` позволяет запланировать выполнение корутины, не дожидаясь её завершения в месте создания задачи.

### 2. Coroutine vs Task

Это важное различие на собеседовании.

**Coroutine** — объект асинхронного вычисления.

**Task** — объект, который управляет выполнением корутины в Event Loop.

```python
import asyncio


async def hello():
    print("Hello")


async def main():
    coroutine = hello()

    print(type(coroutine))

    task = asyncio.create_task(coroutine)

    print(type(task))

    await task


asyncio.run(main())
```

Упрощённо:

```text
async def hello()
        |
        v
     hello()
        |
        v
 Coroutine Object
        |
        | asyncio.create_task()
        v
      Task
        |
        v
    Event Loop
        |
        v
   Выполнение
```

Вызов `hello()` сам по себе не выполняет тело функции.

А `create_task()` регистрирует корутину для выполнения в Event Loop.

### 3. Что происходит под капотом

Когда вызывается:

```python
task = asyncio.create_task(worker())
```

Происходит следующее:

1. `worker()` создаёт объект корутины.
2. `create_task()` получает текущий работающий Event Loop.
3. Создаётся объект `Task`.
4. Task планируется на выполнение.
5. Event Loop запускает корутину, когда получает управление.
6. При `await` незавершённой операции задача приостанавливается.
7. После готовности ожидаемой операции задача возобновляется.
8. Результат или исключение сохраняется в Task.

Схема:

```text
asyncio.create_task()
          |
          v
     Создание Task
          |
          v
     Планирование
          |
          v
       Event Loop
          |
          v
   Выполнение корутины
          |
          ├── await → ожидание
          |             |
          |             v
          |        Возобновление
          |
          ├── return → результат
          |
          ├── raise → исключение
          |
          └── cancel → запрос отмены
```

По умолчанию выполнение вновь созданной задачи не обязано начинаться непосредственно внутри вызова `create_task()`. Оно начинается, когда планировщик предоставляет ей управление.

В некоторых конфигурациях Python поддерживается eager execution, поэтому детали момента начала выполнения могут отличаться.

### 4. Как получить результат Task

Объект Task можно ожидать через `await`.

```python
import asyncio


async def calculate():
    await asyncio.sleep(1)
    return 42


async def main():
    task = asyncio.create_task(calculate())

    result = await task

    print(result)


asyncio.run(main())
```

Результат:

```text
42
```

После завершения задачи результат также можно получить через:

```python
task.result()
```

Но важно понимать разницу:

- `await task` ожидает завершения задачи.
- `task.result()` не ожидает выполнения и вызовет `InvalidStateError`, если задача ещё не завершилась.
- Если задача завершилась с исключением, `result()` повторно поднимет это исключение.
- Если задача отменена, будет поднято `CancelledError`.

### 5. Проверка состояния Task

У Task есть методы для управления состоянием.

```python
import asyncio


async def worker():
    await asyncio.sleep(1)
    return "OK"


async def main():
    task = asyncio.create_task(worker())

    print(task.done())       # False
    print(task.cancelled())  # False

    await task

    print(task.done())       # True
    print(task.result())     # OK


asyncio.run(main())
```

Основные методы:

| Метод | Назначение |
|---|---|
| `done()` | Проверить, завершилась ли задача |
| `cancel()` | Запросить отмену |
| `cancelled()` | Проверить, завершилась ли задача отменой |
| `result()` | Получить результат завершённой задачи |
| `exception()` | Получить исключение завершённой задачи |
| `get_name()` | Получить имя задачи |
| `get_coro()` | Получить связанную корутину |

Важно: `done() == True` означает, что задача завершилась, но не обязательно успешно.

### 6. Отмена задачи — cancel()

Задачу можно отменить:

```python
import asyncio


async def worker():
    try:
        while True:
            print("Работаю...")
            await asyncio.sleep(1)

    except asyncio.CancelledError:
        print("Получена отмена")
        raise


async def main():
    task = asyncio.create_task(worker())

    await asyncio.sleep(2)

    task.cancel()

    try:
        await task
    except asyncio.CancelledError:
        print("Задача отменена")


asyncio.run(main())
```

Что происходит:

1. Создаётся Task.
2. Задача начинает выполняться.
3. Через две секунды вызывается `task.cancel()`.
4. Task получает запрос на отмену.
5. В корутину обычно передаётся `CancelledError` на ближайшей подходящей точке ожидания.
6. Корутина может выполнить очистку ресурсов.
7. Если исключение не подавлено, задача завершается в состоянии отмены.

**Важно:** `cancel()` не убивает задачу мгновенно. Отмена в `asyncio` кооперативная.

### 7. Исключения внутри Task

Если задача завершается с ошибкой, её нужно обработать.

```python
import asyncio


async def worker():
    await asyncio.sleep(1)
    raise ValueError("Что-то пошло не так")


async def main():
    task = asyncio.create_task(worker())

    try:
        await task
    except ValueError as error:
        print(f"Ошибка: {error}")


asyncio.run(main())
```

Если создать задачу, не дождаться её и не получить исключение, можно столкнуться с предупреждением вида:

```text
Task exception was never retrieved
```

Поэтому не стоит создавать задачи без понимания того, кто отвечает за их завершение и обработку ошибок.

### 8. Фоновые задачи — Background Tasks

Иногда необходимо запустить операцию, которая должна выполняться независимо от текущего участка кода.

Например:
- отправить уведомление;
- обновить кэш;
- выполнить периодическую проверку;
- обработать некритичную операцию.

Пример:

```python
import asyncio


background_tasks = set()


async def send_notification():
    await asyncio.sleep(2)
    print("Уведомление отправлено")


async def main():
    task = asyncio.create_task(send_notification())

    background_tasks.add(task)
    task.add_done_callback(background_tasks.discard)

    await asyncio.sleep(3)


asyncio.run(main())
```

Здесь:

- `create_task()` планирует выполнение;
- `background_tasks` хранит сильную ссылку на задачу;
- `add_done_callback()` удаляет задачу из множества после завершения.

Но это только минимальный пример управления ссылками. В реальном приложении также нужно обрабатывать исключения и корректно завершать фоновые задачи.

**Важный Backend-нюанс:** `create_task()` не создаёт надёжную очередь фоновых заданий. Если процесс приложения завершится, незавершённая задача может быть потеряна.

Для задач, которые должны переживать перезапуск сервиса, обычно используют Celery, RabbitMQ, Redis Streams, NATS JetStream или другие системы с механизмами хранения и повторной доставки.

### 9. create_task() vs gather()

Оба инструмента помогают организовать конкурентное выполнение, но используются по-разному.

**create_task():**

```python
task = asyncio.create_task(worker())
```

Создаёт и планирует одну задачу, предоставляя объект для управления её жизненным циклом.

**gather():**

```python
results = await asyncio.gather(
    worker(),
    worker(),
    worker(),
)
```

Координирует несколько awaitable-объектов и собирает их результаты.

| Характеристика | create_task() | gather() |
|---|---|---|
| Создаёт Task | Да | Автоматически планирует переданные корутины |
| Возвращает | Task | Awaitable для списка результатов |
| Можно отменить отдельную задачу | Да | Если сохранить отдельные Task |
| Собирает несколько результатов | Нет, не самостоятельно | Да |
| Основное назначение | Управление отдельной задачей | Координация группы операций |

### 10. create_task() vs TaskGroup

`asyncio.TaskGroup` появился в Python 3.11.

```python
import asyncio


async def worker(name):
    await asyncio.sleep(1)
    return name


async def main():
    async with asyncio.TaskGroup() as group:
        task1 = group.create_task(worker("A"))
        task2 = group.create_task(worker("B"))

    print(task1.result())
    print(task2.result())


asyncio.run(main())
```

Главное отличие — управление жизненным циклом группы.

`TaskGroup`:
- дожидается завершения дочерних задач;
- при обычной необработанной ошибке одной задачи отменяет остальные незавершённые;
- собирает ошибки в `ExceptionGroup` или `BaseExceptionGroup`;
- не позволяет незаметно оставить задачи выполняться после выхода из группы.

Для нескольких логически связанных операций `TaskGroup` часто предпочтительнее отдельных вызовов `create_task()`.

### 11. create_task() и Event Loop

Важно понимать, что Task не равен Thread.

```text
              Event Loop
                  |
        ┌─────────┼─────────┐
        |         |         |
        v         v         v
      Task A    Task B    Task C
        |         |         |
      await     await     await
        |         |         |
        └─────────┼─────────┘
                  |
                  v
          Возобновление задач
```

В обычном однопоточном Event Loop в конкретный момент исполняется одна задача с Python-кодом.

Когда задача достигает `await` и ожидаемая операция ещё не завершена, Event Loop может переключиться на другую готовую задачу.

Это позволяет эффективно обслуживать множество I/O-bound операций.

Но если задача выполняет тяжёлые синхронные вычисления без передачи управления, она блокирует Event Loop.

### 12. Практический пример для Backend

Представим, что необходимо получить данные пользователя и его заказы из двух независимых внешних API.

```python
import asyncio


async def get_user(user_id):
    await asyncio.sleep(1)
    return {"id": user_id, "name": "Ilya"}


async def get_orders(user_id):
    await asyncio.sleep(2)
    return [
        {"id": 101, "user_id": user_id},
        {"id": 102, "user_id": user_id},
    ]


async def get_dashboard(user_id):
    user_task = asyncio.create_task(
        get_user(user_id)
    )

    orders_task = asyncio.create_task(
        get_orders(user_id)
    )

    user = await user_task
    orders = await orders_task

    return {
        "user": user,
        "orders": orders,
    }


async def main():
    result = await get_dashboard(1)
    print(result)


asyncio.run(main())
```

Обе операции запускаются конкурентно.

Общее время будет примерно две секунды вместо трёх, поскольку ожидания перекрываются.

Для production-кода важно дополнительно продумать обработку ошибок и отмену второй задачи, если первая завершилась неуспешно. Для этого удобно использовать `TaskGroup`.

### 13. Типичные ошибки

**Ошибка 1. Считать, что вызов async-функции сразу запускает её выполнение.**

```python
coroutine = worker()
```

Это только создание объекта корутины.

**Ошибка 2. Считать create_task() созданием отдельного потока.**

Task выполняется в Event Loop, а не автоматически в новом потоке.

**Ошибка 3. Создавать задачи и забывать о них.**

Это может привести к необработанным исключениям и проблемам с жизненным циклом.

**Ошибка 4. Считать cancel() мгновенным завершением.**

Отмена кооперативная и может требовать времени.

**Ошибка 5. Использовать create_task() для CPU-bound операций в надежде ускорить вычисления.**

Синхронные CPU-bound вычисления могут блокировать Event Loop.

**Ошибка 6. Считать create_task() заменой Celery или брокера сообщений.**

Task существует внутри процесса приложения и не обеспечивает долговременное хранение заданий.

**Ошибка 7. Использовать один AsyncSession SQLAlchemy конкурентно в нескольких задачах.**

`AsyncSession` — изменяемый объект, представляющий состояние транзакции. Для конкурентных задач обычно нужны отдельные экземпляры сессий.

## 🎤 Вопросы на собеседовании

**1. Что делает asyncio.create_task()?**

Создаёт объект Task из корутины и планирует её выполнение в текущем Event Loop.

**2. Чем Coroutine отличается от Task?**

Coroutine представляет асинхронное вычисление, а Task управляет его выполнением.

**3. Запускает ли create_task() новый поток?**

Нет. Задача выполняется в Event Loop.

**4. Когда задача начинает выполняться?**

Когда Event Loop предоставляет ей управление. Точный момент зависит от планирования и конфигурации выполнения задач.

**5. Что возвращает create_task()?**

Объект `asyncio.Task`.

**6. Как получить результат Task?**

Через `await task`, а после завершения — через `task.result()`.

**7. Как отменить задачу?**

Вызвать `task.cancel()` и корректно обработать `CancelledError`.

**8. Что произойдёт, если задача завершится с исключением?**

Исключение сохраняется в Task. Его можно получить при `await task`, через `result()` или `exception()`.

**9. Чем create_task() отличается от gather()?**

`create_task()` создаёт отдельную задачу, а `gather()` координирует несколько awaitable-объектов и собирает результаты.

**10. Когда лучше использовать TaskGroup?**

Когда несколько задач логически связаны и требуется согласованное управление их завершением, ошибками и отменой.

**11. Можно ли использовать create_task() для фоновых задач в FastAPI?**

Да, но необходимо управлять их жизненным циклом. Для надёжной фоновой обработки с сохранением заданий при перезапуске процесса лучше использовать специализированную очередь задач.

**12. Почему create_task() полезен для I/O-bound операций?**

Потому что пока одна задача ожидает сетевой запрос или другую неблокирующую операцию, Event Loop может выполнять другие задачи.