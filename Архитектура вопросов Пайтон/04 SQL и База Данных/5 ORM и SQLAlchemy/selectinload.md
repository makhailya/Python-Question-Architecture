# selectinload в SQLAlchemy 🔗

**Короткий ответ на собеседовании:**

> `selectinload()` — стратегия eager loading в SQLAlchemy. Она сначала получает основные объекты, затем отдельным запросом загружает связанные объекты через `WHERE ... IN (...)` и связывает их в ORM. Особенно полезна для коллекций [[One-to-Many]] и [[Many-to-Many]], где [[joinedload]]() может привести к большому количеству строк из-за [[JOIN]].

## 🎯 Формула для собеседования

**`selectinload()` — это стратегия eager loading в SQLAlchemy, которая загружает связанные объекты отдельным SQL-запросом с `WHERE ... IN (...)`, используя идентификаторы уже загруженных объектов.**

```text
selectinload()
      ↓
1-й SELECT → основные объекты
      ↓
собираем их PK
      ↓
2-й SELECT ... WHERE FK IN (...)
      ↓
SQLAlchemy связывает объекты в Python
      ↓
N+1 ↓
```

Особенно хорошо подходит для **коллекций**: `One-to-Many` и `Many-to-Many`.

---

## 🎤 Суперкоротко

Допустим:

```python
class User(Base):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(primary_key=True)

    orders: Mapped[list["Order"]] = relationship(
        back_populates="user",
    )


class Order(Base):
    __tablename__ = "orders"

    id: Mapped[int] = mapped_column(primary_key=True)

    user_id: Mapped[int] = mapped_column(
        ForeignKey("users.id"),
    )

    user: Mapped["User"] = relationship(
        back_populates="orders",
    )
```

Без eager loading:

```python
users = session.scalars(
    select(User)
).all()

for user in users:
    print(user.orders)
```

может возникнуть:

```text
1 запрос → users
N запросов → orders для каждого user
```

С `selectinload()`:

```python
users = session.scalars(
    select(User).options(
        selectinload(User.orders)
    )
).all()
```

SQLAlchemy делает примерно:

```sql
SELECT *
FROM users;

SELECT *
FROM orders
WHERE user_id IN (1, 2, 3, 4);
```

---

# 1. Что такое eager loading

**Eager loading** — предварительная загрузка связанных объектов.

Без eager loading:

```text
User
 ↓
обращение к orders
 ↓
дополнительный SELECT
```

С `selectinload()`:

```text
SELECT users
      ↓
получили User IDs
      ↓
SELECT orders WHERE user_id IN (...)
      ↓
orders уже загружены
```

Поэтому при дальнейшем:

```python
user.orders
```

SQLAlchemy использует уже загруженные данные.

---

# 2. Какую проблему решает

Основная проблема — **N+1 Query Problem**.

Например, получили 100 пользователей:

```python
users = session.scalars(
    select(User)
).all()
```

А затем:

```python
for user in users:
    print(user.orders)
```

Без eager loading может получиться:

```text
1 + 100 = 101 запрос
```

С:

```python
selectinload(User.orders)
```

обычно:

```text
1 запрос → users
1 запрос → orders
------------------
2 запроса
```

То есть вместо N отдельных запросов выполняется запрос для всей коллекции.

---

# 3. Почему называется selectinload

Название можно разобрать буквально:

```text
select + in + load
```

То есть:

```text
SELECT
  ...
WHERE related_id IN (...)
```

Например:

```sql
SELECT *
FROM orders
WHERE user_id IN (10, 15, 27, 31);
```

Именно поэтому стратегия особенно хорошо подходит для загрузки коллекций связанных объектов.

---

# 4. Как работает по шагам

Допустим, запросили:

```python
users = session.scalars(
    select(User).options(
        selectinload(User.orders)
    )
).all()
```

### Шаг 1

SQLAlchemy получает пользователей:

```sql
SELECT id, name
FROM users;
```

Результат:

```text
id
--
1
2
3
4
```

### Шаг 2

SQLAlchemy собирает их идентификаторы.

```text
1, 2, 3, 4
```

### Шаг 3

Выполняет дополнительный запрос:

```sql
SELECT *
FROM orders
WHERE user_id IN (1, 2, 3, 4);
```

### Шаг 4

SQLAlchemy сопоставляет:

```text
User 1 → Orders 1, 2
User 2 → Orders 3
User 3 → Orders 4, 5, 6
User 4 → нет заказов
```

После этого:

```python
user.orders
```

доступно без отдельного запроса.

---

# 5. Основное применение — коллекции

Например:

```text
User
 ├── Order
 ├── Order
 └── Order
```

или:

```text
Author
 ├── Book
 ├── Book
 └── Book
```

Для такого случая:

```python
selectinload(Author.books)
```

часто является естественным выбором.

---

# 6. One-to-Many

Например:

