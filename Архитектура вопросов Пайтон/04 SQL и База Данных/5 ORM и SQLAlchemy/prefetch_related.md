# prefetch_related в Django ORM 🔗

## 🎯 Формула для собеседования

**`prefetch_related()` — это механизм Django ORM для eager loading связанных объектов через отдельные SQL-запросы. Django получает основные объекты и связанные объекты отдельными запросами, а затем связывает их в Python.**

Главное применение:

```text id="4p8f3s"
prefetch_related()
       ↓
отдельные SELECT
       ↓
связанные объекты
       ↓
объединение в Python
       ↓
N+1 ↓
```

Особенно полезен для **`ManyToMany` и обратных `ForeignKey`**, где `select_related()` неприменим.

---

## 🎤 Суперкоротко

Есть авторы и книги:

```python id="v7j8e1"
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

Без оптимизации:

```python id="7r9m2q"
authors = Author.objects.all()

for author in authors:
    for book in author.books.all():
        print(book.title)
```

Возможен **N+1**:

```text id="n0m5lq"
1 запрос → authors
N запросов → books каждого автора
```

С `prefetch_related()`:

```python id="2b4h8s"
authors = Author.objects.prefetch_related("books")
```

Django обычно выполняет:

```text id="5q2m8r"
SELECT ... FROM author;

SELECT ... FROM book
WHERE author_id IN (...);
```

После этого связывает книги с авторами в Python.

---

# 1. Какую проблему решает

Основная задача — устранение **N+1 Query Problem** при работе с коллекциями связанных объектов.

Например:

```python id="h3v7y2"
authors = Author.objects.all()

for author in authors:
    print(author.books.all())
```

Если авторов 100:

```text id="j8n3k5"
1 запрос → получить 100 авторов
100 запросов → получить книги
------------------------------
101 запрос
```

С:

```python id="5z1q8v"
authors = Author.objects.prefetch_related("books")
```

получается примерно:

```text id="x6p2m4"
1 запрос → authors
1 запрос → books
----------------
2 запроса
```

---

# 2. Почему здесь не используется JOIN

Главная идея `prefetch_related()`:

> Получить связанные объекты отдельным запросом и затем сопоставить их в Python.

Например:

```text id="r3s7k1"
Authors:

id    name
1     Ilya
2     Alex
3     Maria
```

Второй запрос:

```text id="w9f2c6"
Books:

id    title       author_id
10    Book A      1
11    Book B      1
12    Book C      2
13    Book D      3
```

Django связывает:

```text id="q4m8t2"
Ilya
 ├── Book A
 └── Book B

Alex
 └── Book C

Maria
 └── Book D
```

То есть объединение результатов происходит на стороне Python ORM.

---

# 3. Для каких связей используется

`prefetch_related()` особенно важен для:

### ManyToMany

```python id="g2c7v9"
author.books.all()
```

если связь многие-ко-многим.

### Reverse ForeignKey

Например:

```python id="m5r1x8"
author.books.all()
```

где:

```python id="n8q3s6"
class Book(models.Model):
    author = models.ForeignKey(
        Author,
        on_delete=models.CASCADE,
        related_name="books",
    )
```

То есть:

```text id="a6j2w4"
Author
   ↓
many Books
```

### Также может использоваться с ForeignKey

`prefetch_related()` умеет загружать и обычные `ForeignKey`, но для одиночной связи обычно более естественен:

```python id="x4v7k2"
select_related()
```

---

# 4. ManyToMany

Например:

```python id="u5k8c1"
class Student(models.Model):
    name = models.CharField(max_length=100)


class Course(models.Model):
    name = models.CharField(max_length=100)
    students = models.ManyToManyField(Student)
```

Получаем курсы:

```python id="z3n6q9"
courses = Course.objects.prefetch_related(
    "students",
)
```

Теперь:

```python id="p7m2v5"
for course in courses:
    for student in course.students.all():
        print(student.name)
```

Django заранее загрузил студентов.

---

# 5. Reverse ForeignKey

Например:

```python id="c8x4m1"
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

Используем:

```python id="y2r6k8"
authors = Author.objects.prefetch_related(
    "books",
)
```

Теперь:

```python id="s5n9q3"
for author in authors:
    for book in author.books.all():
        print(book.title)
```

не должен порождать отдельный запрос для каждого автора.

---

# 6. select_related vs prefetch_related

Это один из самых популярных вопросов на собеседовании.

|                   | `select_related()`    | `prefetch_related()`        |
| ----------------- | --------------------- | --------------------------- |
| Основной механизм | SQL `JOIN`            | Отдельные SQL-запросы       |
| `ForeignKey`      | ✅                     | ✅                           |
| `OneToOne`        | ✅                     | ✅                           |
| `ManyToMany`      | ❌                     | ✅                           |
| Reverse FK        | ❌                     | ✅                           |
| Объединение       | БД                    | Python                      |
| Типичный случай   | одна связанная запись | коллекция связанных записей |

