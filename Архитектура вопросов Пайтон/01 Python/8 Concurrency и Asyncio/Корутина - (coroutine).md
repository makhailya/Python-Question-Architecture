# 🔄 Корутина

## 🎯 Ответ на собеседовании

**Корутина (coroutine)** — специальная функция или объект, выполнение которого можно **приостанавливать и возобновлять**.

В Python корутина обычно создаётся с помощью `async def`:

```python
async def get_data():
    await asyncio.sleep(1)
    return "data"
```

Вызов такой функции **не выполняет её сразу**:

```python
coroutine = get_data()
```

Он создаёт **coroutine object** — объект корутины.

Чтобы корутина реально выполнялась, её нужно запустить через `await`, `Task` или другой механизм `asyncio`.

---

## 📌 `async def` → coroutine function

```python
async def get_data():
    return "data"
```

Это **coroutine function** — функция-корутина.

При вызове:

```python
result = get_data()
```

получаем:

```text
coroutine function
       ↓ вызов
coroutine object
```

А не результат `"data"`.

---

## ▶️ Запуск через `await`

Внутри другой корутины:

```python
async def main():
    result = await get_data()
    print(result)
```

`await` запускает/ожидает корутину и получает её результат.

```text
main()
  ↓
await get_data()
  ↓
get_data выполняется
  ↓
return "data"
  ↓
result = "data"
```

---

## 🔀 Корутина может уступать управление

Главная особенность:

```python
async def task():
    print("start")

    await asyncio.sleep(2)

    print("end")
```

В момент:

```python
await asyncio.sleep(2)
```

корутина приостанавливается.

Event Loop получает возможность выполнять другие задачи.

```text
Task 1 → await → ⏸️
                  ↓
             Event Loop
                  ↓
Task 2 → выполняется
                  ↓
Task 1 → продолжается
```

---

## 🧩 Coroutine vs Task

Это **не одно и то же**.

### Coroutine

Объект, представляющий асинхронное выполнение:

```python
coro = get_data()
```

### Task

Обёртка над корутиной, которую `asyncio` планирует для выполнения:

```python
task = asyncio.create_task(get_data())
```

Упрощённо:

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
```

Task позволяет корутине выполняться **конкурентно с другими задачами**.

---

## 📌 Пример конкурентного выполнения

```python
async def task1():
    await asyncio.sleep(2)
    print("task 1")


async def task2():
    await asyncio.sleep(1)
    print("task 2")


async def main():
    t1 = asyncio.create_task(task1())
    t2 = asyncio.create_task(task2())

    await t1
    await t2
```

Event Loop может организовать выполнение так:

```text
0 сек → task1 → await
        task2 → await

1 сек → task2 → завершилась

2 сек → task1 → завершилась
```

---

## ⚠️ Корутина ≠ поток

Корутина:

* не является отдельным потоком;
* обычно выполняется внутри Event Loop;
* переключается кооперативно через `await`;
* имеет очень маленькие накладные расходы.

Поток:

* является единицей выполнения ОС;
* управляется ОС;
* имеет собственный стек;
* может использовать блокирующие операции, не блокируя другие потоки.

---

## 🆚 Coroutine / Task / Thread

|                     | Coroutine  | Task                      | Thread                       |
| ------------------- | ---------- | ------------------------- | ---------------------------- |
| Уровень             | Python     | `asyncio`                 | ОС                           |
| Выполняется         | Event Loop | Event Loop                | ОС                           |
| Параллельность      | ❌          | ❌ сама по себе            | ⚠️ зависит от задачи/CPython |
| Переключение        | `await`    | Event Loop                | Планировщик ОС               |
| Основное применение | Async I/O  | Управление async-задачами | I/O / параллельная работа    |

---

## 🧠 Важный нюанс

Вот это:

```python
async def get_data():
    ...
```

**не является самой корутиной**.

Это **функция-корутина**.

А вот:

```python
get_data()
```

создаёт **объект корутины**.

Это различие часто проверяют на собеседованиях.

---

## 🎤 Суперкоротко

> **Корутина — это объект/единица асинхронного выполнения, которую можно приостанавливать и возобновлять. В Python корутины создаются coroutine-функциями `async def`, а `await` позволяет приостановить текущую корутину и передать управление Event Loop. Для планирования корутины как отдельной задачи используется `asyncio.create_task()`.**
