# 🔍 SELECT в SQL

## 🎯 Ответ на собеседовании

**SELECT** — SQL-команда для получения данных из одной или нескольких таблиц.

С помощью `SELECT` можно выбрать нужные столбцы и строки, объединить таблицы через `JOIN`, сгруппировать данные, отсортировать результат и ограничить количество строк.

---

## 🎤 Суперкоротко

```python
SELECT column1, column2
FROM table
WHERE condition
ORDER BY column1
LIMIT 10;
```

Логически запрос обрабатывается примерно в таком порядке:

```text
FROM / JOIN
    ↓
WHERE
    ↓
GROUP BY
    ↓
HAVING
    ↓
SELECT
    ↓
ORDER BY
    ↓
LIMIT
```

---

# 1. 📌 Базовый SELECT

Получить все столбцы:

```python
SELECT *
FROM users;
```

Получить конкретные столбцы:

```python
SELECT id, name, email
FROM users;
```

Лучше выбирать только необходимые столбцы, а не использовать `SELECT *`, если все данные не нужны.

---

# 2. 🔎 SELECT + WHERE

`WHERE` фильтрует строки.

```python
SELECT id, name
FROM users
WHERE age >= 18;
```

Можно использовать:

```python
SELECT *
FROM users
WHERE age >= 18
  AND city = 'Moscow';
```

---

# 3. 🔢 DISTINCT

`DISTINCT` удаляет дубликаты в результирующем наборе.

```python
SELECT DISTINCT city
FROM users;
```

Результат:

```text
Moscow
London
Berlin
```

Если выбрать несколько столбцов:

```python
SELECT DISTINCT city, age
FROM users;
```

уникальной считается комбинация:

```text
(city, age)
```

---

# 4. 🔗 SELECT + JOIN

`SELECT` может получать данные из нескольких таблиц.

```python
SELECT
    users.name,
    orders.amount
FROM users
JOIN orders
    ON orders.user_id = users.id;
```

Здесь:

```text
users
   │
   └── JOIN
         │
       orders
```

---

# 5. 📊 SELECT + GROUP BY

`GROUP BY` объединяет строки в группы.

Например, количество заказов каждого пользователя:

```python
SELECT
    user_id,
    COUNT(*) AS orders_count
FROM orders
GROUP BY user_id;
```

Результат:

```text
user_id | orders_count
--------+-------------
1       | 5
2       | 3
3       | 10
```

---

# 6. 🧮 SELECT + агрегатные функции

Основные агрегаты:

```text
COUNT()
SUM()
AVG()
MIN()
MAX()
```

Например:

```python
SELECT COUNT(*)
FROM users;
```

Средняя стоимость заказа:

```python
SELECT AVG(amount)
FROM orders;
```

Максимальный заказ:

```python
SELECT MAX(amount)
FROM orders;
```

---

# 7. 🎯 HAVING

`HAVING` фильтрует **группы**, а `WHERE` — отдельные строки.

Например, найти пользователей, у которых больше 5 заказов:

```python
SELECT
    user_id,
    COUNT(*) AS orders_count
FROM orders
GROUP BY user_id
HAVING COUNT(*) > 5;
```

Логика:

```text
WHERE
→ фильтрует строки

GROUP BY
→ формирует группы

HAVING
→ фильтрует группы
```

---

# 8. ↕️ ORDER BY

Сортировка результата.

По возрастанию:

```python
SELECT *
FROM users
ORDER BY age ASC;
```

По убыванию:

```python
SELECT *
FROM users
ORDER BY age DESC;
```

`ASC` — по возрастанию.

`DESC` — по убыванию.

---

# 9. 🔢 LIMIT

Ограничивает количество возвращаемых строк.

```python
SELECT *
FROM users
LIMIT 10;
```

Получим максимум 10 строк.

Часто используется вместе с сортировкой:

```python
SELECT *
FROM users
ORDER BY created_at DESC
LIMIT 10;
```

→ последние 10 пользователей.

---

# 10. ⏭️ OFFSET

Пропускает определённое количество строк.

```python
SELECT *
FROM users
ORDER BY id
LIMIT 10
OFFSET 20;
```

Логика:

```text
пропустить 20
↓
взять следующие 10
```

Это используется для классической **OFFSET pagination**.

⚠️ При очень больших `OFFSET` запросы могут становиться дорогими, потому что БД всё равно должна пройти/пропустить большое количество строк.

---

# 11. 🧩 Алиасы

С помощью `AS` можно дать столбцу или таблице временное имя.

