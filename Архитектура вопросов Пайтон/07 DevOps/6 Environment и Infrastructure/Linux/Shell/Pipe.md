# 🔗 Pipe

## 🎤 Короткий ответ

**Pipe (`|`)** — механизм Unix/Linux, который позволяет передать **stdout одного процесса в stdin другого процесса**.

Например:

```bash
ps aux | grep python
```

Здесь:

```text
ps aux
   │
   │ stdout
   ▼
  pipe
   │
   │ stdin
   ▼
grep python
```

Pipe позволяет объединять небольшие программы в цепочки и строить обработку данных без промежуточных файлов.

---

## 🗣️ Ответ на собеседовании

Pipe — это механизм межпроцессного взаимодействия, который позволяет связать стандартный вывод одного процесса со стандартным вводом другого.

В Shell он используется через оператор `|`. Например:

```bash
ps aux | grep python
```

`ps` записывает результат в свой stdout, Shell направляет этот поток в pipe, а `grep` читает данные из stdin.

Pipe обычно является однонаправленным каналом. В Linux анонимный pipe создаётся ядром и имеет файловые дескрипторы для чтения и записи.

Главное отличие pipe от перенаправления `>` в том, что при `>` вывод направляется в файл, а при `|` — непосредственно другому процессу.

Для Backend-разработчика pipe важен при работе с Linux, Shell, процессами, логами, CI/CD и Unix-инструментами.

---

## 🧭 Где я нахожусь

```text
07 DevOps
└── 06 Environment и Infrastructure
    ├── Linux
    ├── Файловая система
    ├── Процессы
    ├── Права доступа
    ├── Bash
    │   └── Shell
    │       ├── stdin / stdout / stderr
    │       └── Pipe ← Я здесь
    ├── SSH
    ├── Сеть
    └── systemd
```

---

# 📚 Разбор поглубже

## 1. Что такое Pipe

В Linux pipe — это механизм передачи данных между процессами.

Упрощённая модель:

```text
Процесс A
   │
   │ write()
   ▼
┌─────────────┐
│    Pipe     │
│   buffer    │
└─────────────┘
   │
   │ read()
   ▼
Процесс B
```

Один процесс пишет данные в pipe, другой читает их.

---

# 2. Pipe в Shell

Самый известный синтаксис:

```bash
command1 | command2
```

Например:

```bash
ls -la | grep ".py"
```

Логика:

```text
ls -la
   ↓
 stdout
   ↓
  pipe
   ↓
 stdin
   ↓
grep ".py"
```

`grep` не получает вывод `ls` через файл. Данные передаются через pipe.

---

# 3. Что делает Shell

Когда Shell видит:

```bash
ps aux | grep python
```

он организует процессы и их стандартные потоки примерно так:

```text
             ┌─────────────┐
             │   ps aux    │
             └──────┬──────┘
                    │
                  stdout
                    │
                    ▼
               ┌─────────┐
               │  pipe   │
               └────┬────┘
                    │
                  stdin
                    │
                    ▼
             ┌─────────────┐
             │ grep python │
             └─────────────┘
```

То есть Shell не «передаёт текст сам».

Он **настраивает файловые дескрипторы процессов так, чтобы stdout одного был соединён с stdin другого**.

---

# 4. Pipe и файловые дескрипторы

В Linux стандартные файловые дескрипторы:

```text
0 → stdin
1 → stdout
2 → stderr
```

Для pipeline:

```bash
command1 | command2
```

получается концептуально:

```text
command1
    FD 1 ─────→ pipe ─────→ FD 0
                              command2
```

То есть:

```text
stdout command1
        ↓
      pipe
        ↓
stdin command2
```

Это важная связь между темами:

```text
Shell
  ↓
File Descriptors
  ↓
stdin/stdout
  ↓
Pipe
  ↓
Processes
```

---

# 5. Pipe — не файл

Pipe похож на поток данных, но это не обычный файл на диске.

При:

```bash
ps aux | grep python
```

результат `ps` не сохраняется в файл.

Он проходит через механизм pipe и читается `grep`.

```text
ps
 ↓
pipe buffer
 ↓
grep
```

После завершения процессов этот конкретный anonymous pipe исчезает.

---

# 6. Pipe vs Redirect

Это важное различие.

### Redirect

```bash
ls > files.txt
```

Поток:

```text
ls
 ↓
stdout
 ↓
file
```

### Pipe

```bash
ls | grep ".py"
```

Поток:

```text
ls
 ↓
stdout
 ↓
pipe
 ↓
stdin
 ↓
grep
```

| Механизм | Куда идут данные         |
| -------- | ------------------------ |
| `>`      | В файл                   |
| `>>`     | В конец файла            |
| `<`      | Из файла в stdin         |
| `\|`     | В stdin другого процесса |

---

# 7. Несколько Pipe

Pipe можно объединять в цепочку:

```bash
cat access.log | grep ERROR | wc -l
```

Получается:

```text
cat
 ↓
pipe
 ↓
grep
 ↓
pipe
 ↓
wc
```

Или:

