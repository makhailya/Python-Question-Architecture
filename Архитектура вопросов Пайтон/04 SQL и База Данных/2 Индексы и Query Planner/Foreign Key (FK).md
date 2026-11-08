# 🔗 Foreign Key (FK) — внешний ключ в SQL

## 🎤 Короткий ответ

**Foreign Key (внешний ключ)** — это ограничение в реляционной базе данных, которое обеспечивает **ссылочную целостность (Referential Integrity)** между таблицами.

Внешний ключ указывает на первичный или другой подходящий уникальный ключ в связанной таблице.

Например, в таблице `orders` поле `user_id` может ссылаться на `users.id`.

Foreign Key гарантирует, что заказ не будет ссылаться на несуществующего пользователя, если значение внешнего ключа не равно `NULL`.

Также внешний ключ определяет, что должно происходить со связанными записями при удалении или изменении родительской записи.

## 🗣️ Ответ на собеседовании

Foreign Key — это внешний ключ, который используется для связи таблиц и обеспечения ссылочной целостности данных.

Например, у меня есть таблица пользователей `users` и таблица заказов `orders`. В таблице заказов хранится `user_id`, который ссылается на `users.id`.

Если я попытаюсь создать заказ с несуществующим `user_id`, PostgreSQL не позволит выполнить такую операцию.

При этом внешний ключ не обязательно должен быть уникальным: один пользователь может иметь много заказов с одинаковым `user_id`.

Для удаления связанных записей можно настроить разные стратегии: `CASCADE`, `SET NULL`, `RESTRICT` или `NO ACTION`.

В PostgreSQL Foreign Key не создаёт индекс на ссылающемся столбце автоматически. Поэтому, если по этому полю часто выполняются JOIN или фильтрация, индекс обычно добавляют отдельно.

Foreign Key — это прежде всего механизм обеспечения целостности данных, а не просто способ соединять таблицы через JOIN.

## 🧭 Где я нахожусь

```text
04 SQL и База Данных
├── 01 SQL
│   ├── DDL / DML / DCL / TCL
│   ├── CREATE TABLE
│   ├── Ограничения (Constraints)
│   │   ├── PRIMARY KEY
│   │   ├── FOREIGN KEY ← Я здесь
│   │   ├── UNIQUE
│   │   ├── NOT NULL
│   │   └── CHECK
│   │
│   ├── Связи между таблицами
│   │   ├── One-to-One (1:1)
│   │   ├── One-to-Many (1:N)
│   │   └── Many-to-Many (M:N)
│   │
│   └── JOIN
│       ├── INNER JOIN
│       └── LEFT JOIN
│
├── 02 Индексы и Query Planner
│   └── Индексы на Foreign Key
│
└── 03 Транзакции и Concurrency
    └── Ссылочная целостность
```

## 📚 Разбор поглубже

### 1. Что такое Foreign Key

Представим две таблицы.

**users — пользователи**

| id (PK) | name |
|---|---|
| 1 | Иван |
| 2 | Анна |
| 3 | Пётр |

**orders — заказы**

| id (PK) | user_id (FK) | amount |
|---|---|---|
| 101 | 1 | 1500 |
| 102 | 1 | 2300 |
| 103 | 2 | 800 |

Поле `orders.user_id` ссылается на `users.id`.

Схема:

```text
users                         orders
┌─────────────────┐           ┌─────────────────┐
│ id (PK)         │◄──────────│ user_id (FK)    │
│ name            │           │ id (PK)         │
└─────────────────┘           │ amount          │
                              └─────────────────┘

          Один пользователь → много заказов
```

Это связь **One-to-Many (1:N)**.

Один пользователь может иметь несколько заказов, но каждый заказ в этой модели принадлежит одному пользователю.

### 2. Создание Foreign Key

Пример PostgreSQL:

```sql
CREATE TABLE users (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name TEXT NOT NULL
);

CREATE TABLE orders (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    user_id BIGINT NOT NULL REFERENCES users(id),
    amount NUMERIC(10, 2) NOT NULL
);
```

Разберём ключевую строку:

```sql
user_id BIGINT NOT NULL REFERENCES users(id)
```

- `user_id` — столбец в таблице заказов.
- `BIGINT` — тип данных идентификатора.
- `NOT NULL` — каждый заказ обязан иметь пользователя.
- `REFERENCES users(id)` — значение должно ссылаться на существующую запись таблицы `users`.

Альтернативный синтаксис:

```sql
CREATE TABLE orders (
    id BIGINT PRIMARY KEY,
    user_id BIGINT NOT NULL,
    amount NUMERIC(10, 2) NOT NULL,

    CONSTRAINT fk_orders_user
        FOREIGN KEY (user_id)
        REFERENCES users(id)
);
```

Здесь мы явно задаём имя ограничения `fk_orders_user`.

Это удобно при миграциях и диагностике ошибок.

### 3. Как Foreign Key защищает данные

Сначала создадим пользователя:

```sql
INSERT INTO users (name)
VALUES ('Иван')
RETURNING id;
```

Предположим, PostgreSQL вернул `id = 1`.

Теперь добавим заказ:

```sql
INSERT INTO orders (user_id, amount)
VALUES (1, 1500);
```

Запрос выполнится успешно.

Но если выполнить:

```sql
INSERT INTO orders (user_id, amount)
VALUES (999, 1500);
```

PostgreSQL вернёт ошибку внешнего ключа, если пользователя с `id = 999` нет.

Пример сообщения:

```text
ERROR: insert or update on table "orders"
violates foreign key constraint "orders_user_id_fkey"

DETAIL: Key (user_id)=(999) is not present
in table "users".
```

Таким образом, Foreign Key предотвращает появление некорректных ссылок.

### 4. Primary Key vs Foreign Key

| Характеристика | Primary Key | Foreign Key |
|---|---|---|
| Назначение | Идентификация записи | Ссылка на другую запись |
| Уникальность | Обязательна | Не обязательна |
| NULL | Запрещён | Разрешён, если нет NOT NULL |
| Количество на таблицу | Один | Может быть несколько |
| Составной ключ | Да | Да |
| Индекс в PostgreSQL | Создаётся автоматически | На ссылающемся столбце не создаётся автоматически |

**Главное отличие:**

`PRIMARY KEY` отвечает на вопрос: «Какая это запись?»

`FOREIGN KEY` отвечает на вопрос: «С какой записью она связана?»

### 5. Что происходит при удалении родительской записи

Предположим, пользователь с `id = 1` имеет несколько заказов.

Что произойдёт при выполнении:

```sql
DELETE FROM users
WHERE id = 1;
```

Это зависит от настройки Foreign Key.

В PostgreSQL доступны следующие основные варианты.

| Действие | Поведение |
|---|---|
| `ON DELETE NO ACTION` | По умолчанию запрещает нарушение ссылки при проверке ограничения |
| `ON DELETE RESTRICT` | Не позволяет удалить родительскую запись, пока существуют ссылки |
| `ON DELETE CASCADE` | Удаляет связанные дочерние записи |
| `ON DELETE SET NULL` | Устанавливает внешний ключ в NULL |
| `ON DELETE SET DEFAULT` | Устанавливает значение по умолчанию |

Разберём подробнее.

### 6. ON DELETE CASCADE

```sql
CREATE TABLE orders (
    id BIGINT PRIMARY KEY,
    user_id BIGINT NOT NULL
        REFERENCES users(id)
        ON DELETE CASCADE
);
```

Если удалить пользователя:

```sql
DELETE FROM users
WHERE id = 1;
```

PostgreSQL автоматически удалит связанные заказы.

```text
DELETE user id=1
       |
       v
Найти связанные orders
       |
       v
Удалить связанные orders
       |
       v
Удалить пользователя
```

**Когда полезно:**

Когда дочерние записи не имеют самостоятельного смысла без родительской.

Например, элементы временной корзины.

**Когда опасно:**

Если каскадное удаление может уничтожить важные бизнес-данные.

Например, при удалении пользователя интернет-магазина обычно не стоит автоматически удалять всю историю оплаченных заказов.

