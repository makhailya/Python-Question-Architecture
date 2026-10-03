# 💾 Volumes в Docker

## 🎤 Короткий ответ

**Docker Volume** — это механизм хранения данных контейнера отдельно от его файловой системы. Данные Volume сохраняются после удаления или пересоздания контейнера.

Главная идея:

> **Контейнер можно удалить, а данные в Volume останутся.**

Например, PostgreSQL:

```python
docker container
      ↓
PostgreSQL
      ↓
Docker Volume
      ↓
Данные сохраняются
```

---

## 🎯 Формула для собеседования

> **Container — временный, Volume — постоянное хранение данных.**

```python
Container → Volume → Persistent Data
```

---

# ❓ Зачем нужны Volumes

Файловая система контейнера является **эфемерной**.

Например:

```python
PostgreSQL Container
        ↓
      DB data
```

Удаляем контейнер:

```python
docker rm postgres
```

Если данные хранились только внутри контейнера — они могут быть потеряны.

С Volume:

```python
PostgreSQL Container
        ↓
      Volume
        ↓
   PostgreSQL data
```

Контейнер можно удалить:

```python
docker rm postgres
```

и затем создать новый:

```python
docker run ...
```

подключив тот же Volume.

Данные останутся.

---

# 📦 Volume

Создать Volume:

```python
docker volume create postgres_data
```

Посмотреть:

```python
docker volume ls
```

Информация:

```python
docker volume inspect postgres_data
```

Удалить:

```python
docker volume rm postgres_data
```

---

# 🐘 Volume для PostgreSQL

Например:

```python
docker run -d \
  --name postgres \
  -e POSTGRES_PASSWORD=secret \
  -v postgres_data:/var/lib/postgresql/data \
  postgres:15
```

Здесь:

```text
postgres_data
      ↓
/var/lib/postgresql/data
```

`/var/lib/postgresql/data` — каталог внутри контейнера, где PostgreSQL хранит свои данные.

---

# 🔗 Что означает `-v`

Формат:

```python
-v SOURCE:DESTINATION
```

Например:

```python
-v postgres_data:/var/lib/postgresql/data
```

где:

```text
postgres_data
      ↓
Volume на стороне Docker

/var/lib/postgresql/data
      ↓
путь внутри контейнера
```

---

# 🗂️ Named Volume

**Named Volume** — именованный Docker Volume.

```python
docker volume create my_data
```

Использование:

```python
docker run -v my_data:/app/data myapp
```

Преимущество — Volume имеет понятное имя и легко переиспользуется.

Например:

```text
my_data
my_postgres_data
redis_data
```

---

# 📁 Bind Mount

Другой вариант — подключить конкретную директорию хоста.

```python
-v ./data:/app/data
```

Здесь:

```text
./data
   ↓
директория на компьютере

/app/data
   ↓
директория внутри контейнера
```

Например:

```python
docker run \
  -v $(pwd)/src:/app/src \
  myapp
```

Файлы на компьютере напрямую доступны контейнеру.

---

# 🆚 Volume vs Bind Mount

|                                 | Volume | Bind Mount |
| ------------------------------- | ------ | ---------- |
| Управляется Docker              | ✅      | ❌          |
| Хранится в Docker storage       | ✅      | ❌          |
| Указываем конкретный путь хоста | ❌      | ✅          |
| Удобен для DB                   | ✅      | ⚠️         |
| Удобен для разработки           | ⚠️     | ✅          |
| Переносимость                   | Выше   | Ниже       |
| Контроль файлов хостом          | Меньше | Больше     |

Упрощённо:

```python
Volume:
Docker → Data

Bind Mount:
Host Path → Container Path
```

---

# 📝 Пример для разработки

Допустим, Python-приложение находится на компьютере:

```text
project/
├── app/
├── requirements.txt
└── Dockerfile
```

Можно подключить исходный код:

```python
docker run \
  -v ./app:/app/app \
  myapp
```

Теперь изменения на компьютере видны внутри контейнера.

Это удобно для development и hot reload.

---

# 🐳 Volumes в Docker Compose

В Compose Volume описывается в секции:

```python
services:
  db:
    image: postgres:15
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

Здесь:

```text
services
   ↓
db
   ↓
postgres_data
   ↓
/var/lib/postgresql/data
```

Docker Compose создаёт и подключает Volume к контейнеру.

---

# 🐘 FastAPI + PostgreSQL + Volume

Типичный Compose:

```python
services:
  api:
    build: .
    ports:
      - "8000:8000"
    depends_on:
      - db

  db:
    image: postgres:15
    environment:
      POSTGRES_DB: app
      POSTGRES_USER: user
      POSTGRES_PASSWORD: secret
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

Получается:

```text
FastAPI Container
       ↓
   PostgreSQL
       ↓
postgres_data
       ↓
Persistent Database
```

---

# 🔄 Что происходит при `docker compose down`

```python
docker compose down
```

Обычно удаляются:

* контейнеры;
* сети Compose.

**Named Volumes по умолчанию не удаляются.**

Поэтому данные PostgreSQL сохраняются.

Но:

```python
docker compose down -v
```

удаляет также Volume.

То есть:

```text
docker compose down
        ↓
Containers удалены
Volume сохранён
```

А:

```text
docker compose down -v
        ↓
Containers удалены
Volume удалён
        ↓
Данные потеряны
```