```text
access.log
     ↓
    cat
     ↓
   grep ERROR
     ↓
     wc -l
     ↓
   количество
```

Каждая программа выполняет свою небольшую задачу.

---

# 8. Пример с логами

Допустим, есть:

```text
access.log
```

Нужно найти ошибки:

```bash
grep ERROR access.log
```

Посчитать их:

```bash
grep ERROR access.log | wc -l
```

Получается:

```text
grep
 ↓
найти строки ERROR
 ↓
pipe
 ↓
wc -l
 ↓
посчитать строки
```

Это типичный Unix-подход:

> одна программа делает одну задачу, а pipe позволяет объединять программы.

---

# 9. Pipe и stdin

Вторая команда получает данные через **stdin**.

Например:

```bash
echo "hello" | grep hello
```

`echo` пишет:

```text
hello
```

в stdout.

Shell направляет stdout в pipe.

`grep` читает это через stdin.

```text
echo
 │
 │ stdout
 ▼
pipe
 │
 │ stdin
 ▼
grep
```

---

# 10. Pipe и stdout/stderr

По умолчанию оператор:

```bash
|
```

соединяет именно **stdout**, а не stderr.

Например:

```bash
command | grep error
```

означает:

```text
stdout command → stdin grep
```

Но stderr остаётся отдельным потоком.

Если нужно направить и stderr:

```bash
command 2>&1 | grep error
```

Схема:

```text
stdout ───────┐
              ├──→ pipe → grep
stderr ──┐    │
         └────┘
```

В Bash также можно использовать:

```bash
command |& grep error
```

Это сокращённая форма для pipeline со stdout и stderr.

---

# 11. Pipe — однонаправленный

Классический anonymous pipe предназначен для передачи данных в одном направлении:

```text
A
 │
 ▼
Pipe
 │
 ▼
B
```

Если двум процессам нужна двусторонняя связь, обычно используют другие IPC-механизмы или создают два pipe:

```text
A ─────→ Pipe 1 ─────→ B
A ←───── Pipe 2 ←───── B
```

Для более сложного IPC существуют:

* Unix domain sockets;
* socketpair;
* named pipes;
* очереди;
* shared memory.

---

# 12. Anonymous Pipe

Pipe, созданный непосредственно процессом через системный вызов, обычно называют **anonymous pipe**.

В Unix/Linux он часто создаётся через:

```c
pipe()
```

Ядро возвращает два файловых дескриптора:

```text
fd[0] → чтение
fd[1] → запись
```

Концептуально:

```text
fd[1]
  │
  │ write
  ▼
┌───────────┐
│   pipe    │
└───────────┘
  │
  │ read
  ▼
fd[0]
```

---

# 13. Named Pipe — FIFO

В Linux существует также **named pipe**, или FIFO.

Он имеет имя в файловой системе.

Создать:

```bash
mkfifo mypipe
```

После этого:

```bash
echo "hello" > mypipe
```

а в другом процессе:

```bash
cat mypipe
```

Получается:

```text
Process A
   │
   │ write
   ▼
mypipe (FIFO)
   │
   │ read
   ▼
Process B
```

Главное отличие:

```text
Anonymous pipe → обычно используется между связанными процессами

Named pipe → имеет имя и может использоваться независимыми процессами
```

---

# 14. Что происходит при заполнении Pipe

Pipe имеет ограниченный буфер в памяти ядра.

Если writer пишет быстрее, чем reader читает:

```text
Writer
  ↓↓↓↓↓
┌───────────┐
│   buffer  │ ← заполняется
└───────────┘
      ↓
    Reader
```

Когда буфер заполнен, запись может **заблокироваться**, пока reader не освободит место.

Это важное свойство IPC.

Аналогично, если reader пытается читать из пустого pipe, поведение зависит от режима blocking/non-blocking.

---

# 15. Blocking

По умолчанию операции с pipe могут быть блокирующими.

Например:

```text
Writer
  ↓
Pipe заполнен
  ↓
write() блокируется
  ↓
Reader читает данные
  ↓
появляется место
  ↓
Writer продолжает работу
```

Поэтому pipe — не просто «буфер между командами», а полноценный механизм синхронизации потоков данных между процессами.

---

# 16. EOF в Pipe

Если все файловые дескрипторы записи закрыты, reader в итоге получает **EOF** после прочтения оставшихся данных.

Упрощённо:

```text
Writer
   │
   │ close(write_fd)
   ▼
 Pipe
   │
   │ EOF
   ▼
Reader
```

Это позволяет читающему процессу понять:

> больше данных не будет.

---

# 17. Pipe в Python

Python также предоставляет интерфейсы для работы с pipe.

Например, низкоуровневый:

```python
import os

read_fd, write_fd = os.pipe()

os.write(write_fd, b"hello")

data = os.read(read_fd, 5)

print(data)
```

Здесь:

```text
os.pipe()
   ↓
read_fd
write_fd
```

`write_fd` используется для записи, `read_fd` — для чтения.

---

# 18. Pipe и `subprocess`

Для Backend-разработчика более практичен модуль `subprocess`.

