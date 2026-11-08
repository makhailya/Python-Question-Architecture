# ⚡ asyncio.TaskGroup в Python

## 🎤 Короткий ответ

**`asyncio.TaskGroup`** — асинхронный контекстный менеджер для управления группой связанных задач в Python. Появился в **Python 3.11**.

Он реализует принцип **Structured Concurrency (структурированной конкурентности)**: задачи создаются внутри определённой области, и перед выходом из неё Python дожидается их завершения.

Если одна из задач завершается с обычным необработанным исключением, `TaskGroup` отменяет остальные незавершённые задачи и объединяет ошибки в `ExceptionGroup`.

## 🗣️ Ответ на собеседовании

`asyncio.TaskGroup` используется для конкурентного выполнения нескольких асинхронных операций с контролируемым жизненным циклом.

Например, Backend-сервису нужно одновременно получить пользователя, заказы и платежи из разных сервисов.

Я могу создать три задачи внутри `async with asyncio.TaskGroup()`. Все они будут выполняться конкурентно, а выход из блока произойдёт только после их завершения.

Главное отличие от `asyncio.gather()` — обработка ошибок. Если одна задача завершится с обычным исключением, `TaskGroup` отменит остальные незавершённые задачи, дождётся их завершения и передаст ошибки вызывающему коду.

Это помогает избежать ситуации, когда одна операция уже упала, а связанные с ней задачи продолжают работать.

Результаты можно получить через `task.result()` после выхода из контекстного менеджера.

На практике я бы использовал `TaskGroup` для нескольких логически связанных I/O-bound операций, когда их нужно выполнять конкурентно и завершать согласованно.

## 🧭 Где я нахожусь

```text
01 Python
└── 08 Concurrency и Asyncio
    ├── Event Loop
    ├── Coroutine
    ├── await
    ├── Task
    ├── Future
    └── Управление асинхронными задачами
        ├── asyncio.create_task()
        ├── asyncio.gather()
        ├── asyncio.wait()
        ├── asyncio.as_completed()
        ├── asyncio.TaskGroup ← Я здесь
        ├── Отмена задач
        ├── ExceptionGroup
        └── Structured Concurrency
```

## 📚 Разбор поглубже

### 1. Зачем нужен TaskGroup

Допустим, необходимо выполнить три независимых HTTP-запроса.

Каждый запрос занимает одну секунду.

При последовательном выполнении:

```python
user = await get_user()
orders = await get_orders()
payments = await get_payments()
```

Общее время ожидания составит примерно три секунды.

С `TaskGroup` можно запустить операции конкурентно:

```python
import asyncio


async def worker(name):
    await asyncio.sleep(1)
    return f"{name}: OK"


async def main():
    async with asyncio.TaskGroup() as group:
        user_task = group.create_task(worker("User"))
        orders_task = group.create_task(worker("Orders"))
        payments_task = group.create_task(worker("Payments"))

    print(user_task.result())
    print(orders_task.result())
    print(payments_task.result())


asyncio.run(main())
```

Результат:

```text
User: OK
Orders: OK
Payments: OK
```

Общее время — примерно одна секунда.

Почему?

Потому что все три задачи ожидают завершения неблокирующих операций конкурентно.

**Важно:** `TaskGroup` не создаёт новые потоки или процессы. Задачи выполняются через Event Loop.

### 2. Как работает TaskGroup

Упрощённая схема:

```text
       async with TaskGroup()
                 |
                 v
          Создание задач
                 |
         ┌───────┼───────┐
         v       v       v
       Task A  Task B  Task C
         |       |       |
       await   await   await
         |       |       |
         └───────┼───────┘
                 |
                 v
       Ожидание завершения
                 |
          ┌──────┴──────┐
          |             |
          v             v
       Успех          Ошибка
          |             |
          v             v
      Результаты    Отмена других
                        |
                        v
                Ожидание отмены
                        |
                        v
                  ExceptionGroup
```

При нормальном выполнении `TaskGroup` дожидается всех задач.

При обычной необработанной ошибке одной задачи он отменяет остальные незавершённые задачи.

### 3. Синтаксис

```python
async with asyncio.TaskGroup() as group:
    task1 = group.create_task(coroutine1())
    task2 = group.create_task(coroutine2())
```

Разберём:

- `async with` — асинхронный контекстный менеджер.
- `asyncio.TaskGroup()` — создаёт группу задач.
- `group.create_task()` — добавляет задачу в группу.
- Выход из блока — точка, в которой гарантируется завершение задач группы, если выполнение не прервано внешними обстоятельствами.

