# ⚡ asyncio.wait() в Python

## 🎤 Короткий ответ

**`asyncio.wait()`** — функция из стандартной библиотеки Python, которая позволяет ожидать завершения нескольких асинхронных задач по заданному условию.

Она поддерживает три режима ожидания:

- `ALL_COMPLETED` — дождаться всех задач.
- `FIRST_COMPLETED` — дождаться первой завершённой задачи.
- `FIRST_EXCEPTION` — дождаться первой задачи с исключением.

Функция возвращает два множества: `done` (завершённые задачи) и `pending` (ещё не завершённые).

В отличие от `asyncio.gather()`, `asyncio.wait()` не собирает результаты в список и не отменяет оставшиеся задачи автоматически.

## 🗣️ Ответ на собеседовании

`asyncio.wait()` используется для управления ожиданием группы асинхронных задач.

Например, если Backend-сервис отправляет запросы к нескольким независимым источникам данных, иногда достаточно дождаться первого ответа, а не завершения всех запросов.

Для этого можно создать задачи через `asyncio.create_task()` и передать их в `asyncio.wait()` с параметром `return_when=asyncio.FIRST_COMPLETED`.

Функция вернёт два множества: `done` с завершёнными задачами и `pending` с теми, которые продолжают выполняться.

При необходимости оставшиеся задачи можно отменить через `cancel()` и затем дождаться обработки отмены.

Также `asyncio.wait()` поддерживает таймаут. В отличие от `asyncio.wait_for()`, при его истечении функция возвращает текущие множества `done` и `pending`, не выбрасывая `TimeoutError` и не отменяя незавершённые задачи.

В современных версиях Python `asyncio.wait()` принимает задачи и Future-объекты, а не обычные объекты корутин.

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
        ├── asyncio.create_task()
        ├── asyncio.gather()
        ├── asyncio.wait() ← Я здесь
        ├── asyncio.as_completed()
        ├── asyncio.TaskGroup
        ├── Отмена задач
        └── Таймауты
```

## 📚 Разбор поглубже

### 1. Зачем нужен asyncio.wait()

Представим, что у нас есть три асинхронные операции:

- Запрос A — 3 секунды.
- Запрос B — 1 секунда.
- Запрос C — 2 секунды.

Иногда нужно дождаться всех трёх запросов. Но бывают ситуации, когда достаточно первого завершившегося запроса.

`asyncio.wait()` позволяет выбрать условие, при котором ожидание закончится.

Общая схема:

```text
              asyncio.wait()
                     |
             ┌───────┼───────┐
             v       v       v
           Task A  Task B  Task C
             3с      1с      2с
                     |
                     v
              FIRST_COMPLETED
                     |
             ┌───────┴───────┐
             v               v
           done            pending
          {Task B}       {Task A, Task C}
```

Задачи из `pending` продолжают выполняться, если их явно не отменить.

### 2. Синтаксис asyncio.wait()

```python
done, pending = await asyncio.wait(
    tasks,
    timeout=None,
    return_when=asyncio.ALL_COMPLETED,
)
```

Параметры:

| Параметр | Назначение |
|---|---|
| `tasks` | Непустая коллекция Task или Future |
| `timeout` | Максимальное время ожидания в секундах |
| `return_when` | Условие завершения ожидания |

Возвращаемое значение:

```python
done, pending
```

- `done` — множество завершённых задач.
- `pending` — множество незавершённых задач.

Завершённая задача может содержать успешный результат, исключение или состояние отмены.

### 3. ALL_COMPLETED — ожидание всех задач

Это режим по умолчанию.

```python
import asyncio


async def worker(name, delay):
    await asyncio.sleep(delay)
    return f"{name}: готово"


async def main():
    tasks = [
        asyncio.create_task(worker("A", 3)),
        asyncio.create_task(worker("B", 1)),
        asyncio.create_task(worker("C", 2)),
    ]

    done, pending = await asyncio.wait(
        tasks,
        return_when=asyncio.ALL_COMPLETED,
    )

    for task in done:
        print(task.result())

    print("Осталось задач:", len(pending))