### 7. ON DELETE SET NULL

```sql
CREATE TABLE orders (
    id BIGINT PRIMARY KEY,
    user_id BIGINT
        REFERENCES users(id)
        ON DELETE SET NULL
);
```

После удаления пользователя его заказы останутся, но значение `user_id` станет `NULL`.

Пример:

| id | user_id | amount |
|---|---|---|
| 101 | NULL | 1500 |
| 102 | NULL | 2300 |

Важно: столбец должен допускать `NULL`.

Если одновременно указать `NOT NULL`, попытка установить `NULL` при удалении пользователя завершится ошибкой.

### 8. RESTRICT vs NO ACTION

Эти варианты похожи, но между ними есть нюанс.

**RESTRICT** запрещает удаление или изменение родительской записи, если на неё существуют ссылки.

**NO ACTION** — поведение по умолчанию. Проверка выполняется в соответствии с моментом проверки ограничения.

В PostgreSQL ограничение с `NO ACTION` может быть объявлено `DEFERRABLE`, что позволяет отложить проверку до более позднего момента транзакции.

`RESTRICT` не допускает такого откладывания соответствующей проверки.

На обычном собеседовании достаточно сказать:

> По умолчанию Foreign Key не позволяет нарушить ссылочную целостность. `CASCADE` распространяет удаление на дочерние записи, а `SET NULL` обнуляет ссылки.

### 9. ON UPDATE CASCADE

Внешний ключ может реагировать не только на удаление, но и на изменение ключа родительской записи.

```sql
CREATE TABLE orders (
    id BIGINT PRIMARY KEY,
    user_id BIGINT
        REFERENCES users(id)
        ON UPDATE CASCADE
);
```

Если изменить идентификатор пользователя:

```sql
UPDATE users
SET id = 100
WHERE id = 1;
```

PostgreSQL автоматически обновит соответствующие значения `orders.user_id` с `1` на `100`.

На практике первичные ключи обычно стараются не изменять.

Но понимать механизм `ON UPDATE CASCADE` полезно.

### 10. Foreign Key и JOIN

Foreign Key и JOIN часто используются вместе, но это разные механизмы.

**Foreign Key** обеспечивает целостность ссылок.

**JOIN** объединяет данные из таблиц при выполнении запроса.

Пример:

```sql
SELECT
    users.name,
    orders.id,
    orders.amount
FROM orders
JOIN users
    ON orders.user_id = users.id;
```

Результат:

| name | id | amount |
|---|---|---|
| Иван | 101 | 1500 |
| Иван | 102 | 2300 |
| Анна | 103 | 800 |

Важно: SQL позволяет выполнять JOIN даже без объявленного Foreign Key.

Но без ограничения база данных не гарантирует корректность ссылок.

### 11. Foreign Key и индексы

В PostgreSQL при создании Foreign Key **индекс на ссылающемся столбце автоматически не создаётся**.

Например:

```sql
CREATE TABLE orders (
    id BIGINT PRIMARY KEY,
    user_id BIGINT REFERENCES users(id)
);
```

Здесь автоматически создаётся индекс для `orders.id`, но не обязательно для `orders.user_id`.

Если часто выполняются запросы:

```sql
SELECT *
FROM orders
WHERE user_id = 10;
```

может быть полезен индекс:

```sql
CREATE INDEX idx_orders_user_id
ON orders(user_id);
```

Индекс также может ускорить проверки ссылок при удалении или изменении ключа пользователя.

Однако индекс имеет стоимость: занимает место и добавляет накладные расходы при изменении данных.

Поэтому решение о создании индекса зависит от запросов и нагрузки.

### 12. One-to-One, One-to-Many, Many-to-Many

Foreign Key участвует в моделировании разных типов связей.

**One-to-One (1:1)**

Один пользователь — один профиль.

```sql
CREATE TABLE profiles (
    id BIGINT PRIMARY KEY,
    user_id BIGINT NOT NULL UNIQUE
        REFERENCES users(id)
);
```

`UNIQUE` запрещает создание двух профилей для одного пользователя.

