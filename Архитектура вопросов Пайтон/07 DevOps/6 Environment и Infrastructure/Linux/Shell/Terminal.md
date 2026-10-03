# 🖥️ Terminal

## 🎤 Короткий ответ

**Terminal (терминал)** — это интерфейс, через который пользователь взаимодействует с командной строкой и Shell.

Сам Terminal **не выполняет команды**. Он принимает ввод с клавиатуры, передаёт его Shell и отображает результат.

Упрощённо:

```text
Пользователь
     ↓
 Terminal
     ↓
  Shell
     ↓
 Linux / программы
```

Например, когда мы открываем Terminal и вводим:

```bash
ls
```

Terminal передаёт текст Shell, Shell запускает `ls`, а Terminal отображает полученный вывод.

---

## 🗣️ Ответ на собеседовании

Terminal — это интерфейс ввода-вывода, через который пользователь взаимодействует с командной оболочкой.

Важно отличать Terminal от Shell. Terminal отвечает за отображение текста и передачу пользовательского ввода, а Shell интерпретирует команды и запускает программы.

Например, при выполнении `ls` Terminal передаёт введённые символы Shell. Shell находит программу `ls`, запускает её, а stdout процесса возвращается через терминальную сессию и отображается пользователю.

Исторически терминал был физическим устройством, а современные Terminal.app, iTerm2 и Linux Terminal — программные терминальные эмуляторы.

При SSH ситуация похожая: локальный Terminal подключается к удалённой системе, а на сервере запускается Shell.

---

## 🧭 Где я нахожусь

```text
07 DevOps
└── 06 Environment и Infrastructure
    ├── Linux
    │   ├── Файловая система
    │   ├── Процессы
    │   ├── Права доступа
    │   └── Сеть
    ├── Terminal ← Я здесь
    │   └── Shell
    │       ├── Bash
    │       ├── stdin / stdout / stderr
    │       └── Pipe
    ├── SSH
    └── systemd
```

---

# 📚 Разбор поглубже

## 1. Что такое Terminal

Терминал предоставляет текстовый интерфейс для взаимодействия с системой.

Современный Terminal обычно является **terminal emulator** — программой, которая эмулирует поведение классического терминального устройства.

Примеры:

* Terminal.app в macOS;
* iTerm2 в macOS;
* GNOME Terminal;
* Konsole;
* Windows Terminal.

Схема:

```text
┌──────────────────────┐
│   Terminal Emulator  │
│                      │
│ $ python app.py      │
│ Application started  │
│ $                    │
└──────────┬───────────┘
           │
           ▼
         Shell
```

---

# 2. Terminal vs Shell

Это главное различие.

### Terminal

Отвечает за:

* ввод с клавиатуры;
* отображение текста;
* обработку терминальных управляющих последовательностей;
* взаимодействие с terminal session.

### Shell

Отвечает за:

* интерпретацию команд;
* переменные;
* `PATH`;
* pipes;
* redirection;
* запуск программ;
* shell scripts;
* условия и циклы.

Например:

```text
Terminal
   │
   │ "ls -la"
   ▼
Bash
   │
   │ запускает
   ▼
ls
   │
   │ stdout
   ▼
Terminal
```

---

# 3. Terminal не является Shell

Когда пользователь открывает приложение Terminal, внутри него обычно запускается Shell.

Например:

```text
Terminal.app
      ↓
     zsh
      ↓
     bash
```

Но это не означает, что Terminal и Bash — одна программа.

Можно сменить Shell, не меняя Terminal.

Например, один и тот же Terminal может запускать:

```text
bash
zsh
fish
```

---

# 4. Terminal и операционная система

Terminal также не является Linux.

Можно представить уровни:

```text
Пользователь
     ↓
Terminal Emulator
     ↓
Shell
     ↓
System Calls / OS
     ↓
Kernel
     ↓
Hardware
```

Например, команда:

```bash
ls
```

в итоге приводит к взаимодействию программы `ls` с ядром Linux через системные вызовы.

---

# 5. Историческое значение Terminal

Исторически терминал был физическим устройством.

Например:

```text
Компьютер
    │
    │
    ▼
┌─────────────┐
│ Терминал    │
│ клавиатура  │
│ экран       │
└─────────────┘
```

Он позволял удалённо или локально взаимодействовать с компьютером.

Современный Terminal — программная эмуляция такого устройства.

Поэтому термин **terminal emulator** точнее описывает современные приложения вроде Terminal.app.

---

# 6. TTY

В Linux терминальная инфраструктура тесно связана с понятием **TTY**.