asyncio.run(main())
```

Пример результата:

```text
B: готово
A: готово
C: готово
Осталось задач: 0
```

Порядок вывода может отличаться, поскольку `done` — множество, которое не сохраняет порядок завершения или создания задач.

Общее время выполнения — примерно 3 секунды.

### 4. FIRST_COMPLETED — ожидание первой задачи

Этот режим позволяет продолжить работу, когда завершилась хотя бы одна задача.

```python
import asyncio


async def worker(name, delay):
    await asyncio.sleep(delay)
    return name


async def main():
    tasks = [
        asyncio.create_task(worker("A", 3)),
        asyncio.create_task(worker("B", 1)),
        asyncio.create_task(worker("C", 2)),
    ]

    done, pending = await asyncio.wait(
        tasks,
        return_when=asyncio.FIRST_COMPLETED,
    )

    for task in done:
        print("Завершилась:", task.result())

    for task in pending:
        task.cancel()

    await asyncio.gather(
        *pending,
        return_exceptions=True,
    )


asyncio.run(main())
```

Пример результата:

```text
Завершилась: B
```

Что происходит:

1. Создаются три задачи.
2. Они планируются на конкурентное выполнение.
3. `wait()` ожидает первую завершённую задачу.
4. Задача B завершается примерно через секунду.
5. `wait()` возвращает `done` и `pending`.
6. Незавершённые задачи получают запрос на отмену.
7. `gather()` ожидает завершения процедуры отмены.

**Важно:** `FIRST_COMPLETED` означает первую завершённую задачу, а не первую успешно выполненную.

Если задача завершилась с ошибкой или была отменена, это тоже считается завершением.

### 5. FIRST_EXCEPTION — ожидание первой ошибки

Этот режим используется, когда нужно завершить ожидание при первой ошибке.

```python
import asyncio


async def worker(name, delay, fail=False):
    await asyncio.sleep(delay)

    if fail:
        raise ValueError(f"Ошибка в задаче {name}")

    return name


async def main():
    tasks = [
        asyncio.create_task(worker("A", 3)),
        asyncio.create_task(worker("B", 1, fail=True)),
        asyncio.create_task(worker("C", 2)),
    ]

    done, pending = await asyncio.wait(
        tasks,
        return_when=asyncio.FIRST_EXCEPTION,
    )

    for task in done:
        if not task.cancelled():
            error = task.exception()

            if error is not None:
                print("Ошибка:", error)
            else:
                print("Результат:", task.result())

    for task in pending:
        task.cancel()

    await asyncio.gather(
        *pending,
        return_exceptions=True,
    )


asyncio.run(main())
```

Пример результата:

```text
Ошибка: Ошибка в задаче B
```

Особенности:

- Если одна задача завершается с исключением, `wait()` может вернуть управление.
- Если ни одна задача не завершилась с исключением, `FIRST_EXCEPTION` ведёт себя как `ALL_COMPLETED`.
- Отмена задачи сама по себе не считается обычным исключением для срабатывания `FIRST_EXCEPTION`.
- Оставшиеся задачи не отменяются автоматически.

### 6. timeout — ограничение времени ожидания

`asyncio.wait()` поддерживает параметр `timeout`.

```python
import asyncio


async def worker():
    await asyncio.sleep(5)
    return "OK"


async def main():
    task = asyncio.create_task(worker())

    done, pending = await asyncio.wait(
        {task},
        timeout=2,
    )

    print("Завершено:", len(done))
    print("Ожидают:", len(pending))

    for task in pending:
        task.cancel()

    await asyncio.gather(
        *pending,
        return_exceptions=True,
    )


asyncio.run(main())
```

Результат:

```text
Завершено: 0
Ожидают: 1
```

Почему?

Задача должна выполняться пять секунд, но `wait()` ожидает только две.

По истечении таймаута функция возвращает управление.

При этом `asyncio.wait()` **не отменяет задачи автоматически и не выбрасывает `TimeoutError`**.

### 7. asyncio.wait() vs asyncio.wait_for()

Названия похожи, но поведение различается.

**`asyncio.wait()`**

```python
done, pending = await asyncio.wait(
    {task},
    timeout=2,
)
```

Возвращает множества завершённых и незавершённых задач.

**`asyncio.wait_for()`**

```python
try:
    result = await asyncio.wait_for(
        worker(),
        timeout=2,
    )
