# Django ORM 🐍

## 🎯 Формула для собеседования

**Django ORM — это слой Django, который позволяет работать с базой данных через Python-модели и QuerySet, не пиша SQL вручную.**

Главные элементы:

**Model → QuerySet → SQL → Database**

ORM строит SQL-запросы из Python-кода, а результаты преобразует обратно в объекты моделей.

---

## 🎤 Суперкоротко

**Django ORM** — встроенный ORM-фреймворк Django.

Основные понятия:

* **Model** — Python-класс, соответствующий таблице БД.
* **Field** — колонка таблицы.
* **QuerySet** — ленивое описание запроса к БД.
* **Manager** — интерфейс для получения QuerySet, обычно `objects`.
* **Migration** — изменение структуры БД на основе моделей.
* `filter()` — фильтрация.
* `get()` — получение одного объекта.
* `create()` — создание.
* `update()` — массовое обновление.
* `delete()` — удаление.
* `select_related()` — JOIN для `ForeignKey` / `OneToOne`.
* `prefetch_related()` — отдельные запросы для связанных объектов.

---

# 1. Model

Модель — Python-класс, описывающий структуру данных.

```python
from django.db import models


class User(models.Model):
    name = models.CharField(max_length=100)
    email = models.EmailField(unique=True)
    age = models.PositiveIntegerField()
```

Django создаст таблицу примерно такого вида:

```python
id
name
email
age
```

Модель отвечает одновременно за:

* структуру данных;
* связи между таблицами;
* ограничения;
* взаимодействие с БД через ORM.

---

# 2. Field

Поле модели соответствует колонке таблицы.

Примеры:

```python
name = models.CharField(max_length=100)
age = models.IntegerField()
is_active = models.BooleanField(default=True)
created_at = models.DateTimeField(auto_now_add=True)
```

Популярные поля:

| Django Field    | SQL-аналог  |
| --------------- | ----------- |
| `CharField`     | VARCHAR     |
| `TextField`     | TEXT        |
| `IntegerField`  | INTEGER     |
| `BooleanField`  | BOOLEAN     |
| `DateTimeField` | TIMESTAMP   |
| `DecimalField`  | DECIMAL     |
| `UUIDField`     | UUID        |
| `JSONField`     | JSON/JSONB  |
| `ForeignKey`    | FOREIGN KEY |

---

# 3. Manager

**Manager** — интерфейс для работы с QuerySet.

Стандартный менеджер:

```python
User.objects
```

Например:

```python
User.objects.all()
User.objects.filter(age__gte=18)
User.objects.get(id=1)
```

`objects` — это экземпляр `Manager`.

Можно создать собственный Manager:

```python
class ActiveUserManager(models.Manager):

    def get_queryset(self):
        return super().get_queryset().filter(is_active=True)
```

---

# 4. QuerySet

**QuerySet** — объект, который описывает запрос к БД.

Например:

```python
users = User.objects.filter(age__gte=18)
```

Это ещё не обязательно означает, что SQL уже выполнился.

QuerySet **ленивый (lazy)**.

```python
users = User.objects.filter(age__gte=18)

# SQL ещё может не выполняться

for user in users:
    print(user.name)
```

При итерации QuerySet Django выполняет SQL-запрос.

---

# 5. Ленивое выполнение

Один из важных вопросов на собеседовании.

```python
users = User.objects.filter(age__gte=18)
```

Создание QuerySet само по себе обычно не обращается к БД.

Запрос выполняется, когда QuerySet **эвалуируется**.

Например:

```python
list(users)
```

```python
for user in users:
    ...
```

```python
len(users)
```

```python
bool(users)
```

Также существуют операции, которые возвращают результат сразу:

```python
User.objects.get(id=1)
User.objects.count()
User.objects.exists()
```

---

# 6. filter()

`filter()` возвращает QuerySet с подходящими объектами.

```python
users = User.objects.filter(age__gte=18)
```

Пример SQL-концепции:

```sql
SELECT *
FROM users
WHERE age >= 18;
```