Исторически TTY — сокращение от **teletypewriter**.

Сегодня TTY может обозначать терминальное устройство или соответствующий интерфейс.

Посмотреть текущий терминал:

```bash
tty
```

Например:

```text
/dev/ttys001
```

или в Linux:

```text
/dev/pts/0
```

---

# 7. Pseudo-Terminal — PTY

При работе с современным Terminal или SSH часто используется **pseudo-terminal (PTY)**.

Упрощённо:

```text
Terminal Emulator
        │
        ▼
      PTY
        │
        ▼
      Shell
```

PTY позволяет программам вести себя так, будто они работают с настоящим терминальным устройством.

Это важно, например, для:

* интерактивного Bash;
* SSH;
* `vim`;
* `top`;
* интерактивных Python-сессий.

---

# 8. Terminal и stdin/stdout/stderr

Terminal обычно связан со стандартными потоками процесса Shell:

```text
stdin   → ввод с клавиатуры
stdout  → обычный вывод
stderr  → ошибки
```

Упрощённо:

```text
Keyboard
   ↓
stdin
   ↓
Shell
   ↓
Process
   ↓
stdout/stderr
   ↓
Terminal
   ↓
Screen
```

Например:

```bash
python app.py
```

Python может читать stdin:

```python
name = input("Name: ")
```

а результат выводить в stdout:

```python
print("Hello")
```

---

# 9. Что происходит при вводе команды

Допустим, пользователь вводит:

```bash
echo "Hello"
```

Происходит примерно следующее:

```text
1. Keyboard
      ↓
2. Terminal emulator
      ↓
3. stdin / PTY
      ↓
4. Shell
      ↓
5. Shell разбирает команду
      ↓
6. запускается echo
      ↓
7. stdout
      ↓
8. Terminal emulator
      ↓
9. экран
```

---

# 10. Terminal и Pipe

Pipe связывает **процессы**, а Terminal обычно является конечной точкой пользовательского ввода/вывода.

Например:

```bash
ps aux | grep python
```

Здесь:

```text
Terminal
   ↓
Shell
   ↓
ps ──→ pipe ──→ grep
                 ↓
              stdout
                 ↓
              Terminal
```

Terminal сам не является pipe.

Shell организует pipe между процессами.

---

# 11. Terminal и SSH

При SSH локальный Terminal может использоваться для работы с удалённым сервером:

```text
Local Computer
┌─────────────┐
│  Terminal   │
└──────┬──────┘
       │
       │ SSH
       ▼
┌─────────────┐
│ Linux Server│
│    Shell    │
└─────────────┘
```

Например:

```bash
ssh user@server
```

После подключения пользователь получает удалённую shell-сессию.

При этом Terminal находится локально, а Shell и выполняемые команды — на сервере.

---

# 12. Terminal и SSH без интерактивного Shell

SSH не обязательно запускает интерактивный shell.

Например:

```bash
ssh user@server "systemctl status nginx"
```

Здесь команда выполняется на удалённой машине, а результат передаётся обратно в локальный Terminal.

Схема:

```text
Local Terminal
      ↓
      SSH
      ↓
Remote command
      ↓
stdout
      ↓
SSH
      ↓
Local Terminal
```

---

# 13. Terminal и GUI

Terminal — это **CLI (Command-Line Interface)**.

В отличие от GUI:

```text
GUI
→ кнопки
→ окна
→ меню
→ мышь

CLI
→ команды
→ аргументы
→ stdin/stdout
```

Для Backend-разработчика CLI особенно важен при работе с:

* Linux;
* Git;
* Docker;
* SSH;
* systemd;
* PostgreSQL;
* CI/CD.

---

# 14. Terminal и shell history

История команд обычно является функцией Shell, а не самого Terminal.

Например, Bash/Zsh позволяют:

```bash
history
```

или использовать:

```text
↑
↓
```

для перехода между предыдущими командами.

То есть:

```text
Terminal
→ принимает нажатие клавиши

Shell
→ обрабатывает историю команд
```

---

# 15. Terminal и Ctrl+C

Когда пользователь нажимает:

```text
Ctrl+C
```

Terminal/TTY-механизм передаёт соответствующий сигнал foreground process group, обычно приводящий к `SIGINT`.

Например:

```bash
python server.py
```

запущен в foreground.

Пользователь нажимает:

```text
Ctrl+C
```

и процесс обычно получает:

```text
SIGINT
```

Это важно:

> `Ctrl+C` — не просто «кнопка остановки программы». Это механизм терминального управления, связанный с сигналами Unix.

---