except TimeoutError:
    print("Таймаут")
```

При истечении таймаута `wait_for()` запрашивает отмену ожидаемой операции и поднимает `TimeoutError`, если отмена завершилась соответствующим образом.

| Характеристика | wait() | wait_for() |
|---|---|---|
| Основная задача | Ожидание группы задач | Ожидание операции с таймаутом |
| Возвращает | `done`, `pending` | Результат операции |
| Отменяет при таймауте | Нет | Обычно да |
| Выбрасывает TimeoutError | Нет | Да |
| Поддерживает FIRST_COMPLETED | Да | Нет |

### 8. asyncio.wait() vs asyncio.gather()

Это один из частых вопросов на собеседовании.

**`asyncio.gather()`**

```python
results = await asyncio.gather(
    worker_a(),
    worker_b(),
)
```

Используется для сбора результатов нескольких операций.

**`asyncio.wait()`**

```python
task1 = asyncio.create_task(worker_a())
task2 = asyncio.create_task(worker_b())

done, pending = await asyncio.wait(
    {task1, task2},
    return_when=asyncio.FIRST_COMPLETED,
)
```

Используется для управления условиями ожидания.

| Характеристика | gather() | wait() |
|---|---|---|
| Принимает корутины напрямую | Да | Нет |
| Возвращает результаты | Список | Множества Task/Future |
| Сохраняет порядок аргументов | Да | Нет |
| Ожидание первой завершённой | Нет | Да |
| Таймаут отдельным параметром | Нет | Да |
| Автоматически отменяет pending при раннем возврате | Нет | Нет |

### 9. asyncio.wait() vs asyncio.as_completed()

`asyncio.wait()` позволяет дождаться определённого условия и получить две группы задач.

`asyncio.as_completed()` позволяет обрабатывать результаты по мере завершения операций.

Пример:

```python
import asyncio


async def worker(name, delay):
    await asyncio.sleep(delay)
    return name


async def main():
    coroutines = [
        worker("A", 3),
        worker("B", 1),
        worker("C", 2),
    ]

    for completed in asyncio.as_completed(coroutines):
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

Если нужно обработать все результаты по мере готовности, `as_completed()` обычно удобнее.

Если необходимо получить группы `done` и `pending`, например для отмены оставшихся запросов, удобнее `wait()`.

### 10. Практический пример в Backend

Представим, что Backend-сервис обращается к нескольким зеркалам одного API.

Нужно получить первый успешный ответ.

Здесь есть важный нюанс: `FIRST_COMPLETED` может вернуть ошибку. Поэтому нельзя считать первую завершённую задачу обязательно успешной.

```python
import asyncio


async def request_server(name, delay, fail=False):
    await asyncio.sleep(delay)

    if fail:
        raise ConnectionError(f"{name} недоступен")

    return f"Ответ от {name}"


async def get_first_success():
    tasks = {
        asyncio.create_task(request_server("A", 3)),
        asyncio.create_task(request_server("B", 1, fail=True)),
        asyncio.create_task(request_server("C", 2)),
    }

    try:
        while tasks:
            done, tasks = await asyncio.wait(
                tasks,
                return_when=asyncio.FIRST_COMPLETED,
            )

            for task in done:
                try:
                    result = task.result()
                except ConnectionError:
                    continue

                return result

        raise RuntimeError("Все серверы недоступны")

    finally:
        for task in tasks:
            task.cancel()

        await asyncio.gather(
            *tasks,
            return_exceptions=True,
        )


async def main():
    result = await get_first_success()
    print(result)


asyncio.run(main())
```

Результат:

```text
Ответ от C
```

Логика:

1. Запускаются запросы к трём серверам.
2. Сервер B отвечает первым, но с ошибкой.
3. Ошибка обрабатывается, ожидание продолжается.
4. Сервер C возвращает успешный ответ.
5. Функция возвращает его результат.
6. Оставшиеся незавершённые задачи отменяются.

Этот подход называют **hedged requests** или конкурентными запросами к альтернативным источникам, если запросы действительно эквивалентны и безопасны для повторного выполнения.

