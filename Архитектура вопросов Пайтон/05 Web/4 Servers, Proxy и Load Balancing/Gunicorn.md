# 🦄 Gunicorn

## 🎯 Ответ на собеседовании

**Gunicorn (Green Unicorn)** — это production WSGI-сервер для Python-приложений. Он принимает HTTP-запросы и передаёт их Python-приложению через WSGI.

Чаще всего Gunicorn используют с Django и Flask.

Типичная схема:

```text
Клиент
   ↓
Nginx
   ↓
Gunicorn
   ↓
Django
```

Gunicorn использует модель **master + workers**: master-процесс управляет worker-процессами, а workers непосредственно обрабатывают запросы.

Главное:

> **Gunicorn — WSGI-сервер, который запускает Python-приложение и организует обработку запросов через worker-процессы.**

---

## 🎤 Суперкоротко

```text
Gunicorn = production WSGI-сервер

Nginx
  ↓
Gunicorn
  ↓
Django / Flask
```

```text
Gunicorn
   ↓
Master
   ├── Worker
   ├── Worker
   ├── Worker
   └── Worker
```

---

# 🌐 Зачем нужен Gunicorn

Django не должен самостоятельно заниматься всей серверной работой в production.

Gunicorn выступает посредником:

```text
HTTP Client
     ↓
   Gunicorn
     ↓
    WSGI
     ↓
   Django
```

Он:

* запускает приложение;
* создаёт workers;
* принимает соединения;
* распределяет работу между workers;
* перезапускает workers при необходимости;
* позволяет масштабировать приложение количеством workers.

---

# 🔌 Gunicorn и WSGI

Это два разных понятия.

**WSGI** — стандарт интерфейса.

**Gunicorn** — сервер, который использует этот интерфейс.

```text
WSGI
 ↓
стандарт взаимодействия


Gunicorn
 ↓
реализация WSGI-сервера
```

Например, Django предоставляет:

```text
project.wsgi:application
```

Gunicorn запускает этот объект.

---

# 🏗️ Master и Workers

Одна из ключевых особенностей Gunicorn — архитектура:

```text
              Gunicorn
                  │
               Master
          ┌───────┼───────┐
          ↓       ↓       ↓
       Worker  Worker  Worker
```

### Master

Master-процесс:

* управляет workers;
* запускает workers;
* следит за их состоянием;
* перезапускает завершившиеся workers;
* обрабатывает управляющие сигналы.

Master обычно **не обрабатывает HTTP-запросы непосредственно**.

### Worker

Worker — процесс, который непосредственно обслуживает запросы приложения.

Например:

```text
Gunicorn
   │
   ├── Worker 1
   ├── Worker 2
   ├── Worker 3
   └── Worker 4
```

Чем больше подходящих workers, тем больше запросов приложение потенциально может обрабатывать одновременно.

---

# 🚀 Запуск Django

Допустим, структура:

```text
project/
├── manage.py
└── project/
    ├── settings.py
    ├── urls.py
    └── wsgi.py
```

Gunicorn можно запустить:

```python
gunicorn project.wsgi:application
```

Здесь:

```text
project.wsgi
     ↓
модуль wsgi.py

application
     ↓
WSGI application
```

---

# ⚙️ Workers

Можно указать количество workers:

```python
gunicorn --workers 4 project.wsgi:application
```

Получится:

```text
Master
 ├── Worker 1
 ├── Worker 2
 ├── Worker 3
 └── Worker 4
```

Workers — отдельные процессы.

Это позволяет использовать несколько CPU-ядер и обрабатывать несколько запросов одновременно.

---

# 🔄 Как обрабатывается запрос

Допустим:

```text
GET /users/
```

Происходит примерно следующее:

```text
Client
  ↓
Nginx
  ↓
Gunicorn
  ↓
Worker
  ↓
WSGI
  ↓
Django
  ↓
Response
  ↓
Worker
  ↓
Nginx
  ↓
Client
```

Gunicorn принимает запрос и передаёт его worker'у, который запускает обработку Django-приложением.

---

# 🌐 Gunicorn + Nginx

В production часто используют:

```text
                    Internet
                       ↓
                    Nginx
                       ↓
                   Gunicorn
                       ↓
                    Django
                       ↓
                   PostgreSQL
```

### Nginx

Отвечает, например, за:

* TLS termination;
* раздачу статических файлов;
* reverse proxy;
* ограничения и buffering;
* обработку внешних HTTP-соединений.

### Gunicorn

Отвечает за:

* запуск Python-приложения;
* workers;
* передачу запросов Django;
* выполнение Python-кода приложения.

---

# 🧩 Почему не использовать `runserver`

Django имеет встроенный development server:

```python
python manage.py runserver
```

Но он предназначен прежде всего для разработки.

В production обычно используют отдельный сервер приложений:

```text
Development:

Django runserver


Production:

Nginx
  ↓
Gunicorn
  ↓
Django
```

---

# ⚡ Gunicorn и ASGI

Классический Gunicorn — **WSGI-сервер**.

Однако Gunicorn также может использоваться в современных Python-стэках с ASGI через подходящий worker-класс.

Например, часто встречается схема:

```text
Nginx
  ↓
Gunicorn
  ↓
Uvicorn Worker
  ↓
FastAPI
```

Здесь важно не путать роли:

```text
Gunicorn
↓
process manager / server


Uvicorn Worker
↓
ASGI implementation
```

Для простого запуска FastAPI также можно использовать непосредственно Uvicorn:

```python
uvicorn main:app
```

---

# 🆚 Gunicorn vs Uvicorn

| Gunicorn                    | Uvicorn                              |
| --------------------------- | ------------------------------------ |
| WSGI-сервер                 | ASGI-сервер                          |
| Классически Django/Flask    | FastAPI/Starlette                    |
| Синхронная WSGI-модель      | Async ASGI-модель                    |
| Master + workers            | ASGI server                          |
| Часто используется за Nginx | Может работать напрямую или за Nginx |

Упрощённо:

```text
Django / WSGI
      ↓
  Gunicorn


FastAPI / ASGI
      ↓
  Uvicorn
```

---

# 🧠 Worker ≠ Thread

В базовой конфигурации Gunicorn workers — это **отдельные процессы**.

```text
Master
 ├── Process
 ├── Process
 ├── Process
 └── Process
```

Но Gunicorn поддерживает разные типы workers и конфигурации, включая варианты с threads и async workers.

На собеседовании лучше сказать:

> **По умолчанию Gunicorn использует worker-процессы; конкретный тип worker'а зависит от выбранной worker-класса и конфигурации.**

---

# 📈 Масштабирование

Если один worker способен обрабатывать ограниченное количество запросов одновременно, можно увеличить количество workers:

```text
1 Worker
   ↓
ограниченная параллельность


4 Workers
   ↓
больше одновременно обрабатываемых запросов
```

Но нельзя просто бесконечно увеличивать их количество.

Каждый worker потребляет:

* RAM;
* CPU;
* другие системные ресурсы.

Поэтому количество workers подбирают с учётом нагрузки и ресурсов сервера.

---

# 🛡️ Что происходит при падении Worker

Одна из задач master-процесса — следить за workers.

Упрощённо:

```text
Master
   ↓
Worker 1
Worker 2
Worker 3
   ↓
Worker 2 crashed
   ↓
Master обнаруживает проблему
   ↓
запускает новый Worker
```

Это повышает устойчивость приложения.

---

# 🔄 Graceful Restart

Gunicorn поддерживает корректное перезапускание workers.

Идея:

```text
старый Worker
      ↓
перестаёт принимать новую работу
      ↓
завершает текущие запросы
      ↓
останавливается

новый Worker
      ↓
начинает принимать запросы
```

Это позволяет обновлять приложение с минимальным влиянием на пользователей.

---

# 🎯 Главное

```text
Gunicorn
   ↓
Master
   ↓
Workers
   ↓
WSGI
   ↓
Django / Flask
```

Запомнить:

```text
WSGI     → стандарт интерфейса
Gunicorn → WSGI-сервер
Worker   → обрабатывает запросы
Master   → управляет workers
Nginx    → reverse proxy перед Gunicorn
```

### Формула для собеседования

> **Gunicorn — production WSGI-сервер для Python-приложений. Он использует master-процесс и worker-процессы, которые обрабатывают запросы приложения. Часто работает в связке Nginx → Gunicorn → Django.**

```text
Nginx
  ↓
Gunicorn
  ↓
Workers
  ↓
Django
  ↓
Database
```
