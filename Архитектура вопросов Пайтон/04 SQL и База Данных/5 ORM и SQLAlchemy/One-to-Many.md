# One-to-Many 🔗

## 🎯 Формула для собеседования

**One-to-Many (один-ко-многим) — это связь, при которой одной записи родительской таблицы соответствует много записей дочерней таблицы, а каждая дочерняя запись принадлежит одному родителю.**

```text id="7m3kq2"
1 Author
   │
   ├── Book 1
   ├── Book 2
   └── Book 3
```

В реляционной БД связь обычно реализуется через **Foreign Key на стороне Many**:

```text id="2x8v91"
Author.id
    ↑
    │
Book.author_id
```

---

## 🎤 Суперкоротко

Пример:

```text id="c6p4n8"
Author        Book
------        ----
id            id
name          title
              author_id → Author.id
```

Один автор:

```text id="k9r2m5"
Author 1
   ↓
Book 1
Book 2
Book 3
```

Но одна книга:

```text id="v4n7x1"
Book 1
   ↓
один Author
```

Поэтому:

```text id="q8m3s6"
Author 1 → N Books
```

---

# 1. Почему называется One-to-Many

Название описывает количество связанных объектов:

```text id="j5k8p2"
ONE
 ↓
MANY
```

Например:

```text id="r3x7m1"
Один пользователь
       ↓
много заказов
```

```text id="w6q2n9"
Один заказ
       ↓
много товаров
```

```text id="t4v8c3"
Одна категория
       ↓
много товаров
```

---

# 2. Где хранится Foreign Key

Ключевое правило:

> **Foreign Key находится на стороне MANY.**

Например:

```text id="b8n4q6"
Author
------
id
name


Book
----
id
title
author_id  ← Foreign Key
```

Почему?

Потому что каждая книга должна знать, **какому автору она принадлежит**.

```text id="p7m2x5"
Book.author_id → Author.id
```

---

# 3. Реализация в SQL

Создадим таблицы:

```sql id="v6q9r2"
CREATE TABLE author (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE book (
    id SERIAL PRIMARY KEY,
    title VARCHAR(200),
    author_id INTEGER REFERENCES author(id)
);
```

Получаем:

```text id="n3k8m5"
author
  │
  │ 1
  │
  │
  │ N
  ↓
book
```

---

# 4. Django ORM

В Django One-to-Many реализуется через `ForeignKey`.

```python id="x4m7q2"
class Author(models.Model):
    name = models.CharField(max_length=100)


class Book(models.Model):
    title = models.CharField(max_length=200)

    author = models.ForeignKey(
        Author,
        on_delete=models.CASCADE,
        related_name="books",
    )
```

Здесь:

```text id="p8c3v6"
Author
   ↓
many Books
```

А:

```python id="7k2n9m"
Book.author
```

возвращает одного автора.

---

# 5. Прямое направление

У книги:

```python id="q5x8r1"
book.author
```

получаем:

```text id="z3m7c4"
Book → Author
```

То есть:

```text id="h6n2v9"
Many → One
```

Несмотря на то что вся relationship называется **One-to-Many**, если смотреть от дочернего объекта, направление получается Many-to-One.

---

# 6. Обратное направление

У автора:

```python id="b4q7m1"
author.books.all()
```

получаем:

```text id="v9x3n6"
Author → Books
```

То есть:

```text id="m2k8c5"
One → Many
```

Именно поэтому `related_name` очень удобен:

```python id="c7r4p9"
related_name="books"
```

позволяет писать:

```python id="f8n3x2"
author.books.all()
```

---

# 7. Без related_name

Если `related_name` не указать:

```python id="9qv5m7"
author = models.ForeignKey(
    Author,
    on_delete=models.CASCADE,
)
```

Django создаст обратное имя на основе модели:

```python id="s2k8x4"
author.book_set.all()
```

То есть:

```python id="j6m3p8"
author.book_set.all()
```

С `related_name` код обычно читается лучше:

```python id="c9x4n7"
author.books.all()
```

---

# 8. on_delete

У `ForeignKey` обязательно указывается поведение при удалении связанного объекта.

Например:

```python id="w5r2m8"
author = models.ForeignKey(
    Author,
    on_delete=models.CASCADE,
)
```

### CASCADE

Если удалить автора:

```text id="h8q3v1"
Author
   ↓ DELETE
Books
   ↓
DELETE
```

Книги также удаляются.

Другие варианты:

```text id="n6x2m9"
CASCADE
PROTECT
RESTRICT
SET_NULL
SET_DEFAULT
SET(...)
DO_NOTHING
```

Например:

```python id="4m8q2x"
author = models.ForeignKey(
    Author,
    null=True,
    on_delete=models.SET_NULL,
)
```

При удалении автора:

```text id="r7v3k5"
author_id → NULL
```

а книга остаётся.

---

# 9. SQL JOIN для One-to-Many

Допустим:

```text id="y2m6q8"
Author 1
 ├── Book 1
 ├── Book 2
 └── Book 3
```

JOIN:

```sql id="c4v8n1"
SELECT
    author.name,
    book.title
FROM author
JOIN book
    ON book.author_id = author.id;
```

Результат:

```text id="p7x3m9"
Author 1 | Book 1
Author 1 | Book 2
Author 1 | Book 3
```

Обрати внимание:

**строка родителя повторяется для каждой дочерней записи.**

Это называется **row multiplication**.

---

# 10. One-to-Many и JOIN Load

В SQLAlchemy можно использовать:

```python id="m9c4x7"
select(Author).options(
    joinedload(Author.books)
)
```

Но если у автора много книг:

```text id="n3v8q2"
Author 1
 ├── Book 1
 ├── Book 2
 ├── Book 3
 └── ...
```

JOIN создаёт много строк.

Поэтому для коллекций часто используют:

```python id="x6k2m5"
selectinload(Author.books)
```

Получается:

```text id="r8q3v1"
SELECT authors
       ↓
SELECT books
WHERE author_id IN (...)
```

---

# 11. One-to-Many и N+1

Очень важная связь с оптимизацией ORM.

Например:

```python id="w4m7p2"
authors = session.scalars(
    select(Author)
).all()

for author in authors:
    for book in author.books:
        print(book.title)
```

Без eager loading может возникнуть:

```text id="q9x3n6"
1 SELECT → authors

N SELECT:
    books для Author 1
    books для Author 2
    books для Author 3
    ...
```

Итого:

```text id="v5k8m2"
1 + N запросов
```

Для One-to-Many в SQLAlchemy часто:

```python id="b3q7x9"
selectinload(Author.books)
```

В Django:

```python id="m8c2r5"
Author.objects.prefetch_related("books")
```

---

# 12. One-to-Many в Django: prefetch_related

Для обратного `ForeignKey`:

```python id="n4v7k2"
authors = Author.objects.prefetch_related(
    "books",
)
```

Django делает основной запрос:

```text id="x8m3q6"
SELECT authors ...
```

и дополнительный:

```text id="c5r9n1"
SELECT books
WHERE author_id IN (...)
```

Затем связывает книги с авторами.

---

# 13. One-to-Many в SQLAlchemy: selectinload

Аналогичный подход:

```python id="p2k7m4"
stmt = select(Author).options(
    selectinload(Author.books)
)

authors = session.scalars(stmt).all()
```

Концептуально:

```text id="j6x3q8"
SELECT author
       ↓
получили author.id
       ↓
SELECT book
WHERE author_id IN (...)
```

---

# 14. One-to-Many vs Many-to-One

Это две стороны одной связи.

Например:

```text id="n8c4m2"
Author 1
   ↓
   N
Books
```

Если смотрим со стороны `Author`:

```text id="z5q7v3"
One-to-Many
```

Если смотрим со стороны `Book`:

```text id="r2m9x6"
Many-to-One
```

В Django:

```python id="t4k8p1"
author.books.all()  # One → Many

book.author          # Many → One
```

---

# 15. One-to-Many vs Many-to-Many

### One-to-Many

У одного родителя много детей:

```text id="y6n3q8"
Author
 ↓
Book
Book
Book
```

Foreign Key находится в `Book`.

### Many-to-Many

У каждого объекта может быть много связанных объектов с обеих сторон:

```text id="k4r7m2"
Student ─── Course
   │   ╲   ╱  │
   │    ╲ ╱   │
   │     ╳    │
   │    ╱ ╲   │
```

Обычно нужна промежуточная таблица.

В Django:

```python id="v8c2n5"
students = models.ManyToManyField(Student)
```

---

# 16. Ограничение One-to-Many