Для операций с побочными эффектами, например создания платежей, такая схема без дополнительных гарантий может быть опасной.

### 11. Что происходит под капотом

`asyncio.wait()` не создаёт отдельные потоки и не выполняет CPU-bound код параллельно.

Он работает с задачами и Future-объектами, зарегистрированными в Event Loop.

Упрощённо:

```text
asyncio.wait(tasks)
        |
        v
Регистрация ожидания
        |
        v
Event Loop выполняет задачи
        |
        ├── Task A → await
        ├── Task B → await
        └── Task C → await
        |
        v
Задачи завершаются
        |
        v
Проверка return_when
        |
        ├── Условие не выполнено → ждём
        |
        └── Условие выполнено
                    |
                    v
              done, pending
```

`wait()` наблюдает за состоянием задач и возвращает управление, когда выполняется условие ожидания или истекает таймаут.

Он не управляет бизнес-логикой задач и не отменяет их автоматически.

### 12. Типичные ошибки

**Ошибка 1. Передавать корутины напрямую.**

Неправильно:

```python
await asyncio.wait([
    worker_a(),
    worker_b(),
])
```

В современных версиях Python передача обычных объектов корутин в `asyncio.wait()` запрещена.

Правильно:

```python
tasks = [
    asyncio.create_task(worker_a()),
    asyncio.create_task(worker_b()),
]

done, pending = await asyncio.wait(tasks)
```

**Ошибка 2. Считать done списком результатов.**

`done` содержит объекты Task/Future, а не возвращённые значения корутин.

Результаты получают через `task.result()` после проверки состояния.

**Ошибка 3. Считать FIRST_COMPLETED первым успешным результатом.**

Задача может завершиться исключением или отменой.

**Ошибка 4. Забывать про pending.**

После возврата `wait()` незавершённые задачи могут продолжать выполняться.

Нужно явно решить, что с ними делать: дождаться, отменить или передать управление их жизненным циклом другому компоненту.

**Ошибка 5. Считать timeout автоматической отменой.**

`asyncio.wait(timeout=...)` не отменяет незавершённые задачи.

**Ошибка 6. Не обрабатывать исключения завершённых задач.**

`wait()` не поднимает автоматически исключения дочерних задач. Их необходимо извлекать и обрабатывать.

**Ошибка 7. Передавать пустую коллекцию.**

`asyncio.wait()` не принимает пустой набор задач и выбрасывает `ValueError`.

## 🎤 Вопросы на собеседовании

**1. Что делает asyncio.wait()?**

Ожидает выполнения заданного условия для набора асинхронных задач и возвращает множества `done` и `pending`.

**2. Что возвращает asyncio.wait()?**

Два множества: завершённые и незавершённые задачи.

**3. Какие режимы return_when существуют?**

`ALL_COMPLETED`, `FIRST_COMPLETED`, `FIRST_EXCEPTION`.

**4. Чем FIRST_COMPLETED отличается от FIRST_EXCEPTION?**

`FIRST_COMPLETED` срабатывает при любом завершении задачи, а `FIRST_EXCEPTION` — при завершении задачи с исключением. Если ошибок нет, последний режим ожидает все задачи.

**5. Отменяет ли wait() оставшиеся задачи?**

Нет. Управление незавершёнными задачами остаётся за вызывающим кодом.

**6. Чем wait() отличается от gather()?**

`gather()` собирает результаты в порядке аргументов, а `wait()` возвращает множества задач и позволяет управлять условиями ожидания.

**7. Чем wait() отличается от wait_for()?**

`wait()` ожидает группу задач и не отменяет их при таймауте. `wait_for()` ограничивает ожидание операции по времени и обычно отменяет её при превышении таймаута.

**8. Можно ли передавать корутины напрямую в asyncio.wait()?**

В современных версиях Python нельзя. Нужно предварительно создать Task или использовать совместимые Future-объекты.

**9. Как получить результат завершённой задачи?**

Через `task.result()`, предварительно убедившись, что задача завершилась и не была отменена.

**10. Где asyncio.wait() полезен в Backend?**

Когда нужно дождаться первого ответа, ограничить ожидание группы запросов, обработать частичные результаты или управлять незавершёнными операциями.

