# 🧵 Threading в Python

## 🎤 Короткий ответ

`threading` — это механизм многопоточности в Python, который позволяет выполнять несколько потоков внутри одного процесса.

Потоки используют **общее адресное пространство процесса**, поэтому могут напрямую работать с общими объектами. При этом у каждого потока есть собственный стек и состояние выполнения.

В CPython потоки особенно полезны для **I/O-bound задач**: сетевых запросов, работы с файлами, ожидания БД и т.д. Для CPU-bound задач на чистом Python обычный `threading` ограничивается GIL, поэтому чаще используют `multiprocessing`.

Главная проблема потоков — **совместный доступ к данным**, из-за которого возникают race condition и необходимость синхронизации через `Lock`, `RLock`, `Semaphore`, `Condition`, `Queue` и другие механизмы.

---

## 🗣️ Ответ на собеседовании

> `threading` — это реализация многопоточности в Python, когда внутри одного процесса работают несколько потоков.
>
> Все потоки одного процесса используют общее адресное пространство, поэтому они могут работать с одними и теми же объектами. Это удобно, но одновременно создаёт проблемы синхронизации.
>
> В CPython есть [[GIL — Global Interpreter Lock]], поэтому несколько потоков не выполняют Python bytecode параллельно в обычной конфигурации с включённым GIL. Поэтому threading особенно полезен для I/O-bound задач: пока один поток ждёт сеть, диск или другой внешний ресурс, другой поток может выполнять работу.
>
> Для CPU-bound задач на чистом Python обычно используют [[Многопроцессность (Multiprocessing)]], где процессы имеют отдельные интерпретаторы и память.
>
> При работе нескольких потоков с общими данными нужно учитывать [[Race Condition|Race Condition]] и использовать механизмы синхронизации, например `Lock` или `Queue`.

---

## 🧭 Где я нахожусь

```text
01 Python
└── 08 Concurrency и Asyncio
    │
    ├── Многозадачность
    │   ├── Конкурентность
    │   ├── Параллелизм
    │   └── Виды многозадачности
    │
    ├── Многопоточность
    │   ├── Thread
    │   ├── Shared Memory
    │   ├── GIL
    │   ├── Race Condition
    │   ├── Data Race
    │   └── Синхронизация
    │       ├── Lock
    │       ├── RLock
    │       ├── Semaphore
    │       ├── Condition
    │       └── Queue
    │
    ├── Многопроцессность
    │
    └── Asyncio
        ├── Event Loop
        ├── Coroutine
        ├── Awaitable
        ├── Task
        └── Future

                    ↑
                 Я здесь
```

---

# 📚 Разбор поглубже

## 1. Что такое Thread

**Thread (поток)** — это отдельная последовательность выполнения инструкций внутри процесса.

У процесса могут быть:

```text
Process
│
├── Thread 1
├── Thread 2
├── Thread 3
└── Thread 4
```

Потоки одного процесса:

* используют общую память процесса;
* имеют собственный стек;
* имеют собственный instruction pointer / состояние выполнения;
* могут выполняться конкурентно;
* могут обращаться к одним и тем же объектам Python.

---

## 2. Процесс и поток

Главное различие:

|                 | Process                               | Thread                          |
| --------------- | ------------------------------------- | ------------------------------- |
| Память          | отдельная                             | общая внутри процесса           |
| Создание        | дороже                                | дешевле                         |
| Обмен данными   | IPC                                   | общие объекты                   |
| Изоляция        | высокая                               | низкая                          |
| Ошибка одного   | обычно не затрагивает другие процессы | может затронуть общий процесс   |
| CPU parallelism | да                                    | ограничен GIL в обычном CPython |
| I/O-bound       | подходит                              | очень подходит                  |

Упрощённо:

```text
Process A                  Process B
┌─────────────┐            ┌─────────────┐
│ Thread 1    │            │ Thread 1    │
│ Thread 2    │            │ Thread 2    │
│ Thread 3    │            │ Thread 3    │
└─────────────┘            └─────────────┘
     Memory                      Memory
```

Память процессов изолирована.

А внутри одного процесса:

