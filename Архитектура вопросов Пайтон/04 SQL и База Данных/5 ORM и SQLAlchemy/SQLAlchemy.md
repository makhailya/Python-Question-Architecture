# 🐍 SQLAlchemy

## 🎯 Формула для собеседования

> **SQLAlchemy — это Python-библиотека для работы с реляционными базами данных. Она предоставляет Core для построения и выполнения SQL-запросов и ORM для работы с БД через Python-объекты. Основные компоненты ORM — `Engine`, `Session`, модели и запросы. SQLAlchemy также умеет управлять connection pool и транзакциями.**

---

## 🎤 Суперкоротко

```text id="f9m3k2"
Python
   ↓
SQLAlchemy
   ├── Core
   │    ↓
   │   SQL
   │
   └── ORM
        ↓
      Models
        ↓
    PostgreSQL
```

Главные понятия:

```text id="r7c4v1"
Engine
Session
Model
Query
Transaction
Connection Pool
```

---

# 1. 🤔 Что такое SQLAlchemy

**SQLAlchemy** — библиотека для работы Python-приложения с реляционными БД.

Она позволяет:

```text id="k3p8s5"
Python application
       ↓
   SQLAlchemy
       ↓
 PostgreSQL / MySQL / SQLite / ...
```

При этом SQLAlchemy **не является самой БД**.

Например:

```text id="m5v2q9"
FastAPI
   ↓
SQLAlchemy
   ↓
PostgreSQL
```

---

# 2. 🧩 SQLAlchemy Core и ORM

У SQLAlchemy есть два основных уровня.

## Core

Core предоставляет SQL expression language и низкоуровневые механизмы работы с БД.

Можно явно описывать SQL-операции:

```python
from sqlalchemy import select

stmt = select(users).where(users.c.age >= 18)
```

SQLAlchemy затем скомпилирует выражение в SQL для конкретной БД.

---

## ORM

ORM позволяет работать с таблицами через Python-классы.

Например:

```python
class User:
    id: int
    name: str
    email: str
```

Идея:

```text id="x8c4n2"
Python object
      ↕
     ORM
      ↕
Database row
```

---

# 3. 🔌 Engine

`Engine` — центральная точка взаимодействия SQLAlchemy с БД.

Он связывает приложение с конкретной database configuration и управляет подключениями.

Упрощённо:

```text id="q6m3v8"
Application
     ↓
  Engine
     ↓
Connection Pool
     ↓
PostgreSQL
```

Пример:

```python
from sqlalchemy import create_engine

engine = create_engine(
    "postgresql+psycopg://user:password@localhost/mydb"
)
```

---

# 4. 📦 Engine и Connection Pool

Это особенно связано с предыдущей темой.

SQLAlchemy `Engine` обычно использует **Connection Pool**.

```text id="w4p7k1"
Engine
  ↓
Pool
 ┌──┬──┬──┬──┐
 │C1│C2│C3│C4│
 └──┴──┴──┴──┘
  ↓
PostgreSQL
```

Когда приложению нужен connection:

```text id="z9m2c5"
Engine
  ↓
acquire connection
  ↓
SQL
  ↓
release
  ↓
Pool
```

Поэтому `Engine` — это не просто «одно соединение с БД».

---

# 5. 🧠 Session

`Session` — основной объект SQLAlchemy ORM для работы с объектами и транзакциями.

Упрощённо:

```text id="p3k8v6"
Application
     ↓
  Session
     ↓
  Engine
     ↓
PostgreSQL
```

Через `Session` можно:

* загружать объекты;
* добавлять новые;
* изменять;
* удалять;
* выполнять запросы;
* управлять транзакцией.

---

# 6. 🔄 Жизненный цикл Session

Типичный сценарий:

```text id="h5q2m9"
создать Session
      ↓
выполнить запросы
      ↓
изменить объекты
      ↓
commit
      ↓
закрыть Session
```

Например:

```python
from sqlalchemy.orm import Session

with Session(engine) as session:
    users = session.scalars(
        select(User)
    ).all()
```

После завершения работы session закрывается.

---

# 7. 💾 Транзакция

SQLAlchemy позволяет работать с транзакциями.

Например:

```python
with Session(engine) as session:
    user = User(name="Ivan")

    session.add(user)

    session.commit()
```

Если произошла ошибка:

```python
with Session(engine) as session:
    try:
        session.add(user)
        session.commit()
    except Exception:
        session.rollback()
        raise
```

Основная идея:

```text id="r8m3c6"
Session
   ↓
BEGIN
   ↓
SQL
   ↓
COMMIT
```

или:

```text id="t4v9k2"
Session
   ↓
SQL
   ↓
ROLLBACK
```

---

# 8. 🧱 Model

В ORM таблица обычно представляется Python-классом.

Например:

```python
from sqlalchemy.orm import DeclarativeBase
from sqlalchemy.orm import Mapped
from sqlalchemy.orm import mapped_column


class Base(DeclarativeBase):
    pass


class User(Base):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str]
    email: Mapped[str]
```

