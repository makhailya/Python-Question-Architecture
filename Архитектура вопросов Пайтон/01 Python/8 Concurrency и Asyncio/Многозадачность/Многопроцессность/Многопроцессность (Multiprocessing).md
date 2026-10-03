# ⚙️ Многопроцессность (Multiprocessing)

## 🎤 Короткий ответ

**Многопроцессность (multiprocessing)** — это способ конкурентного и параллельного выполнения задач, при котором программа запускает несколько независимых процессов.

Каждый процесс имеет **собственное адресное пространство и собственный интерпретатор Python**. Поэтому процессы могут выполнять Python-код **параллельно на разных [[CPU-bound]]** и не ограничиваются одним общим [[GIL — Global Interpreter Lock]].

Главный недостаток — процессы тяжелее потоков, а обмен данными между ними сложнее: используются **IPC** — `Queue`, `Pipe`, shared memory и другие механизмы.

---

## 🗣️ Ответ на собеседовании

> Multiprocessing — это выполнение задач в нескольких независимых процессах.
>
> В отличие от потоков, процессы имеют отдельные адресные пространства и собственное состояние интерпретатора. Поэтому несколько процессов могут реально выполнять Python-код параллельно на разных ядрах CPU.
>
> Это особенно полезно для CPU-bound задач: сложных вычислений, обработки изображений, анализа данных и других операций, где основное время тратится на CPU.
>
> Обратная сторона — создание процесса дороже, чем создание потока, а процессы не имеют общей памяти по умолчанию. Для обмена данными используют IPC: `Queue`, `Pipe`, shared memory и другие механизмы.
>
> В Python для этого используется модуль `multiprocessing`, а для управления пулом worker-процессов можно использовать `multiprocessing.Pool` или `ProcessPoolExecutor`.

---

## 🧭 Где я нахожусь

```text id="n4k8qp"
01 Python
└── 08 Concurrency и Asyncio
    │
    ├── Многозадачность
    │   ├── Конкурентность
    │   └── Параллелизм
    │
    ├── Многопоточность
    │   ├── Thread
    │   ├── Shared Memory
    │   ├── GIL
    │   ├── Race Condition
    │   ├── Data Race
    │   └── Синхронизация
    │
    ├── Многопроцессность ← Я здесь
    │   ├── Process
    │   ├── Pool
    │   ├── IPC
    │   │   ├── Queue
    │   │   └── Pipe
    │   ├── Shared Memory
    │   └── ProcessPoolExecutor
    │
    └── Asyncio
        ├── Event Loop
        ├── Coroutine
        ├── Awaitable
        ├── Task
        └── Future
```

---

# 📚 Разбор поглубже

## 1. Что такое процесс

**Process (процесс)** — это выполняющаяся программа, которой операционная система выделяет собственное адресное пространство и ресурсы.

Упрощённо:

```text id="v7c2ma"
Process
│
├── Code
├── Heap
├── Stack
├── Python objects
├── File descriptors
└── Python interpreter
```

Если запустить несколько процессов:

```text id="q8r4tx"
Process 1
└── Memory 1

Process 2
└── Memory 2

Process 3
└── Memory 3
```

Память между ними изолирована.

---

# 2. Что такое multiprocessing

`multiprocessing` позволяет создать несколько процессов:

```text id="c9m2pk"
Main Process
     │
     ├── Process 1
     ├── Process 2
     ├── Process 3
     └── Process 4
```

Если компьютер имеет несколько CPU-ядер:

```text id="s4f7nv"
CPU Core 1 ← Process 1
CPU Core 2 ← Process 2
CPU Core 3 ← Process 3
CPU Core 4 ← Process 4
```

Процессы могут действительно выполнять вычисления параллельно.

---

# 3. Почему multiprocessing помогает с GIL

В обычном CPython с включённым GIL:

```text id="h7p3kx"
Process
│
├── Thread 1 ──┐
├── Thread 2 ──┼── один GIL
└── Thread 3 ──┘
```

