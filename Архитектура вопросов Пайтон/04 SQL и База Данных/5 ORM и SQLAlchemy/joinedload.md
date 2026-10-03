# joinedload (Join Load) в SQLAlchemy 🔗

## 🎯 Формула для собеседования

**`joinedload()` — это стратегия eager loading в SQLAlchemy, которая загружает связанные объекты через SQL `JOIN` в рамках основного запроса.**

```text id="q7m2x4"
joinedload()
    ↓
SQL JOIN
    ↓
eager loading
    ↓
связанный объект загружается заранее
    ↓
N+1 ↓
```

Основная идея:

> **`joinedload()` = заранее загрузить relationship через JOIN.**

---

## 🎤 Суперкоротко

В SQLAlchemy есть модели:

```python
from sqlalchemy.orm import Mapped, mapped_column, relationship


class User(Base):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(primary_key=True)

    profile: Mapped["Profile"] = relationship(
        back_populates="user",
    )


class Profile(Base):
    __tablename__ = "profiles"

    id: Mapped[int] = mapped_column(primary_key=True)
    user_id: Mapped[int] = mapped_column(ForeignKey("users.id"))

    user: Mapped["User"] = relationship(
        back_populates="profile",
    )
```

Без `joinedload()`:

```python
users = session.scalars(
    select(User)
).all()
```

При обращении:

```python
user.profile
```

может возникнуть дополнительный SQL-запрос.

С `joinedload()`:

```python
users = session.scalars(
    select(User).options(
        joinedload(User.profile)
    )
).all()
```

SQLAlchemy заранее загрузит `Profile` через `JOIN`.

---

# 1. Какую проблему решает

Главная проблема — **N+1 Query Problem**.

Например:

```python
users = session.scalars(
    select(User)
).all()

for user in users:
    print(user.profile)
```

Без eager loading возможна схема:

```text id="n3v8k2"
1 запрос → получить пользователей

N запросов:
    user 1 → profile
    user 2 → profile
    user 3 → profile
    ...
```

Итого:

```text id="m5q7c1"
1 + N запросов
```

С:

```python
joinedload(User.profile)
```

связанные профили загружаются вместе с пользователями.

---

# 2. Простой пример

```python
from sqlalchemy import select
from sqlalchemy.orm import joinedload

stmt = (
    select(User)
    .options(
        joinedload(User.profile)
    )
)

users = session.scalars(stmt).all()
```

Теперь:

```python
for user in users:
    print(user.profile)
```

не требует отдельного запроса для каждого `User`.

---

# 3. Что происходит в SQL

Концептуально SQLAlchemy формирует запрос с `JOIN`:

```sql
SELECT ...
FROM users
LEFT OUTER JOIN profiles
    ON users.id = profiles.user_id;
```

Важная деталь:

**тип JOIN зависит от настроек relationship / `joinedload()`.**

По умолчанию `joinedload()` обычно использует `LEFT OUTER JOIN`.

Можно запросить `INNER JOIN`:

```python
joinedload(
    User.profile,
    innerjoin=True,
)
```

---

# 4. Это eager loading

Есть два основных подхода к загрузке relationship.

### Lazy loading

Связанный объект загружается при обращении:

```python
user.profile
```

```text id="a7k3m9"
SELECT User
     ↓
user.profile
     ↓
SELECT Profile
```

### Eager loading

Связанный объект загружается заранее:

```python
joinedload(User.profile)
```

```text id="c4n8q2"
SELECT User + Profile
        ↓
user.profile
```

То есть:

```text id="x5m7r3"
Lazy loading
    → загрузить при обращении

Eager loading
    → загрузить заранее
```

---

# 5. joinedload() не меняет смысл основного запроса

Это важный момент SQLAlchemy.

Допустим:

```python
stmt = select(User).options(
    joinedload(User.profile)
)
```

`joinedload()` нужен для **загрузки relationship**, а не для изменения логики выборки `User`.

То есть:

```python
joinedload(User.profile)
```

не означает:

> «Выбери только пользователей, у которых есть Profile».

Он означает:

> «Если загружаешь User, загрузи их Profile вместе с ними».

Если нужно именно фильтровать по связанной таблице, обычно используется явный:

```python
join()
```

---

# 6. joinedload() vs join()

Это частый вопрос.

### `join()`

Используется для построения SQL-запроса и изменения набора результатов.

```python
stmt = (
    select(User)
    .join(User.profile)
)
```

Например, можно фильтровать:

```python
stmt = (
    select(User)
    .join(User.profile)
    .where(Profile.city == "Moscow")
)
```

### `joinedload()`

Используется для eager loading:

```python
stmt = (
    select(User)
    .options(
        joinedload(User.profile)
    )
)
```

Главное различие:

```text id="p8m4x2"
join()
    ↓
изменяет SQL-запрос / его семантику

joinedload()
    ↓
оптимизирует загрузку relationship
```

---

# 7. joinedload для ForeignKey

Например:

```python
class Order(Base):
    __tablename__ = "orders"

    id: Mapped[int] = mapped_column(primary_key=True)

    user_id: Mapped[int] = mapped_column(
        ForeignKey("users.id")
    )

    user: Mapped["User"] = relationship()
```

Запрос:

```python
stmt = (
    select(Order)
    .options(
        joinedload(Order.user)
    )
)

orders = session.scalars(stmt).all()
```

Теперь:

```python
for order in orders:
    print(order.user.name)
```

не вызывает отдельный запрос для каждого заказа.

---

# 8. joinedload для One-to-One

Например:

```python
stmt = select(User).options(
    joinedload(User.profile)
)
```

Для `OneToOne` это естественный сценарий:

```text id="u4n7c2"
User
 ↓
один Profile
```

JOIN позволяет получить обе сущности сразу.

---

# 9. joinedload для коллекций

`joinedload()` также может использоваться для relationship, возвращающего коллекцию.

Например:

```python
class User(Base):
    orders: Mapped[list["Order"]] = relationship()
```

Можно написать:

```python
stmt = select(User).options(
    joinedload(User.orders)
)
```

Но здесь появляется важная проблема.

---

# 10. Row Multiplication

Допустим:

```text id="k6r2m8"
User 1
 ├── Order 1
 ├── Order 2
 └── Order 3
```

SQL JOIN даст примерно:

```text id="q9v4n1"
User 1 | Order 1
User 1 | Order 2
User 1 | Order 3
```

То есть строк SQL становится больше.

Если:

```text id="m3x7c5"
100 users
×
100 orders
```

результат JOIN может значительно разрастись.

Это называется **row multiplication / row explosion**.

---

# 11. unique() при joinedload коллекции

В SQLAlchemy 2.x при `joinedload()` коллекционной relationship обычно необходимо вызвать:

```python
result = session.execute(
    select(User).options(
        joinedload(User.orders)
    )
)

users = result.unique().scalars().all()
```

Или:

```python
users = session.scalars(
    select(User).options(
        joinedload(User.orders)
    )
).unique().all()
```

Почему?

Потому что SQL JOIN возвращает несколько строк для одного `User`:

```text id="w5q8c2"
User 1 → Order 1
User 1 → Order 2
User 1 → Order 3
```

`unique()` убирает дублирование родительских ORM-объектов в результате.

---

# 12. joinedload vs selectinload

Это особенно важно для собеседования.

|                     | `joinedload()`     | `selectinload()`                 |
| ------------------- | ------------------ | -------------------------------- |
| Стратегия           | JOIN               | отдельный SELECT                 |
| Основной запрос     | расширяется JOIN   | остаётся отдельным               |
| Количество запросов | обычно 1           | обычно 2+                        |
| FK / OneToOne       | ✅                  | ✅                                |
| Коллекции           | ✅                  | ✅                                |
| Row multiplication  | возможна           | значительно меньше этой проблемы |
| Хорошо для          | одиночных связей   | коллекций                        |
| Аналог Django       | `select_related()` | `prefetch_related()`             |

Очень полезная аналогия:

```text id="c7m2v9"
Django:
select_related()
        ≈
SQLAlchemy:
joinedload()
```

И:

```text id="n4x8q1"
Django:
prefetch_related()
        ≈
SQLAlchemy:
selectinload()
```

Это не абсолютно идентичные API, но концептуально очень близкие стратегии.

---

# 13. Вложенные связи

Можно загружать relationship цепочкой:

```python
stmt = select(Order).options(
    joinedload(Order.user)
    .joinedload(User.profile)
)
```

Получается:

```text id="r6k3m8"
Order
  ↓
User
  ↓
Profile
```

SQLAlchemy построит необходимые JOIN.

---

# 14. Комбинирование joinedload и selectinload

Можно выбрать стратегию отдельно для разных связей.

Например:

```python
stmt = select(User).options(
    joinedload(User.profile),
    selectinload(User.orders),
)
```

Здесь:

```text id="p2v7n4"
User
 ├── profile
 │      ↓
 │   joinedload → JOIN
 │
 └── orders
        ↓
   selectinload → отдельный SELECT
```

Это часто лучше, чем пытаться загрузить всё через JOIN.

---

# 15. innerjoin=True

По умолчанию `joinedload()` обычно использует `LEFT OUTER JOIN`.

Можно использовать `INNER JOIN`:

```python
stmt = select(Order).options(
    joinedload(
        Order.user,
        innerjoin=True,
    )
)
```

Получается концептуально:

```sql
SELECT ...
FROM orders
JOIN users
    ON orders.user_id = users.id;
```

Это имеет смысл, если связанный объект гарантированно существует и такая семантика подходит запросу.

---

# 16. joinedload не всегда быстрее

Важно не превращать:

```python
joinedload()
```

в правило:

> «Всегда используй JOIN — это быстрее».

Нет.

Для небольших одиночных связей JOIN может быть очень удобен.

