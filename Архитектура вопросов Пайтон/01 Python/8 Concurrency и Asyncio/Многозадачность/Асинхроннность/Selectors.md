# 🔌 Selectors

## 🎤 Короткий ответ

**Selector** — механизм ОС для ожидания событий от множества файловых дескрипторов одновременно.

В Python он доступен через модуль `selectors` и используется как абстракция над механизмами мультиплексирования I/O операционной системы.

Идея:

```text
Много сокетов
     ↓
  Selector
     ↓
ОС сообщает:
какие готовы к I/O
     ↓
обрабатываем только готовые
```

Это позволяет одному потоку обслуживать большое количество сетевых соединений, не создавая отдельный поток или процесс для каждого сокета.

В Linux под капотом обычно используется **epoll**, а на macOS — **kqueue**.

---

## 🗣️ Ответ на собеседовании

Selector — это механизм **I/O multiplexing**, который позволяет одному потоку следить сразу за множеством файловых дескрипторов и ждать, когда один или несколько из них будут готовы к операции ввода-вывода.

Например, сервер может иметь тысячи TCP-соединений. Вместо того чтобы блокироваться на каждом сокете отдельно, он регистрирует их в selector:

```text
socket 1 ─┐
socket 2 ─┤
socket 3 ─┼──→ Selector ──→ ОС
socket 4 ─┤
socket N ─┘
```

ОС сообщает, какие дескрипторы готовы к чтению или записи. После этого программа обрабатывает только их.

В Python есть модуль `selectors`, который предоставляет переносимый интерфейс над механизмами ОС. На Linux обычно используется `epoll`, на macOS — `kqueue`.

Этот механизм является одной из основ асинхронного сетевого программирования. В частности, event loop может использовать selector, чтобы эффективно ждать готовности сетевых операций.

---

## 🧭 Где я нахожусь

```text
01 Python
└── 08 Concurrency и Asyncio
    ├── Многопоточность
    ├── Многопроцессность
    ├── Asyncio
    │   ├── Event Loop
    │   ├── Coroutine
    │   ├── Task
    │   ├── Future
    │   ├── await
    │   ├── Неблокирующий I/O
    │   ├── I/O Multiplexing
    │   │   └── Selectors ← Я здесь
    │   └── epoll / kqueue
    └── GIL
```

---

# 📚 Разбор поглубже

## 1. Какую проблему решает Selector

Представим сервер с большим количеством клиентов:

```text
Client A ── socket A
Client B ── socket B
Client C ── socket C
...
Client 10000 ── socket 10000
```

Нужно понимать, с каким сокетом сейчас можно работать.

Наивный вариант:

```python
while True:
    read(socket_a)
    read(socket_b)
    read(socket_c)
```

Проблема — `read()` может заблокироваться на сокете, в котором пока нет данных.

Можно создать отдельный поток на каждый сокет, но при большом количестве соединений это становится дорого и сложно.

Selector решает проблему иначе:

```text
                ┌── socket A
                ├── socket B
Applications ───┼── socket C
                ├── socket D
                └── socket E
                       ↓
                    Selector
                       ↓
                      ОС
                       ↓
             A и D готовы к чтению
                       ↓
                обрабатываем A и D
```

---

# 2. Что такое I/O multiplexing

**I/O multiplexing** — возможность одному потоку ожидать события сразу на множестве I/O-источников.

То есть вместо:

```text
Thread 1 → socket 1
Thread 2 → socket 2
Thread 3 → socket 3
```

можно сделать:

```text
              socket 1
              socket 2
              socket 3
                  ↓
               Selector
                  ↓
              один поток
```

ОС отслеживает состояние дескрипторов и сообщает, какие готовы.

---

# 3. File Descriptor

Selector работает не непосредственно с Python-объектами, а с **file descriptors (FD)**.

File descriptor — целочисленный идентификатор открытого ресурса процесса.

Например:

```text
0 → stdin
1 → stdout
2 → stderr
```

Сетевой socket также получает file descriptor:

```text
socket → FD 7
```

Selector может следить за этим FD.

