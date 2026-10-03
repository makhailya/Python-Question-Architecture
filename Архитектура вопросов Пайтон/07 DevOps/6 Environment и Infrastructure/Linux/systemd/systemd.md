# ⚙️ systemd

## 🎤 Короткий ответ

**systemd** — это система инициализации и менеджер служб в Linux. Она запускается одной из первых после загрузки ядра и управляет жизненным циклом системных сервисов: запуском, остановкой, перезапуском и автозапуском.

Основной объект systemd — **unit**. Для сервисов используется `service unit`, описываемый файлом `.service`.

Основные команды:

```bash
systemctl start nginx
systemctl stop nginx
systemctl restart nginx
systemctl status nginx
systemctl enable nginx
```

То есть systemd отвечает за то, **какие сервисы запущены, когда они запускаются и как ими управлять**.

---

## 🗣️ Ответ на собеседовании

`systemd` — это система инициализации и менеджер служб в Linux. После загрузки ядра Linux запускает `systemd` как основной процесс системы, обычно с PID 1.

Он управляет различными системными объектами, которые называются **units**. Например, `.service` используется для служб, `.socket` — для socket activation, `.timer` — для задач по расписанию, `.mount` — для точек монтирования.

Для управления units используется `systemctl`. Например, `systemctl start nginx` запускает сервис, `stop` останавливает, `restart` перезапускает, `status` показывает состояние.

Отдельно важно различать `start` и `enable`: `start` запускает сервис сейчас, а `enable` добавляет его в конфигурацию автозапуска при загрузке системы.

Для Backend-разработчика systemd важен потому, что Python-приложение, Nginx, PostgreSQL, Redis и другие компоненты сервера могут работать как systemd-сервисы. systemd также может автоматически перезапускать упавшие сервисы, управлять зависимостями и собирать их логи через journald.

---

## 🧭 Где я нахожусь

```text
07 DevOps
└── 06 Environment и Infrastructure
    ├── Linux
    │   ├── Файловая система
    │   ├── Процессы
    │   ├── Права доступа
    │   ├── Bash
    │   └── SSH
    ├── Сеть
    └── systemd ← Я здесь
        ├── systemctl
        ├── Units
        ├── Services
        ├── Targets
        ├── Dependencies
        ├── Journald
        └── Service lifecycle
```

---

# 📚 Разбор поглубже

## 1. Что такое systemd

`systemd` выполняет несколько связанных задач:

* инициализация системы;
* запуск системных служб;
* остановка служб;
* управление зависимостями;
* автоматический перезапуск;
* управление ресурсами процессов;
* ведение журналов через `journald`;
* запуск задач по времени;
* управление mount points и sockets.

Главная идея:

```text
Linux Kernel
     ↓
systemd (PID 1)
     ↓
┌──────────────┬──────────────┬──────────────┐
│    nginx     │  postgresql  │   myapp      │
└──────────────┴──────────────┴──────────────┘
```

---

# 2. systemd и PID 1

После загрузки ядра Linux запускается первый userspace-процесс.

В системе с systemd:

```text
PID 1
  ↓
systemd
```

Проверить:

```bash
ps -p 1 -o pid,comm,args
```

или:

```bash
systemctl status
```

PID 1 имеет особое значение:

* запускает и контролирует системные сервисы;
* управляет процессами;
* участвует в обработке завершившихся процессов;
* обеспечивает переход системы между состояниями.

---

# 3. Unit

**Unit** — объект, которым управляет systemd.

Существуют разные типы units:

```text
.service  → служба
.socket   → socket
.target   → группа/состояние units
.timer    → запуск по времени
.mount    → mount point
.path     → мониторинг пути
.device   → устройство
```

Посмотреть units:

```bash
systemctl list-units
```

Посмотреть установленные unit-файлы:

```bash
systemctl list-unit-files
```

---

# 4. Service

Для Backend-разработчика наиболее важен тип:

```text
.service
```

Например:

```text
nginx.service
postgresql.service
redis.service
myapp.service
```

Unit-файл определяет:

* что запускать;
* от какого пользователя;
* рабочую директорию;
* переменные окружения;
* зависимости;
* что делать при падении;
* когда запускать.