```python
class Author(Base):
    books: Mapped[list["Book"]] = relationship()
```

Запрос:

```python
authors = session.scalars(
    select(Author).options(
        selectinload(Author.books)
    )
).all()
```

Получаем:

```text
SELECT authors ...

SELECT books
WHERE author_id IN (...)
```

---

# 7. Many-to-Many

`selectinload()` также подходит для `Many-to-Many`.

Например:

```python
class Student(Base):
    courses: Mapped[list["Course"]] = relationship(
        secondary=student_course,
    )
```

Можно:

```python
students = session.scalars(
    select(Student).options(
        selectinload(Student.courses)
    )
).all()
```

SQLAlchemy выполнит необходимые дополнительные запросы, включая работу с промежуточной таблицей.

---

# 8. selectinload vs joinedload

Это один из главных вопросов на собеседовании.

### `joinedload()`

```text
основной SELECT
      ↓
JOIN
      ↓
связанные данные
```

### `selectinload()`

```text
основной SELECT
      ↓
получили IDs
      ↓
отдельный SELECT
WHERE ... IN (...)
```

Сравнение:

|                    | `joinedload()`      | `selectinload()`        |
| ------------------ | ------------------- | ----------------------- |
| Стратегия          | `JOIN`              | отдельный `SELECT`      |
| Запросов           | обычно 1            | обычно 2+               |
| Коллекции          | можно, но осторожно | ⭐ очень хороший вариант |
| Row multiplication | возможна            | существенно меньше      |
| FK / OneToOne      | хорошо              | хорошо                  |
| One-to-Many        | осторожно           | часто предпочтительнее  |
| Many-to-Many       | возможно            | часто предпочтительнее  |

---

# 9. Почему selectinload хорош для коллекций

Допустим:

```text
User 1 → 100 Orders
User 2 → 200 Orders
User 3 → 150 Orders
```

`joinedload()` может создать большой JOIN:

```text
User 1 | Order 1
User 1 | Order 2
User 1 | Order 3
...
User 2 | Order 101
...
```

Получается много строк, содержащих повторяющиеся данные пользователей.

`selectinload()` делает:

```text
SELECT users
```

и отдельно:

```text
SELECT orders
WHERE user_id IN (1, 2, 3)
```

Поэтому нет необходимости размножать строки основного результата через JOIN.

---

# 10. Важное отличие от joinedload: unique()

При `joinedload()` коллекции SQL JOIN может создавать несколько строк на один родительский объект.

Поэтому в SQLAlchemy 2.x часто требуется:

```python
users = session.scalars(
    select(User).options(
        joinedload(User.orders)
    )
).unique().all()
```

При `selectinload()` такой проблемы с дублированием основного результата из-за JOIN нет:

```python
users = session.scalars(
    select(User).options(
        selectinload(User.orders)
    )
).all()
```

Это одно из практических преимуществ `selectinload()` для коллекций.

---

# 11. Вложенные relationship

Можно загружать несколько уровней:

```python
stmt = select(User).options(
    selectinload(User.orders)
        .selectinload(Order.items)
)
```

Получается:

```text
User
 ↓
Orders
 ↓
Items
```

SQLAlchemy выполняет необходимые SELECT-запросы для каждого уровня.

---

# 12. Комбинирование с joinedload

Можно использовать разные стратегии одновременно.

Например:

```python
stmt = select(User).options(
    joinedload(User.profile),
    selectinload(User.orders),
)
```

Здесь:

```text
User
 ├── profile
 │      ↓
 │   joinedload()
 │      ↓
 │     JOIN
 │
 └── orders
        ↓
   selectinload()
        ↓
   SELECT ... IN (...)
```

Это вполне нормальный подход.

---

# 13. selectinload и фильтрация

Сам `selectinload()` предназначен для загрузки relationship, а не для фильтрации основного результата.

Например:

```python
stmt = select(User).options(
    selectinload(User.orders)
)
```

означает:

> Получить пользователей и заранее загрузить их заказы.

Если нужно выбрать только пользователей с определёнными заказами, логика запроса строится отдельно:

```python
stmt = (
    select(User)
    .join(User.orders)
    .where(Order.status == "paid")
    .options(
        selectinload(User.orders)
    )
)
```

Здесь:

```text
join()
    ↓
фильтрация

selectinload()
    ↓
eager loading
```

Это разные задачи.

---

# 14. selectinload не изменяет смысл основного результата

Как и `joinedload()`, `selectinload()` является **loader strategy**.

Его назначение:

```text
как загрузить relationship
```

а не:

```text
какие User должны попасть в результат
```

Это важная концепция SQLAlchemy.

---

# 15. Chunking / большие IN

Если объектов очень много, список:

```sql
WHERE user_id IN (...)
```

может стать большим.

SQLAlchemy может разбивать загрузку связанных объектов на несколько запросов в зависимости от используемой стратегии и ограничений.

