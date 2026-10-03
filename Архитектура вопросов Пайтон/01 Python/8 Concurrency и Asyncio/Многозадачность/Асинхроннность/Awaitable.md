# ⏳ Awaitable в Python

## 🎤 Короткий ответ

**Awaitable** — это объект, который можно передать в [[await]].

То есть:

```python
result = await obj
```

будет работать только в том случае, если `obj` является **awaitable**.

Основные виды awaitable в `asyncio`:

* **[[Корутина - (coroutine)]]** — объект, полученный при вызове `async def`;
* **[[Task]]** — запланированная [[Event Loop — цикл событий]] [[Корутина - (coroutine)]];
* **[[Future]]** — объект, представляющий результат асинхронной операции, который будет доступен позже.

Главная формула:

```text
awaitable
    ↓
можно использовать с await
```

---

## 🗣️ Ответ на собеседовании

Awaitable — это объект, который поддерживает механизм ожидания и может использоваться с оператором [[await]].

В Python основными awaitable являются [[Корутина - (coroutine)]], `asyncio.Task` и `asyncio.Future`.

Coroutine получается при вызове функции, объявленной через `async def`. Task — это coroutine, которую [[Event Loop — цикл событий]] запланировал для выполнения. [[Future]] представляет результат асинхронной операции, который появится в будущем.

Когда мы пишем [[await]], текущая coroutine приостанавливается до тех пор, пока ожидаемый объект не станет готов, а [[Event Loop — цикл событий]] получает возможность выполнять другие задачи.

Поэтому важно различать саму [[Корутина - (coroutine)]], [[Task]] и [[Future]]: все они [[Awaitable]], но выполняют разные роли в модели `asyncio`.

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
        ├── Awaitable ← Я здесь
        │   │
        │   ├── Coroutine
        │   ├── Task
        │   └── Future
        │
        ├── await
        ├── Task
        ├── Future
        ├── Selector
        └── Non-blocking I/O
```

---

# 📚 Разбор поглубже

## 1. Что такое Awaitable

Слово **awaitable** буквально означает:

> «то, что можно ожидать через `await`».

Например:

```python
async def fetch_data():
    return "data"


async def main():
    result = await fetch_data()
```

Здесь:

```python
fetch_data()
```

возвращает coroutine object.

А coroutine object является awaitable.

Поэтому:

```python
await fetch_data()
```

корректно.

---

# 2. Awaitable — это не конкретный тип

Очень важно.

`Awaitable` — это скорее **категория объектов**, которые можно использовать с `await`.

```text
Awaitable
│
├── Coroutine
├── Task
└── Future
```

Поэтому нельзя воспринимать `Awaitable` просто как ещё один конкретный класс вроде `list` или `dict`.

В Python существует абстракция:

```python
from collections.abc import Awaitable
```

Она описывает объекты, которые поддерживают протокол ожидания.

---

# 3. Coroutine как Awaitable

Создадим async-функцию:

```python
async def fetch_data():
    return "data"
```

Важно различать:

```python
fetch_data
```

и:

```python
fetch_data()
```

Первое — функция.

Второе — coroutine object.

```python
coroutine = fetch_data()
```

Теперь:

```text
fetch_data
    ↓
async function
    ↓ вызов
coroutine object
    ↓
Awaitable
```

Поэтому:

```python
result = await coroutine
```

работает.

---

# 4. Task как Awaitable

`Task` — это объект, который представляет запланированное выполнение coroutine.

Например:

```python
import asyncio


async def fetch_data():
    await asyncio.sleep(1)
    return "data"


async def main():
    task = asyncio.create_task(fetch_data())

    result = await task
```

Здесь:

```python
asyncio.create_task(fetch_data())
```

создаёт `Task`.

Упрощённо:

```text
Coroutine
    ↓
create_task()
    ↓
Task
    ↓
Event Loop
```

Task начинает планироваться Event Loop независимо от текущей coroutine.

А:

```python
await task
```

означает:

> «Дождаться результата этой Task».

---

# 5. Future как Awaitable

`Future` — низкоуровневый объект, который представляет **будущий результат асинхронной операции**.

Упрощённо:

```text
Future
│
├── сейчас результата нет
│
├── операция выполняется
│
├── результат готов
│
└── Future получает результат
```

Например:

```python
import asyncio


async def main():
    loop = asyncio.get_running_loop()

    future = loop.create_future()

    future.set_result("готово")

    result = await future

    print(result)
```

Здесь:

```python
future
```

является awaitable.

---

# 6. Зачем нужны разные виды Awaitable

На первый взгляд возникает вопрос:

> Если всё можно `await`, зачем вообще нужны Coroutine, Task и Future?

Потому что они представляют **разные уровни абстракции**.

### Coroutine

Описывает асинхронное выполнение:

```text
«Вот код, который должен выполняться асинхронно»
```

### Task

Представляет coroutine, которую Event Loop уже запланировал:

```text
«Эту coroutine нужно выполнять как отдельную задачу»
```

### Future

Представляет будущий результат:

```text
«Результат этой операции появится позже»
```

Упрощённо:

```text
Coroutine
    │
    │ create_task()
    ▼