Несколько потоков конкурируют за GIL.

У процессов ситуация другая:

```text id="m2z8cq"
Process 1
└── CPython
    └── GIL 1

Process 2
└── CPython
    └── GIL 2

Process 3
└── CPython
    └── GIL 3
```

У каждого процесса собственный интерпретатор и собственное состояние.

Поэтому:

```text id="6w4s1b"
Core 1 → Process 1
Core 2 → Process 2
Core 3 → Process 3
```

могут выполнять Python bytecode параллельно.

---

# 4. Для каких задач нужен multiprocessing

Главная категория:

> **CPU-bound задачи**

То есть задачи, где большую часть времени процессор занят вычислениями.

Например:

```python id="p1m4kv"
def calculate():
    result = 0

    for i in range(100_000_000):
        result += i * i

    return result
```

Другие примеры:

* математические вычисления;
* обработка изображений;
* некоторые ML/AI-задачи;
* обработка больших массивов;
* CPU-heavy ETL;
* сложный парсинг;
* вычислительные алгоритмы.

Упрощённо:

```text id="x8r2cf"
CPU-bound
    ↓
нужно загрузить CPU
    ↓
несколько процессов
    ↓
несколько CPU cores
```

---

# 5. Multiprocessing vs Threading

Это один из самых важных вопросов на собеседовании.

|                    | Threading                                | Multiprocessing              |
| ------------------ | ---------------------------------------- | ---------------------------- |
| Единица выполнения | Thread                                   | Process                      |
| Память             | общая                                    | отдельная                    |
| GIL                | общий в процессе                         | отдельный у каждого процесса |
| CPU parallelism    | ограничен GIL в обычном CPython          | да                           |
| I/O-bound          | хорошо                                   | возможно, но часто избыточно |
| CPU-bound          | обычно не лучший вариант для pure Python | хорошо подходит              |
| Создание           | дешевле                                  | дороже                       |
| Обмен данными      | простой через общую память               | IPC                          |
| Изоляция           | ниже                                     | выше                         |

Упрощённое правило:

```text id="r9m3kf"
I/O-bound
   ↓
Threading / Asyncio

CPU-bound
   ↓
Multiprocessing
```

Но это **эвристика**, а не абсолютное правило: конкретный выбор зависит от характера нагрузки, используемых библиотек, стоимости IPC и архитектуры приложения.

---

# 6. Создание Process

В Python:

```python id="c5w8za"
from multiprocessing import Process


def worker():
    print("Работаю в отдельном процессе")


process = Process(target=worker)

process.start()
process.join()
```

Здесь:

```python id="2a6z8q"
process.start()
```

запускает отдельный процесс.

А:

```python id="t9x3pm"
process.join()
```

заставляет текущий процесс дождаться его завершения.

---

# 7. `start()` и `run()`

Как и у `Thread`, здесь есть важная разница.

```python id="n6r2jb"
process.start()
```

запускает отдельный процесс.

А:

```python id="a8c5pd"
process.run()
```

не создаёт новый процесс — он выполняет target в текущем процессе.

Поэтому:

```text id="q5w8vk"
start()
  ↓
новый Process
  ↓
target()


run()
  ↓
текущий Process
```

---

# 8. `join()`

`join()` используется для ожидания завершения процесса.

```python id="x4j6nt"
process.start()
process.join()

print("Процесс завершён")
```

Схема:

```text id="s6c8qp"
Main Process
     │
     ├── start()
     │
     ├── join() ────────────────┐
     │                          │
     │                     Process
     │                       работа
     │                          │
     └── продолжение ◄──────────┘
```

---

# 9. `Process` не разделяет обычные Python-переменные

Это принципиальное отличие от threading.

```python id="h1r5zd"
counter = 0
```

Если создать два процесса, они не будут автоматически изменять одну и ту же переменную.

```text id="e8c2mq"
Process 1
counter = 0

Process 2
counter = 0
```