---

# 4. `selectors` в Python

Python предоставляет высокоуровневый интерфейс:

```python
import selectors

selector = selectors.DefaultSelector()
```

`DefaultSelector` выбирает наиболее подходящий механизм для текущей платформы.

Условно:

```text
Linux
  ↓
epoll

macOS
  ↓
kqueue
```

То есть код Python может использовать одинаковый API, не работая напрямую с конкретным системным механизмом.

---

# 5. Регистрация сокета

Чтобы selector отслеживал socket, его нужно зарегистрировать:

```python
import selectors
import socket

selector = selectors.DefaultSelector()

sock = socket.socket()
sock.setblocking(False)

selector.register(
    sock,
    selectors.EVENT_READ
)
```

Здесь:

```text
sock
 ↓
register()
 ↓
selector
 ↓
отслеживает EVENT_READ
```

---

# 6. EVENT_READ и EVENT_WRITE

Основные события:

```python
selectors.EVENT_READ
selectors.EVENT_WRITE
```

### `EVENT_READ`

Означает:

> можно выполнять операцию чтения без блокировки.

Например, в socket появились входящие данные.

### `EVENT_WRITE`

Означает:

> socket готов к записи без блокировки.

Например, можно отправлять данные.

Можно зарегистрировать оба:

```python
selector.register(
    sock,
    selectors.EVENT_READ | selectors.EVENT_WRITE
)
```

---

# 7. Получение готовых событий

Основной метод:

```python
events = selector.select()
```

Он ждёт, пока появятся готовые к I/O дескрипторы.

После этого:

```python
for key, mask in events:
    ...
```

`key` содержит информацию о зарегистрированном объекте.

`mask` показывает, какое событие произошло.

Например:

```python
if mask & selectors.EVENT_READ:
    print("Можно читать")
```

---

# 8. Простейшая схема event loop

Упрощённый event loop выглядит так:

```python
while True:
    events = selector.select()

    for key, mask in events:
        callback = key.data
        callback(key.fileobj, mask)
```

Логика:

```text
        Event Loop
             ↓
    selector.select()
             ↓
       ОС ждёт I/O
             ↓
     появились события
             ↓
      готовые sockets
             ↓
         callback
             ↓
        обработка
             ↓
    снова select()
```

Это фундаментальная идея event-driven I/O.

---

# 9. `timeout`

Можно передать timeout:

```python
events = selector.select(timeout=1)
```

Это означает:

> ждать события максимум одну секунду.

Если событий нет:

```python
events == []
```

Можно использовать:

```python
selector.select(timeout=0)
```

для неблокирующей проверки.

---

# 10. `key`

При регистрации:

```python
selector.register(
    sock,
    selectors.EVENT_READ,
    data="client_1"
)
```

`data` можно использовать для хранения контекста.

После `select()`:

```python
for key, mask in selector.select():
    print(key.fileobj)
    print(key.events)
    print(key.data)
```

Например:

```text
key.fileobj → socket
key.events  → EVENT_READ
key.data    → "client_1"
```

---

# 11. `register`, `modify`, `unregister`

### Зарегистрировать

```python
selector.register(
    sock,
    selectors.EVENT_READ
)
```

### Изменить события

```python
selector.modify(
    sock,
    selectors.EVENT_READ | selectors.EVENT_WRITE
)
```

### Удалить

```python
selector.unregister(sock)
```

Например, клиент отключился:

```text
client disconnected
       ↓
selector.unregister(socket)
       ↓
selector больше не отслеживает FD
```

---

# 12. Полный упрощённый пример

```python
import selectors
import socket

selector = selectors.DefaultSelector()

server = socket.socket()
server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
server.bind(("127.0.0.1", 8000))
server.listen()
server.setblocking(False)

selector.register(
    server,
    selectors.EVENT_READ
)

while True:
    events = selector.select()

    for key, mask in events:
        sock = key.fileobj

        if sock is server:
            client, address = server.accept()
            client.setblocking(False)

            selector.register(
                client,
                selectors.EVENT_READ
            )

        else:
            data = sock.recv(4096)

            if data:
                sock.sendall(data)
            else:
                selector.unregister(sock)
                sock.close()
```