```text
             Process
                │
       ┌────────┴────────┐
       │ Shared Memory   │
       │                 │
       ├── Thread 1      │
       ├── Thread 2      │
       └── Thread 3      │
```

---

# 3. Создание потока

В Python используется модуль `threading`.

```python
import threading


def worker():
    print("Работаю в отдельном потоке")


thread = threading.Thread(target=worker)

thread.start()
thread.join()
```

Здесь:

```python
thread = threading.Thread(...)
```

создаёт объект потока.

```python
thread.start()
```

запускает поток.

```python
thread.join()
```

заставляет текущий поток дождаться его завершения.

---

# 4. `start()` и `run()` — важная разница

Это очень частый вопрос на собеседовании.

### `start()`

```python
thread.start()
```

Запускает выполнение потока.

### `run()`

```python
thread.run()
```

Просто выполняет метод `run()` в **текущем потоке**.

То есть:

```python
thread.start()
```

→ создаётся отдельное выполнение потока.

А:

```python
thread.run()
```

→ никакого нового потока фактически не запускается.

Можно представить:

```text
start()
  │
  └──> новый Thread
          │
          └──> run()


run()
  │
  └──> текущий Thread
```

---

# 5. `join()`

`join()` нужен, чтобы дождаться завершения другого потока.

```python
import threading
import time


def worker():
    time.sleep(2)
    print("Готово")


thread = threading.Thread(target=worker)

thread.start()
thread.join()

print("Продолжаем работу")
```

Без `join()` основной поток может продолжить выполнение, пока worker ещё работает.

С `join()`:

```text
Main Thread
    │
    ├── start worker
    │
    ├── ждёт join()
    │
    │      Worker Thread
    │           │
    │           └── работа
    │
    └── продолжает работу
```

---

# 6. Несколько потоков

```python
import threading
import time


def worker(number):
    print(f"Поток {number} начал работу")
    time.sleep(1)
    print(f"Поток {number} закончил работу")


threads = []

for i in range(3):
    thread = threading.Thread(target=worker, args=(i,))
    thread.start()
    threads.append(thread)

for thread in threads:
    thread.join()
```

Получаем:

```text
Main
 │
 ├── Thread 1 ─── работа ───┐
 ├── Thread 2 ─── работа ───┤
 └── Thread 3 ─── работа ───┘
                             │
                           join
```

Потоки могут **конкурентно продвигаться**.

Это не означает автоматически, что Python bytecode выполняется одновременно на нескольких ядрах.

---

# 7. Threading и GIL

В обычном CPython с GIL существует глобальная блокировка интерпретатора.

Упрощённо:

```text
Thread 1 ──┐
Thread 2 ──┼──> GIL ──> Python bytecode
Thread 3 ──┘
```

В конкретный момент времени только один поток выполняет Python bytecode под GIL.

Но это **не означает, что threading бесполезен**.

Главное применение:

```text
Thread 1
   │
   └── HTTP request
          │
          └── ждём...

Thread 2
   │
   └── работает
```

Когда один поток находится в ожидании I/O, другой может использовать процессор.

---

# 8. I/O-bound и CPU-bound

## I/O-bound

Основное время программа ждёт внешний ресурс:

* HTTP;
* база данных;
* файл;
* socket;
* API;
* DNS;
* устройство.

Например:

```python
response = requests.get(url)
```

Большая часть времени может уходить на ожидание сети.

Здесь threading может быть полезен.

---

## CPU-bound

Основное время тратится на вычисления:

```python
for i in range(100_000_000):
    result += i ** 2
```

В обычном CPython с GIL threading не даёт ожидаемого CPU parallelism для чистого Python-кода.

Для CPU-bound задач часто рассматривают:

```text
CPU-bound
    │
    └── multiprocessing
```

---

# 9. Shared Memory

Главное преимущество потоков — они могут обращаться к общей памяти процесса.

Например:

```python
counter = 0
```

Несколько потоков могут обращаться к `counter`.

Но это одновременно источник проблем.

```text
Process
│
├── Thread 1 ──┐
│              │
├── Thread 2 ──┼──> shared data
│              │
└── Thread 3 ──┘
```

