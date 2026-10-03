# ❓ NULL в SQL

## 🎯 Ответ на собеседовании

**`NULL` в SQL означает отсутствие или неизвестность значения.**

`NULL` — это **не**:

* `0`;
* пустая строка `''`;
* `False`;
* значение по умолчанию.

Проверять `NULL` нужно специальными операторами:

```python
IS NULL
IS NOT NULL
```

Например:

```python
SELECT *
FROM users
WHERE email IS NULL;
```

---

## 🎤 Суперкоротко

```text
NULL = значение отсутствует / неизвестно
```

Правильно:

```python
WHERE email IS NULL
```

Неправильно:

```python
WHERE email = NULL
```

---

# 📌 Пример

Таблица:

```text
id | name   | email
---+--------+----------------
1  | Илья   | ilya@test.ru
2  | Анна   | anna@test.ru
3  | Максим | NULL
```

У Максима `email` отсутствует.

Найти таких пользователей:

```python
SELECT *
FROM users
WHERE email IS NULL;
```

Результат:

```text
3 | Максим | NULL
```

---

# 🔹 NULL ≠ 0

Это разные значения.

```text
NULL → значения нет / неизвестно
0    → числовое значение ноль
```

Например:

```text
age = NULL
```

означает:

> возраст неизвестен или не указан.

А:

```text
age = 0
```

означает:

> возраст равен нулю.

---

# 🔹 NULL ≠ пустая строка

Это тоже разные вещи.

```text
NULL
```

означает отсутствие значения.

А:

```text
''
```

означает существующую строку длиной `0`.

Например:

```text
email = NULL
```

и:

```text
email = ''
```

— не одно и то же.

---

# 🔹 Почему нельзя использовать `= NULL`

Очень важный момент.

Нельзя писать:

```python
SELECT *
FROM users
WHERE email = NULL;
```

Почему?

SQL использует специальную **трёхзначную логику**:

```text
TRUE
FALSE
UNKNOWN
```

Сравнение:

```python
email = NULL
```

даёт:

```text
UNKNOWN
```

а не `TRUE`.

Поэтому используется:

```python
IS NULL
```

---

# 🔹 Трёхзначная логика SQL

В обычной логике:

```text
TRUE
FALSE
```

В SQL:

```text
TRUE
FALSE
UNKNOWN
```

Например:

```text
5 = 5
```

→ `TRUE`

```text
5 = 10
```

→ `FALSE`

```text
5 = NULL
```

→ `UNKNOWN`

Потому что SQL не может установить, равен ли `5` неизвестному значению.

---

# 🔹 NULL в арифметике

Операции с `NULL` обычно дают `NULL`.

Например:

```python
SELECT 100 + NULL;
```

Результат:

```text
NULL
```

Потому что невозможно вычислить:

```text
100 + неизвестно
```

Аналогично:

```python
SELECT 100 * NULL;
```

→ `NULL`

```python
SELECT NULL / 10;
```

→ `NULL`

---

# 🔹 NULL в сравнении

Например:

```python
SELECT *
FROM users
WHERE age > 18;
```

Если:

```text
age = NULL
```

условие:

```text
NULL > 18
```

имеет результат:

```text
UNKNOWN
```

Такая строка не проходит `WHERE`.

---

# 🔹 IS NULL

Проверить отсутствие значения:

```python
SELECT *
FROM users
WHERE email IS NULL;
```

---

# 🔹 IS NOT NULL

Проверить наличие значения:

```python
SELECT *
FROM users
WHERE email IS NOT NULL;
```

Например:

```text
Илья   | ilya@test.ru
Анна   | anna@test.ru
Максим | NULL
```

Запрос:

```python
SELECT *
FROM users
WHERE email IS NOT NULL;
```

вернёт:

```text
Илья
Анна
```

---

# 🔹 NULL и LEFT JOIN

Это особенно важно после изучения `LEFT JOIN`.

```python
SELECT
    u.name,
    o.id
FROM users AS u
LEFT JOIN orders AS o
    ON u.id = o.user_id;
```

Если у пользователя нет заказа:

```text
name   | order_id
-------+---------
Илья   | 10
Анна   | 11
Максим | NULL
```

`NULL` здесь означает:

> В правой таблице не найдено соответствующей строки.

Поэтому можно найти пользователей без заказов:

```python
SELECT u.*
FROM users AS u
LEFT JOIN orders AS o
    ON u.id = o.user_id
WHERE o.id IS NULL;
```

---

# 🔹 NULL и COUNT

Очень важная особенность агрегатных функций.

Пусть:

```text
amount
------
500
700
NULL
```

Тогда:

```python
SELECT COUNT(amount)
FROM orders;
```

вернёт:

```text
2
```

Потому что `COUNT(column)` **не считает `NULL`**.

Но:

```python
SELECT COUNT(*)
FROM orders;
```

вернёт:

```text
3
```

Потому что `COUNT(*)` считает строки.

---

# 🔹 NULL и SUM / AVG

Большинство агрегатных функций игнорируют `NULL`.

Например:

```text
amount
------
500
700
NULL
```

```python
SELECT SUM(amount)
FROM orders;
```

Получим:

```text
1200
```

А:

```python
SELECT AVG(amount)
FROM orders;
```

считает среднее по:

```text
500
700
```

а не по трём строкам.

---

# 🔹 NULL и COALESCE

`COALESCE` позволяет заменить `NULL` другим значением.

Например:

```python
SELECT
    name,
    COALESCE(email, 'Email не указан') AS email
FROM users;
```

Получим:

```text
Илья   | ilya@test.ru
Анна   | anna@test.ru
Максим | Email не указан
```

Общий принцип:

```python
COALESCE(value, default_value)
```

То есть:

> Если `value` не `NULL`, вернуть его. Иначе вернуть `default_value`.

---

# 🔹 Несколько вариантов COALESCE

Можно передать несколько значений:

```python
SELECT COALESCE(phone, email, 'Контактов нет')
FROM users;
```

SQL будет искать первое значение, которое не является `NULL`:

```text
phone
  ↓
если NULL → email
  ↓
если NULL → 'Контактов нет'
```

---

# 🔹 NULL и UNIQUE

В PostgreSQL `NULL` имеет особенности при `UNIQUE`.

Например:

```python
CREATE TABLE users (
    id INTEGER PRIMARY KEY,
    email TEXT UNIQUE
);
```

Можно иметь несколько строк с:

```text
email = NULL
```

потому что `NULL` не считается обычным равным значением.

Например:

```text
1 | test@mail.ru
2 | NULL
3 | NULL
```

Это допустимо в PostgreSQL при обычном `UNIQUE`.

---

# 🔹 NULL и DISTINCT

`DISTINCT` обрабатывает `NULL` как одно значение в результирующем наборе.

Например:

```text
email
----------------
a@test.ru
NULL
NULL
b@test.ru
```

Запрос:

```python
SELECT DISTINCT email
FROM users;
```

даст:

```text
a@test.ru
NULL
b@test.ru
```

---

# 🔹 NULL и ORDER BY

При сортировке `NULL` можно явно расположить.

```python
SELECT *
FROM users
ORDER BY email NULLS FIRST;
```

или:

```python
SELECT *
FROM users
ORDER BY email NULLS LAST;
```

Это особенно полезно, если нужно явно определить положение отсутствующих значений.

---

# 🔹 NULL и CASE

`NULL` можно обрабатывать через `CASE`:

```python
SELECT
    name,
    CASE
        WHEN email IS NULL THEN 'Нет email'
        ELSE 'Email есть'
    END AS status
FROM users;
```

Получим:

```text
Илья   | Email есть
Анна   | Email есть
Максим | Нет email
```

---

# 🔹 NULL в Python и SQL

В Python аналогом SQL `NULL` является:

```python
None
```

Например, при работе с PostgreSQL через Python:

```python
user_email = None
```

обычно соответствует:

```text
SQL NULL
```

Но это не означает, что SQL `NULL` полностью идентичен Python `None` — это концепции разных систем.

В ORM, например Django ORM, проверка выглядит иначе:

```python
User.objects.filter(email__isnull=True)
```

---

# ⚠️ Типичная ошибка

Неправильно:

```python
SELECT *
FROM users
WHERE email = NULL;
```

Правильно:

```python
SELECT *
FROM users
WHERE email IS NULL;
```

И:

```python
SELECT *
FROM users
WHERE email IS NOT NULL;
```

---

# 🔹 NULL и логические условия

Из-за `UNKNOWN` поведение `AND` и `OR` может быть неожиданным.

Например:

```text
TRUE AND UNKNOWN
→ UNKNOWN
```

```text
FALSE AND UNKNOWN
→ FALSE
```

```text
TRUE OR UNKNOWN
→ TRUE
```

```text
FALSE OR UNKNOWN
→ UNKNOWN
```

Поэтому условия с `NULL` требуют осторожности.

---

# 🧠 Ментальная модель

Не думай о `NULL` как о конкретном значении.

Лучше думать:

```text
NULL
 ↓
"значение отсутствует / неизвестно"
```

Поэтому:

```text
NULL = 5
```

нельзя нормально оценить как `TRUE` или `FALSE`.

Получается:

```text
UNKNOWN
```

А чтобы проверить именно отсутствие значения:

```python
IS NULL
```

---

# 🎤 Как ответить на собеседовании

**Вопрос: Что такое NULL?**

> `NULL` в SQL обозначает отсутствие или неизвестность значения. Это не ноль и не пустая строка. Для проверки используются `IS NULL` и `IS NOT NULL`.

**Вопрос: Почему нельзя `column = NULL`?**

> Потому что сравнение с `NULL` даёт `UNKNOWN` в трёхзначной логике SQL. Поэтому для проверки нужно использовать `IS NULL`.

**Вопрос: Считает ли `COUNT(column)` NULL?**

> Нет, `COUNT(column)` игнорирует `NULL`. `COUNT(*)` считает строки, включая строки, где значение конкретного столбца равно `NULL`.

**Вопрос: Как заменить NULL значением по умолчанию?**

> Использовать `COALESCE`.

```python
SELECT COALESCE(email, 'unknown')
FROM users;
```

---

# 🎯 Главное

```text
NULL
→ отсутствие / неизвестность значения
```

Не путать:

```text
NULL ≠ 0
NULL ≠ ''
NULL ≠ FALSE
```

Проверка:

```python
IS NULL
IS NOT NULL
```

Не:

```python
= NULL
<> NULL
```

Замена:

```python
COALESCE(value, default)
```

И главное:

```text
SQL использует трёхзначную логику:

TRUE
FALSE
UNKNOWN
```

Именно поэтому `NULL` — одна из наиболее важных особенностей SQL, которую нужно уверенно объяснять на собеседовании.