---

# 5. Пример service-файла

Например, Python-приложение:

```ini
[Unit]
Description=My FastAPI Application
After=network.target

[Service]
User=app
WorkingDirectory=/opt/myapp
ExecStart=/opt/myapp/.venv/bin/python app.py
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

Основные секции:

```text
[Unit]
    общая информация и зависимости

[Service]
    параметры запуска процесса

[Install]
    правила подключения unit к target
```

---

# 6. `systemctl`

Основной инструмент управления systemd:

```bash
systemctl
```

### Запустить

```bash
sudo systemctl start nginx
```

### Остановить

```bash
sudo systemctl stop nginx
```

### Перезапустить

```bash
sudo systemctl restart nginx
```

### Проверить статус

```bash
systemctl status nginx
```

---

# 7. `start` vs `enable`

Это один из частых вопросов на собеседовании.

### `start`

Запускает сервис **сейчас**:

```bash
sudo systemctl start myapp
```

После перезагрузки это само по себе не гарантирует автоматический запуск.

### `enable`

Настраивает запуск сервиса при соответствующем этапе загрузки системы:

```bash
sudo systemctl enable myapp
```

Можно сделать оба действия:

```bash
sudo systemctl enable --now myapp
```

Это означает:

```text
enable → добавить в автозапуск
now    → запустить прямо сейчас
```

---

# 8. `disable`

Отключает автозапуск:

```bash
sudo systemctl disable myapp
```

При этом уже работающий сервис обычно не останавливается.

Чтобы одновременно отключить и остановить:

```bash
sudo systemctl disable --now myapp
```

---

# 9. `status`

Очень важная команда для диагностики:

```bash
systemctl status myapp
```

Она показывает:

* загружен ли unit;
* активен ли сервис;
* PID процесса;
* время запуска;
* последние сообщения журнала;
* статус завершения.

Например:

```text
Active: active (running)
```

означает, что сервис сейчас работает.

Возможные состояния:

```text
active
inactive
failed
activating
deactivating
```

---

# 10. `restart` vs `reload`

### restart

Полностью перезапускает процесс:

```bash
sudo systemctl restart nginx
```

Условно:

```text
process
   ↓
stop
   ↓
start
```

### reload

Просит сервис перечитать конфигурацию без полного перезапуска процесса:

```bash
sudo systemctl reload nginx
```

Но это зависит от самого сервиса — он должен поддерживать reload.

---

# 11. `daemon-reload`

Очень важно не путать:

```bash
systemctl reload
```

и:

```bash
systemctl daemon-reload
```

Если изменили unit-файл:

```bash
sudo nano /etc/systemd/system/myapp.service
```

systemd нужно сообщить, что конфигурация unit-файлов изменилась:

```bash
sudo systemctl daemon-reload
```

После этого можно:

```bash
sudo systemctl restart myapp
```

То есть:

```text
Изменили .service
      ↓
daemon-reload
      ↓
restart сервиса
```

---

# 12. `ExecStart`

Определяет команду запуска процесса.

Например:

```ini
[Service]
ExecStart=/opt/myapp/.venv/bin/gunicorn \
    -k uvicorn.workers.UvicornWorker \
    app:app
```

systemd запускает именно этот процесс.

---

# 13. Пользователь процесса

Сервис необязательно запускать от `root`.

Например:

```ini
[Service]
User=app
Group=app
```

Это важно с точки зрения безопасности.

Вместо:

```text
systemd
   ↓
root
   ↓
Python application
```

можно:

```text
systemd
   ↓
app user
   ↓
Python application
```

Если приложение будет скомпрометировано, права процесса будут ограничены правами пользователя `app`.

---

# 14. WorkingDirectory

Можно определить рабочую директорию:

```ini
[Service]
WorkingDirectory=/opt/myapp
```

Тогда относительные пути процесса будут вычисляться относительно неё.

Например:

```text
/opt/myapp
├── app.py
├── config/
└── logs/
```

---

# 15. Переменные окружения

В unit можно задать environment variables:

```ini
[Service]
Environment="APP_ENV=production"
Environment="PORT=8000"
```

Также можно использовать отдельный файл:

```ini
[Service]
EnvironmentFile=/etc/myapp/environment
```

Например:

```text
APP_ENV=production
PORT=8000
```

Для секретов в production обычно следует продумывать отдельный механизм управления секретами, а не бездумно хранить их прямо в unit-файле.

---

# 16. Restart

systemd может автоматически перезапускать процесс.

Например:

```ini
[Service]
Restart=on-failure
```

Если приложение аварийно завершилось:

```text
myapp
  ↓
