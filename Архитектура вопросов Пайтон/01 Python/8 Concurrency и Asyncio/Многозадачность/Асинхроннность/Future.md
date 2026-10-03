# 🔮 Future в asyncio

## 🎤 Короткий ответ

**`Future`** — это объект, который представляет **результат асинхронной операции, доступный в будущем**.

В момент создания результата ещё нет:

```text
Future
  │
  ├── pending → результата нет
  │
  └── done → результат готов
```

`Future` является **[[Awaitable]]**, поэтому его можно ожидать через [[await]]:

```python
result = await future
```

В отличие от [[Task]], `Future` **не предназначен для самостоятельного выполнения [[Корутина - (coroutine)]]**. Он является контейнером/обещанием будущего результата, который кто-то должен установить.

---

## 🗣️ Ответ на собеседовании

`Future` в `asyncio` — это низкоуровневый объект, представляющий результат асинхронной операции, который станет доступен позже.

Когда создаётся `Future`, он обычно находится в состоянии `pending`. Какая-то другая часть системы выполняет операцию и затем устанавливает результат через `set_result()` или исключение через `set_exception()`.

Поскольку `Future` является awaitable, coroutine может написать:

```python
result = await future
```

и приостановиться до тех пор, пока `Future` не будет завершён.

Главное отличие от `Task` в том, что `Task` предназначена для выполнения coroutine и управляется Event Loop, а `Future` в основном представляет **готовящийся результат**. В обычном прикладном коде `Future` встречается реже, потому что чаще мы работаем с coroutine, Task и высокоуровневыми API `asyncio`.

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
        ├── Awaitable
        │   │
        │   ├── Coroutine
        │   ├── Task
        │   └── Future ← Я здесь
        │
        ├── await
        ├── Selector
        └── Non-blocking I/O
```

---

# 📚 Разбор поглубже

## 1. Что такое Future

Само название хорошо отражает идею:

> **Future — результат, который будет доступен в будущем.**

Например, мы запускаем некоторую асинхронную операцию:

```text
запрос
  │
  ▼
Future
  │
  │ результата пока нет
  ▼
   WAIT
  │
  │ операция завершилась
  ▼
результат
```

У `Future` есть состояние, которое изменяется со временем.

---

# 2. Состояния Future

Упрощённо:

```text
Future
  │
  ▼
PENDING
  │
  ├──────────────┐
  │              │
  ▼              ▼
RESULT         EXCEPTION
  │              │
  └──────┬───────┘
         ▼
        DONE
```

В начале:

```python
future.done()
```

обычно возвращает:

```python
False
```

После установки результата:

```python
future.set_result("готово")
```

Future становится завершённым:

```python
future.done()
# True
```

---

# 3. Создание Future

Обычно Future создаётся через текущий Event Loop:

```python
import asyncio


async def main():
    loop = asyncio.get_running_loop()

    future = loop.create_future()

    print(future.done())

    future.set_result("готово")

    print(future.done())

    result = await future

    print(result)


asyncio.run(main())
```

Получим примерно:

```text
False
True
готово
```

Здесь мы вручную показали жизненный цикл Future.

---

# 4. `set_result()`

Результат Future устанавливается через:

```python
future.set_result(value)
```

Например:

```python
future.set_result(42)
```

После этого:

```python
result = await future
```

вернёт:

```text
42
```

Схема:

```text
Future
  │
  │ set_result(42)
  ▼
DONE
  │
  │ await
  ▼
42
```

---

# 5. `set_exception()`

Future может завершиться не результатом, а исключением:

```python
future.set_exception(ValueError("Ошибка"))
```

Если coroutine ожидает такой Future:

```python
result = await future
```

то исключение будет выброшено в месте `await`.

Например:

```python
import asyncio


async def main():
    loop = asyncio.get_running_loop()

    future = loop.create_future()
    future.set_exception(ValueError("Ошибка"))

    try:
        await future
    except ValueError as error:
        print(error)


asyncio.run(main())
```

---

# 6. Почему `await future` может ждать

Рассмотрим:

```python
import asyncio


async def main():
    loop = asyncio.get_running_loop()

    future = loop.create_future()

    asyncio.get_running_loop().call_later(
        2,
        future.set_result,
        "готово",
    )

    result = await future

    print(result)


asyncio.run(main())
```

Упрощённо:

```text
main()
 │
 ▼
await future
 │
 ▼
main приостанавливается
 │
 ▼
Event Loop продолжает работу
 │
 │ 2 секунды
 ▼
future.set_result("готово")
 │
 ▼
Future → DONE
 │
 ▼
main продолжается
 │
 ▼
result = "готово"
```

Это хорошо показывает главную идею Future.

---

# 7. Future является Awaitable

Мы уже знаем:

```text
Awaitable
│
├── Coroutine
├── Task
└── Future
```

Поэтому:

```python
result = await future
```

корректно.

Когда coroutine встречает:

```python
await future
```

она может приостановиться до тех пор, пока Future не завершится.

---

# 8. Кто устанавливает результат Future?

Это принципиальный момент.

Future сама не выполняет работу.

Например:

```text
Future
  │
  └── результата нет
```

Кто-то другой должен сделать:

```python
future.set_result(...)
```

или:

```python
future.set_exception(...)
```

Поэтому:

```text
Future
   ↓
представляет результат

Task
   ↓
выполняет coroutine
```

Это ключевое различие.

---

# 9. Future vs Task

Это один из наиболее вероятных вопросов на собеседовании.

|                          | `Task`               | `Future`                          |
| ------------------------ | -------------------- | --------------------------------- |
| Основная роль            | Выполнение coroutine | Представление будущего результата |
| Запускает coroutine      | Да                   | Нет                               |
| Awaitable                | Да                   | Да                                |
| Управляется Event Loop   | Да                   | Да                                |
| Можно получить результат | Да                   | Да                                |
| Может быть отменён       | Да                   | Да                                |

Упрощённо:

```text
Task
 ↓
