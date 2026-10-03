# 📡 Signals

## 🎤 Короткий ответ

**Signal (сигнал)** — это механизм Linux/Unix для асинхронного уведомления процесса о событии.

Процесс получает сигнал и может:

* выполнить стандартное действие;
* обработать сигнал самостоятельно;
* проигнорировать некоторые сигналы.

Например:

```text id="f1e9r2"
Ctrl+C
   ↓
SIGINT
   ↓
процесс
   ↓
обычно завершение
```

Сигнал можно отправить командой:

```bash id="p7r2g4"
kill -TERM PID
```

Важно: `kill` не обязательно означает «убить процесс». Это команда **отправки сигнала** процессу.

---

## 🗣️ Ответ на собеседовании

Signal — это механизм IPC в Unix/Linux, который позволяет асинхронно уведомлять процесс о событии.

У каждого сигнала есть номер и имя, например `SIGTERM`, `SIGINT`, `SIGKILL`, `SIGSTOP`.

Сигнал можно отправить командой `kill`. Например:

```bash id="a2c9v7"
kill -TERM 1234
```

По умолчанию `SIGTERM` просит процесс корректно завершиться. Процесс может обработать этот сигнал и выполнить cleanup: закрыть соединения, сохранить состояние и освободить ресурсы.

`SIGKILL` отличается тем, что процесс не может его обработать, проигнорировать или перехватить — ядро принудительно завершает процесс.

В Backend и DevOps сигналы особенно важны при остановке сервисов, graceful shutdown, работе systemd, Docker и управлении процессами.

---

## 🧭 Где я нахожусь

```text id="m3p8xq"
07 DevOps
└── 06 Environment и Infrastructure
    ├── Linux
    │   ├── Файловая система
    │   ├── Процессы
    │   │   └── Signals ← Я здесь
    │   ├── Права доступа
    │   ├── Shell
    │   │   └── Terminal
    │   ├── SSH
    │   └── Сеть
    └── systemd
```

---

# 📚 Разбор поглубже

## 1. Что такое Signal

Signal — это способ уведомить процесс о некотором событии.

Упрощённо:

```text id="w5m9xz"
Kernel / Process
       │
       │ signal
       ▼
    Process
```

Например:

```text id="s2d6va"
Пользователь
     │
   Ctrl+C
     ↓
  Terminal
     ↓
   SIGINT
     ↓
  Python
```

Сигналы являются асинхронным механизмом: процесс не обязательно находится в момент получения сигнала в той точке программы, где он ожидает это событие.

---

# 2. Основные сигналы

Для Backend-разработчика особенно важно знать:

| Signal    | Номер* | Назначение                                                            |
| --------- | -----: | --------------------------------------------------------------------- |
| `SIGINT`  |      2 | прерывание процесса пользователем                                     |
| `SIGTERM` |     15 | просьба корректно завершиться                                         |
| `SIGKILL` |      9 | принудительное завершение                                             |
| `SIGSTOP` |   19** | приостановка процесса                                                 |
| `SIGCONT` |   18** | продолжение после остановки                                           |
| `SIGHUP`  |      1 | исторически hangup; также используется для некоторых сценариев reload |
| `SIGQUIT` |      3 | завершение с core dump по умолчанию                                   |
| `SIGCHLD` |   17** | изменение состояния дочернего процесса                                |

* Номера некоторых сигналов зависят от архитектуры и Unix-системы.
** На Linux x86/amd64 указаны типичные номера.

---

# 3. SIGINT

`SIGINT` — **Interrupt**.

Обычно отправляется при:

```text id="j8z4k1"
Ctrl+C
```

Например:

```bash id="x8c2w6"
python server.py
```

Нажимаем:

```text id="t1q4m7"
Ctrl+C
```

Упрощённо:

```text id="x4j6va"
Terminal
   ↓
SIGINT
   ↓
Python process
```

По умолчанию процесс завершается.

Но программа может обработать `SIGINT`.

---

# 4. SIGTERM

`SIGTERM` — стандартный сигнал для **корректного завершения процесса**.

Например:

```bash id="v6n2qa"
kill -TERM 1234
```

или:

```bash id="k4z9jp"
kill 1234
```

Поскольку `kill PID` по умолчанию отправляет `SIGTERM`.

Процесс может перехватить сигнал и выполнить cleanup:

```text id="a9m3kf"
SIGTERM
   ↓
Application
   ↓
закрыть соединения
   ↓
завершить текущие операции
   ↓
flush buffers
   ↓
exit
```

Именно поэтому `SIGTERM` обычно предпочтительнее `SIGKILL` для штатной остановки сервисов.

---

# 5. SIGKILL

`SIGKILL` — принудительное завершение процесса.

```bash id="g2v8ns"
kill -9 1234
```

или:

```bash id="r7c3pm"
kill -KILL 1234
```