Task
    │
    │ await
    ▼
результат
```

---

# 7. Что происходит при `await`

Рассмотрим:

```python
result = await task
```

Упрощённая схема:

```text
Current Coroutine
       │
       ▼
    await task
       │
       ▼
текущая coroutine
приостанавливается
       │
       ▼
Event Loop
       │
       ├── Task A
       ├── Task B
       └── Task C
```

Когда `task` завершится:

```text
Task завершена
      │
      ▼
Event Loop
      │
      ▼
Current Coroutine
      │
      ▼
result = ...
```

---

# 8. Awaitable и `__await__`

На более низком уровне возможность использовать объект с `await` связана с методом:

```python
__await__()
```

Можно создать собственный awaitable-объект:

```python
class MyAwaitable:
    def __await__(self):
        async def inner():
            return "готово"

        return inner().__await__()
```

Теперь объект можно ожидать:

```python
async def main():
    result = await MyAwaitable()
    print(result)
```

Это уже более низкоуровневая часть протокола `asyncio`.

Для обычного backend-кода самостоятельно реализовывать `__await__()` требуется редко, но понимать его существование полезно для собеседования.

---

# 9. Awaitable и `async def`

Важно:

```python
async def fetch():
    return 10
```

не означает, что сама функция является awaitable.

Вот это:

```python
fetch
```

— функция.

А это:

```python
fetch()
```

— coroutine object, который можно `await`.

То есть:

```text
async def
    ↓
coroutine function
    ↓ вызов
coroutine object
    ↓
awaitable
```

---

# 10. Awaitable и обычное значение

Обычный объект нельзя просто так передать в `await`:

```python
async def main():
    result = await 10
```

Это ошибка, потому что `int` не является awaitable.

То же самое:

```python
await "hello"
```

не работает.

Сравнение:

```text
await coroutine()  → ✅
await task         → ✅
await future       → ✅

await 10           → ❌
await "hello"      → ❌
await [1, 2, 3]    → ❌
```

---

# 11. Awaitable и асинхронность

Awaitable является частью общей модели:

```text
                    asyncio
                       │
                       ▼
                  Event Loop
                       │
                ┌──────┴──────┐
                ▼             ▼
              Task         Future
                │             │
                └──────┬──────┘
                       ▼
                    await
                       │
                       ▼
                  результат
```

Но `await` сам по себе не означает:

```text
«запустить параллельно»
```

`await` означает:

```text
«дождаться awaitable»
```

А конкурентное выполнение появляется благодаря тому, как Event Loop планирует несколько задач.

---

# 12. Coroutine vs Task vs Future

| Объект    | Что представляет               | Awaitable |
| --------- | ------------------------------ | --------- |
| Coroutine | Асинхронное выполнение функции | ✅         |
| Task      | Запланированная coroutine      | ✅         |
| Future    | Будущий результат операции     | ✅         |

Главное различие:

```text
Coroutine → что выполнить
Task      → запланированное выполнение
Future    → будущий результат
```

---

# 13. Связь `await` → Awaitable

Теперь вся предыдущая тема складывается:

```text
await
 │
 └── ожидает Awaitable
          │
          ├── Coroutine
          ├── Task
          └── Future
```

Например:

```python
async def fetch():
    await asyncio.sleep(1)
    return "data"
```

Здесь:

```text
fetch()
   ↓
Coroutine
   ↓
await
   ↓
результат
```

А если:

```python
task = asyncio.create_task(fetch())
```

получаем:

```text
fetch()
   ↓
Coroutine
   ↓
create_task()
   ↓
Task
   ↓
await task
   ↓
результат
```

---

## 🎤 Вопросы на собеседовании

### Что такое Awaitable?

Awaitable — объект, который можно использовать с оператором `await`.

### Какие основные Awaitable есть в asyncio?

Coroutine, `Task` и `Future`.

### Является ли coroutine awaitable?

Да.

### Является ли Task awaitable?

Да.

### Является ли Future awaitable?

Да.

### Является ли обычный `int` awaitable?

Нет.

```python
await 10  # TypeError
```

### Чем coroutine отличается от Task?

Coroutine — объект асинхронного выполнения, полученный при вызове `async def`.

Task — coroutine, которая была запланирована Event Loop как отдельная задача.

### Чем Task отличается от Future?

Task предназначена для управления выполнением coroutine.

Future представляет будущий результат асинхронной операции.

### Что делает `await`?

Ожидает awaitable. Текущая coroutine приостанавливается, а Event Loop может выполнять другие задачи.

### Что такое `__await__()`?

Это специальный метод, связанный с протоколом await. Он позволяет объекту участвовать в механизме `await`.

---

## 🧠 Формула

```text
Awaitable
    ↓
объект, который можно await
    │
    ├── Coroutine → асинхронное выполнение
    ├── Task      → запланированная coroutine
    └── Future    → будущий результат
```

А главное различие:

```text
await
 ↓
«дождаться Awaitable»

Task
 ↓
«запланировать Coroutine»

Future
 ↓
«представить будущий результат»
```