Можно передавать несколько условий:

```python
User.objects.filter(
    age__gte=18,
    is_active=True,
)
```

Условия объединяются через `AND`.

---

# 7. get()

`get()` используется, когда ожидается **ровно один объект**.

```python
user = User.objects.get(id=1)
```

Возможны исключения:

```python
User.DoesNotExist
```

если объекта нет.

И:

```python
User.MultipleObjectsReturned
```

если найдено больше одного объекта.

Поэтому:

```python
get()
```

не подходит для обычного поиска списка объектов.

---

# 8. first() и last()

Можно получить первый или последний объект:

```python
user = User.objects.order_by("id").first()
```

Если объектов нет:

```python
user is None
```

В отличие от `get()`, `first()` не выбрасывает `DoesNotExist`.

---

# 9. create()

Создание объекта:

```python
user = User.objects.create(
    name="Ilya",
    email="ilya@example.com",
    age=31,
)
```

Django создаёт SQL `INSERT`.

Альтернативный вариант:

```python
user = User(
    name="Ilya",
    email="ilya@example.com",
    age=31,
)

user.save()
```

---

# 10. update()

Массовое обновление:

```python
User.objects.filter(
    is_active=False,
).update(
    is_active=True,
)
```

Это выполняется одним SQL `UPDATE`.

Важно:

`QuerySet.update()` **не вызывает `save()` каждого объекта**.

Поэтому не следует рассчитывать на обычную логику `save()` для такого обновления.

---

# 11. delete()

Удаление:

```python
User.objects.filter(
    is_active=False,
).delete()
```

Можно удалить конкретный объект:

```python
user.delete()
```

---

# 12. Lookup

Django ORM использует специальные lookup-операторы.

```python
User.objects.filter(age__gte=18)
```

Формат:

```text
field__lookup=value
```

Основные:

| Lookup       | Значение                    |
| ------------ | --------------------------- |
| `exact`      | равно                       |
| `gt`         | `>`                         |
| `gte`        | `>=`                        |
| `lt`         | `<`                         |
| `lte`        | `<=`                        |
| `in`         | входит в список             |
| `contains`   | содержит                    |
| `icontains`  | содержит без учёта регистра |
| `startswith` | начинается с                |
| `endswith`   | заканчивается               |
| `isnull`     | NULL                        |
| `range`      | диапазон                    |

Пример:

```python
User.objects.filter(age__gte=18)
```

```python
User.objects.filter(
    email__icontains="@gmail.com",
)
```

```python
User.objects.filter(
    id__in=[1, 2, 3],
)
```

---

# 13. AND / OR / NOT

По умолчанию несколько условий:

```python
User.objects.filter(
    age__gte=18,
    is_active=True,
)
```

означают:

```text
age >= 18 AND is_active = true
```

Для сложных условий используется `Q`.

```python
from django.db.models import Q

User.objects.filter(
    Q(age__lt=18) | Q(age__gte=60)
)
```

Получается:

```text
age < 18 OR age >= 60
```

NOT:

```python
User.objects.filter(
    ~Q(is_active=True)
)
```

---

# 14. order_by()

Сортировка:

```python
User.objects.order_by("age")
```

По возрастанию.

По убыванию:

```python
User.objects.order_by("-age")
```

Несколько полей:

```python
User.objects.order_by(
    "age",
    "-name",
)
```

---

# 15. values() и values_list()

По умолчанию ORM возвращает объекты моделей:

```python
users = User.objects.all()
```

Можно получить словари:

```python
users = User.objects.values(
    "id",
    "name",
)
```

Результат концептуально:

```python
[
    {"id": 1, "name": "Ilya"},
    {"id": 2, "name": "Alex"},
]
```

Или tuples:

```python
User.objects.values_list(
    "id",
    "name",
)
```

---

# 16. exists()

Если нужно только проверить наличие объекта:

```python
if User.objects.filter(email=email).exists():
    ...
```

Это предпочтительнее, чем:

```python
if User.objects.filter(email=email):
    ...
```

или загрузки всех объектов ради проверки существования.

`exists()` предназначен именно для проверки наличия.

---

# 17. count()

Количество объектов:

```python
count = User.objects.filter(
    is_active=True,
).count()
```

ORM сформирует агрегирующий SQL-запрос.

Концептуально:

```sql
SELECT COUNT(*)
FROM users
WHERE is_active = true;
```

---

# 18. Связи между моделями

Django поддерживает:

* `ForeignKey` — многие к одному;
* `OneToOneField` — один к одному;
* `ManyToManyField` — многие ко многим.

Пример:

```python
class Author(models.Model):
    name = models.CharField(max_length=100)


class Book(models.Model):
    title = models.CharField(max_length=200)
    author = models.ForeignKey(
        Author,
        on_delete=models.CASCADE,
    )
```

Один автор может иметь много книг.

---

# 19. Работа с ForeignKey

Получить книги автора:

```python
author.books.all()
```

Если указать `related_name`:

```python
author = models.ForeignKey(
    Author,
    on_delete=models.CASCADE,
    related_name="books",
)
```

тогда:

```python
author.books.all()
```

---

# 20. Фильтрация через связи

Можно фильтровать связанные модели:

```python
Book.objects.filter(
    author__name="Ilya",
)
```

Или:

```python
Book.objects.filter(
    author__age__gte=18,
)
```

Django самостоятельно строит необходимые JOIN.

---

# 21. N+1 Problem

Одна из самых важных проблем Django ORM.

Например:

```python
books = Book.objects.all()

for book in books:
    print(book.author.name)
```

Может привести к:

```text
1 запрос → получить книги
N запросов → получить author для каждой книги
```

Итого:

```text
1 + N запросов
```

Это **N+1 problem**.

---

# 22. select_related()

Для `ForeignKey` и `OneToOne` используется:

```python
select_related()
```

Например:

```python
books = Book.objects.select_related(
    "author",
)
```

Теперь Django может получить книги и авторов через SQL `JOIN`.

Концептуально:

```sql
SELECT ...
FROM book
JOIN author
    ON book.author_id = author.id;
```

Это обычно позволяет получить данные одним запросом.

### Главное

```text
select_related()
    ↓
SQL JOIN
    ↓
ForeignKey / OneToOne
```

---

# 23. prefetch_related()

Для `ManyToMany` и обратных связей обычно используется:

```python
prefetch_related()
```

Например:

```python
authors = Author.objects.prefetch_related(
    "books",
)
```

Концептуально Django выполняет несколько запросов:

```text
SELECT authors ...
SELECT books ... WHERE author_id IN (...)
```

Затем ORM связывает результаты в Python.

### Главное

```text
prefetch_related()
    ↓
отдельные SELECT
    ↓
связи собираются в Python
```

---

# 24. select_related vs prefetch_related

|                         | `select_related`  | `prefetch_related`         |
| ----------------------- | ----------------- | -------------------------- |
| Основной механизм       | JOIN              | отдельные запросы          |
| `ForeignKey`            | ✅                 | ✅                          |
| `OneToOne`              | ✅                 | ✅                          |
| `ManyToMany`            | ❌                 | ✅                          |
| Reverse FK              | ❌                 | ✅                          |
| Количество запросов     | обычно 1          | обычно 2+                  |
| Row multiplication      | возможна при JOIN | нет такой проблемы от JOIN |
| Где объединяются данные | БД                | Python                     |

На собеседовании:

> `select_related()` использует SQL JOIN и подходит прежде всего для одиночных связей, а `prefetch_related()` делает отдельные запросы и подходит для коллекций — `ManyToMany` и reverse ForeignKey.

---

# 25. aggregate()

`aggregate()` возвращает агрегированный результат для всего QuerySet.

```python
from django.db.models import Avg

result = User.objects.aggregate(
    average_age=Avg("age"),
)
```

Результат:

```python
{
    "average_age": 31.5,
}
```