Это два независимых состояния.

Например:

```python id="v2s9kc"
counter = 0


def worker():
    global counter
    counter += 1
```

Изменение `counter` внутри worker-процесса не изменит автоматически `counter` в родительском процессе.

---

# 10. Почему память изолирована

Каждый процесс получает собственное адресное пространство:

```text id="u5g3nz"
Process A
┌──────────────────┐
│ Memory A         │
└──────────────────┘

Process B
┌──────────────────┐
│ Memory B         │
└──────────────────┘
```

Это даёт преимущества:

* изоляцию;
* независимость состояния;
* меньший риск прямого повреждения памяти другого процесса.

Но создаёт проблему:

> Как передавать данные между процессами?

Ответ — **IPC**.

---

# 11. IPC — Inter-Process Communication

**IPC** — способы взаимодействия между процессами.

Основные механизмы:

```text id="v3k7sa"
IPC
│
├── Queue
├── Pipe
├── Shared Memory
├── Socket
└── другие механизмы
```

---

# 12. Queue

`multiprocessing.Queue` позволяет безопасно передавать объекты между процессами.

```python id="y7f2pk"
from multiprocessing import Process, Queue


def worker(queue):
    queue.put("result")


if __name__ == "__main__":
    queue = Queue()

    process = Process(
        target=worker,
        args=(queue,)
    )

    process.start()

    result = queue.get()

    process.join()

    print(result)
```

Схема:

```text id="w3q8mz"
Process A
    │
    │ put()
    ▼
┌─────────┐
│  Queue  │
└─────────┘
    │
    │ get()
    ▼
Process B
```

Это пример **message passing**.

---

# 13. Pipe

`Pipe` создаёт канал связи между двумя участниками.

```python id="f9x3vb"
from multiprocessing import Process, Pipe


def worker(conn):
    conn.send("Hello")
    conn.close()


if __name__ == "__main__":
    parent_conn, child_conn = Pipe()

    process = Process(
        target=worker,
        args=(child_conn,)
    )

    process.start()

    print(parent_conn.recv())

    process.join()
```

Схема:

```text id="q7n4yc"
Process A
    │
    │ send()
    ▼
  Pipe
    │
    ▼
Process B
    │
    │ recv()
```

`Pipe` особенно естественен для связи между двумя участниками.

---

# 14. Queue vs Pipe

| Queue                                        | Pipe                                                   |
| -------------------------------------------- | ------------------------------------------------------ |
| Очередь сообщений                            | Канал связи                                            |
| Удобен для producer/consumer                 | Хорош для прямого обмена                               |
| Может использоваться несколькими участниками | Обычно рассматривается как связь между двумя endpoints |
| Высокоуровневее                              | Более низкоуровневый механизм                          |

---

# 15. Shared Memory

Можно не передавать данные через Queue/Pipe, а создать область общей памяти.

```text id="s2k9vn"
Process A ──┐
            ├── Shared Memory
Process B ──┘
```

В Python:

```python id="q4b7nc"
from multiprocessing import shared_memory
```

Это особенно интересно для больших объёмов данных, когда копирование и сериализация становятся дорогими.

Но:

> Shared Memory не отменяет необходимость синхронизации.

Если два процесса одновременно изменяют одни данные:

```text id="k6c1pv"
Process A ── WRITE ──┐
                     ├── Shared Memory
Process B ── WRITE ──┘
```

может возникнуть конкурентная проблема.

---

# 16. Pickle и multiprocessing

При передаче объектов между процессами Python часто требуется их **сериализация**.

Например:

```text id="g7v5ma"
Process A
   │
   ├── Python object
   ↓
pickle
   ↓
IPC
   ↓
unpickle
   ↓
Process B
```

Это удобно, но имеет цену:

* CPU;
* память;
* копирование данных;
* время сериализации/десериализации.

Поэтому передача огромных объектов между процессами может стать bottleneck.

---

# 17. Pool

Если нужно обработать много однотипных задач, удобно использовать пул процессов.