Формула:

```text id="m8v2q5"
select_related()
    ↓
JOIN
    ↓
одна связанная запись
```

```text id="f4k7n2"
prefetch_related()
    ↓
несколько SELECT
    ↓
коллекция связанных записей
```

---

# 7. Важный нюанс: количество запросов

Не стоит говорить:

> "`prefetch_related()` всегда делает ровно два запроса."

Правильнее:

> "`prefetch_related()` выполняет дополнительные запросы для связанных объектов и затем объединяет результаты в Python.

Например, для простой связи может быть:

```text id="n6r1w8"
SELECT authors ...
SELECT books WHERE author_id IN (...)
```

Но при нескольких `prefetch_related()` запросов может быть больше.

---

# 8. Несколько связей

Можно предварительно загрузить несколько отношений:

```python id="q3v8m2"
Author.objects.prefetch_related(
    "books",
    "articles",
)
```

И вложенные связи:

```python id="k9s4p6"
Author.objects.prefetch_related(
    "books__genres",
)
```

Получается:

```text id="z1x7c3"
Author
  ↓
Books
  ↓
Genres
```

Django выполнит необходимые дополнительные запросы и соберёт связи.

---

# 9. Prefetch()

Для более сложного контроля существует объект:

```python id="r5m8q2"
from django.db.models import Prefetch
```

Например:

```python id="v7c3n9"
authors = Author.objects.prefetch_related(
    Prefetch(
        "books",
        queryset=Book.objects.filter(
            published=True,
        ),
    )
)
```

Теперь предварительно загружаются только опубликованные книги.

---

# 10. to_attr

Результат `Prefetch` можно сохранить в отдельный атрибут:

```python id="b4n9x6"
authors = Author.objects.prefetch_related(
    Prefetch(
        "books",
        queryset=Book.objects.filter(
            published=True,
        ),
        to_attr="published_books",
    )
)
```

Теперь:

```python id="q8m2v5"
for author in authors:
    for book in author.published_books:
        print(book.title)
```

Это удобно, когда нужно несколько разных вариантов одной связи.

Например:

```python id="n3k7r1"
Prefetch(
    "books",
    queryset=Book.objects.filter(published=True),
    to_attr="published_books",
)
```

и отдельно:

```python id="p6x2m8"
Prefetch(
    "books",
    queryset=Book.objects.filter(published=False),
    to_attr="draft_books",
)
```

---

# 11. Кэширование результата

После `prefetch_related()` связанные объекты обычно помещаются Django в кэш.

Например:

```python id="w5q8c2"
authors = Author.objects.prefetch_related("books")

for author in authors:
    print(author.books.all())
```

После предварительной загрузки повторное обращение к этой связи может использовать уже загруженные данные, а не делать новый запрос.

Но важно понимать:

```python id="r7m3x9"
author.books.all()
```

и:

```python id="h4n8q1"
author.books.filter(title__icontains="Python")
```

— это разные QuerySet.

Если изменить запрос фильтрацией, предварительно загруженный кэш `.all()` не обязательно будет пригоден для нового запроса.

---

# 12. Когда prefetch может быть неэффективным

`prefetch_related()` тоже не является магической оптимизацией.

Если связанных объектов огромное количество:

```text id="c5v9m2"
1 000 000 authors
        ↓
10 000 000 books
```

предварительная загрузка всего набора может потребовать много:

* RAM;
* сетевого трафика;
* времени;
* ресурсов PostgreSQL;
* времени на сборку объектов Python.

Поэтому нужно смотреть на реальный workload.

---

# 13. Prefetch + фильтрация

Можно комбинировать:

```python id="j8q2v6"
authors = (
    Author.objects
    .filter(is_active=True)
    .prefetch_related("books")
)
```

Сначала выбираются нужные авторы, затем связанные книги для полученного набора.

---

# 14. Prefetch + Prefetch

Можно делать разные выборки одной связи:

```python id="d6r3k8"
authors = Author.objects.prefetch_related(
    Prefetch(
        "books",
        queryset=Book.objects.filter(
            published=True,
        ),
        to_attr="published_books",
    ),
    Prefetch(
        "books",
        queryset=Book.objects.filter(
            published=False,
        ),
        to_attr="draft_books",
    ),
)
```

Теперь:

```python id="s9m4x2"
author.published_books
author.draft_books
```

---

# 15. prefetch_related и N+1

### Плохо

```python id="t2v7k4"
authors = Author.objects.all()

for author in authors:
    for book in author.books.all():
        print(book.title)
```

Потенциально:

```text id="x5n8q1"
1 + N запросов
```

### Хорошо

```python id="m3r6c9"
authors = Author.objects.prefetch_related(
    "books",
)
```

Теперь:

```text id="v8q2k5"
основной SELECT
       +
SELECT связанных объектов
       ↓
сопоставление в Python
```