Для больших коллекций JOIN способен создать огромное количество строк.

Поэтому выбор зависит от:

* кардинальности relationship;
* количества связанных объектов;
* размера строк;
* количества JOIN;
* фильтрации;
* индексов;
* плана выполнения;
* реального workload.

---

# 17. Пример проблемы с большими коллекциями

Допустим:

```text id="v8m3q5"
User
 ↓
10 000 Orders
```

Запрос:

```python
select(User).options(
    joinedload(User.orders)
)
```

может вернуть 10 000 строк для одного пользователя.

В таком случае часто более подходящая стратегия:

```python
select(User).options(
    selectinload(User.orders)
)
```

То есть:

```text id="j5r2n8"
joinedload
    ↓
один большой JOIN

selectinload
    ↓
User SELECT
    +
Order SELECT WHERE user_id IN (...)
```

---

# 18. Проверка SQL

Для анализа можно посмотреть SQLAlchemy statement:

```python
stmt = select(User).options(
    joinedload(User.profile)
)

print(stmt)
```

А для реального анализа производительности нужно смотреть SQL, который ушёл в БД, и план выполнения PostgreSQL.

Например:

```sql
EXPLAIN ANALYZE
...
```

---

# 19. Типичная ошибка

### ❌ Использовать joinedload для всего

```python
select(User).options(
    joinedload(User.profile),
    joinedload(User.orders),
    joinedload(User.groups),
    joinedload(User.permissions),
)
```

Если это несколько коллекций, JOIN может создать огромное количество комбинаций строк.

Например:

```text id="f7n2c4"
1 User
× 100 Orders
× 20 Groups
× 10 Permissions

= потенциально 20 000 строк
```

Поэтому для коллекций часто выбирают:

```python
selectinload()
```

---

# 20. Типичный backend-пример

Допустим, endpoint возвращает заказы:

```python
stmt = (
    select(Order)
    .options(
        joinedload(Order.user),
    )
    .where(
        Order.status == "paid",
    )
)

orders = session.scalars(stmt).all()
```

Теперь при сериализации:

```python
for order in orders:
    print(
        order.id,
        order.user.name,
    )
```

не возникает N+1 для `user`.

---

# 21. Аналогия с Django

Если ты знаешь Django ORM, запомнить очень просто:

```text id="u2m8q5"
Django
──────────────
select_related()
       ↓
     JOIN
```

примерно соответствует:

```text id="c5r7n3"
SQLAlchemy
──────────────
joinedload()
       ↓
     JOIN
```

И:

```text id="q8x4m2"
Django
prefetch_related()
       ↓
отдельные SELECT
```

примерно соответствует:

```text id="n6v3k9"
SQLAlchemy
selectinload()
       ↓
отдельные SELECT
```

---

# 🔥 Главное для собеседования

### Что такое `joinedload()`?

> `joinedload()` — стратегия eager loading в SQLAlchemy, которая загружает связанные объекты через SQL JOIN, чтобы избежать дополнительных запросов при обращении к relationship.

### Какую проблему решает?

**N+1 Query Problem.**

### Основной механизм?

```text id="x4q7m1"
joinedload()
    ↓
SQL JOIN
```

### Для каких связей?

Хорошо подходит для:

* `ForeignKey`;
* `OneToOne`;
* небольших коллекций, если JOIN не приводит к чрезмерному росту результата.

### Что важно для коллекций?

`JOIN` может размножать строки, поэтому при `joinedload()` коллекции в SQLAlchemy 2.x обычно нужен:

```python
result.unique()
```

### Чем отличается `joinedload()` от `join()`?

```text id="m9c2v6"
join()
    ↓
формирует JOIN
    ↓
может менять набор результатов / использоваться для фильтрации
```

```text id="r5n8k3"
joinedload()
    ↓
eager loading relationship
    ↓
не предназначен для изменения семантики основного результата
```

### Чем отличается от `selectinload()`?

```text id="p7x3m9"
joinedload()
    → JOIN
    → обычно один SQL-запрос
```

```text id="k2v6q8"
selectinload()
    → отдельный SELECT
    → WHERE ... IN (...)
    → часто лучше для коллекций
```

---

## 🧠 Финальная формула

```text id="z8m4r2"
              SQLAlchemy ORM
                    │
             relationship
                    │
          ┌─────────┴─────────┐
          ↓                   ↓
    joinedload()         selectinload()
          ↓                   ↓
       JOIN              SELECT ... IN
          ↓                   ↓
  eager loading        eager loading
          ↓                   ↓
    N+1 ↓              N+1 ↓
```

**Короткий ответ на собеседовании:**

> `joinedload()` — это стратегия eager loading в SQLAlchemy. Она загружает relationship через SQL JOIN и позволяет избежать N+1 запросов. Особенно удобна для `ForeignKey` и `OneToOne`. Для больших коллекций нужно учитывать row multiplication, поэтому часто вместо `joinedload()` используют [[selectinload]]()`.