Ключевое свойство:

> Процесс не может обработать, проигнорировать или перехватить `SIGKILL`.

То есть приложение не сможет выполнить свой обычный graceful shutdown.

Упрощённо:

```text id="q1v6cx"
SIGKILL
   ↓
Kernel
   ↓
Process terminated
```

Поэтому `SIGKILL` обычно используют, когда нормальное завершение не сработало.

---

# 6. SIGTERM vs SIGKILL

Это один из важных вопросов на собеседовании.

### SIGTERM

```text id="a8m2pq"
"Заверши работу корректно"
```

Процесс может:

* обработать сигнал;
* закрыть соединения;
* освободить ресурсы;
* завершить текущую работу;
* сохранить состояние.

### SIGKILL

```text id="c5v9ld"
"Заверши процесс немедленно"
```

Процесс не может выполнить собственный handler.

Поэтому:

```text id="z7n3kx"
SIGTERM
   ↓
graceful shutdown
```

предпочтительнее:

```text id="u4b8rm"
SIGKILL
   ↓
forced termination
```

---

# 7. `kill`

Название команды может вводить в заблуждение.

```bash id="s3h6wn"
kill PID
```

не означает буквально «убить».

Команда:

> отправляет указанный сигнал процессу.

По умолчанию:

```bash id="p9c2xa"
kill 1234
```

эквивалентно:

```bash id="m6k8tr"
kill -TERM 1234
```

Можно отправить другой сигнал:

```bash id="v5d1qp"
kill -HUP 1234
kill -INT 1234
kill -STOP 1234
kill -CONT 1234
```

---

# 8. Узнать PID

Например:

```bash id="h4x7qn"
ps aux
```

или:

```bash id="b8m2zc"
pgrep python
```

Получив PID:

```bash id="k5n9vf"
kill -TERM 1234
```

---

# 9. `kill -l`

Посмотреть список сигналов:

```bash id="p2x8ds"
kill -l
```

Например:

```text id="n7m4qc"
1) SIGHUP
2) SIGINT
3) SIGQUIT
...
4) SIGKILL
5) SIGTERM
```

---

# 10. Кто может отправлять сигнал

Сигналы отправляются с учётом прав доступа.

Обычный пользователь не может произвольно посылать сигналы процессам других пользователей.

Например:

```bash id="c9k2vb"
kill 1234
```

может завершиться ошибкой:

```text id="f6p3yx"
Operation not permitted
```

Для административного управления может потребоваться:

```bash id="q8m1zd"
sudo kill -TERM 1234
```

---

# 11. Signal Handler

Процесс может установить обработчик сигнала.

В Python:

```python id="e4n7js"
import signal

def handle_sigterm(signum, frame):
    print("Получен SIGTERM")

signal.signal(signal.SIGTERM, handle_sigterm)
```

Теперь при получении `SIGTERM` Python вызовет обработчик.

Концептуально:

```text id="y6r2mw"
SIGTERM
   ↓
signal handler
   ↓
cleanup
   ↓
exit
```

---

# 12. Signals в Python

Python предоставляет модуль:

```python id="v8q3ka"
import signal
```

Например:

```python id="n5x9bf"
import signal
import time

running = True

def stop(signum, frame):
    global running
    running = False

signal.signal(signal.SIGTERM, stop)

while running:
    print("Working...")
    time.sleep(1)

print("Graceful shutdown")
```

При `SIGTERM`:

```text id="s4m8cx"
running = False
      ↓
цикл заканчивается
      ↓
программа завершается
```

---

# 13. Важный нюанс Python

В CPython обработчики сигналов имеют важную особенность: Python-level signal handlers выполняются в **главном потоке интерпретатора**.

Поэтому обработка сигналов в многопоточном приложении имеет нюансы.

Упрощённо:

```text id="p3c7vk"
Process
├── Main Thread
│      ↑
│   signal handler
│
├── Worker Thread
└── Worker Thread
```

Это особенно важно при работе с:

* threading;
* asyncio;
* web servers.

---

# 14. SIGSTOP

`SIGSTOP` приостанавливает процесс:

```bash id="d7m3qa"
kill -STOP 1234
```

Процесс перестаёт выполняться.

В отличие от обычного сигнала завершения, процесс не завершается — он **останавливается**.

---

# 15. SIGCONT

Продолжить остановленный процесс:

```bash id="j9v5cx"
kill -CONT 1234
```

Получается:

```text id="x6q2nz"
Running
   ↓
SIGSTOP
   ↓
Stopped
   ↓
SIGCONT
   ↓
Running
```

---

# 16. Почему SIGSTOP особенный

Как и `SIGKILL`, `SIGSTOP` нельзя перехватить или проигнорировать.

То есть приложение не может установить обычный handler для:

```text id="m2c8vy"
SIGSTOP
SIGKILL
```

Это позволяет ядру гарантированно остановить или завершить процесс.

---

# 17. SIGHUP

Исторически `SIGHUP` означал:

> Hang Up — разрыв терминального соединения.

Но современные серверные приложения также могут использовать `SIGHUP` для других целей, например для перечитывания конфигурации.

Важно:

> конкретное поведение `SIGHUP` зависит от приложения.

Например, некоторые daemon/service могут трактовать:

```text id="w5p9ca"
SIGHUP
   ↓
reload configuration
```

Но нельзя считать, что любой процесс автоматически делает reload при `SIGHUP`.

---

# 18. SIGCHLD

`SIGCHLD` отправляется родительскому процессу, когда состояние дочернего процесса изменяется, например при его завершении.

Упрощённо:

```text id="r8q3kn"
Parent
  │
  └── Child
        │
        ↓
       exit
        │
        ↓
     SIGCHLD
        │
        ↓
     Parent
```

Это связано с механизмом:

```text id="b7m4xz"
wait()
waitpid()
```

и обработкой завершившихся дочерних процессов.

Это особенно важно для понимания:

* process lifecycle;
* zombies;
* process managers.

---

# 19. Signals и Zombie Process

Представим:

```text id="j4p8cy"
Parent
   │
   └── Child
          ↓
         exit
```

После завершения ребёнка родитель должен забрать его exit status через `wait()`/`waitpid()`.

Если этого не произошло, процесс может временно находиться в состоянии **zombie**.

`SIGCHLD` позволяет родителю узнать об изменении состояния дочернего процесса.

---

# 20. Signals и Shell

Shell активно использует сигналы для управления foreground-процессами.

Например:

```bash id="w2c7vm"
python server.py
```

Нажимаем:

```text id="z5x8qa"
Ctrl+C
```

Получаем:

```text id="v3m6pn"
SIGINT
```

Другой пример:

```text id="s7n2xd"
Ctrl+Z
   ↓
SIGTSTP
   ↓
процесс приостанавливается
```

Затем:

```bash id="q8c4ml"
fg
```

возвращает job на foreground.

---

# 21. SIGTSTP

`SIGTSTP` обычно возникает при:

```text id="f2m7xk"
Ctrl+Z
```

Он просит процесс остановиться.

В отличие от `SIGSTOP`, `SIGTSTP` можно обработать.

Типичная схема:

```text id="n6q3wp"
Running
   ↓
Ctrl+Z
   ↓
SIGTSTP
   ↓
Stopped
   ↓
fg / bg
   ↓
Running
```

---

# 22. Signals и systemd

Signals особенно важны при управлении сервисами.

Например:

```bash id="h5x8pv"
systemctl stop myapp
```

systemd обычно использует сигнал для штатного завершения процесса, а при необходимости после заданного периода может применить более жёсткое завершение.

Упрощённо:

```text id="m8q2cz"
systemctl stop
      ↓
 SIGTERM
      ↓
graceful shutdown
      ↓
если процесс не завершился
      ↓
SIGKILL
```

Точная последовательность и таймаут зависят от конфигурации unit.

---

# 23. Signals и Docker

Docker также использует сигналы при остановке контейнеров.

Типичная концепция:

```text id="c4n8wy"
docker stop
     ↓
SIGTERM
     ↓
graceful shutdown
     ↓
timeout
     ↓
SIGKILL
```

Поэтому Backend-приложение должно корректно реагировать на `SIGTERM`.

Это особенно важно для:

* FastAPI;
* Gunicorn;
* Celery workers;
* долгих задач;
* database connections.

---

# 24. Graceful Shutdown

**Graceful shutdown** — корректное завершение приложения.

Например:

```text id="e5v9cr"
SIGTERM
   ↓
перестать принимать новую работу
   ↓
дождаться допустимых текущих операций
   ↓
закрыть DB connections
   ↓
закрыть Redis connections
   ↓
flush buffers
   ↓
exit
```

Для Backend это особенно важно.

Если просто использовать:

```text id="x9c2mq"
SIGKILL
```

процесс может быть завершён до выполнения cleanup.

---

# 25. Signals и asyncio

В асинхронном приложении можно зарегистрировать обработчик сигнала через event loop.

Например:

```python id="r3v7km"
import asyncio
import signal

async def main():
    stop_event = asyncio.Event()

    loop = asyncio.get_running_loop()

    loop.add_signal_handler(
        signal.SIGTERM,
        stop_event.set,
    )

    await stop_event.wait()

asyncio.run(main())
```

Схема:

```text id="j8m4qx"
SIGTERM
   ↓
Event Loop
   ↓
asyncio.Event
   ↓
main()
   ↓
graceful shutdown
```

На практике конкретная реализация graceful shutdown часто делегируется серверу приложений, например Gunicorn/Uvicorn.