```python id="z4q8wk"
from multiprocessing import Pool


def square(x):
    return x * x


if __name__ == "__main__":
    with Pool(processes=4) as pool:
        results = pool.map(square, range(10))

    print(results)
```

Схема:

```text id="m7p3cx"
            Pool
             │
     ┌───────┼───────┐
     ↓       ↓       ↓
 Process  Process  Process
    1        2        3
```

Пул переиспользует worker-процессы.

Это удобнее, чем вручную создавать отдельный процесс для каждой маленькой задачи.

---

# 18. ProcessPoolExecutor

Современный высокоуровневый вариант:

```python id="n2v6sd"
from concurrent.futures import ProcessPoolExecutor


def square(x):
    return x * x


if __name__ == "__main__":
    with ProcessPoolExecutor(max_workers=4) as executor:
        results = executor.map(square, range(10))

    print(list(results))
```

По концепции он похож на `ThreadPoolExecutor`, но worker'ы являются процессами.

```text id="k4s7pm"
ThreadPoolExecutor
        ↓
      Threads


ProcessPoolExecutor
        ↓
     Processes
```

---

# 19. Почему процессы дороже потоков

Создание процесса требует больше ресурсов.

У процесса есть:

* собственное адресное пространство;
* собственный Python interpreter;
* свои ресурсы;
* IPC для обмена данными.

У потока:

```text id="c8q2rm"
Process
├── Thread 1
├── Thread 2
└── Thread 3
```

потоки используют инфраструктуру уже существующего процесса.

Поэтому:

```text id="x9p5nv"
Thread
→ дешевле

Process
→ дороже
```

Но процессы дают более сильную изоляцию и CPU parallelism.

---

# 20. `if __name__ == "__main__"`

При использовании multiprocessing важно защищать точку запуска:

```python id="r6w3pz"
if __name__ == "__main__":
    process = Process(target=worker)
    process.start()
    process.join()
```

Это особенно важно для способов запуска процессов, где новый интерпретатор импортирует основной модуль.

Без такой защиты код создания новых процессов может выполняться повторно при импорте и приводить к проблемам с созданием процессов.

---

# 21. Start methods

В `multiprocessing` существуют разные способы запуска процессов.

Основные:

```text id="a3w9kc"
start method
│
├── spawn
├── fork
└── forkserver
```

### `spawn`

Создаётся новый Python interpreter.

### `fork`

Процесс создаётся через fork текущего процесса.

### `forkserver`

Используется отдельный server process, который создаёт worker-процессы через fork.

Выбор start method имеет архитектурные последствия, особенно для состояния процесса, потоков и сторонних библиотек.

---

# 22. Важный нюанс для macOS

На macOS важно помнить, что `spawn` является стандартным start method в современных версиях Python.

Поэтому привычная Linux-модель:

```text
fork → дочерний процесс получает состояние родителя
```

не должна автоматически переноситься на macOS.

Для кроссплатформенного multiprocessing особенно важно:

```python id="h9c4zy"
if __name__ == "__main__":
    ...
```

и аккуратно проектировать код, который запускается при импорте модуля.

---

# 23. Multiprocessing и Shared Mutable State

Главное различие с threading:

### Threading

```text id="f5c2n8"
Process
│
├── Thread 1 ──┐
├── Thread 2 ──┼── Shared State
└── Thread 3 ──┘
```

### Multiprocessing

```text id="e2s7kc"
Process 1 ── Memory 1
Process 2 ── Memory 2
Process 3 ── Memory 3
```

Shared state между процессами нужно организовывать специально.

Это уменьшает количество случайных конфликтов памяти, но делает обмен данными более сложным.

---

# 24. Когда multiprocessing может быть плохим выбором

Не стоит автоматически использовать процессы для любой задачи.

Например, если:

```text id="v5h9ca"
задача очень маленькая
```

а стоимость:

```text id="m2x7qp"
создание процесса
+
IPC
+
serialization
+
copying
```