Популярные агрегаты:

```python
Count
Sum
Avg
Min
Max
```

---

# 26. annotate()

`annotate()` добавляет вычисляемое поле к каждому объекту/группе результата.

Например, количество книг у каждого автора:

```python
from django.db.models import Count

authors = Author.objects.annotate(
    books_count=Count("books"),
)
```

Теперь:

```python
for author in authors:
    print(author.name, author.books_count)
```

### Разница

```text
aggregate()
    ↓
один агрегированный результат

annotate()
    ↓
вычисляемое значение для каждого объекта/группы
```

---

# 27. F()

`F()` позволяет ссылаться на значение другого поля **на стороне БД**.

Например:

```python
from django.db.models import F

Product.objects.update(
    price=F("price") + 100,
)
```

Вместо:

```text
SELECT price
→ Python
→ price + 100
→ UPDATE
```

операция выполняется непосредственно в БД.

Это также помогает избежать некоторых race conditions при атомарных обновлениях.

---

# 28. Case / When

Условная логика на стороне БД:

```python
from django.db.models import Case, When, Value

users = User.objects.annotate(
    category=Case(
        When(age__lt=18, then=Value("child")),
        default=Value("adult"),
    )
)
```

Концептуально соответствует SQL `CASE`.

---

# 29. Транзакции

Django предоставляет API для транзакций:

```python
from django.db import transaction
```

Основной вариант:

```python
with transaction.atomic():
    ...
```

Все операции внутри блока выполняются как одна транзакция.

Если возникает исключение:

```text
ROLLBACK
```

Если всё успешно:

```text
COMMIT
```

---

# 30. select_for_update()

Для блокировки строк используется:

```python
with transaction.atomic():
    user = User.objects.select_for_update().get(
        id=1,
    )

    user.balance -= 100
    user.save()
```

`select_for_update()` использует блокировку строк на уровне БД.

Важно:

```text
select_for_update()
≠
select_related()
```

`select_related()` — загрузка связанных объектов через JOIN.

`select_for_update()` — блокировка выбранных строк.

---

# 31. Миграции

Модель:

```python
class User(models.Model):
    name = models.CharField(max_length=100)
```

Изменили:

```python
class User(models.Model):
    name = models.CharField(max_length=100)
    age = models.IntegerField()
```

Создаём миграцию:

```python
python manage.py makemigrations
```

Применяем:

```python
python manage.py migrate
```

### Разница

```text
makemigrations
    ↓
создаёт описание изменения схемы

migrate
    ↓
применяет изменение к БД
```

---

# 32. Raw SQL

Django ORM не запрещает SQL.

Можно выполнить SQL напрямую:

```python
User.objects.raw(
    "SELECT * FROM app_user WHERE age >= %s",
    [18],
)
```

Но в обычной ситуации предпочтительно использовать ORM, если он позволяет выразить запрос.

---

# 33. Как посмотреть SQL

QuerySet можно посмотреть:

```python
queryset = User.objects.filter(age__gte=18)

print(queryset.query)
```

Это полезно для понимания того, какой SQL генерирует ORM.

Для анализа производительности используется также:

```python
queryset.explain()
```

---

# 34. Django ORM и SQLAlchemy

|               | Django ORM                                  | SQLAlchemy               |
| ------------- | ------------------------------------------- | ------------------------ |
| ORM           | встроен в Django                            | отдельная библиотека     |
| Model         | Django Model                                | SQLAlchemy Model         |
| QuerySet      | есть                                        | нет прямого аналога      |
| Manager       | есть                                        | нет прямого аналога      |
| Session       | нет отдельного ORM Session как в SQLAlchemy | есть                     |
| Migrations    | Django Migrations                           | обычно Alembic           |
| Web framework | тесно интегрирован с Django                 | независим                |
| Query API     | ORM QuerySet API                            | SQL Expression / ORM API |

Главное различие:

**Django ORM тесно интегрирован с Django и использует `Model.objects → QuerySet`.**