Это упрощённый event-driven сервер.

Идея здесь важнее конкретной реализации:

```text
server socket
     ↓
accept
     ↓
client sockets
     ↓
selector
     ↓
готовые sockets
     ↓
recv/send
```

---

# 13. Что происходит под капотом

Python:

```python
selector.select()
```

не реализует механизм ожидания I/O самостоятельно.

`selectors` выбирает системный механизм.

Упрощённо:

```text
Python
  ↓
selectors
  ↓
DefaultSelector
  ↓
epoll / kqueue / poll / select
  ↓
Kernel
  ↓
Network sockets
```

На Linux наиболее важен:

```text
epoll
```

На macOS:

```text
kqueue
```

---

# 14. `select`, `poll`, `epoll`, `kqueue`

Это разные механизмы ОС для I/O multiplexing.

Упрощённо:

| Механизм | Где                         |
| -------- | --------------------------- |
| `select` | переносимый старый механизм |
| `poll`   | Unix/Linux                  |
| `epoll`  | Linux                       |
| `kqueue` | macOS/BSD                   |

Python `selectors` скрывает большую часть платформенных различий.

Поэтому вместо непосредственного использования:

```python
epoll(...)
```

можно использовать:

```python
selectors.DefaultSelector()
```

---

# 15. Почему epoll эффективнее select при большом количестве соединений

Упрощённо, `select()` работает с набором дескрипторов, который приходится проверять.

При большом количестве соединений это может приводить к лишней работе.

`epoll` использует другой механизм ожидания событий: ядро сообщает о готовых дескрипторах.

Упрощённая модель:

```text
select:

10000 sockets
     ↓
проверить множество FD
     ↓
найти готовые


epoll:

10000 sockets зарегистрированы
          ↓
       kernel
          ↓
готовые FD
          ↓
приложение
```

Это одна из причин, почему `epoll` хорошо подходит для большого количества сетевых соединений.

---

# 16. Selector и блокирующий I/O

Важно понимать:

> Selector сам по себе не делает произвольный код неблокирующим.

Например:

```python
events = selector.select()

data = socket.recv(4096)

```

обычно предполагает использование **non-blocking socket**.

Если после получения события выполнить другую долгую блокирующую операцию:

```python
data = slow_function()
```

event loop может зависнуть.

Поэтому асинхронная архитектура должна избегать блокирующих операций внутри event loop.

---

# 17. Selector и asyncio

Это особенно важно для собеседования.

Упрощённая архитектура:

```text
asyncio
   ↓
Event Loop
   ↓
Selector
   ↓
epoll / kqueue
   ↓
Kernel
   ↓
Sockets
```

Например:

```python
await reader.read()
```

не означает:

```text
поток стоит и блокируется
```

В типичном случае event loop регистрирует интересующее I/O-событие и переключается на другую работу.

Когда ОС сообщает:

```text
socket готов
```

event loop продолжает соответствующую coroutine.

---

# 18. Selector ≠ Event Loop

Это важное различие.

### Selector

Отвечает за:

> какие I/O-дескрипторы готовы?

### Event Loop

Отвечает за более широкую координацию:

* coroutine;
* Task;
* callbacks;
* timers;
* I/O events;
* scheduling.

Схема:

```text
              Event Loop
             /    |     \
            /     |      \
       Tasks    Timers   Selector
                           ↓
                      I/O readiness
```

То есть selector — **часть механизма event loop**, а не весь event loop.

---

# 19. Selector ≠ asyncio

`selectors` можно использовать непосредственно:

```python
import selectors
```

и построить собственный event-driven цикл.

`asyncio` предоставляет значительно более высокий уровень абстракции.

```text
selectors
   ↓
низкоуровневый I/O multiplexing


asyncio
   ↓
coroutines
Tasks
await
Event Loop
I/O
timers
```

Поэтому обычно Backend-разработчик работает с `asyncio`, а не напрямую с selector.