# 16. Foreground и Background

Terminal может иметь foreground process group.

Например:

```bash
python server.py
```

процесс работает в foreground:

```text
Terminal
   ↓
Shell
   ↓
python
```

Если выполнить:

```bash
python server.py &
```

процесс запускается в background:

```text
Terminal
   ↓
Shell
   ├── foreground
   └── python → background
```

Управлять jobs можно командами:

```bash
jobs
fg
bg
```

---

# 17. Terminal и цвета

Когда команда выводит цветной текст:

```bash
ls --color=auto
```

Terminal отображает ANSI escape sequences.

Программа отправляет управляющие последовательности, а terminal emulator интерпретирует их.

Например:

```text
stdout
   ↓
ANSI escape sequences
   ↓
Terminal emulator
   ↓
цветной текст
```

Поэтому Terminal умеет отображать:

* цвета;
* курсор;
* очистку экрана;
* перемещение курсора;
* изменение стилей текста.

---

# 18. Terminal и псевдографика

Программы вроде:

```bash
top
htop
vim
nano
```

не просто выводят текст сверху вниз.

Они взаимодействуют с terminal emulator как с интерактивным устройством:

```text
keyboard
   ↓
Terminal
   ↓
PTY
   ↓
program
   ↓
PTY
   ↓
Terminal
   ↓
screen
```

Поэтому такие программы требуют терминальной среды.

---

# 19. Terminal vs Console

Термины часто используются неточно.

В общем случае:

```text
Terminal
→ интерфейс ввода/вывода

Console
→ может обозначать системную/локальную консоль или терминал в более широком смысле

Shell
→ командный интерпретатор
```

В повседневной разработке слово **terminal** обычно означает приложение или терминальную сессию, в которой пользователь работает с CLI.

---

# 20. Практический пример для Backend

Представим, что на Linux-сервере работает FastAPI.

Разработчик подключается:

```bash
ssh user@server
```

Дальше в Terminal:

```bash
cd /opt/myapp
```

Проверяет процесс:

```bash
ps aux | grep gunicorn
```

Проверяет порт:

```bash
ss -ltnp | grep 8000
```

Проверяет API:

```bash
curl http://localhost:8000
```

Смотрит systemd-логи:

```bash
journalctl -u myapp -f
```

Здесь Terminal является интерфейсом, а реальные действия выполняются Shell, системными утилитами и ядром Linux.

---

# 21. Ключевое различие всех понятий

Полезно держать в голове такую цепочку:

```text
Terminal
   ↓
предоставляет интерфейс
   ↓
Shell
   ↓
интерпретирует команды
   ↓
Process
   ↓
выполняет программу
   ↓
Kernel
   ↓
управляет ресурсами
```

Например:

```text
Terminal
   ↓
Bash
   ↓
python app.py
   ↓
Python process
   ↓
Linux Kernel
```

---

## 🎤 Вопросы на собеседовании

### Что такое Terminal?

Программный или физический интерфейс ввода-вывода, через который пользователь взаимодействует с системой. В современных ОС обычно речь идёт о terminal emulator.

### Чем Terminal отличается от Shell?

**Terminal** предоставляет интерфейс ввода/вывода, а **Shell** интерпретирует команды и запускает программы.

```text
Terminal
   ↓
Shell
   ↓
Program
```

### Что такое terminal emulator?

Программа, которая эмулирует поведение терминального устройства. Например, Terminal.app или GNOME Terminal.

### Что такое TTY?

Исторически — teleprinter/terminal interface; в Unix/Linux терминальное устройство или соответствующий интерфейс.

### Что такое PTY?

**Pseudo-Terminal** — программная терминальная пара, позволяющая процессу работать так, будто он взаимодействует с терминальным устройством.

### Что происходит при `ssh user@server`?

SSH устанавливает соединение с сервером и обычно создаёт удалённую shell-сессию. Локальный Terminal остаётся интерфейсом пользователя.

### Что происходит при `Ctrl+C`?

Foreground process group обычно получает сигнал `SIGINT`.

### Является ли Terminal операционной системой?

Нет.

### Является ли Terminal Shell?

Нет.

### Что такое CLI?

**Command-Line Interface** — интерфейс взаимодействия с системой через текстовые команды.

### Где находится Shell относительно Terminal?

Обычно:

```text
Terminal Emulator
       ↓
     Shell
       ↓
   Programs
```

### Почему Terminal важен Backend-разработчику?

Потому что через Terminal постоянно выполняются команды для работы с Linux, Git, Docker, SSH, systemd, базами данных, процессами и CI/CD.