больше, чем сама работа, multiprocessing может сделать программу **медленнее**.

Поэтому важна стоимость задачи относительно overhead.

---

# 25. CPU-bound vs I/O-bound

Главная схема:

```text id="q8c5nm"
                    Задача
                      │
             ┌────────┴────────┐
             ↓                 ↓
         CPU-bound          I/O-bound
             │                 │
             ↓                 ↓
      Multiprocessing       Threading
             │              / Asyncio
             ↓
       CPU parallelism
```

Но это не абсолютное правило.

Например, I/O-heavy программа может использовать multiprocessing по другим причинам — изоляция, архитектура, распределение нагрузки и т.д.

---

# 26. Полная картина

```text id="c7m2vq"
                 Concurrency
                      │
        ┌─────────────┼─────────────┐
        ↓             ↓             ↓
    Threading    Multiprocessing   Asyncio
        │             │              │
        │             │              ├── Event Loop
        │             │              ├── Task
        │             │              └── await
        │             │
        │             ├── Process
        │             ├── Pool
        │             ├── IPC
        │             │   ├── Queue
        │             │   └── Pipe
        │             └── Shared Memory
        │
        ├── Shared Memory
        ├── GIL
        ├── Race Condition
        └── Synchronization
```

---

# 🎤 Вопросы на собеседовании

### Что такое multiprocessing?

Выполнение задач в нескольких независимых процессах, каждый из которых имеет собственное адресное пространство и интерпретатор.

### Для каких задач используют multiprocessing?

Прежде всего для CPU-bound задач, когда требуется использовать несколько CPU-ядер для выполнения Python-кода.

### Почему multiprocessing помогает с GIL?

У каждого процесса свой интерпретатор и своё состояние, поэтому процессы могут параллельно выполнять Python bytecode.

### Чем процесс отличается от потока?

Процесс имеет отдельное адресное пространство, поток работает внутри процесса и разделяет его память с другими потоками.

### Разделяют ли процессы обычные Python-переменные?

Нет. По умолчанию память процессов изолирована.

### Как процессы обмениваются данными?

Через IPC: `Queue`, `Pipe`, shared memory, sockets и другие механизмы.

### Что такое IPC?

Inter-Process Communication — механизмы обмена данными и координации между процессами.

### Что такое `multiprocessing.Queue`?

Очередь для обмена данными между процессами.

### Что такое `Pipe`?

Канал связи между процессами, позволяющий отправлять и получать сообщения через endpoints.

### Что такое Shared Memory?

Специальная область памяти, доступная нескольким процессам.

### Почему процессы дороже потоков?

У каждого процесса собственная память и интерпретатор, а обмен данными требует IPC и часто сериализации.

### Что такое `Pool`?

Пул worker-процессов, позволяющий переиспользовать процессы для выполнения множества задач.

### Что такое `ProcessPoolExecutor`?

Высокоуровневый интерфейс из `concurrent.futures` для выполнения задач в пуле процессов.

### Зачем нужен `if __name__ == "__main__"`?

Чтобы код запуска процессов выполнялся только в основном модуле и не запускался повторно при импорте дочерним процессом.

### Может ли multiprocessing работать с Shared Memory?

Да. Например, через `multiprocessing.shared_memory`.

---

## 🧠 Формула для собеседования

> **Multiprocessing = несколько независимых процессов + отдельная память + отдельный Python interpreter + реальный CPU parallelism.**
>
> Поэтому:
>
> **CPU-bound → multiprocessing часто подходит лучше threading.**
>
> Главный минус:
>
> **Process дороже Thread + память изолирована + нужен IPC.**
>
> IPC:
>
> **Queue / Pipe / Shared Memory / Socket**
>
> И главное:
>
> **Thread → shared memory по умолчанию.**
>
> **Process → isolated memory по умолчанию.**
>
> **Shared memory между процессами → нужно создавать специально и отдельно решать вопрос синхронизации.**