---

# 20. Selector ≠ многопоточность

Selector позволяет одному потоку обслуживать множество I/O-источников.

Например:

```text
1 thread
   ↓
Selector
   ├── socket 1
   ├── socket 2
   ├── socket 3
   └── socket 10000
```

Это отличается от:

```text
10000 sockets
   ↓
10000 threads
```

Но selector не делает CPU-bound код параллельным.

Если один callback выполняет:

```python
for _ in range(10**10):
    ...
```

event loop будет занят этой работой.

---

# 21. Selector и I/O-bound задачи

Selector особенно полезен для **I/O-bound** задач:

* HTTP-соединения;
* TCP-сокеты;
* WebSocket;
* сетевые серверы;
* большое количество одновременных соединений.

Например:

```text
1000 clients
    ↓
network I/O
    ↓
selector
    ↓
один event loop
```

Пока один клиент ждёт сеть, можно обрабатывать другой.

---

# 22. Важная цепочка для собеседования

Нужно уметь объяснить:

```text
Socket
   ↓
File Descriptor
   ↓
Selector
   ↓
epoll / kqueue
   ↓
Kernel
   ↓
I/O readiness
   ↓
Event Loop
   ↓
Coroutine / Task
```

И в обратную сторону:

```text
Данные пришли в socket
        ↓
Kernel помечает FD готовым
        ↓
epoll/kqueue сообщает событие
        ↓
Selector получает событие
        ↓
Event Loop обрабатывает его
        ↓
возобновляется coroutine
```

---

# 23. Типичная ошибка на собеседовании

❌ Неправильно:

> Selector выполняет асинхронность.

Точнее:

> Selector предоставляет механизм ожидания готовности I/O на множестве файловых дескрипторов. Event loop использует этот механизм для организации неблокирующего I/O.

---

❌ Неправильно:

> epoll создаёт отдельный поток на каждый socket.

Нет.

`epoll` — механизм ядра Linux для мониторинга большого количества FD.

---

❌ Неправильно:

> Selector запускает несколько задач параллельно.

Нет.

Он сообщает о готовности I/O. Планированием coroutine/tasks занимается более высокий уровень — например, event loop.

---

## 🎤 Вопросы на собеседовании

### Что такое Selector?

Механизм I/O multiplexing для ожидания событий на множестве файловых дескрипторов.

### Зачем нужен Selector?

Чтобы одному потоку эффективно обслуживать множество I/O-соединений и не блокироваться отдельно на каждом сокете.

### Что находится под `selectors.DefaultSelector()`?

Платформенный механизм I/O multiplexing. Например, `epoll` на Linux и `kqueue` на macOS.

### Что делает `selector.select()`?

Ждёт и возвращает зарегистрированные файловые дескрипторы, которые готовы к указанным операциям I/O.

### Что такое `EVENT_READ`?

Событие, означающее, что соответствующий дескриптор готов к чтению без блокировки.

### Что такое `EVENT_WRITE`?

Событие, означающее, что дескриптор готов к записи без блокировки.

### Selector и Event Loop — одно и то же?

Нет.

**Selector** отслеживает готовность I/O, а **Event Loop** координирует задачи, callbacks, таймеры и I/O-события.

### Как Selector связан с asyncio?

Упрощённо:

```text
asyncio Event Loop
       ↓
    Selector
       ↓
epoll / kqueue
       ↓
     Kernel
       ↓
    Sockets
```

### Почему Selector полезен для сетевого сервера?

Потому что сервер может обслуживать множество соединений одним потоком, обрабатывая только те sockets, которые в данный момент готовы к I/O.

### Selector делает CPU-bound задачи параллельными?

Нет. Selector предназначен для multiplexing I/O, а не для выполнения CPU-bound вычислений.

### Какая главная идея I/O multiplexing?

Не ждать каждый socket отдельно, а передать множество FD ОС и получить информацию о тех, которые готовы к работе.

```text
много FD
   ↓
kernel
   ↓
готовые FD
   ↓
event loop
   ↓
обработка
```