Если несколько потоков одновременно изменяют данные, порядок операций может иметь значение.

---

# 10. Race Condition

**Race condition** возникает, когда результат зависит от порядка выполнения конкурентных операций.

Например:

```python
counter += 1
```

Концептуально это не одна магическая операция:

```text
read counter
    ↓
calculate counter + 1
    ↓
write counter
```

Если два потока делают это одновременно, возможна потеря обновления.

```text
counter = 0

Thread 1: read 0
Thread 2: read 0

Thread 1: write 1
Thread 2: write 1

Итог: 1
```

Хотя логически ожидалось:

```text
2
```

Поэтому общий изменяемый state требует синхронизации.

---

# 11. Lock

`Lock` позволяет ограничить критическую секцию.

```python
import threading

counter = 0
lock = threading.Lock()


def increment():
    global counter

    with lock:
        counter += 1
```

Теперь:

```text
Thread 1 ──> LOCK ──> изменение ──> UNLOCK
Thread 2 ──> ждёт
Thread 3 ──> ждёт
```

После освобождения lock другой поток может войти в критическую секцию.

---

# 12. Что такое критическая секция

**Критическая секция** — участок кода, который работает с общим ресурсом и должен выполняться согласованно.

Например:

```python
with lock:
    balance -= amount
```

Здесь изменение общего `balance` может быть критической секцией.

Важно:

> Lock защищает не сам объект, а участок взаимодействия с общим состоянием.

---

# 13. Другие механизмы синхронизации

В `threading` есть несколько примитивов:

```text
Synchronization
│
├── Lock
├── RLock
├── Semaphore
├── Condition
├── Event
└── Barrier
```

Отдельно важен:

```text
Queue
```

`Queue` часто используется как безопасный канал обмена данными между потоками.

---

# 14. Queue

Вместо того чтобы несколько потоков напрямую изменяли общий список:

```text
Thread 1 ──┐
Thread 2 ──┼──> shared list
Thread 3 ──┘
```

можно использовать очередь:

```text
Producer
    │
    ▼
┌─────────┐
│  Queue  │
└─────────┘
    │
    ▼
Consumer
```

Пример:

```python
from queue import Queue
import threading


queue = Queue()


def producer():
    for i in range(5):
        queue.put(i)


def consumer():
    while True:
        item = queue.get()

        if item is None:
            break

        print(item)

        queue.task_done()


producer_thread = threading.Thread(target=producer)
consumer_thread = threading.Thread(target=consumer)

producer_thread.start()
consumer_thread.start()

producer_thread.join()

queue.put(None)

consumer_thread.join()
```

`Queue` предоставляет потокобезопасные операции для обмена данными.

---

# 15. ThreadPoolExecutor

В реальных приложениях часто не создают вручную огромное количество `Thread`.

Для этого есть:

```python
from concurrent.futures import ThreadPoolExecutor
```

Пример:

```python
from concurrent.futures import ThreadPoolExecutor


def worker(number):
    return number * 2


with ThreadPoolExecutor(max_workers=4) as executor:
    results = executor.map(worker, range(10))

    for result in results:
        print(result)
```

Модель:

```text
ThreadPoolExecutor
│
├── Worker Thread 1
├── Worker Thread 2
├── Worker Thread 3
└── Worker Thread 4
        ▲
        │
      tasks
```

Плюс в том, что пул управляет жизненным циклом потоков за нас.

---

# 16. Thread vs Asyncio

Это важно не смешивать.

### Threading

```text
OS / Python Threads
        │
        ├── Thread 1
        ├── Thread 2
        └── Thread 3
```

### Asyncio

```text
Thread
  │
  └── Event Loop
       │
       ├── Task 1
       ├── Task 2
       └── Task 3
```

В threading переключением выполнения занимается среда выполнения/ОС и сам интерпретатор.

В asyncio задачи обычно переключаются **кооперативно** через `await`.

---

# 17. Thread vs Multiprocessing

Главное различие:

```text
Threading

Process
├── Thread
├── Thread
└── Thread

     ↓

Shared memory
```

против:

```text
Multiprocessing

Process 1 ─── Memory 1
Process 2 ─── Memory 2
Process 3 ─── Memory 3
```

### Threading

Хорошо подходит, когда:

* нужно много I/O;
* требуется общий memory space;
* задачи относительно лёгкие;
* удобно использовать thread pool.

### Multiprocessing

Часто используют, когда:

* задача CPU-bound;
* нужно использовать несколько CPU cores;
* нужна изоляция памяти процессов.

---

# 18. Типичные ошибки

### ❌ Threading = parallelism

Нет.

Threading обеспечивает многопоточность и позволяет выполнять задачи конкурентно.

---

### ❌ GIL делает threading бесполезным

Нет.

Для I/O-bound задач threading вполне применим.

---

### ❌ GIL защищает данные приложения

Нет.

GIL не является заменой `Lock`.

```python
lock = threading.Lock()
```

используют для синхронизации общего состояния.

---

### ❌ `run()` запускает поток

Нет.

```python
thread.start()
```

запускает поток.

```python
thread.run()
```

вызывает выполнение в текущем потоке.

---

### ❌ `join()` останавливает поток

Нет.

`join()` заставляет **вызывающий поток ждать завершения другого потока**.

---

### ❌ Чем больше потоков, тем быстрее

Не обязательно.

Слишком большое количество потоков приводит к:

* context switching;
* расходу памяти;
* конкуренции за ресурсы;
* дополнительным накладным расходам;
* contention на locks.

---

# 19. Полная картина

```text
Многозадачность
│
├── Конкурентность
│
├── Параллелизм
│
├── Threading
│   │
│   ├── Thread
│   ├── Shared Memory
│   ├── GIL
│   │
│   ├── Race Condition
│   ├── Data Race
│   │
│   └── Synchronization
│       ├── Lock
│       ├── RLock
│       ├── Semaphore
│       ├── Condition
│       └── Queue
│
├── Multiprocessing
│   ├── Process
│   ├── IPC
│   ├── Queue
│   ├── Pipe
│   └── SharedMemory
│
└── Asyncio
    ├── Event Loop
    ├── Coroutine
    ├── Awaitable
    ├── Task
    ├── Future
    └── await
```

---

# 🎤 Вопросы на собеседовании

### 1. Что такое threading?

Механизм многопоточности, позволяющий выполнять несколько потоков внутри одного процесса.

### 2. Чем поток отличается от процесса?

Потоки одного процесса разделяют память, процессы имеют отдельные адресные пространства.

### 3. Что делает `start()`?

Запускает поток.

### 4. Что делает `join()`?

Ждёт завершения потока.

### 5. Чем `start()` отличается от `run()`?

`start()` запускает отдельный поток, а прямой вызов `run()` выполняет код в текущем потоке.

### 6. Для каких задач полезен threading?

В первую очередь для I/O-bound задач.

### 7. Почему threading плохо подходит для CPU-bound pure Python в обычном CPython с GIL?

Потому что GIL ограничивает одновременное выполнение Python bytecode несколькими потоками.

### 8. Что такое shared memory?

Общая память процесса, доступная его потокам.

### 9. Какая проблема возникает из-за shared memory?

Race condition и необходимость синхронизации.

### 10. Что такое Lock?

Примитив синхронизации, позволяющий ограничить одновременный доступ к критической секции.

### 11. GIL и Lock — это одно и то же?

Нет.

**GIL** относится к выполнению Python bytecode в CPython, а **Lock** используется приложением для синхронизации доступа к общему состоянию.

### 12. Что такое ThreadPoolExecutor?

Высокоуровневый механизм управления пулом потоков, позволяющий отправлять задачи worker-потокам без ручного управления каждым `Thread`.

---

## 🧠 Формула для собеседования

> **Threading = несколько потоков внутри одного процесса + общая память + конкурентное выполнение + особенно полезно для I/O-bound задач + синхронизация общего состояния.**
>
> **Главная проблема — shared state → race condition → Lock / Queue / другие механизмы синхронизации.**
>
> **CPU-bound pure Python в обычном CPython → GIL → чаще multiprocessing.**