crash
  ↓
systemd
  ↓
restart
  ↓
myapp
```

Другой вариант:

```ini
Restart=always
```

Но политика перезапуска должна соответствовать поведению конкретного сервиса.

---

# 17. Зависимости

systemd умеет определять порядок запуска.

Например:

```ini
[Unit]
After=network.target
```

Это означает, что данный unit должен запускаться после указанного target.

Важно различать:

```text
After=
Requires=
Wants=
```

### `After=`

Определяет **порядок** запуска/остановки.

### `Requires=`

Создаёт зависимость между units.

### `Wants=`

Выражает более мягкую зависимость.

Ключевой момент:

> `After=` само по себе не означает, что зависимый unit будет запущен.

---

# 18. Targets

**Target** — специальный тип unit, используемый для группировки units и описания состояния системы.

Например:

```text
multi-user.target
graphical.target
```

Упрощённо:

```text
multi-user.target
      │
      ├── ssh.service
      ├── nginx.service
      ├── cron.service
      └── ...
```

`WantedBy=multi-user.target` в `[Install]` означает, что сервис можно привязать к этому target при `enable`.

---

# 19. Journald

В systemd есть компонент:

```text
systemd-journald
```

Он собирает системные журналы.

Посмотреть логи сервиса:

```bash
journalctl -u myapp
```

Последние сообщения:

```bash
journalctl -u myapp -n 50
```

Следить в реальном времени:

```bash
journalctl -u myapp -f
```

Логи за текущую загрузку:

```bash
journalctl -b
```

---

# 20. Диагностика упавшего сервиса

Представим:

```bash
sudo systemctl restart myapp
```

и приложение сразу падает.

Проверяем:

```bash
systemctl status myapp
```

Затем:

```bash
journalctl -u myapp -n 100
```

Можно смотреть в реальном времени:

```bash
journalctl -u myapp -f
```

Типичный workflow:

```text
systemctl status
        ↓
journalctl
        ↓
найти ошибку
        ↓
исправить конфигурацию/приложение
        ↓
daemon-reload при изменении unit
        ↓
restart
```

---

# 21. Где лежат unit-файлы

Типичные директории:

```text
/etc/systemd/system/
```

— локальные/admin-created units.

```text
/usr/lib/systemd/system/
```

или на некоторых дистрибутивах:

```text
/lib/systemd/system/
```

— units, установленные пакетами.

Для своего сервиса часто используют:

```text
/etc/systemd/system/myapp.service
```

---

# 22. `systemctl cat`

Посмотреть содержимое unit:

```bash
systemctl cat nginx
```

Это удобно, когда нужно понять, **как именно systemd запускает сервис**.

Также:

```bash
systemctl show nginx
```

показывает свойства unit в более машинно-ориентированном виде.

---

# 23. systemd и Python Backend

Например, есть FastAPI-приложение:

```text
/opt/myapp
├── .venv
├── app
└── ...
```

Можно запускать его через Gunicorn/Uvicorn Worker:

```text
systemd
   ↓
Gunicorn
   ↓
Uvicorn workers
   ↓
FastAPI
```

Пример:

```ini
[Unit]
Description=FastAPI application
After=network.target

[Service]
User=app
Group=app
WorkingDirectory=/opt/myapp
ExecStart=/opt/myapp/.venv/bin/gunicorn \
    -k uvicorn.workers.UvicornWorker \
    -w 4 \
    app.main:app
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

После создания:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now myapp
```

Проверка:

```bash
systemctl status myapp
```

Логи:

```bash
journalctl -u myapp -f
```

---

# 24. systemd vs Docker

Это не одно и то же.

### systemd

Управляет процессами и службами **операционной системы**.

```text
Linux
└── systemd
    ├── nginx
    ├── PostgreSQL
    └── myapp