После успешного выхода можно получить результаты:

```python
result1 = task1.result()
result2 = task2.result()
```

В отличие от `gather()`, `TaskGroup` не возвращает общий список результатов автоматически.

### 4. Что такое Structured Concurrency

**Structured Concurrency** — подход, при котором жизненный цикл конкурентных задач связан с областью кода, где они были созданы.

Без структурированной конкурентности:

```python
async def main():
    asyncio.create_task(background_job())

    print("Основная функция завершилась")
```

Здесь создаётся задача, но не определено, кто отвечает за её завершение и ошибки.

При завершении `asyncio.run()` незавершённые задачи будут отменены. В долгоживущем приложении такая задача может продолжать выполняться без должного контроля.

С `TaskGroup`:

```python
async def main():
    async with asyncio.TaskGroup() as group:
        group.create_task(background_job())

    print("Все задачи группы завершились")
```

Теперь жизненный цикл задачи связан с блоком `async with`.

**Главная идея:** дочерние задачи не должны бесконтрольно переживать область, которая отвечает за их выполнение.

### 5. Обработка ошибок

Это одна из самых важных особенностей `TaskGroup`.

```python
import asyncio


async def success():
    try:
        await asyncio.sleep(5)
        print("Успешно")
    finally:
        print("Очистка ресурсов")


async def failure():
    await asyncio.sleep(1)
    raise ValueError("Ошибка в задаче")


async def main():
    try:
        async with asyncio.TaskGroup() as group:
            group.create_task(success())
            group.create_task(failure())

    except* ValueError as errors:
        print("Поймали ошибку:", errors)


asyncio.run(main())
```

Что произойдёт:

1. Создаются две задачи.
2. Обе начинают выполняться конкурентно.
3. Через секунду `failure()` выбрасывает `ValueError`.
4. `TaskGroup` запрашивает отмену `success()`.
5. В `success()` выполняется блок `finally`.
6. Группа дожидается завершения отмены.
7. Ошибка передаётся через `ExceptionGroup`.

Пример вывода:

```text
Очистка ресурсов
Поймали ошибку: unhandled errors in a TaskGroup (1 sub-exception)
```

Задача `success()` не успеет вывести `Успешно`, потому что её отменят до истечения пяти секунд.

### 6. Что такое ExceptionGroup

`ExceptionGroup` — механизм Python 3.11+, позволяющий объединять несколько исключений.

Он полезен при конкурентном выполнении, потому что несколько задач могут завершиться ошибками.

Для обработки отдельных типов исключений используется `except*`.

```python
try:
    async with asyncio.TaskGroup() as group:
        group.create_task(operation_a())
        group.create_task(operation_b())

except* ValueError as errors:
    print("Ошибки значений:", errors)

except* ConnectionError as errors:
    print("Сетевые ошибки:", errors)
```

`except*` позволяет обработать подходящие исключения внутри группы.

**Важный нюанс:** если одна задача завершилась ошибкой, остальные получают запрос на отмену. Поэтому не обязательно, что все потенциальные ошибки успеют произойти.

Также `KeyboardInterrupt` и `SystemExit` имеют специальное поведение: после завершения остальных задач они передаются дальше, а не обрабатываются как обычные ошибки группы.

### 7. Отмена задач внутри TaskGroup

Отмена работает через `asyncio.CancelledError`.

```python
import asyncio


async def worker():
    try:
        while True:
            print("Работаю...")
            await asyncio.sleep(1)

    except asyncio.CancelledError:
        print("Задача получила отмену")
        raise


async def main():
    async with asyncio.TaskGroup() as group:
        task = group.create_task(worker())

        await asyncio.sleep(2)

        task.cancel()


asyncio.run(main())
```

Здесь мы вручную отменяем одну задачу.

Если задача корректно завершится через `CancelledError`, это само по себе не считается обычной ошибкой группы и не приводит к отмене остальных задач.

**Не следует без необходимости подавлять `CancelledError`.** Это может нарушить работу механизмов структурированной конкурентности и таймаутов.

### 8. TaskGroup vs asyncio.gather()

Это ключевой вопрос для собеседования.

| Характеристика | TaskGroup | gather() |
|---|---|---|
| Конкурентное выполнение | Да | Да |
| Появился | Python 3.11 | Ранее |
| Создание задач | `group.create_task()` | Автоматически для корутин |
| Получение результатов | Через объекты Task | Общий список |
| Ошибка одной задачи | Отменяет остальные незавершённые | По умолчанию не отменяет |
| Обработка нескольких ошибок | ExceptionGroup | Зависит от режима |
| Structured Concurrency | Да | Нет в строгом смысле |