One-to-Many означает:

```text id="p3m7x9"
один Book
    ↓
один Author
```

Но если `author_id` не `NOT NULL`, технически книга может не иметь автора:

```text id="w5q8n2"
Book
author_id = NULL
```

В Django это можно разрешить:

```python id="m4r6c8"
author = models.ForeignKey(
    Author,
    null=True,
    on_delete=models.SET_NULL,
)
```

То есть кардинальность и обязательность связи — не одно и то же.

---

# 17. Индекс Foreign Key

Для One-to-Many часто важен индекс на:

```text id="x7n2m5"
book.author_id
```

Например, запрос:

```sql id="c9q4v8"
SELECT *
FROM book
WHERE author_id = 10;
```

может эффективно использовать индекс.

В Django `ForeignKey` обычно имеет индекс автоматически, если явно не отключить его соответствующей настройкой.

---

# 18. Пример из реального backend

### Пользователь → Заказы

```python id="j3v8n2"
class User(models.Model):
    name = models.CharField(max_length=100)


class Order(models.Model):
    user = models.ForeignKey(
        User,
        on_delete=models.CASCADE,
        related_name="orders",
    )
    total = models.DecimalField(
        max_digits=10,
        decimal_places=2,
    )
```

Получить заказы пользователя:

```python id="q6m2x9"
user.orders.all()
```

Получить пользователя заказа:

```python id="r4k8c1"
order.user
```

---

# 19. Пример: категория → товары

```python id="v7n3m5"
class Category(models.Model):
    name = models.CharField(max_length=100)


class Product(models.Model):
    name = models.CharField(max_length=200)

    category = models.ForeignKey(
        Category,
        on_delete=models.CASCADE,
        related_name="products",
    )
```

Получить товары:

```python id="x2q6m8"
category.products.all()
```

Получить категорию:

```python id="c5r9n4"
product.category
```

---

# 20. Схема в базе

```text id="n8m3q7"
┌──────────────┐
│   AUTHOR     │
├──────────────┤
│ id PK        │
│ name         │
└──────┬───────┘
       │
       │ 1
       │
       │ N
       ↓
┌──────────────┐
│    BOOK      │
├──────────────┤
│ id PK        │
│ title        │
│ author_id FK │
└──────────────┘
```

Главное:

**FK находится в таблице `BOOK`, то есть на стороне MANY.**

---

# 🔥 Главное для собеседования

### Что такое One-to-Many?

> One-to-Many — связь, при которой одной записи родительской таблицы соответствует множество записей дочерней таблицы, а каждая дочерняя запись относится к одному родителю.

### Где находится Foreign Key?

**На стороне MANY.**

```text id="k7x2m5"
Author.id
    ↑
    │
Book.author_id
```

### Django

```python id="p4n8c2"
class Book(models.Model):
    author = models.ForeignKey(
        Author,
        on_delete=models.CASCADE,
        related_name="books",
    )
```

### Получение данных

```python id="w6q3r9"
book.author       # Many → One

author.books.all()  # One → Many
```

### Оптимизация

```text id="z2m8v4"
Django:
prefetch_related()
        ↓
One-to-Many коллекция
```

```text id="c7q3n9"
SQLAlchemy:
selectinload()
        ↓
One-to-Many коллекция
```

`joinedload()` тоже может использоваться, но при больших коллекциях JOIN может вызвать **row multiplication**.

---

## 🧠 Финальная формула

```text id="m5r8x2"
              One-to-Many
                   │
          ┌────────┴────────┐
          ↓                 ↓
       Parent            Child
          │                 │
       Author             Book
          │                 │
          │ 1             N │
          └────────┬────────┘
                   ↓
            Book.author_id
                 FK
```

```text id="q8v3n6"
One-to-Many
     ↓
FK на стороне MANY
     ↓
Django → ForeignKey
     ↓
Django → prefetch_related()
     ↓
SQLAlchemy → selectinload()
```

**Короткий ответ на собеседовании:**

> One-to-Many — это связь «один ко многим»: например, один автор имеет много книг. В реляционной БД она реализуется через Foreign Key в таблице `Book`, то есть на стороне Many. В Django используется `ForeignKey`, а для предварительной загрузки обратной коллекции обычно `prefetch_related()`. В SQLAlchemy аналогичная задача часто решается через `selectinload()`.
