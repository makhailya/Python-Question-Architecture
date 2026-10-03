# ⏳ `await` в Python

## 🎤 Короткий ответ

`await` — оператор, который используется внутри `async def` для **ожидания awaitable-объекта**.

Когда [[Корутина - (coroutine)]] доходит до `await`, она **приостанавливает своё выполнение**, а управление возвращается [[Event Loop — цикл событий]]. Event Loop в это время может выполнять другие готовые задачи.

```python
async def get_data():
    result = await fetch_data()
    return result
```

Важно: `await` **не блокирует весь Event Loop**, если ожидаемая операция действительно асинхронная и неблокирующая.

---

## 🗣️ Ответ на собеседовании

`await` используется внутри асинхронной функции `async def` и позволяет дождаться результата другой асинхронной операции.

Когда [[Корутина - (coroutine)]] встречает `await`, она приостанавливает своё выполнение до завершения ожидаемой операции. При этом [[Event Loop — цикл событий]] получает возможность заняться другими задачами.

Например, если [[Корутина - (coroutine)]] отправила асинхронный запрос к базе данных и ждёт ответа, [[Event Loop — цикл событий]] может переключиться на обработку других запросов.

При этом важно понимать, что `await` сам по себе не делает любую функцию асинхронной. Ожидать нужно [[Awaitable]] — например [[Корутина - (coroutine)]], [[Task]] или другой объект, поддерживающий протокол ожидания.

И ещё важный момент: если внутри [[await]] находится блокирующая синхронная операция, [[Event Loop — цикл событий]] всё равно может заблокироваться. Поэтому асинхронный код должен использовать неблокирующие операции либо выносить блокирующую работу в отдельный поток или процесс.

---

## 🧭 Где я нахожусь

```text
Python
│
└── 08 Concurrency и Asyncio
    │
    ├── Многозадачность
    │   ├── Конкурентность
    │   └── Параллелизм
    │
    └── Asyncio
        │
        ├── Event Loop
        │
        ├── Coroutine
        │
        ├── await ← Я здесь
        │
        ├── Task
        ├── Future
        ├── Selector
        ├── Non-blocking I/O
        └── I/O Multiplexing
```

---

# 📚 Разбор поглубже

## 1. Что делает `await`

Возьмём простой пример:

```python
import asyncio


async def get_data():
    await asyncio.sleep(2)
    return "Данные"
```

Когда выполнение доходит до:

```python
await asyncio.sleep(2)
```

происходит примерно следующее:

```text
get_data()
    │
    ▼
await sleep(2)
    │
    ├── coroutine приостанавливается
    │
    └── управление возвращается Event Loop
                │
                ├── Task B
                ├── Task C
                └── другие задачи
```

Через две секунды операция становится готовой, и Event Loop сможет продолжить `get_data()`.

---

# 2. `await` не означает «остановить программу»

Это распространённая ошибка.

Неправильно представлять:

```text
await
 ↓
вся программа остановилась
```

Правильнее:

```text
await
 ↓
текущая coroutine приостановилась
 ↓
Event Loop продолжает работать
 ↓
другие задачи могут выполняться
```

Именно это делает `await` важным механизмом конкурентности.

---

# 3. Пример с двумя задачами

```python
import asyncio


async def task_a():
    print("A: начало")
    await asyncio.sleep(2)
    print("A: конец")


async def task_b():
    print("B: начало")
    await asyncio.sleep(1)
    print("B: конец")


async def main():
    await asyncio.gather(
        task_a(),
        task_b(),
    )


asyncio.run(main())
```

Упрощённо происходящее выглядит так:

```text
Время →

Task A: ███ await ───────── ███
Task B: ███ await ─── ███

Event Loop:
          ↑
          └── пока A ждёт,
              занимается B
```

Поэтому две операции ожидания могут выполняться **конкурентно**, не блокируя друг друга.

---

# 4. Что можно `await`

`await` работает с **awaitable**-объектами.

К основным awaitable относятся:

```text
Coroutine
Task
Future
```

Например:

### Coroutine

```python
async def fetch():
    return "data"


async def main():
    result = await fetch()
```

### Task

```python
task = asyncio.create_task(fetch())

result = await task
```

### Future

`Future` представляет результат асинхронной операции, который будет доступен позже.

```python
future = asyncio.Future()

result = await future
```

---

# 5. Почему `await` нельзя использовать где угодно

Например, так нельзя:

```python
def main():
    result = await fetch()
```

Будет ошибка.

`await` используется внутри асинхронного контекста:

```python
async def main():
    result = await fetch()
```

То есть:

```text
def
 ↓
обычная функция
 ↓
await нельзя

async def
 ↓
coroutine function
 ↓
await можно
```

---

# 6. `await` и coroutine

Очень важно различать:

```python
async def fetch():
    return "data"
```

Само объявление:

```python
async def
```

создаёт **coroutine function**.

Когда мы вызываем её:

```python
fetch()
```

получаем **coroutine object**.

Она ещё не выполнила тело функции обычным способом.

Чтобы передать управление её выполнению, coroutine должна быть запланирована или awaited:

```python
await fetch()
```

или:

```python
asyncio.create_task(fetch())
```

---

# 7. Что происходит под капотом

Упрощённая модель:

```text
Coroutine
    │
    ▼
await
    │
    ▼
Awaitable
    │
    ▼
Task / Future
    │
    ▼
Event Loop
    │
    ▼
ожидание I/O
    │
    ▼
I/O завершилось
    │
    ▼
Task становится готовой
    │
    ▼
Event Loop продолжает coroutine
```

В реальности механизм сложнее и включает планирование задач, callbacks, Futures и взаимодействие с механизмами ОС для неблокирующего I/O.

---