«Выполни эту coroutine»

Future
 ↓
«Результат этой операции будет позже»
```

---

# 10. Связь Task и Future

Здесь есть важный нюанс: в `asyncio` `Task` является специализированной разновидностью Future.

Упрощённо можно представить:

```text
Future
   │
   └── Task
        │
        └── выполняет coroutine
```

То есть Task предоставляет интерфейс Future для результата, но дополнительно управляет выполнением coroutine.

Поэтому Task можно:

```python
await task
```

и получить результат coroutine.

---

# 11. Future — низкоуровневый механизм

В прикладном коде обычно не приходится вручную создавать Future.

Чаще пишут:

```python
async def fetch_data():
    ...
```

и работают через:

```python
await fetch_data()
```

или:

```python
task = asyncio.create_task(fetch_data())
```

Future чаще встречается внутри:

* `asyncio`;
* библиотек;
* event loop;
* адаптеров между callback-based и async-кодом;
* низкоуровневых механизмов I/O.

Поэтому на backend-собеседовании важно **понимать модель Future**, но необязательно постоянно использовать его вручную.

---

# 12. Future и callback

У Future можно зарегистрировать callback, который будет вызван после завершения.

Например:

```python
def on_done(future):
    print("Future завершён")


future.add_done_callback(on_done)
```

Схема:

```text
Future
  │
  │ pending
  │
  ▼
set_result()
  │
  ▼
DONE
  │
  ▼
callback
```

Это один из механизмов, связывающих Future с Event Loop.

---

# 13. Получение результата через `result()`

После завершения можно получить результат:

```python
result = future.result()
```

Но важно:

```python
future.result()
```

не является аналогом:

```python
await future
```

Если Future ещё не завершена, `result()` не будет ждать её завершения.

Поэтому:

```text
await future
    ↓
может приостановить coroutine и дождаться

future.result()
    ↓
получить уже готовый результат
```

Если Future ещё не завершена, вызов `result()` приведёт к ошибке состояния.

---

# 14. Future и отмена

Future можно отменить:

```python
future.cancel()
```

Проверить:

```python
future.cancelled()
```

Если Future была отменена:

```python
future.cancelled()
# True
```

При ожидании отменённой Future обычно возникает:

```python
asyncio.CancelledError
```

Это связывает Future с общей моделью отмены асинхронных операций.

---

# 15. Future в общей картине asyncio

Теперь можно собрать все предыдущие темы:

```text
                    asyncio
                       │
                       ▼
                  Event Loop
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
      Coroutine       Task        Future
          │            │            │
          │            │            │
          │        выполняет        │
          │        coroutine        │
          │                         │
          └────────────┬────────────┘
                       │
                       ▼
                    await
                       │
                       ▼
                 результат
```

Более точно:

```text
Coroutine
    │
    │ create_task()
    ▼
Task ──────────────► выполняет coroutine
 │
 │ await
 ▼
результат


Future
 │
 │ set_result()
 ▼
результат
```

---

# 16. Самая важная аналогия

Можно представить ресторан:

```text
Coroutine
→ заказ, который нужно приготовить

Task
→ заказ уже принят кухней и поставлен в работу

Future
→ номерок/обещание:
  «когда заказ будет готов,
   ты получишь результат»

await
→ «подожди, пока заказ будет готов»
```

При этом ожидание не означает остановку всего ресторана:

```text
ты ждёшь заказ
      ↓
кухня продолжает работать
      ↓
другие заказы готовятся
      ↓
твой заказ готов
      ↓
ты получаешь результат
```

Для понимания `asyncio` это достаточно хорошая ментальная модель.

---

# 🎤 Вопросы на собеседовании

### Что такое Future?

Future — awaitable-объект, представляющий результат асинхронной операции, который будет доступен позже.

### Future выполняет операцию?

Нет. Future представляет будущий результат. Кто-то другой должен установить этот результат.

### Как установить результат Future?

```python
future.set_result(value)
```

### Как установить исключение?

```python
future.set_exception(error)
```

### Можно ли сделать `await future`?

Да. Future является awaitable.

### Что произойдёт при `await` незавершённого Future?

Текущая coroutine приостановится, а Event Loop сможет выполнять другие задачи. После завершения Future coroutine продолжит работу.

### Чем Future отличается от Task?

Task предназначена для выполнения coroutine и является специализированным awaitable-объектом с интерфейсом Future.

Future в первую очередь представляет будущий результат операции.

### Кто устанавливает результат Future?

Другая часть асинхронной системы: callback, Event Loop, библиотека или другой код.

### Что делает `future.result()`?

Возвращает уже установленный результат Future. Сам по себе этот метод не является механизмом асинхронного ожидания.

### Что такое `set_result()`?

Метод, который переводит Future в завершённое состояние и устанавливает её результат.

### Что происходит при `set_exception()`?

Future завершается с исключением, и при `await` этого Future исключение будет получено ожидающей coroutine.

### Можно ли отменить Future?

Да:

```python
future.cancel()
```

---

## 🧠 Формула

```text
Future
   │
   ├── PENDING
   │     │
   │     └── результата ещё нет
   │
   └── DONE
         │
         ├── result
         └── exception
```

И главное:

```text
Coroutine
    ↓
что выполнить

Task
    ↓
запланированное выполнение coroutine

Future
    ↓
будущий результат

await
    ↓
дождаться Awaitable
```

> **Task выполняет coroutine, Future представляет будущий результат, а `await` позволяет дождаться этого результата.**