Получается соответствие:

```text id="n7c3p5"
Python class
     ↓
User
     ↓
users table
```

А:

```text id="x2m8q4"
User instance
     ↓
database row
```

---

# 9. 🔎 SELECT в SQLAlchemy

Современный SQLAlchemy использует конструкцию `select()`.

```python
from sqlalchemy import select

stmt = select(User)
```

Выполнение:

```python
result = session.execute(stmt)
```

Для получения ORM-объектов удобно:

```python
users = session.scalars(stmt).all()
```

---

# 10. 🔍 WHERE

Например:

```python
stmt = (
    select(User)
    .where(User.age >= 18)
)
```

Концептуально:

```text id="m6q4v8"
SQLAlchemy expression
        ↓
SELECT ...
FROM users
WHERE age >= 18
```

SQLAlchemy строит SQL, а PostgreSQL выполняет его.

---

# 11. 🔗 JOIN

SQLAlchemy позволяет описывать `JOIN`.

Например:

```python
stmt = (
    select(User, Order)
    .join(Order, Order.user_id == User.id)
)
```

Концептуально получится:

```text id="c8v2m7"
SELECT ...
FROM users
JOIN orders
    ON orders.user_id = users.id;
```

---

# 12. 📦 Relationships

ORM позволяет описывать связи между моделями.

Например:

```python
from sqlalchemy.orm import relationship


class User(Base):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(primary_key=True)

    orders: Mapped[list["Order"]] = relationship(
        back_populates="user"
    )


class Order(Base):
    __tablename__ = "orders"

    id: Mapped[int] = mapped_column(primary_key=True)
    user_id: Mapped[int] = mapped_column()

    user: Mapped[User] = relationship(
        back_populates="orders"
    )
```

Теперь можно концептуально обращаться:

```python
user.orders
```

и:

```python
order.user
```

---

# 13. 🐌 SQLAlchemy и N+1

ORM может столкнуться с классической **N+1 проблемой**.

Например:

```python
users = session.scalars(
    select(User)
).all()

for user in users:
    print(user.orders)
```

Если `orders` загружается лениво, ORM может сделать:

```text id="u5k8m2"
SELECT users;

SELECT orders WHERE user_id = 1;
SELECT orders WHERE user_id = 2;
SELECT orders WHERE user_id = 3;
...
```

Получаем:

```text id="e3q7n9"
N + 1 запросов
```

---

# 14. 🚀 Eager Loading

SQLAlchemy предоставляет стратегии eager loading.

В частности:

```text id="k6m3p8"
joinedload()
selectinload()
```

Они соответствуют концепциям, которые мы разбирали ранее.

### `joinedload()`

Загружает связанные данные через `JOIN`.

```python
from sqlalchemy.orm import joinedload

stmt = (
    select(User)
    .options(joinedload(User.orders))
)
```

Концептуально:

```text id="p9v4c2"
User
 ↓
JOIN
 ↓
Orders
```

---

### `selectinload()`

Загружает связанные данные отдельным запросом через `IN`.

```python
from sqlalchemy.orm import selectinload

stmt = (
    select(User)
    .options(selectinload(User.orders))
)
```

Концептуально:

```text id="r2m7k5"
SELECT users;

SELECT orders
WHERE user_id IN (...);
```

То есть:

```text id="x4c8n1"
joinedload()
→ JOIN Load

selectinload()
→ Selection Load
```

Это очень полезная связь с предыдущими темами.

---

# 15. 🧠 Identity Map

`Session` поддерживает **Identity Map**.

Идея:

> В рамках одной Session SQLAlchemy старается представлять одну строку БД одним объектом Python для данной identity.

Например:

```text id="a7k3m9"
users.id = 1
```

Если этот пользователь уже загружен в Session, повторное обращение к нему может использовать существующий объект.

Концептуально:

```text id="z5q8c2"
DB row id=1
     ↓
 Session Identity Map
     ↓
 Python User object
```

Это помогает избежать создания нескольких разных Python-объектов для одной database identity внутри одной Session.

---

# 16. 📝 Unit of Work

SQLAlchemy Session также реализует концепцию **Unit of Work**.

Приложение изменяет объекты:

```python
user.name = "Petr"
```

SQLAlchemy отслеживает изменения.

При:

```python
session.commit()
```

ORM определяет необходимые изменения и отправляет SQL в БД.

Упрощённо:

```text id="m8c4p1"
Python objects
      ↓
Session tracks changes
      ↓
flush
      ↓
SQL
      ↓
COMMIT
```

---

# 17. 🔄 Flush vs Commit

Это важное различие.

### `flush()`

Отправляет накопленные изменения из Session в БД **в рамках текущей транзакции**.

Но транзакция ещё не обязательно зафиксирована.

```python
session.flush()
```

### `commit()`

Фиксирует транзакцию:

```python
session.commit()
```

Упрощённо:

```text id="v6k2p9"
session.add()
      ↓
   flush()
      ↓
SQL отправлен в БД
      ↓
  commit()
      ↓
изменения зафиксированы
```

---

# 18. ❌ Rollback

Если транзакцию нужно отменить:

```python
session.rollback()
```

Например:

```python
try:
    session.add(user)
    session.commit()
except Exception:
    session.rollback()
    raise
```

После ошибки транзакционное состояние Session необходимо корректно обработать перед дальнейшим использованием.

---

# 19. 🧩 SQLAlchemy и драйвер БД

SQLAlchemy обычно работает поверх **DBAPI-драйвера**.

Например для PostgreSQL:

```text id="q3m7v8"
SQLAlchemy
     ↓
psycopg
     ↓
PostgreSQL
```

То есть SQLAlchemy не заменяет драйвер.

Упрощённо:

```text id="f8c2m4"
Application
     ↓
SQLAlchemy
     ↓
DB Driver
     ↓
PostgreSQL
```

---

# 20. 🏗️ SQLAlchemy в FastAPI

Типичная архитектура:

```text id="n4p8c2"
FastAPI
   ↓
Dependency
   ↓
Session
   ↓
SQLAlchemy
   ↓
Engine
   ↓
Connection Pool
   ↓
PostgreSQL
```

Например:

```python
def get_session():
    with Session(engine) as session:
        yield session
```

Endpoint получает Session через dependency injection.

---

# 21. 📊 Core vs ORM

|                | SQLAlchemy Core       | SQLAlchemy ORM             |
| -------------- | --------------------- | -------------------------- |
| Уровень        | Ниже                  | Выше                       |
| Работа с SQL   | Expression API        | Через модели               |
| Python objects | Не обязательно        | Основной подход            |
| Relationships  | Нет ORM relationships | ✅                          |
| Identity Map   | ❌                     | ✅                          |
| Unit of Work   | ❌ как ORM-механизм    | ✅                          |
| Контроль SQL   | Высокий               | Тоже высокий, но через ORM |
| Абстракция     | Ниже                  | Выше                       |

Важно:

> ORM в SQLAlchemy построена поверх Core, поэтому это не две полностью независимые библиотеки.

---

# 22. ⚠️ SQLAlchemy не гарантирует хороший SQL

ORM позволяет писать удобный Python-код, но это не означает, что запрос автоматически оптимален.

Например, можно случайно получить:

```text id="j7m2c5"
N+1
```

или:

```text id="x3q8v1"
слишком большой JOIN
```

Поэтому backend-разработчик должен понимать SQL:

```text id="p5k9m2"
SQLAlchemy
    ↓
SQL
    ↓
EXPLAIN
    ↓
PostgreSQL
```

ORM не отменяет необходимость знать:

* `SELECT`;
* `JOIN`;
* индексы;
* транзакции;
* isolation levels;
* `EXPLAIN`;
* N+1;
* connection pooling.

---

# 23. 🧠 Основные объекты SQLAlchemy

```text id="c7m4p9"
Engine
│
├── Connection Pool
│
└── Connections

Session
│
├── ORM objects
├── Identity Map
├── Unit of Work
└── Transactions

Model
│
└── Table mapping

select()
│
└── SQL expression
```

---

# 24. 🔥 Как всё связывается вместе

```text id="v8q3m6"
                 FastAPI
                    │
                    ▼
                  Session
                    │
                    ▼
                SQLAlchemy
              ┌─────┴─────┐
              │           │
             ORM         Core
              │           │
              └─────┬─────┘
                    ▼
                  Engine
                    │
                    ▼
             Connection Pool
                    │
                    ▼
               DB Driver
                    │
                    ▼
                PostgreSQL
```

---

# 🎯 Главное

```text id="k4m8p2"
SQLAlchemy
│
├── Python library для работы с БД
│
├── Core
│    └── SQL Expression Language
│
├── ORM
│    ├── Models
│    ├── Session
│    ├── Relationships
│    ├── Identity Map
│    └── Unit of Work
│
├── Engine
│    └── Connection Pool
│
├── Transactions
│    ├── commit
│    └── rollback
│
└── Eager Loading
     ├── joinedload()
     └── selectinload()
```

### Связь с предыдущими темами

```text id="r9c3v7"
Connection Pool
       ↓
SQLAlchemy Engine
       ↓
PostgreSQL connections

N+1
       ↓
joinedload()
       ↓
JOIN Load

N+1
       ↓
selectinload()
       ↓
Selection Load
```

### Короткая формула

> **SQLAlchemy — это Python-библиотека для работы с реляционными БД, включающая Core и ORM. В ORM `Session` управляет объектами и транзакциями, `Engine` обеспечивает взаимодействие с БД и connection pool, а модели отображаются на таблицы. Для связанных данных SQLAlchemy предоставляет стратегии вроде `joinedload()` и `selectinload()`.**