Поэтому нельзя утверждать:

> `selectinload()` всегда выполняет ровно два SQL-запроса.

Корректнее:

> Он выполняет отдельные SELECT-запросы для связанных объектов, используя `IN` по идентификаторам родительских объектов; при необходимости запросы могут быть разбиты на несколько частей.

---

# 16. Когда selectinload может быть плохим выбором

Несмотря на преимущества, это не универсальное решение.

Проблемы возможны при:

* очень большом количестве parent-объектов;
* очень больших коллекциях;
* огромных `IN`;
* большом количестве уровней eager loading;
* ненужной предварительной загрузке данных.

Например:

```text
1 000 000 Users
        ↓
10 000 000 Orders
```

Не стоит автоматически делать:

```python
selectinload(User.orders)
```

и загружать всё.

Нужно учитывать pagination, фильтрацию и реальный workload.

---

# 17. Pagination

Если API возвращает пользователей страницами:

```python
stmt = (
    select(User)
    .options(
        selectinload(User.orders)
    )
    .limit(50)
    .offset(100)
)
```

то `selectinload()` будет загружать связанные данные для **выбранного набора пользователей**, а не для всей таблицы.

Это одна из причин, почему стратегия хорошо сочетается с pagination.

---

# 18. Lazy loading vs selectinload

### Lazy

```python
users = session.scalars(
    select(User)
).all()
```

Затем:

```python
user.orders
```

может вызвать SQL.

### Selectinload

```python
users = session.scalars(
    select(User).options(
        selectinload(User.orders)
    )
).all()
```

`orders` загружаются заранее.

```text
lazy:
    SELECT users
       ↓
    user.orders
       ↓
    SELECT orders

selectinload:
    SELECT users
       ↓
    SELECT orders WHERE ... IN (...)
       ↓
    user.orders
```

---

# 19. Аналогия с Django

Если ты уже знаешь Django ORM, запомнить удобно:

```text
Django
───────────────
select_related()
       ↓
JOIN
```

примерно соответствует:

```text
SQLAlchemy
───────────────
joinedload()
       ↓
JOIN
```

А:

```text
Django
───────────────
prefetch_related()
       ↓
отдельные SELECT
```

концептуально ближе всего к:

```text
SQLAlchemy
───────────────
selectinload()
       ↓
SELECT ... WHERE IN (...)
```

Это очень полезная связка для собеседования.

---

# 20. Три стратегии в одном месте

```text
                 SQLAlchemy relationship
                         │
            ┌────────────┼────────────┐
            ↓            ↓            ↓
          lazy       joinedload   selectinload
            │            │            │
       при обращении     JOIN       SELECT IN
            │            │            │
       N+1 риск       eager        eager
                       loading      loading
```

### Lazy loading

```python
select(User)
```

Связь загружается при обращении.

### joinedload

```python
select(User).options(
    joinedload(User.profile)
)
```

Связь через JOIN.

### selectinload

```python
select(User).options(
    selectinload(User.orders)
)
```

Связь отдельным SELECT через `IN`.

---

# 🔥 Главное для собеседования

### Что такое `selectinload()`?

> `selectinload()` — стратегия eager loading в SQLAlchemy, которая загружает связанные объекты отдельным SELECT-запросом с `WHERE ... IN (...)`, используя идентификаторы уже загруженных родительских объектов.

### Какую проблему решает?

**N+1 Query Problem.**

### Как работает?

```text
SELECT parents
      ↓
получили IDs
      ↓
SELECT children
WHERE parent_id IN (...)
      ↓
SQLAlchemy связывает объекты
```

### Для чего особенно подходит?

* `One-to-Many`;
* `Many-to-Many`;
* большие коллекции, где JOIN может вызвать сильное размножение строк.

### `selectinload()` vs `joinedload()`

```text
joinedload()
    → JOIN
    → один основной запрос
    → возможен row multiplication
```

```text
selectinload()
    → отдельный SELECT
    → WHERE ... IN (...)
    → меньше проблем с row multiplication
```

### Аналогия с Django

```text
Django select_related()
        ≈
SQLAlchemy joinedload()
```

```text
Django prefetch_related()
        ≈
SQLAlchemy selectinload()
```

---

## 🧠 Финальная формула

```text
selectinload()
      ↓
Eager Loading
      ↓
SELECT родителей
      ↓
получить их IDs
      ↓
SELECT связанных объектов
WHERE foreign_key IN (...)
      ↓
связать объекты в Python
      ↓
N+1 ↓
```

**Короткий ответ на собеседовании:**

> `selectinload()` — стратегия eager loading в SQLAlchemy. Она сначала получает основные объекты, затем отдельным запросом загружает связанные объекты через `WHERE ... IN (...)` и связывает их в ORM. Особенно полезна для коллекций `One-to-Many` и `Many-to-Many`, где [[joinedload]]()` может привести к большому количеству строк из-за JOIN.