Пример `gather()`:

```python
results = await asyncio.gather(
    get_user(),
    get_orders(),
    get_payments(),
)
```

Пример `TaskGroup`:

```python
async with asyncio.TaskGroup() as group:
    user_task = group.create_task(get_user())
    orders_task = group.create_task(get_orders())
    payments_task = group.create_task(get_payments())

results = [
    user_task.result(),
    orders_task.result(),
    payments_task.result(),
]
```

**Когда выбирать `gather()`?**

Когда удобно собрать результаты независимых операций в список, особенно если нужно отдельно обрабатывать ошибки через `return_exceptions=True`.

**Когда выбирать `TaskGroup`?**

Когда операции логически связаны и при ошибке одной нужно согласованно завершить остальные.

### 9. TaskGroup vs asyncio.create_task()

`asyncio.create_task()` создаёт отдельную задачу:

```python
task = asyncio.create_task(worker())
```

Но ответственность за её жизненный цикл остаётся у вызывающего кода.

`TaskGroup` создаёт и управляет группой:

```python
async with asyncio.TaskGroup() as group:
    task = group.create_task(worker())
```

Группа гарантирует, что выход из контекстного менеджера не произойдёт до завершения её задач.

Это особенно полезно, когда создаётся несколько связанных операций.

### 10. Можно ли создавать задачи динамически?

Да.

Например, количество запросов определяется во время выполнения:

```python
import asyncio


async def fetch_user(user_id):
    await asyncio.sleep(0.5)
    return {"id": user_id}


async def main():
    user_ids = [1, 2, 3, 4, 5]

    async with asyncio.TaskGroup() as group:
        tasks = [
            group.create_task(fetch_user(user_id))
            for user_id in user_ids
        ]

    results = [task.result() for task in tasks]

    print(results)


asyncio.run(main())
```

Результаты:

```text
[{'id': 1}, {'id': 2}, {'id': 3}, {'id': 4}, {'id': 5}]
```

Задачи можно добавлять, пока группа активна, в том числе из других задач, которым передали ссылку на эту группу.

### 11. TaskGroup и Semaphore

`TaskGroup` не ограничивает количество одновременно выполняемых операций.

Если создать 1000 задач, они все будут запланированы на выполнение.

Для ограничения конкурентности можно использовать `asyncio.Semaphore`.

```python
import asyncio


async def fetch_data(number, semaphore):
    async with semaphore:
        await asyncio.sleep(1)
        return number


async def main():
    semaphore = asyncio.Semaphore(5)

    async with asyncio.TaskGroup() as group:
        tasks = [
            group.create_task(fetch_data(i, semaphore))
            for i in range(20)
        ]

    results = [task.result() for task in tasks]

    print(results)


asyncio.run(main())
```

Здесь:

- `TaskGroup` управляет жизненным циклом 20 задач.
- `Semaphore(5)` ограничивает количество задач внутри защищённого участка до пяти.
- При обычной необработанной ошибке одной задачи группа отменит остальные незавершённые.

Это полезная комбинация для Backend-сервисов, которые обращаются к внешним API.

### 12. TaskGroup и таймауты

Для ограничения времени выполнения группы можно использовать `asyncio.timeout()`.

```python
import asyncio


async def worker():
    await asyncio.sleep(10)
    return "OK"


async def main():
    try:
        async with asyncio.timeout(2):
            async with asyncio.TaskGroup() as group:
                group.create_task(worker())
                group.create_task(worker())

    except TimeoutError:
        print("Превышено время ожидания")


asyncio.run(main())
```

Через две секунды таймаут запросит отмену текущей задачи.

`TaskGroup` обработает отмену и дождётся завершения дочерних задач.

Затем `asyncio.timeout()` преобразует соответствующую отмену в `TimeoutError`.

Важно: отмена кооперативная, поэтому фактическое время завершения может превышать две секунды.

### 13. Практический пример в Backend

Представим FastAPI-приложение, которое собирает данные для личного кабинета.

Нужно получить:

- информацию о пользователе;
- список заказов;
- историю платежей.

```python
import asyncio


async def get_user(user_id):
    await asyncio.sleep(0.5)
    return {
        "id": user_id,
        "name": "Ilya",
    }


async def get_orders(user_id):
    await asyncio.sleep(1)
    return [
        {"id": 101, "user_id": user_id},
        {"id": 102, "user_id": user_id},
    ]


async def get_payments(user_id):
    await asyncio.sleep(0.7)
    return [
        {"id": 501, "amount": 1500},
    ]


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


async def main():
    dashboard = await get_dashboard(1)
    print(dashboard)


asyncio.run(main())
```