```

### Docker

Изолирует и запускает контейнеры.

```text
Linux
└── Docker
    ├── container 1
    ├── container 2
    └── container 3
```

В production они могут использоваться вместе:

```text
Linux
   ↓
systemd
   ↓
Docker
   ↓
Containers
```

При этом внутри контейнера обычно не требуется запускать полноценный systemd.

---

# 25. systemd vs Supervisor

Оба инструмента могут управлять процессами, но находятся в разных контекстах.

```text
systemd
→ системный init/service manager

Supervisor
→ отдельный process manager
```

На современном Linux systemd обычно является базовым системным механизмом управления службами.

---

# 26. Частая ошибка: `start` ≠ `enable`

```bash
systemctl start nginx
```

означает:

> запустить сейчас.

```bash
systemctl enable nginx
```

означает:

> настроить автозапуск.

А:

```bash
systemctl enable --now nginx
```

означает:

> настроить автозапуск и запустить сейчас.

---

# 27. Частая ошибка: `reload` ≠ `daemon-reload`

```text
reload
→ перечитать конфигурацию самого сервиса

daemon-reload
→ перечитать unit-файлы systemd
```

Например, изменили:

```text
/etc/systemd/system/myapp.service
```

Нужно:

```bash
sudo systemctl daemon-reload
```

А затем, если необходимо применить изменения к процессу:

```bash
sudo systemctl restart myapp
```

---

# 28. Полезные команды

| Задача                    | Команда                        |
| ------------------------- | ------------------------------ |
| Запустить                 | `systemctl start myapp`        |
| Остановить                | `systemctl stop myapp`         |
| Перезапустить             | `systemctl restart myapp`      |
| Перечитать конфиг сервиса | `systemctl reload myapp`       |
| Посмотреть статус         | `systemctl status myapp`       |
| Включить автозапуск       | `systemctl enable myapp`       |
| Отключить автозапуск      | `systemctl disable myapp`      |
| Включить + запустить      | `systemctl enable --now myapp` |
| Перечитать unit-файлы     | `systemctl daemon-reload`      |
| Посмотреть unit           | `systemctl cat myapp`          |
| Логи сервиса              | `journalctl -u myapp`          |
| Логи в реальном времени   | `journalctl -u myapp -f`       |
| Все units                 | `systemctl list-units`         |
| Unit-файлы                | `systemctl list-unit-files`    |

---

## 🎤 Вопросы на собеседовании

### Что такое systemd?

Система инициализации и менеджер служб Linux, управляющий процессами, сервисами, зависимостями и другими системными units.

### Почему у systemd особенный PID?

В обычной Linux-системе systemd работает как PID 1 и является первым userspace-процессом.

### Что такое unit?

Объект, которым управляет systemd: service, socket, timer, target, mount и другие.

### Что такое `.service`?

Unit, описывающий системную службу и правила её запуска.

### Чем `start` отличается от `enable`?

`start` запускает сервис сейчас, а `enable` настраивает его автозапуск при загрузке системы.

### Что делает `systemctl status`?

Показывает состояние unit, информацию о процессе и последние связанные сообщения журнала.

### Зачем нужен `daemon-reload`?

Чтобы systemd перечитал изменённые unit-файлы.

### Чем `reload` отличается от `daemon-reload`?

`reload` относится к конфигурации конкретного сервиса, а `daemon-reload` — к конфигурации самого systemd и его unit-файлам.

### Как настроить автоматический перезапуск?

Например:

```ini
[Service]
Restart=on-failure
```

### Где смотреть логи systemd-сервиса?

Через `journalctl`:

```bash
journalctl -u myapp
```

### Как запустить Python Backend как systemd-сервис?

Создать `.service` в `/etc/systemd/system/`, указать `User`, `WorkingDirectory`, `ExecStart`, при необходимости `Restart`, затем выполнить:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now myapp
```

### Как диагностировать, почему сервис не запускается?

Сначала:

```bash
systemctl status myapp
```

затем:

```bash
journalctl -u myapp -n 100
```

После этого анализировать ошибку запуска, права доступа, пути, environment variables, зависимости и конфигурацию приложения.