Например:

```python
import subprocess

result = subprocess.run(
    ["ls", "-la"],
    capture_output=True,
    text=True,
)

print(result.stdout)
```

Python получает stdout процесса.

Можно строить pipeline между subprocess:

```python
import subprocess

ps = subprocess.Popen(
    ["ps", "aux"],
    stdout=subprocess.PIPE,
    text=True,
)

grep = subprocess.Popen(
    ["grep", "python"],
    stdin=ps.stdout,
    stdout=subprocess.PIPE,
    text=True,
)

output, _ = grep.communicate()

print(output)
```

Концептуально это тот же механизм:

```text
ps aux
  ↓
PIPE
  ↓
grep python
```

---

# 19. Pipe vs Queue

Оба механизма могут использоваться для IPC, но предназначены для разных моделей.

### Pipe

Поток данных:

```text
A → B
```

### Queue

Очередь сообщений/объектов:

```text
Producer
   ↓
 Queue
   ↓
Consumer
```

Для Python multiprocessing:

```python
from multiprocessing import Queue
```

Queue обычно удобнее, когда нужно передавать отдельные сообщения или задачи между процессами.

---

# 20. Pipe vs Socket

### Pipe

Обычно используется для IPC между процессами одной машины:

```text
Process A
   ↓
 Pipe
   ↓
Process B
```

### Socket

Может использоваться:

```text
Process A
   ↓
Socket
   ↓
Process B
```

и процессы могут находиться на разных машинах:

```text
Server
  ↓
Network
  ↓
Client
```

Для локального IPC также существуют Unix domain sockets.

---

# 21. Pipe и Shell — главное различие

Нужно различать:

```text
Pipe как механизм ОС
```

и:

```text
| как оператор Shell
```

Когда мы пишем:

```bash
ps aux | grep python
```

`|` — это **синтаксис Shell**, который говорит оболочке:

> соедини stdout первой команды со stdin второй.

Сам pipe реализуется средствами операционной системы.

---

# 22. Практический пример

Команда:

```bash
journalctl -u myapp | grep ERROR | tail -20
```

Создаёт pipeline:

```text
journalctl
    │
    │ stdout
    ▼
   pipe
    │
    ▼
grep ERROR
    │
    │ stdout
    ▼
   pipe
    │
    ▼
 tail -20
```

Результат:

> взять логи `myapp`, оставить строки с `ERROR` и показать последние 20.

Это очень типичный workflow при диагностике Backend-сервисов.

---

# 23. Типичные ошибки

### Ошибка 1 — считать pipe файлом

Pipe — это канал IPC с буфером в ядре, а не обычный файл на диске.

### Ошибка 2 — считать `|` перенаправлением в файл

```bash
command > file
```

и:

```bash
command | other_command
```

— разные механизмы.

### Ошибка 3 — думать, что pipe передаёт stderr

Обычный:

```bash
command | grep error
```

передаёт только stdout.

### Ошибка 4 — считать pipe двусторонним

Классический pipe — однонаправленный.

### Ошибка 5 — путать pipe и socket

Pipe — механизм IPC; socket — более универсальный механизм коммуникации, в том числе по сети.

---

# 🎤 Вопросы на собеседовании

### Что такое Pipe?

Механизм IPC, который позволяет передавать поток данных между процессами. В Shell оператор `|` соединяет stdout одного процесса со stdin другого.

### Что делает `|`?

```bash
command1 | command2
```

Направляет stdout `command1` в stdin `command2`.

### Что находится между процессами?

В Linux pipe реализуется ядром и имеет буфер для передаваемых данных.

### Pipe — это файл?

Нет. Он представлен файловыми дескрипторами и управляется ядром, но не является обычным файлом на диске.

### Какие файловые дескрипторы используются?

```text
0 → stdin
1 → stdout
2 → stderr
```

В pipeline обычно:

```text
stdout первого → pipe → stdin второго
```

### Передаёт ли `|` stderr?

Нет, обычный `|` передаёт stdout.

Для объединения stderr со stdout:

```bash
command 2>&1 | grep error
```

или в Bash:

```bash
command |& grep error
```

### Что произойдёт, если pipe заполнится?

Writer может заблокироваться до тех пор, пока reader не прочитает часть данных и не освободит место в буфере.

### Что такое anonymous pipe?

Pipe без имени, обычно создаваемый процессами для организации IPC, часто между связанными процессами.

### Что такое FIFO?

Named pipe — именованный pipe, который существует как объект файловой системы и позволяет взаимодействовать независимым процессам.

### Чем Pipe отличается от Queue?

Pipe обычно представляет поток данных между процессами, а Queue предоставляет модель очереди сообщений/объектов.

### Чем Pipe отличается от Socket?

Pipe обычно используется для IPC на одной машине, а socket может использоваться как локально, так и для сетевого взаимодействия.

### Что происходит при `ps aux | grep python`?

Shell создаёт pipeline, соединяя stdout процесса `ps` с stdin процесса `grep` через pipe.

```text
ps aux
  │
  ▼
pipe
  │
  ▼
grep python
```
