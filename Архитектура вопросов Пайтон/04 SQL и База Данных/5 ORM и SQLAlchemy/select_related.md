# select_related в Django ORM 🔗

## 🎯 Формула для собеседования

**`select_related()` — это оптимизация Django ORM, которая заранее загружает связанные объекты через SQL `JOIN`, обычно одним запросом. Используется прежде всего для `ForeignKey` и `OneToOneField` и помогает устранить N+1 запросов.**

```text
select_related()
      ↓
   SQL JOIN
      ↓
ForeignKey / OneToOne
      ↓
меньше SQL-запросов
```

---

## 🎤 Суперкоротко

Если есть:

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

Без `select_related()`:

```python
books = Book.objects.all()

for book in books:
    print(book.author.name)
```

может получиться:

```text
1 запрос → получить книги
N запросов → получить авторов

Итого: 1 + N запросов
```

С `select_related()`:

```python
books = Book.objects.select_related("author")
```

Django использует `JOIN` и заранее загружает авторов.

---

# 1. Какую проблему решает

Основное назначение — борьба с **N+1 Query Problem**.

Допустим, в БД есть:

```text
Book
├── Book 1 → Author 1
├── Book 2 → Author 2
├── Book 3 → Author 3
└── Book 4 → Author 1
```

Запрос:

```python
books = Book.objects.all()

for book in books:
    print(book.author.name)
```

может привести к:

```text
SELECT * FROM book;

SELECT * FROM author WHERE id = 1;
SELECT * FROM author WHERE id = 2;
SELECT * FROM author WHERE id = 3;
SELECT * FROM author WHERE id = 1;
```

То есть приложение делает дополнительные запросы при обращении:

```python
book.author
```

---

# 2. С select_related()

Пишем:

```python
books = Book.objects.select_related("author")
```

Теперь Django заранее строит запрос с `JOIN`.

Концептуально:

```sql
SELECT
    book.*,
    author.*
FROM book
JOIN author
    ON book.author_id = author.id;
```

После этого:

```python
for book in books:
    print(book.author.name)
```

не требует отдельного SQL-запроса для каждого автора.

---

# 3. Почему именно JOIN

`select_related()` использует SQL `JOIN`, потому что связанные объекты находятся в связи, которую можно представить одной строкой результата.

Например:

```text
Book
  │
  │ author_id
  ↓
Author
```

Для `ForeignKey` у каждой книги обычно **один конкретный автор**.

Поэтому данные можно получить одним `JOIN`.

---

# 4. Какие связи поддерживает

Основные случаи:

### ForeignKey

```python
Book.objects.select_related("author")
```

### OneToOneField

```python
User.objects.select_related("profile")
```

Также можно проходить несколько связей:

```python
Book.objects.select_related(
    "author__country",
)
```

То есть:

```text
Book
 ↓
Author
 ↓
Country
```

Django построит необходимые JOIN.

---

# 5. select_related vs обычный QuerySet

### Без

```python
books = Book.objects.all()
```

Связанный объект может быть загружен позднее:

```python
book.author
```

### С

```python
books = Book.objects.select_related("author")
```

Автор загружается сразу вместе с книгой.

---

# 6. Важный момент: select_related не делает запрос сразу

Это всё ещё QuerySet:

```python
queryset = Book.objects.select_related("author")
```

Он ленивый.

SQL будет выполнен при evaluation:

```python
for book in queryset:
    ...
```

или:

```python
list(queryset)
```

---

# 7. Несколько связей

Можно указать несколько полей:

```python
Book.objects.select_related(
    "author",
    "publisher",
)
```

Django построит JOIN для обеих связей.

Можно использовать цепочку:

```python
Book.objects.select_related(
    "author__country",
)
```

---

# 8. select_related() без аргументов

Можно написать:

```python
Book.objects.select_related()
```

Django автоматически выберет некоторые подходящие связи.

На практике обычно лучше явно указывать поля:

```python
Book.objects.select_related(
    "author",
)
```

Так запрос более предсказуемый и понятный.

---

# 9. Почему нельзя использовать для ManyToMany

Представим:

```text
Author
  ↓
  ├── Book 1
  ├── Book 2
  └── Book 3
```

У автора **много** книг.

JOIN создаёт несколько строк:

```text
Author 1 | Book 1
Author 1 | Book 2
Author 1 | Book 3
```

При сложных JOIN количество строк может резко увеличиваться.

Поэтому для:

* `ManyToManyField`;
* reverse `ForeignKey`;

обычно используется:

```python
prefetch_related()
```

---

# 10. select_related vs prefetch_related

|                    | `select_related()`                   | `prefetch_related()`                  |
| ------------------ | ------------------------------------ | ------------------------------------- |
| Механизм           | SQL `JOIN`                           | отдельные SQL-запросы                 |
| ForeignKey         | ✅                                    | ✅                                     |
| OneToOne           | ✅                                    | ✅                                     |
| ManyToMany         | ❌                                    | ✅                                     |
| Reverse ForeignKey | ❌                                    | ✅                                     |
| Основная идея      | получить связанные данные через JOIN | получить коллекции отдельным запросом |
| N+1                | устраняет                            | устраняет                             |

Формула:

```text
ForeignKey / OneToOne
        ↓
select_related()
        ↓
JOIN
```

```text
ManyToMany / Reverse FK
        ↓
prefetch_related()
        ↓
отдельные SELECT
```

---

# 11. Пример из реального backend

Допустим, API возвращает список заказов:

```python
class Order(models.Model):
    user = models.ForeignKey(
        User,
        on_delete=models.CASCADE,
    )
    total = models.DecimalField(
        max_digits=10,
        decimal_places=2,
    )
```

Нужно вывести:

```text
Заказ №1 — Ilya — 1500 ₽
Заказ №2 — Alex — 2300 ₽
Заказ №3 — Maria — 900 ₽
```

Плохой вариант:

```python
orders = Order.objects.all()

for order in orders:
    print(order.user.name)
```

Возможен N+1.

Лучше:

```python
orders = Order.objects.select_related("user")

for order in orders:
    print(order.user.name)
```

---

# 12. select_related в Django REST Framework

Например:

```python
class OrderViewSet(ModelViewSet):
    queryset = Order.objects.select_related("user")
    serializer_class = OrderSerializer
```

Это особенно важно для API, где один endpoint может вернуть десятки или сотни объектов.

Без оптимизации сериализатор может вызвать:

```python
order.user
```

для каждого заказа и породить N+1 запросов.

---

# 13. select_related + filter

Их можно спокойно комбинировать:

```python
orders = (
    Order.objects
    .select_related("user")
    .filter(user__is_active=True)
)
```

Django сформирует QuerySet с необходимым JOIN и фильтрацией.

---

# 14. select_related + only()

Можно ограничивать выбираемые поля:

```python
books = Book.objects.select_related(
    "author",
).only(
    "title",
    "author__name",
)
```

Это может уменьшить объём передаваемых данных.

Но `only()` нужно использовать аккуратно: если позже обратиться к отложенному полю, Django может выполнить дополнительный запрос.

---

# 15. Может ли select_related ухудшить запрос?

Да.

`JOIN` — не бесплатная операция.

Если подключить слишком много связанных таблиц:

```python
Order.objects.select_related(
    "user",
    "user__profile",
    "user__company",
    "user__company__address",
)
```

запрос может стать:

* сложнее;
* тяжелее для БД;
* больше по объёму результата;
* менее эффективным, чем несколько отдельных запросов.

Поэтому `select_related()` используют для **реально необходимых** связей.

---

# 16. Главная ловушка на собеседовании

Не говорить:

> "`select_related()` всегда делает один SQL-запрос."

Правильнее:

> "`select_related()` загружает связанные объекты через SQL JOIN, что обычно позволяет получить их в рамках одного SQL-запроса."

Потому что количество запросов зависит от конкретного QuerySet и последующих операций.

---

# 17. Как проверить запрос

Можно посмотреть SQL:

```python
queryset = Book.objects.select_related("author")

print(queryset.query)
```

Для анализа плана:

```python
print(
    queryset.explain()
)
```

А количество запросов удобно проверять в тестах или Django Debug Toolbar.

---

# 18. select_related и JOIN: важное различие

Не путать:

```python
select_related("author")
```

и:

```python
filter(author__name="Ilya")
```

`filter()` говорит:

> Отфильтруй книги по связанному автору.

`select_related()` говорит:

> Загрузи связанного автора заранее.

Они могут использовать JOIN, но **назначение разное**.

Например:

```python
Book.objects.filter(
    author__name="Ilya",
)
```

не означает автоматически, что `book.author` будет оптимизированно загружен для последующего использования.

Если автор нужен в результатах:

```python
Book.objects.select_related(
    "author",
).filter(
    author__name="Ilya",
)
```

---

# 19. Связь с N+1

```text
             Без оптимизации

Book.objects.all()
        ↓
     Book 1 ──→ SELECT Author
     Book 2 ──→ SELECT Author
     Book 3 ──→ SELECT Author
     ...
     Book N ──→ SELECT Author

             1 + N запросов
```

С `select_related()`:

```text
Book.objects.select_related("author")
                ↓
             SQL JOIN
                ↓
       Books + Authors
                ↓
        обычно 1 запрос
```

---

# 🔥 Главное для собеседования

### Что такое `select_related()`?

> Это оптимизация Django ORM, которая заранее загружает связанные объекты через SQL `JOIN`.

### Для каких связей?

```text
ForeignKey
OneToOneField
```

### Какую проблему решает?

```text
N+1 Query Problem
```

### Чем отличается от `prefetch_related()`?

```text
select_related
    → JOIN
    → ForeignKey / OneToOne

prefetch_related
    → отдельные запросы
    → ManyToMany / reverse ForeignKey
```

### Пример

```python
books = Book.objects.select_related("author")

for book in books:
    print(book.author.name)
```

### Самая важная формула

```text
select_related()
        ↓
    SQL JOIN
        ↓
ForeignKey / OneToOne
        ↓
     N+1 ↓
```

**Короткий ответ на собеседовании:**

> `select_related()` — это механизм Django ORM для eager loading связанных объектов через SQL JOIN. Он применяется прежде всего к `ForeignKey` и `OneToOneField` и позволяет избежать N+1 запросов, когда связанные данные нужны вместе с основными объектами.