Общее время выполнения будет примерно одна секунда, потому что самая долгая операция занимает одну секунду.

Если один из сервисов завершится с обычной необработанной ошибкой, `TaskGroup` отменит остальные незавершённые операции.

**Важный нюанс для SQLAlchemy:** не следует использовать один экземпляр `AsyncSession` конкурентно в нескольких задачах. Каждой конкурентной операции, работающей с БД, обычно нужна собственная сессия.

### 14. Типичные ошибки

**Ошибка 1. Считать, что TaskGroup автоматически возвращает список результатов.**

Нет. Результаты получают через сохранённые объекты Task после успешного выхода из группы.

**Ошибка 2. Путать TaskGroup с gather().**

Оба инструмента обеспечивают конкурентное выполнение, но отличаются управлением ошибками и жизненным циклом.

**Ошибка 3. Считать, что TaskGroup создаёт потоки.**

Нет. Он использует Event Loop.

**Ошибка 4. Подавлять CancelledError без необходимости.**

Это может нарушить корректную отмену задач и работу структурированной конкурентности.

**Ошибка 5. Считать, что TaskGroup ограничивает количество одновременных запросов.**

Нет. Для этого используют `Semaphore`, пулы соединений или другие механизмы ограничения нагрузки.

**Ошибка 6. Получать task.result() до завершения задачи.**

Если задача ещё не завершена, будет поднято `InvalidStateError`.

**Ошибка 7. Считать TaskGroup заменой очереди задач.**

`TaskGroup` управляет задачами внутри процесса Python. Он не обеспечивает хранение заданий при перезапуске приложения и не заменяет Celery или брокер сообщений.

**Ошибка 8. Считать, что TaskGroup откатывает выполненные операции при ошибке.**

Отмена задач не откатывает уже совершённые изменения в БД или внешних сервисах.

Для согласованности бизнес-операций нужны транзакции, идемпотентность или другие механизмы.

## 🎤 Вопросы на собеседовании

**1. Что такое asyncio.TaskGroup?**

Асинхронный контекстный менеджер для управления группой задач, реализующий структурированную конкурентность.

**2. В какой версии Python появился TaskGroup?**

В Python 3.11.

**3. Чем TaskGroup отличается от gather()?**

`TaskGroup` при обычной необработанной ошибке одной задачи отменяет остальные незавершённые задачи и управляет их завершением. `gather()` по умолчанию этого не делает.

**4. Что такое Structured Concurrency?**

Подход, при котором жизненный цикл дочерних задач ограничен определённой областью кода.

**5. Как получить результаты TaskGroup?**

Сохранить объекты Task, а после успешного выхода из блока вызвать `task.result()`.

**6. Что произойдёт, если одна задача завершится с ValueError?**

Группа отменит остальные незавершённые задачи, дождётся их завершения и передаст ошибки через `ExceptionGroup`.

**7. Что такое ExceptionGroup?**

Механизм Python для объединения нескольких исключений, который можно обрабатывать через `except*`.

**8. Можно ли использовать Semaphore вместе с TaskGroup?**

Да. `TaskGroup` управляет жизненным циклом задач, а `Semaphore` ограничивает количество одновременно выполняемых операций внутри защищённого участка.

**9. Можно ли использовать TaskGroup для CPU-bound задач?**

Он не обеспечивает параллельное выполнение тяжёлых синхронных вычислений на нескольких ядрах. Для этого могут понадобиться процессы или другие механизмы.

**10. Где TaskGroup полезен в Backend?**

При выполнении нескольких логически связанных асинхронных операций, например запросов к внешним API, когда важно согласованно обрабатывать ошибки и отмену.

## 🧪 Мини-задание

Создай три асинхронные функции:

- `task_a()` — ожидает одну секунду и возвращает `"A"`.
- `task_b()` — ожидает две секунды и выбрасывает `ValueError`.
- `task_c()` — ожидает пять секунд и возвращает `"C"`.

Запусти их через `asyncio.TaskGroup` и обработай ошибку с помощью `except* ValueError`.

**Вопросы для самопроверки:**

1. Успеет ли `task_a()` завершиться?
2. Что произойдёт с `task_c()` после ошибки в `task_b()`?
3. Почему `TaskGroup` отличается в этой ситуации от `asyncio.gather()`?

**Главное, что нужно запомнить:** `asyncio.TaskGroup` — это не просто способ запустить несколько корутин. Это инструмент для управления связанными задачами как единой группой с предсказуемой обработкой ошибок и отмены.