**One-to-Many (1:N)**

Один пользователь — много заказов.

```sql
CREATE TABLE orders (
    id BIGINT PRIMARY KEY,
    user_id BIGINT NOT NULL
        REFERENCES users(id)
);
```

Здесь `user_id` может повторяться.

**Many-to-Many (M:N)**

Один студент может посещать несколько курсов, а на одном курсе может быть несколько студентов.

```sql
CREATE TABLE students (
    id BIGINT PRIMARY KEY
);

CREATE TABLE courses (
    id BIGINT PRIMARY KEY
);

CREATE TABLE student_courses (
    student_id BIGINT REFERENCES students(id),
    course_id BIGINT REFERENCES courses(id),

    PRIMARY KEY (student_id, course_id)
);
```

Таблица `student_courses` связывает две сущности.

Составной Primary Key предотвращает дублирование одной и той же пары.

### 13. Foreign Key в SQLAlchemy 2.x

Пример ORM-моделей:

```python
from sqlalchemy import BigInteger, ForeignKey, Identity, String
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column, relationship


class Base(DeclarativeBase):
    pass


class User(Base):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(
        BigInteger,
        Identity(),
        primary_key=True,
    )

    name: Mapped[str] = mapped_column(String(100))

    orders: Mapped[list["Order"]] = relationship(
        back_populates="user"
    )


class Order(Base):
    __tablename__ = "orders"

    id: Mapped[int] = mapped_column(
        BigInteger,
        Identity(),
        primary_key=True,
    )

    user_id: Mapped[int] = mapped_column(
        BigInteger,
        ForeignKey("users.id"),
        nullable=False,
    )

    user: Mapped["User"] = relationship(
        back_populates="orders"
    )
```

Разберём ключевые элементы.

**`ForeignKey("users.id")`**

Создаёт описание внешнего ключа на уровне схемы БД.

**`relationship()`**

Описывает ORM-связь между Python-объектами.

**`back_populates`**

Связывает два направления ORM-отношения.

Важно различать:

- `ForeignKey` — ограничение базы данных.
- `relationship` — механизм SQLAlchemy для работы со связанными объектами.

Сам по себе `relationship()` не заменяет Foreign Key в базе данных.

### 14. Foreign Key и транзакции

Представим, что нужно создать пользователя и его первый заказ.

Правильно выполнять связанные изменения в одной транзакции.

```sql
BEGIN;

INSERT INTO users (id, name)
OVERRIDING SYSTEM VALUE
VALUES (100, 'Иван');

INSERT INTO orders (user_id, amount)
VALUES (100, 1500);

COMMIT;
```

Здесь предполагается, что таблица `users` использует `GENERATED ALWAYS AS IDENTITY`, а идентификатор `100` свободен. `OVERRIDING SYSTEM VALUE` позволяет явно указать значение такого identity-столбца.

Если операция завершится ошибкой, транзакцию можно откатить.

```sql
ROLLBACK;
```

Foreign Key обеспечивает корректность ссылок, а транзакция — атомарность группы изменений.

Это разные, но взаимодополняющие механизмы.

### 15. Составной Foreign Key

Внешний ключ может состоять из нескольких столбцов.

```sql
CREATE TABLE departments (
    company_id BIGINT,
    department_id BIGINT,
    name TEXT,

    PRIMARY KEY (company_id, department_id)
);

CREATE TABLE employees (
    id BIGINT PRIMARY KEY,
    company_id BIGINT NOT NULL,
    department_id BIGINT NOT NULL,

    FOREIGN KEY (company_id, department_id)
        REFERENCES departments(company_id, department_id)
);
```

Здесь сотрудник ссылается на отдел внутри конкретной компании.

База проверяет существование всей пары:

```text
(company_id, department_id)
```

Это полезно при моделировании составных идентификаторов.

### 16. Типичные ошибки

**Ошибка 1. Считать Foreign Key уникальным.**

Внешний ключ может повторяться. Это необходимо для связи One-to-Many.

**Ошибка 2. Считать Foreign Key обязательным NOT NULL.**