```python
SELECT
    name AS username
FROM users;
```

Для таблиц:

```python
SELECT
    u.name,
    o.amount
FROM users AS u
JOIN orders AS o
    ON o.user_id = u.id;
```

Алиасы особенно полезны при `JOIN`.

---

# 12. 🧠 CASE

`CASE` позволяет выполнять условную логику прямо в запросе.

```python
SELECT
    name,
    CASE
        WHEN age >= 18 THEN 'adult'
        ELSE 'minor'
    END AS category
FROM users;
```

Результат:

```text
name  | category
------+---------
Ivan  | adult
Petr  | minor
```

---

# 13. ❓ NULL в SELECT

`NULL` нельзя проверять через:

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

Или:

```python
SELECT *
FROM users
WHERE email IS NOT NULL;
```

Для замены `NULL` используется `COALESCE`:

```python
SELECT
    COALESCE(email, 'unknown')
FROM users;
```

---

# 14. 🔍 EXISTS

`EXISTS` проверяет, существует ли хотя бы одна подходящая строка.

Например, найти пользователей, у которых есть заказы:

```python
SELECT *
FROM users AS u
WHERE EXISTS (
    SELECT 1
    FROM orders AS o
    WHERE o.user_id = u.id
);
```

`EXISTS` возвращает логический результат:

```text
есть подходящая строка → TRUE
нет → FALSE
```

---

# 15. 📦 Подзапрос

`SELECT` может находиться внутри другого запроса.

Например:

```python
SELECT *
FROM users
WHERE age > (
    SELECT AVG(age)
    FROM users
);
```

Внутренний `SELECT` сначала логически получает средний возраст, внешний использует его для фильтрации.

---

# 16. 🔄 Логический порядок обработки

Очень важный момент для собеседования.

Хотя SQL записывается так:

```python
SELECT ...
FROM ...
WHERE ...
GROUP BY ...
HAVING ...
ORDER BY ...
LIMIT ...;
```

логически обработка идёт примерно так:

```text
1. FROM
      ↓
2. JOIN / ON
      ↓
3. WHERE
      ↓
4. GROUP BY
      ↓
5. HAVING
      ↓
6. SELECT
      ↓
7. DISTINCT
      ↓
8. ORDER BY
      ↓
9. LIMIT / OFFSET
```

Поэтому алиас из `SELECT` обычно нельзя использовать непосредственно в `WHERE`.

Например:

```python
SELECT
    age * 2 AS double_age
FROM users
WHERE double_age > 50;
```

обычно не сработает.

Потому что `WHERE` логически вычисляется раньше `SELECT`.

---

# 17. ⚡ SELECT и индекс

`SELECT` может использовать индекс.

Например:

```python
SELECT *
FROM users
WHERE email = 'ivan@mail.ru';
```

Если существует индекс:

```python
CREATE INDEX idx_users_email
ON users(email);
```

PostgreSQL может выбрать:

```text
Index Scan
```

Но индекс **не гарантированно будет использован**.

Решение принимает planner на основе статистики и стоимости выполнения.

Проверить можно:

```python
EXPLAIN
SELECT *
FROM users
WHERE email = 'ivan@mail.ru';
```

---

# 18. 🧠 SELECT не изменяет данные

Обычный `SELECT` предназначен для чтения.

Для изменения данных используются:

```text
INSERT
UPDATE
DELETE
```

В PostgreSQL также существуют команды, сочетающие изменение данных с возвращением результата через `RETURNING`.

Например:

```python
INSERT INTO users (name)
VALUES ('Ivan')
RETURNING id, name;
```

---

# 🎯 Главное

```text
SELECT
│
├── выбирает данные
│
├── FROM → откуда
├── JOIN → объединить таблицы
├── WHERE → отфильтровать строки
├── GROUP BY → сформировать группы
├── HAVING → отфильтровать группы
├── SELECT → какие данные вернуть
├── ORDER BY → отсортировать
└── LIMIT/OFFSET → ограничить результат
```

### Формула для собеседования

> **SELECT — SQL-оператор для выборки данных. Он позволяет выбрать столбцы, отфильтровать строки через WHERE, объединить таблицы через JOIN, сгруппировать данные через GROUP BY, отфильтровать группы через HAVING, отсортировать результат через ORDER BY и ограничить его через LIMIT/OFFSET.**

```text
SELECT
→ выборка данных
→ фильтрация
→ JOIN
→ агрегация
→ сортировка
→ ограничение результата
```