---

# 26. Signal vs Exception

Это разные механизмы.

### Exception

Возникает внутри выполнения программы:

```python id="a6p9sd"
raise ValueError("invalid value")
```

### Signal

Приходит процессу извне:

```text id="d8q2vn"
another process
      ↓
    signal
      ↓
target process
```

Упрощённо:

```text id="1m5x7c"
Exception → механизм языка/runtime

Signal    → механизм ОС
```

---

# 27. Signal vs Exit Code

Signal и exit code тоже не одно и то же.

Если процесс завершился нормально:

```text id="k3r8yb"
process
   ↓
exit(0)
   ↓
exit code = 0
```

Если процесс был завершён сигналом, shell и некоторые API могут отражать факт signal termination отдельно.

В shell это можно увидеть, например:

```bash id="v7p2md"
command
echo $?
```

Но интерпретация exit status после signal зависит от shell и конкретного сценария; нельзя считать любой ненулевой код просто «номером сигнала».

---

# 28. Полезные команды

### Отправить SIGTERM

```bash id="m9c4xk"
kill -TERM PID
```

### Отправить SIGKILL

```bash id="q7v2nz"
kill -KILL PID
```

### Отправить SIGINT

```bash id="x5r8wp"
kill -INT PID
```

### Остановить процесс

```bash id="a2m6cj"
kill -STOP PID
```

### Продолжить

```bash id="p8y3vr"
kill -CONT PID
```

### Посмотреть сигналы

```bash id="d4n7xs"
kill -l
```

### Найти процесс

```bash id="t6q2km"
pgrep python
```

---

# 29. Практический Backend-сценарий

Есть FastAPI-сервис:

```text id="k9m4vx"
systemd
   ↓
Gunicorn
   ↓
Uvicorn workers
   ↓
FastAPI
```

Разработчик выполняет:

```bash id="u5c8na"
sudo systemctl stop myapp
```

systemd инициирует остановку:

```text id="p3r7md"
SIGTERM
   ↓
Gunicorn
   ↓
workers
   ↓
graceful shutdown
```

Приложение:

```text id="n8v2qx"
1. перестаёт принимать новую работу
2. завершает текущие операции
3. закрывает DB connections
4. закрывает Redis connections
5. завершается
```

Если процесс не завершился в установленный период, systemd может принудительно завершить его.

---

# 30. Главная схема

```text id="y7m3pc"
                SIGNALS
                   │
        ┌──────────┴──────────┐
        ↓                     ↓
    Управление             Уведомление
    процессом              о событии
        │                     │
   ┌────┼────┐          ┌─────┴─────┐
   ↓    ↓    ↓          ↓           ↓
SIGTERM SIGSTOP SIGKILL SIGCHLD    SIGHUP
   │
   ↓
Graceful Shutdown
```

---

## 🎤 Вопросы на собеседовании

### Что такое Signal?

Механизм Unix/Linux для асинхронного уведомления процесса о событии.

### Что делает `kill`?

Отправляет сигнал процессу. Название команды не означает, что она всегда завершает процесс.

### Что происходит при `kill PID`?

По умолчанию процессу отправляется `SIGTERM`.

### Чем `SIGTERM` отличается от `SIGKILL`?

`SIGTERM` можно обработать и использовать для graceful shutdown. `SIGKILL` нельзя перехватить или проигнорировать — ядро принудительно завершает процесс.

### Что происходит при `Ctrl+C`?

Обычно foreground process group получает `SIGINT`.

### Что происходит при `Ctrl+Z`?

Обычно foreground process group получает `SIGTSTP`, после чего процесс приостанавливается.

### Можно ли обработать SIGKILL?

Нет.

### Можно ли обработать SIGSTOP?

Нет.

### Зачем нужен SIGTERM в Backend?

Для graceful shutdown: корректно завершить работу, закрыть соединения, остановить приём новых задач и освободить ресурсы.

### Как отправить SIGTERM?

```bash id="p4z8vn"
kill -TERM PID
```

### Как отправить SIGKILL?

```bash id="r6c2xm"
kill -9 PID
```

### Что такое signal handler?

Функция/обработчик, который выполняется в ответ на определённый сигнал, если этот сигнал допускает обработку.

### Чем Signal отличается от Exception?

Signal — механизм ОС для уведомления процесса извне, Exception — механизм обработки ошибок/событий внутри программы и runtime.

### Как systemd связан с сигналами?

При остановке сервиса systemd использует сигналы для управления процессом. Обычно сначала применяется корректное завершение, а если процесс не завершается в установленный срок, может использоваться принудительное завершение.

### Почему SIGKILL нежелателен для обычной остановки Backend?

Потому что приложение не успевает выполнить graceful shutdown: закрыть соединения, завершить допустимые операции и выполнить cleanup.
