# 📜 journalctl

## 🎤 Короткий ответ

`journalctl` — команда для просмотра и анализа журналов **systemd-journald** в Linux.

Она позволяет смотреть:

* логи конкретного сервиса;
* последние сообщения;
* логи в реальном времени;
* сообщения за определённый период;
* ошибки;
* логи конкретного PID или пользователя;
* сообщения текущей загрузки системы.

Например:

```bash
journalctl -u myapp
```

показывает журнал сервиса `myapp`.

В реальном времени:

```bash
journalctl -u myapp -f
```

Для Backend-разработчика `journalctl` особенно важен при диагностике проблем с **systemd-сервисами**: почему приложение упало, почему не запустилось, что происходило перед ошибкой.

---

## 🗣️ Ответ на собеседовании

`journalctl` — CLI-инструмент для чтения журнала, который собирает `systemd-journald`.

В отличие от простого просмотра файла через `cat` или `tail`, journalctl умеет фильтровать записи по unit, PID, пользователю, времени и другим параметрам.

Например:

```bash
journalctl -u myapp -n 100
```

покажет последние 100 записей сервиса.

Для мониторинга в реальном времени:

```bash
journalctl -u myapp -f
```

Можно также посмотреть сообщения текущей загрузки:

```bash
journalctl -b
```

или только ошибки:

```bash
journalctl -p err
```

В Backend-разработке типичный сценарий — после `systemctl status myapp` использовать `journalctl -u myapp`, чтобы найти причину падения или неправильного запуска приложения.

---

## 🧭 Где я нахожусь

```text
07 DevOps
└── 06 Environment и Infrastructure
    ├── Linux
    │   ├── Файловая система
    │   ├── Процессы
    │   ├── Права доступа
    │   ├── Shell
    │   ├── Сеть
    │   └── Signals
    ├── systemd
    │   ├── Units
    │   ├── Services
    │   ├── systemctl
    │   └── journald
    │       └── journalctl ← Я здесь
    └── SSH
```

---

# 📚 Разбор поглубже

## 1. Что такое journald

`systemd-journald` — системный сервис, который собирает и хранит журнальные записи.

Упрощённо:

```text
Application
     ↓
stdout / stderr
     ↓
systemd-journald
     ↓
Journal
     ↓
journalctl
     ↓
Developer
```

Например, systemd запускает:

```text
myapp.service
      ↓
Python / Gunicorn
      ↓
logs
      ↓
journald
```

А разработчик получает их через:

```bash
journalctl -u myapp
```

---

# 2. journalctl vs log file

Обычный лог-файл:

```bash
tail -f /var/log/myapp.log
```

работает с конкретным текстовым файлом.

`journalctl` работает с журналом `systemd-journald` и позволяет фильтровать структурированные записи.

Например:

```bash
journalctl -u nginx
```

не нужно знать конкретный путь к файлу логов Nginx.

---

# 3. Посмотреть весь журнал

```bash
journalctl
```

Команда выводит записи журнала.

На сервере журнал может быть очень большим, поэтому обычно полезнее сразу использовать фильтры.

Например:

```bash
journalctl -n 100
```

---

# 4. Последние записи — `-n`

Показать последние 50 записей:

```bash
journalctl -n 50
```

Последние 100:

```bash
journalctl -n 100
```

Это удобно при диагностике:

```text
Сервис упал
    ↓
journalctl -n 100
    ↓
смотрим последние события
```

---

# 5. Логи конкретного сервиса — `-u`

Самый важный для Backend вариант:

```bash
journalctl -u myapp
```

`-u` означает фильтрацию по **systemd unit**.

Например:

```bash
journalctl -u nginx
```

или:

```bash
journalctl -u postgresql
```

или:

```bash
journalctl -u myapp
```

---

# 6. Последние записи сервиса

Например:

```bash
journalctl -u myapp -n 100
```

Логика:

```text
-u myapp
→ только сервис myapp

-n 100
→ последние 100 записей
```

Это один из наиболее практичных вариантов:

```bash
journalctl -u myapp -n 100
```

---

# 7. Смотреть логи в реальном времени — `-f`

Аналог `tail -f`:

```bash
journalctl -u myapp -f
```

`-f` означает **follow**.

Схема:

```text
Application
    ↓
new log
    ↓
journald
    ↓
journalctl -f
    ↓
Terminal
```

Команда продолжает работать и показывает новые записи по мере их появления.

Остановить:

```text
Ctrl+C
```

---

# 8. Логи текущей загрузки — `-b`

Показать журнал текущей загрузки системы:

```bash
journalctl -b
```

Это удобно, если проблема появилась после перезагрузки.

Например:

```bash
journalctl -b -u myapp
```

означает:

> показать логи `myapp` за текущую загрузку системы.

---

# 9. Предыдущая загрузка

Можно посмотреть предыдущую boot-сессию:

```bash
journalctl -b -1
```

Следующая:

```text
-b
→ текущая загрузка

-b -1
→ предыдущая загрузка

-b -2
→ загрузка до предыдущей
```

Это полезно при расследовании проблем после reboot.

Посмотреть список загрузок:

```bash
journalctl --list-boots
```

---

# 10. Фильтр по уровню — `-p`

Можно фильтровать сообщения по приоритету.

Например:

```bash
journalctl -p err
```

Показать ошибки и более критичные сообщения.

Основные уровни:

```text
emerg   → 0
alert   → 1
crit    → 2
err     → 3
warning → 4
notice  → 5
info    → 6
debug   → 7
```

Например:

```bash
journalctl -p warning
```

покажет сообщения уровня `warning` и более приоритетные.

---

# 11. Только ошибки конкретного сервиса

Можно объединить фильтры:

```bash
journalctl -u myapp -p err
```

Получаем:

```text
myapp
  ↓
только ошибки
```

Это удобно, когда сервис пишет много информационных сообщений.

---

# 12. Фильтр по времени

Можно указать начало:

```bash
journalctl --since "2026-09-27 10:00:00"
```

И конец:

```bash
journalctl --until "2026-09-27 12:00:00"
```

Вместе:

```bash
journalctl \
    --since "2026-09-27 10:00:00" \
    --until "2026-09-27 12:00:00"
```

Для сервиса:

```bash
journalctl \
    -u myapp \
    --since "1 hour ago"
```

---

# 13. Относительное время

Можно использовать выражения:

```bash
journalctl --since "1 hour ago"
```

или:

```bash
journalctl --since "30 minutes ago"
```

Например:

```bash
journalctl -u myapp --since "10 minutes ago"
```

Очень удобно для оперативной диагностики.

---

# 14. Формат вывода

По умолчанию journalctl показывает примерно:

```text
Sep 27 14:20:31 server myapp[1234]: Application started
```

Можно изменить формат.

Например:

```bash
journalctl -u myapp -o short-iso
```

Получаем timestamp в более удобном формате.

Также существуют другие output formats:

```bash
journalctl -o json
```

или:

```bash
journalctl -o json-pretty
```

---

# 15. JSON

Для автоматизированной обработки:

```bash
journalctl -u myapp -o json
```

Можно получить структурированные записи.

Это полезно, если журнал нужно обрабатывать программно.

---

# 16. Логи конкретного PID

Можно фильтровать по процессу:

```bash
journalctl _PID=1234
```

Это особенно полезно, когда известен PID процесса, который создаёт проблему.

Например:

```text
systemctl status myapp
        ↓
Main PID: 1234
        ↓
journalctl _PID=1234
```

---

# 17. Логи пользователя

Можно фильтровать по UID:

```bash
journalctl _UID=1000
```

Это позволяет найти записи, связанные с конкретным пользователем.

---

# 18. Kernel messages

Для сообщений ядра:

```bash
journalctl -k
```

Например:

```bash
journalctl -k -b
```

покажет kernel messages текущей загрузки.

Это может быть полезно при проблемах с:

* драйверами;
* устройствами;
* сетью;
* файловыми системами;
* hardware.

---

# 19. Follow + service

Один из самых полезных вариантов:

```bash
journalctl -u myapp -f
```

Например, перезапускаем приложение:

```bash
sudo systemctl restart myapp
```

и одновременно смотрим:

```bash
journalctl -u myapp -f
```

Можно сразу увидеть:

```text
Starting...
Loading configuration...
Connecting to PostgreSQL...
ERROR: connection refused
```

---

# 20. Типичный workflow диагностики

Предположим:

```bash
sudo systemctl restart myapp
```

сервис не запускается.

### Шаг 1 — статус

```bash
systemctl status myapp
```

Получаем:

```text
Active: failed
```

### Шаг 2 — журнал

```bash
journalctl -u myapp -n 100
```

### Шаг 3 — только ошибки

```bash
journalctl -u myapp -p err
```

### Шаг 4 — последние события

```bash
journalctl -u myapp --since "10 minutes ago"
```

Получаем:

```text
systemd
   ↓
myapp failed
   ↓
journalctl
   ↓
конкретная ошибка
   ↓
исправление
```

---

# 21. `systemctl status` vs `journalctl`

Их часто используют вместе.

### `systemctl status`

Показывает:

* состояние unit;
* PID;
* время запуска;
* основную информацию;
* небольшой фрагмент последних логов.

```bash
systemctl status myapp
```

### `journalctl`

Предназначен для более полноценного анализа журнала:

```bash
journalctl -u myapp
```

Схема:

```text
systemctl status
        ↓
быстрая диагностика
        ↓
journalctl
        ↓
подробный анализ логов
```

---

# 22. journalctl и Backend

Допустим, есть:

```text
FastAPI
   ↓
Gunicorn
   ↓
systemd
   ↓
journald
```

При ошибке:

```text
FastAPI
   ↓
Exception
   ↓
stderr
   ↓
journald
```

Разработчик выполняет:

```bash
journalctl -u myapp -f
```

и видит traceback или сообщение об ошибке.

Это позволяет диагностировать:

* неправильные environment variables;
* проблемы подключения к PostgreSQL;
* ошибки миграций;
* проблемы импорта Python;
* ошибки конфигурации;
* падение workers;
* ошибки запуска Gunicorn.

---

# 23. journalctl и stdout/stderr

Если systemd запускает сервис, его stdout/stderr может быть направлен в journal в зависимости от конфигурации unit.

Например:

```text
Python
  ↓
stdout/stderr
  ↓
systemd/journald
  ↓
journalctl
```

Поэтому обычный:

```python
print("Application started")
```

может появиться в:

```bash
journalctl -u myapp
```

при соответствующей конфигурации сервиса.

---

# 24. Где физически хранится journal

В зависимости от конфигурации systemd journal может храниться:

```text
/run/log/journal/
```

если используется volatile storage,

или:

```text
/var/log/journal/
```

если включено persistent storage.

Важно:

> `journalctl` не означает просто «читать `/var/log/...`». Он обращается к журналу `systemd-journald`.

---

# 25. Persistent journal

Если journal настроен только как volatile storage, часть журналов может исчезать после перезагрузки.

При persistent storage записи могут сохраняться между reboot.

Проверить конфигурацию можно через:

```bash
cat /etc/systemd/journald.conf
```

---

# 26. Размер журнала

На сервере журнал может занимать значительный объём диска.

Посмотреть использование:

```bash
journalctl --disk-usage
```

Это особенно важно для production-серверов.

---

# 27. Очистка старых журналов

Для ограничения размера можно использовать:

```bash
journalctl --vacuum-size=500M
```

или по времени:

```bash
journalctl --vacuum-time=7d
```

Это удаляет старые записи согласно указанному ограничению.

В production очистку логов лучше организовывать через осознанную retention policy, а не выполнять случайные удаления.

---

# 28. Полезные команды