SQLAlchemy разделяет Core и ORM и предоставляет более явную модель `Engine → Connection/Session`.

---

# 35. Типичный CRUD

```python
# CREATE
user = User.objects.create(
    name="Ilya",
    email="ilya@example.com",
)

# READ
user = User.objects.get(id=1)

users = User.objects.filter(
    is_active=True,
)

# UPDATE
User.objects.filter(
    id=1,
).update(
    name="Alex",
)

# DELETE
User.objects.filter(
    id=1,
).delete()
```

---

# 36. Типичный запрос с оптимизацией

Плохо:

```python
books = Book.objects.all()

for book in books:
    print(book.author.name)
```

Возможен N+1.

Лучше:

```python
books = Book.objects.select_related(
    "author",
)

for book in books:
    print(book.author.name)
```

Для коллекции:

```python
authors = Author.objects.prefetch_related(
    "books",
)

for author in authors:
    for book in author.books.all():
        print(book.title)
```

---

# 37. Важный момент: ORM не отменяет SQL

На собеседовании важно показать, что ORM — не магия.

Даже если код выглядит так:

```python
User.objects.filter(
    age__gte=18,
)
```

Django в конечном итоге формирует SQL.

Поэтому backend-разработчик должен понимать:

* `JOIN`;
* индексы;
* `GROUP BY`;
* `WHERE`;
* транзакции;
* блокировки;
* N+1;
* планы выполнения;
* `EXPLAIN`;
* количество запросов.

---

# 38. Частые ошибки

### ❌ N+1

```python
for book in Book.objects.all():
    print(book.author.name)
```

### ✅

```python
Book.objects.select_related("author")
```

---

### ❌ Получать объект ради проверки существования

```python
try:
    User.objects.get(email=email)
except User.DoesNotExist:
    ...
```

Если нужен только факт существования:

### ✅

```python
User.objects.filter(
    email=email,
).exists()
```

---

### ❌ Обновлять объекты в цикле

```python
for user in users:
    user.is_active = False
    user.save()
```

Это может привести к множеству `UPDATE`.

### ✅

```python
User.objects.filter(
    id__in=user_ids,
).update(
    is_active=False,
)
```

---

# 39. Архитектура Django ORM

```text
Python код
    ↓
Model Manager
    ↓
QuerySet
    ↓
SQL Compiler
    ↓
Database Driver
    ↓
PostgreSQL
```

Например:

```python
User.objects.filter(
    age__gte=18,
)
```

примерно превращается в:

```text
QuerySet
    ↓
WHERE age >= 18
    ↓
SQL
    ↓
PostgreSQL
```

---

# 🔥 Главное для собеседования

1. **Model** — Python-представление таблицы БД.
2. **Field** — колонка таблицы.
3. **Manager** — интерфейс получения QuerySet.
4. **QuerySet** — ленивое описание запроса.
5. `filter()` — возвращает QuerySet.
6. `get()` — ожидает ровно один объект.
7. `create()` — INSERT.
8. `update()` — массовый UPDATE.
9. `delete()` — DELETE.
10. `select_related()` → **JOIN**, `ForeignKey` / `OneToOne`.
11. `prefetch_related()` → **отдельные запросы**, коллекции связей.
12. N+1 — одна из главных проблем производительности ORM.
13. `annotate()` — вычисление для каждого объекта/группы.
14. `aggregate()` — общий агрегированный результат.
15. `F()` — операции с полями непосредственно в БД.
16. `Q()` — сложные `AND` / `OR` / `NOT`.
17. `atomic()` — транзакция.
18. `select_for_update()` — блокировка строк.
19. `makemigrations` создаёт миграции, `migrate` применяет их.
20. ORM не отменяет необходимость понимать SQL и планы выполнения.

### Формула

```text
Django ORM
    ↓
Model + Manager + QuerySet
    ↓
Python API
    ↓
SQL
    ↓
PostgreSQL
```

И отдельно:

```text
N+1
  ↓
select_related → JOIN
prefetch_related → отдельные SELECT
```