---

# 16. Можно комбинировать select_related и prefetch_related

Да.

Например:

```python id="k7m4p2"
authors = Author.objects.prefetch_related(
    "books__publisher",
)
```

Или более сложная комбинация:

```python id="c9x3n6"
orders = (
    Order.objects
    .select_related("user")
    .prefetch_related("items")
)
```

Здесь:

```text id="f2q8v5"
Order
  │
  ├── user
  │     ↓
  │   select_related → JOIN
  │
  └── items
        ↓
      prefetch_related → отдельный SELECT
```

Это распространённый подход в реальных Django-проектах.

---

# 17. Почему prefetch лучше для коллекций

Представим:

```text id="y4n8c2"
Author 1 → 100 books
Author 2 → 200 books
Author 3 → 150 books
```

При JOIN результат может содержать много повторяющихся данных автора:

```text id="z6r1m9"
Author 1 | Book 1
Author 1 | Book 2
Author 1 | Book 3
...
```

При `prefetch_related()`:

```text id="q2v7k4"
SELECT authors
```

и:

```text id="m8c3x5"
SELECT books
WHERE author_id IN (1, 2, 3)
```

Django затем связывает результаты.

Это позволяет избежать некоторых проблем с **row multiplication**, характерных для JOIN коллекций.

---

# 18. Что происходит внутри

Упрощённо:

```text id="n5q8r2"
Author.objects.prefetch_related("books")
                │
                ▼
       SELECT authors
                │
                ▼
        получили author_id
                │
                ▼
       SELECT books
       WHERE author_id IN (...)
                │
                ▼
       Django сопоставляет
       books → authors
                │
                ▼
       author.books.all()
```

То есть Django не делает отдельный запрос для каждого автора.

---

# 19. prefetch_related и база данных

Важно понимать границу ответственности:

```text id="x3m7p9"
PostgreSQL
    ↓
возвращает данные
```

а:

```text id="c8q2v5"
Django ORM
    ↓
сопоставляет связанные объекты
```

Поэтому `prefetch_related()` отличается от обычного SQL JOIN не только количеством запросов, но и **местом, где происходит объединение результатов**.

---

# 20. Как проверить запросы

Можно посмотреть SQL:

```python id="v6n2r8"
queryset = Author.objects.prefetch_related("books")

print(queryset.query)
```

Но `queryset.query` показывает SQL основного QuerySet; дополнительные prefetch-запросы выполняются при evaluation.

Для анализа фактических запросов удобно использовать Django Debug Toolbar или логирование SQL.

---

# 21. Важный нюанс с `.iterator()`

При использовании:

```python id="q4m8x7"
queryset.iterator()
```

поведение `prefetch_related()` зависит от версии Django и параметров `iterator()`.

В современных версиях Django при необходимости нужно учитывать `chunk_size`, поскольку prefetch должен понимать, каким объёмом данных работать.

Для собеседования достаточно помнить:

> `iterator()` и `prefetch_related()` имеют особенности взаимодействия, особенно при потоковой обработке больших QuerySet.

---

# 22. Частая ошибка

Нельзя думать:

> "`prefetch_related()` просто заменяет `select_related()`."

Нет.

Они решают похожую задачу — **уменьшение количества запросов**, но используют разные стратегии.

```text id="s8n3m6"
select_related
    → JOIN
    → данные объединяет БД

prefetch_related
    → отдельные SELECT
    → данные объединяет Django/Python
```

---

# 🔥 Главное для собеседования

### Что такое `prefetch_related()`?

> `prefetch_related()` — механизм Django ORM для предварительной загрузки связанных объектов через отдельные SQL-запросы с последующим объединением результатов в Python.

### Зачем?

Для устранения **N+1** при работе с коллекциями связанных объектов.

### Где особенно используется?

```text id="e7m2q9"
ManyToMany
Reverse ForeignKey
```

### Чем отличается от `select_related()`?

```text id="p4n8c2"
select_related()
    ↓
SQL JOIN
    ↓
ForeignKey / OneToOne
```

```text id="x6r3m7"
prefetch_related()
    ↓
отдельные SELECT
    ↓
ManyToMany / Reverse FK
```

### Пример

```python id="h9q5v2"
authors = Author.objects.prefetch_related(
    "books",
)

for author in authors:
    for book in author.books.all():
        print(book.title)
```

### Продвинутый вариант

```python id="w3k8n6"
from django.db.models import Prefetch

authors = Author.objects.prefetch_related(
    Prefetch(
        "books",
        queryset=Book.objects.filter(
            published=True,
        ),
        to_attr="published_books",
    )
)
```

**Формула:**

```text id="r2m7c5"
prefetch_related()
        ↓
  отдельные SELECT
        ↓
связанные коллекции
        ↓
объединение в Python
        ↓
       N+1 ↓
```