| Задача                     | Команда                           |
| -------------------------- | --------------------------------- |
| Весь journal               | `journalctl`                      |
| Последние 100 записей      | `journalctl -n 100`               |
| Логи сервиса               | `journalctl -u myapp`             |
| Последние 100 сервиса      | `journalctl -u myapp -n 100`      |
| Следить в реальном времени | `journalctl -u myapp -f`          |
| Текущая загрузка           | `journalctl -b`                   |
| Предыдущая загрузка        | `journalctl -b -1`                |
| Список загрузок            | `journalctl --list-boots`         |
| Ошибки                     | `journalctl -p err`               |
| Логи за час                | `journalctl --since "1 hour ago"` |
| Kernel logs                | `journalctl -k`                   |
| По PID                     | `journalctl _PID=1234`            |
| JSON                       | `journalctl -o json`              |
| Размер journal             | `journalctl --disk-usage`         |

---

# 29. Самый полезный набор для Backend

На практике стоит хорошо помнить:

```bash
systemctl status myapp
```

```bash
journalctl -u myapp
```

```bash
journalctl -u myapp -n 100
```

```bash
journalctl -u myapp -f
```

```bash
journalctl -u myapp --since "10 minutes ago"
```

```bash
journalctl -u myapp -p err
```

```bash
journalctl -b -1 -u myapp
```

---

# 30. Практический сценарий

Допустим, после деплоя приложение перестало запускаться.

```bash
sudo systemctl restart myapp
```

Проверяем:

```bash
systemctl status myapp
```

Видим:

```text
Active: failed
```

Смотрим последние записи:

```bash
journalctl -u myapp -n 100
```

Находим:

```text
ModuleNotFoundError: No module named ...
```

Исправляем окружение.

Затем:

```bash
sudo systemctl daemon-reload
sudo systemctl restart myapp
```

И наблюдаем:

```bash
journalctl -u myapp -f
```

Получается полноценная цепочка:

```text
systemd
   ↓
systemctl status
   ↓
journalctl
   ↓
ошибка
   ↓
исправление
   ↓
restart
   ↓
journalctl -f
```

---

## 🎤 Вопросы на собеседовании

### Что такое `journalctl`?

CLI-инструмент для просмотра и фильтрации журнала `systemd-journald`.

### Что такое `journald`?

Сервис systemd, который собирает и хранит журнальные записи.

### Как посмотреть логи systemd-сервиса?

```bash
journalctl -u myapp
```

### Как смотреть логи в реальном времени?

```bash
journalctl -u myapp -f
```

### Как посмотреть последние 100 записей?

```bash
journalctl -n 100
```

Для конкретного сервиса:

```bash
journalctl -u myapp -n 100
```

### Как посмотреть логи текущей загрузки?

```bash
journalctl -b
```

### Как посмотреть предыдущую загрузку?

```bash
journalctl -b -1
```

### Как посмотреть только ошибки?

```bash
journalctl -p err
```

### Как посмотреть логи за последний час?

```bash
journalctl --since "1 hour ago"
```

### Чем `journalctl` отличается от `tail -f`?

`tail -f` следит за конкретным файлом, а `journalctl` работает с журналом `systemd-journald` и позволяет фильтровать записи по unit, времени, PID, приоритету и другим полям.

### Чем `systemctl status` отличается от `journalctl -u`?

`systemctl status` показывает состояние systemd unit и небольшой фрагмент последних сообщений, а `journalctl -u` предназначен для полноценного просмотра и анализа журнала конкретного unit.

### Где хранятся журналы systemd?

В зависимости от конфигурации journal может храниться в `/run/log/journal/` или `/var/log/journal/`.

### Как посмотреть, сколько места занимает journal?

```bash
journalctl --disk-usage
```

### Как посмотреть kernel messages?

```bash
journalctl -k
```

### Как диагностировать упавший Backend-сервис?

```bash
systemctl status myapp
journalctl -u myapp -n 100
journalctl -u myapp -p err
```

После исправления — перезапустить сервис и проверить журнал:

```bash
systemctl restart myapp
journalctl -u myapp -f
```