# 8. `await` и Event Loop

Event Loop — центральный механизм `asyncio`.

Упрощённо:

```text
              Event Loop
                  │
        ┌─────────┼─────────┐
        ▼         ▼         ▼
      Task A    Task B    Task C
        │         │         │
      await     await     CPU
        │         │
        ▼         ▼
       I/O       I/O
        │         │
        └────┬────┘
             │
       I/O завершилось
             │
             ▼
      Task снова готова
```

`await` сообщает механизму выполнения:

> «Эта coroutine сейчас не может продолжать работу; продолжи её, когда ожидаемая операция будет готова».

---

# 9. `await` и блокирующие операции

Вот здесь находится один из самых важных нюансов.

Само наличие `await` **не гарантирует**, что код неблокирующий.

Например, концептуально плохо:

```python
async def main():
    await blocking_function()
```

если `blocking_function()` выполняет обычную долгую синхронную работу и не является настоящим awaitable.

Ещё хуже — выполнить блокирующую операцию непосредственно внутри async-кода:

```python
import time


async def main():
    time.sleep(5)
```

`time.sleep()` блокирует поток.

Если Event Loop работает в этом же потоке:

```text
Event Loop
    │
    ▼
time.sleep(5)
    │
    └── весь Event Loop стоит
```

Другие async-задачи не смогут нормально выполняться.

Для сравнения:

```python
import asyncio


async def main():
    await asyncio.sleep(5)
```

Здесь `asyncio.sleep()` позволяет Event Loop заниматься другими задачами.

---

# 10. `await` ≠ новый поток

`await` не создаёт поток.

```text
await
  ≠
Thread
```

Обычно `asyncio` может выполнять множество задач в **одном потоке**:

```text
Thread
│
└── Event Loop
    ├── Task A
    ├── Task B
    ├── Task C
    └── Task D
```

Поэтому асинхронность и многопоточность — разные механизмы.

---

# 11. `await` ≠ параллелизм

`await` обычно используется для конкурентного выполнения задач, а не для запуска CPU-задач параллельно на разных ядрах.

```text
await
 ↓
кооперативная конкурентность
 ↓
Event Loop
```

Для CPU-bound работы может понадобиться:

```text
Process
 ↓
другое CPU-ядро
 ↓
parallelism
```

---

# 12. Последовательные `await`

Посмотрим:

```python
async def main():
    a = await fetch_a()
    b = await fetch_b()
```

Здесь `fetch_b()` начнёт выполняться после завершения `fetch_a()`.

Упрощённо:

```text
A █████████
            B █████████
```

Это не означает, что две операции выполняются конкурентно.

Если операции независимы, можно создать задачи:

```python
async def main():
    task_a = asyncio.create_task(fetch_a())
    task_b = asyncio.create_task(fetch_b())

    a = await task_a
    b = await task_b
```

Теперь:

```text
A █████████
B █████████
```

Они могут продвигаться конкурентно.

---

# 13. `await` и `asyncio.gather()`

Для нескольких независимых async-операций часто используют:

```python
async def main():
    a, b = await asyncio.gather(
        fetch_a(),
        fetch_b(),
    )
```

Здесь:

```text
gather()
    │
    ├── fetch_a()
    └── fetch_b()
```

Обе coroutine могут быть запланированы для конкурентного выполнения.

Важно понимать:

```text
await gather(...)
```

означает ожидание **результатов группы задач**, а не последовательное выполнение функций.

---

# 14. Модель, которую нужно держать в голове

```text
async def
    ↓
создаём coroutine
    ↓
Task / await
    ↓
Event Loop
    ↓
coroutine выполняется
    ↓
await I/O
    ↓
coroutine приостанавливается
    ↓
Event Loop занимается другими задачами
    ↓
I/O завершилось
    ↓
coroutine снова готова
    ↓
продолжает выполнение
```

Это гораздо полезнее, чем запоминать определение `await` как просто «ожидание».

---

## 🎤 Вопросы на собеседовании

### Что такое `await`?

`await` — оператор Python для ожидания awaitable-объекта внутри асинхронного контекста. Он приостанавливает текущую coroutine и позволяет Event Loop выполнять другие задачи.

### Блокирует ли `await` Event Loop?

Сам по себе корректный `await` неблокирующей операции — нет. Он приостанавливает текущую coroutine, а Event Loop может выполнять другие задачи.

### Создаёт ли `await` новый поток?

Нет.

### Создаёт ли `await` новый процесс?

Нет.

### Что можно передать в `await`?

Awaitable-объект: например coroutine, `Task` или `Future`.

### Можно ли использовать `await` в обычной функции?

Нет. В обычном Python-коде `await` используется внутри `async def`.

### Чем отличаются:

```python
a = await foo()
```

и

```python
task = asyncio.create_task(foo())
```

`await` непосредственно ожидает выполнение awaitable в текущей coroutine.

`create_task()` планирует coroutine как отдельную `Task`, позволяя ей выполняться конкурентно с текущей coroutine.

### Почему это может быть проблемой?

```python
async def main():
    time.sleep(5)
```

Потому что `time.sleep()` блокирует поток Event Loop.

Правильный async-вариант:

```python
async def main():
    await asyncio.sleep(5)
```

### Что произойдёт при двух последовательных `await`?

```python
a = await foo()
b = await bar()
```

`bar()` начнёт выполняться после того, как `foo()` завершится.

Если операции независимы и нужно выполнять их конкурентно, можно использовать `create_task()` или `asyncio.gather()`.

### Самая важная формула

```text
await
  ↓
приостановить текущую coroutine
  ↓
вернуть управление Event Loop
  ↓
дать возможность работать другим задачам
  ↓
дождаться готовности awaitable
  ↓
продолжить coroutine
```