Если столбец допускает `NULL`, ссылка может отсутствовать.

**Ошибка 3. Считать, что Foreign Key автоматически создаёт индекс в PostgreSQL.**

На ссылающемся столбце индекс нужно создавать отдельно, когда это оправдано.

**Ошибка 4. Путать Foreign Key и JOIN.**

Foreign Key проверяет целостность данных, JOIN объединяет строки в запросе.

**Ошибка 5. Использовать CASCADE без понимания последствий.**

Каскадное удаление может удалить большое количество связанных записей.

**Ошибка 6. Считать, что Foreign Key всегда ссылается только на Primary Key.**

В PostgreSQL он также может ссылаться на подходящий уникальный ключ.

**Ошибка 7. Считать Foreign Key механизмом связи между микросервисами.**

Обычный Foreign Key действует внутри одной базы данных. Он не обеспечивает ссылочную целостность между независимыми базами разных микросервисов.

**Ошибка 8. Считать, что SQLAlchemy relationship создаёт Foreign Key автоматически.**

`relationship()` описывает ORM-отношение, а ограничение базы задаётся через `ForeignKey` или `ForeignKeyConstraint`.

## 🎤 Вопросы на собеседовании

**1. Что такое Foreign Key?**

Ограничение, обеспечивающее ссылочную целостность между таблицами.

**2. Чем Foreign Key отличается от Primary Key?**

Primary Key идентифицирует запись, а Foreign Key ссылается на ключ связанной записи.

**3. Может ли Foreign Key содержать NULL?**

Да, если столбец допускает `NULL`.

**4. Может ли Foreign Key повторяться?**

Да. Например, несколько заказов могут принадлежать одному пользователю.

**5. Что делает ON DELETE CASCADE?**

Автоматически удаляет связанные дочерние записи при удалении родительской.

**6. Что делает ON DELETE SET NULL?**

Устанавливает `NULL` в ссылающемся столбце при удалении родительской записи.

**7. Что происходит при удалении родительской записи по умолчанию?**

В PostgreSQL используется `NO ACTION`. Если удаление нарушает ссылочную целостность, операция не проходит проверку ограничения.

**8. Создаётся ли индекс на Foreign Key автоматически?**

В PostgreSQL на ссылающемся столбце — нет.

**9. Может ли Foreign Key ссылаться на UNIQUE?**

Да, если целевой ключ удовлетворяет требованиям PostgreSQL к ссылочному ключу.

**10. Можно ли сделать составной Foreign Key?**

Да, через `FOREIGN KEY (column1, column2)`.

**11. В чём отличие Foreign Key от JOIN?**

Foreign Key обеспечивает целостность данных, JOIN выполняет соединение таблиц в запросе.

**12. Как Foreign Key используется в SQLAlchemy?**

Через `ForeignKey` или `ForeignKeyConstraint` для ограничения БД и `relationship()` для навигации между ORM-объектами.

## 🧪 Мини-задание

Создай две таблицы:

```sql
CREATE TABLE authors (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name TEXT NOT NULL
);

CREATE TABLE books (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    author_id BIGINT NOT NULL
        REFERENCES authors(id)
        ON DELETE CASCADE,
    title TEXT NOT NULL
);
```

Выполни следующие действия:

1. Добавь двух авторов.
2. Добавь три книги первому автору.
3. Попробуй создать книгу с несуществующим `author_id`.
4. Удали первого автора.
5. Проверь, что произошло с его книгами.
6. Измени `ON DELETE CASCADE` на `ON DELETE RESTRICT` и повтори эксперимент.

**Вопросы для самопроверки:**

- Почему несколько книг могут иметь одинаковый `author_id`?
- Почему PostgreSQL запрещает ссылаться на несуществующего автора?
- Чем опасен `ON DELETE CASCADE`?
- Почему индекс на `books.author_id` может быть полезен?

**Главное, что нужно запомнить:** `FOREIGN KEY` — это ограничение ссылочной целостности. Оно не просто связывает таблицы, а гарантирует, что ссылки между их записями остаются корректными.