---

# ⚠️ Volume ≠ Backup

Это очень важный момент для собеседования.

Volume защищает данные от удаления **самого контейнера**, но не от всех проблем.

Например:

```text
Volume
   ↓
ошибочное DELETE
   ↓
данные удалены
```

Удаление данных попадёт в Volume.

Также возможны:

* повреждение данных;
* удаление Volume;
* проблемы с диском;
* ошибки администратора;
* отказ сервера.

Поэтому:

> **Volume — это persistent storage, а не backup.**

Для PostgreSQL нужны отдельные механизмы резервного копирования.

---

# 🔐 Права доступа

При использовании Volume важно учитывать права файлов.

Например:

```text
Host / Docker
      ↓
Volume
      ↓
Container process
```

Процесс внутри контейнера должен иметь необходимые права на каталог.

Особенно это важно для:

* PostgreSQL;
* Redis;
* Elasticsearch;
* приложений, которые пишут файлы.

---

# 📊 Где Docker хранит Volume

Docker сам управляет местом хранения named volumes.

Посмотреть информацию:

```python
docker volume inspect postgres_data
```

Можно увидеть:

```text
Name
Driver
Mountpoint
```

Обычно приложение не должно зависеть от конкретного физического пути Docker на хосте.

---

# 🧹 Удаление неиспользуемых Volumes

Docker может накапливать ненужные Volume.

Посмотреть:

```python
docker volume ls
```

Удалить неиспользуемые:

```python
docker volume prune
```

⚠️ Команда удаляет неиспользуемые Volume, поэтому перед выполнением нужно понимать, какие данные больше не нужны.

---

# 📌 Read-Only Volume

Mount можно сделать только для чтения.

Например:

```python
-v ./config:/app/config:ro
```

`ro` означает:

```text
read only
```

Контейнер сможет читать файлы, но не изменять их.

---

# 🆚 Container filesystem vs Volume

| Container filesystem                     | Volume                           |
| ---------------------------------------- | -------------------------------- |
| Живёт вместе с контейнером               | Независим от контейнера          |
| Удаление контейнера может удалить данные | Данные сохраняются               |
| Подходит для временных файлов            | Подходит для persistent data     |
| Не предназначен для постоянной БД        | Хороший вариант для локальной DB |

---

# ☸️ А как с этим в Kubernetes?

В Kubernetes аналогичная идея реализуется через **Volumes**, но механизм шире.

Например:

```text
Pod
 ↓
Volume
 ↓
Storage
```

Для постоянного хранения обычно используются:

```text
PersistentVolume (PV)
        ↓
PersistentVolumeClaim (PVC)
        ↓
Pod
```

Упрощённо:

```python
PVC
 ↓
PV
 ↓
Storage
 ↓
Pod
```

В Kubernetes контейнер также нельзя рассматривать как место для долговременного хранения важных данных.

---

# 🧠 Важный момент для баз данных

Например, PostgreSQL работает в Docker:

```text
PostgreSQL
     ↓
Volume
     ↓
Disk
```

Если контейнер перезапустился:

```text
Container ❌
     ↓
Container ✅
     ↓
Volume
     ↓
Данные остались
```

Но если физически сломался сервер:

```text
Server ❌
     ↓
Volume тоже недоступен
```

Поэтому в production нужны:

* persistent storage;
* репликация;
* backups;
* monitoring;
* recovery strategy.

---

# 🎤 Как рассказать на собеседовании

> **Docker Volume — это механизм постоянного хранения данных отдельно от файловой системы контейнера. Контейнер можно удалить или пересоздать, а данные в Volume сохранятся. Например, PostgreSQL в Docker обычно хранит `/var/lib/postgresql/data` в named Volume. В Docker есть named volumes и bind mounts. Volume не является backup — для защиты от потери данных нужны отдельные резервные копии и стратегия восстановления.**

---

# ❓ Частые вопросы

### Что такое Volume?

Механизм хранения данных Docker отдельно от жизненного цикла контейнера.

### Зачем Volume?

Чтобы данные не исчезали при удалении/пересоздании контейнера.

### Что будет с Volume после `docker rm`?

Сам контейнер удалится, а отдельный Volume обычно останется.

### Что делает `docker compose down -v`?

Удаляет Compose-контейнеры и связанные с проектом Volume.

### Volume — это backup?

**Нет.**

### Volume vs Bind Mount?

**Volume управляется Docker, Bind Mount подключает конкретный путь файловой системы хоста.**

### Где использовать Volume?

Например:

* PostgreSQL;
* Redis с persistence;
* Elasticsearch;
* пользовательские файлы;
* другие persistent data.

### Можно ли использовать Volume для разработки?

Да. Но для исходного кода часто удобнее Bind Mount:

```python
./src:/app/src
```

---

## 🔑 Главное

```python
Container
    ↓
Filesystem
    ↓
❌ может исчезнуть при удалении контейнера


Container
    ↓
Volume
    ↓
✅ данные сохраняются
```

### Три вещи, которые нужно различать

```python
Volume
→ Docker-managed persistent storage

Bind Mount
→ Host directory → Container directory

Backup
→ отдельная копия данных для восстановления
```

> **Volume сохраняет данные независимо от контейнера, но сам по себе не является резервной копией.**
