# 📊 GROUP BY в SQL

## 🎯 Ответ на собеседовании

**`GROUP BY` группирует строки с одинаковыми значениями в указанных столбцах, чтобы над каждой группой можно было выполнить агрегатные операции: `COUNT`, `SUM`, `AVG`, `MIN`, `MAX`.**

Например:

```python
SELECT
    user_id,
    COUNT(*) AS orders_count
FROM orders
GROUP BY user_id;
```

Здесь все заказы одного пользователя объединяются в одну группу, после чего `COUNT(*)` считает количество заказов в каждой группе.

---

## 🎤 Суперкоротко

```text
GROUP BY
→ разбивает строки на группы
→ одинаковые значения попадают в одну группу
→ агрегаты считаются отдельно для каждой группы
```

Например:

```text
orders
   ↓
GROUP BY user_id
   ↓
группа user=1
группа user=2
группа user=3
   ↓
COUNT / SUM / AVG
```

---

# 📌 Простой пример

Таблица:

```text
orders

id | user_id | amount
---+---------+-------
1  | 10      | 500
2  | 10      | 700
3  | 20      | 300
4  | 20      | 400
5  | 30      | 900
```

Запрос:

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
10      | 2
20      | 2
30      | 1
```

То есть:

```text
user_id = 10
→ 2 строки
→ COUNT = 2

user_id = 20
→ 2 строки
→ COUNT = 2

user_id = 30
→ 1 строка
→ COUNT = 1
```

---

# 🔹 Как работает GROUP BY

Представим исходные данные:

```text
10 | 500
10 | 700
20 | 300
20 | 400
30 | 900
```

После:

```python
GROUP BY user_id
```

концептуально получаем:

```text
Группа 10:
500
700

Группа 20:
300
400

Группа 30:
900
```

Теперь можно применить агрегаты:

```text
Группа 10 → COUNT=2, SUM=1200
Группа 20 → COUNT=2, SUM=700
Группа 30 → COUNT=1, SUM=900
```

---

# 🔹 GROUP BY + COUNT

Самый распространённый вариант:

```python
SELECT
    user_id,
    COUNT(*) AS orders_count
FROM orders
GROUP BY user_id;
```

Получаем количество заказов каждого пользователя.

---

# 🔹 GROUP BY + SUM

Посчитать общую сумму заказов:

```python
SELECT
    user_id,
    SUM(amount) AS total_amount
FROM orders
GROUP BY user_id;
```

Результат:

```text
user_id | total_amount
--------+-------------
10      | 1200
20      | 700
30      | 900
```

---

# 🔹 GROUP BY + AVG

Средняя стоимость заказа:

```python
SELECT
    user_id,
    AVG(amount) AS avg_amount
FROM orders
GROUP BY user_id;
```

Результат:

```text
user_id | avg_amount
--------+-----------
10      | 600
20      | 350
30      | 900
```

---

# 🔹 GROUP BY + MIN / MAX

Минимальный и максимальный заказ:

```python
SELECT
    user_id,
    MIN(amount) AS min_amount,
    MAX(amount) AS max_amount
FROM orders
GROUP BY user_id;
```

Результат:

```text
user_id | min_amount | max_amount
--------+------------+-----------
10      | 500        | 700
20      | 300        | 400
30      | 900        | 900
```

---

# 🔹 GROUP BY по нескольким столбцам

Можно группировать сразу по нескольким полям:

```python
SELECT
    user_id,
    status,
    COUNT(*) AS orders_count
FROM orders
GROUP BY user_id, status;
```

Теперь отдельная группа определяется **комбинацией**:

```text
user_id + status
```

Например:

```text
user_id | status  | orders_count
--------+---------+-------------
10      | paid    | 3
10      | pending | 1
20      | paid    | 2
20      | pending | 4
```

То есть:

```text
(10, paid)
(10, pending)
(20, paid)
(20, pending)
```

— это разные группы.

---

# 🔹 GROUP BY по городу

Например, таблица пользователей:

```text
id | name   | city
---+--------+-------------
1  | Илья   | Екатеринбург
2  | Анна   | Москва
3  | Максим | Екатеринбург
4  | Олег   | Москва
5  | Ирина  | Казань
```

Посчитать пользователей по городам:

```python
SELECT
    city,
    COUNT(*) AS users_count
FROM users
GROUP BY city;
```

Результат:

```text
city           | users_count
---------------+------------
Екатеринбург   | 2
Москва         | 2
Казань         | 1
```

---

# 🔹 GROUP BY и SELECT ⚠️

Очень важное правило.

Если используется:

```python
GROUP BY
```

то столбцы в `SELECT` должны быть либо:

1. указаны в `GROUP BY`;
2. либо использованы внутри агрегатной функции.

Правильно:

```python
SELECT
    user_id,
    COUNT(*)
FROM orders
GROUP BY user_id;
```

`user_id` есть в `GROUP BY`.

---

Например:

```python
SELECT
    user_id,
    amount,
    COUNT(*)
FROM orders
GROUP BY user_id;
```

Проблема:

```text
user_id → понятно, к какой группе относится
amount  → какой именно amount брать из нескольких строк?
```

Для пользователя `10` есть:

```text
500
700
```

Какой `amount` должен вывести SQL?

Однозначного ответа нет.

Поэтому PostgreSQL выдаст ошибку.

Нужно использовать агрегат:

```python
SELECT
    user_id,
    AVG(amount),
    COUNT(*)
FROM orders
GROUP BY user_id;
```

или добавить `amount` в группировку:

```python
SELECT
    user_id,
    amount,
    COUNT(*)
FROM orders
GROUP BY user_id, amount;
```

Но это уже будут другие группы.

---

# 🔹 GROUP BY не обязательно означает COUNT

`GROUP BY` часто воспринимают как:

```text
GROUP BY → COUNT
```

Но это неверно.

Можно использовать:

```text
COUNT
SUM
AVG
MIN
MAX
```

и другие агрегатные выражения.

Например:

```python
SELECT
    category_id,
    SUM(price)
FROM products
GROUP BY category_id;
```

Здесь мы не считаем количество строк — мы суммируем значения.

---

# 🔹 GROUP BY без агрегатной функции

Можно написать:

```python
SELECT city
FROM users
GROUP BY city;
```

Получим уникальные города:

```text
Екатеринбург
Москва
Казань
```

Но если нужна именно уникализация значений, обычно понятнее использовать:

```python
SELECT DISTINCT city
FROM users;
```

Поэтому:

```text
GROUP BY без агрегатов
→ технически возможно

DISTINCT
→ обычно более явно выражает намерение получить уникальные значения
```

---

# 🔹 GROUP BY + WHERE

`WHERE` фильтрует строки **до группировки**.

Например:

```python
SELECT
    user_id,
    COUNT(*) AS orders_count
FROM orders
WHERE amount > 500
GROUP BY user_id;
```

Логика:

```text
orders
   ↓
WHERE amount > 500
   ↓
остались подходящие заказы
   ↓
GROUP BY user_id
   ↓
COUNT(*)
```

То есть заказы дешевле или равные `500` вообще не участвуют в группировке.

---

# 🔹 GROUP BY + HAVING

`HAVING` фильтрует уже сформированные группы.

Например:

```python
SELECT
    user_id,
    COUNT(*) AS orders_count
FROM orders
GROUP BY user_id
HAVING COUNT(*) >= 2;
```

Сначала:

```text
GROUP BY
```

создаёт группы:

```text
10 → 2 заказа
20 → 2 заказа
30 → 1 заказ
```

Затем:

```text
HAVING COUNT(*) >= 2
```

оставляет:

```text
10 → 2
20 → 2
```

---

# 🔥 WHERE vs GROUP BY vs HAVING

Это нужно уверенно различать.

```text
WHERE
→ фильтрует строки

GROUP BY
→ объединяет строки в группы

HAVING
→ фильтрует группы
```

Пример:

```python
SELECT
    user_id,
    COUNT(*) AS orders_count
FROM orders
WHERE amount > 100
GROUP BY user_id
HAVING COUNT(*) >= 5;
```

Логика:

```text
Все orders
    ↓
WHERE amount > 100
    ↓
только дорогие заказы
    ↓
GROUP BY user_id
    ↓
группы пользователей
    ↓
COUNT(*)
    ↓
HAVING COUNT(*) >= 5
    ↓
пользователи с 5+ подходящими заказами
```

---

# 🔹 GROUP BY + ORDER BY

Можно отсортировать группы после агрегирования.

Например, найти пользователей с наибольшим количеством заказов:

```python
SELECT
    user_id,
    COUNT(*) AS orders_count
FROM orders
GROUP BY user_id
ORDER BY orders_count DESC;
```

Результат:

```text
user_id | orders_count
--------+-------------
10      | 15
20      | 10
30      | 4
```

Здесь:

```text
GROUP BY
→ создаёт группы

COUNT
→ считает заказы

ORDER BY
→ сортирует группы
```

---

# 🔹 GROUP BY + LIMIT

Можно получить, например, топ-10 пользователей по количеству заказов:

```python
SELECT
    user_id,
    COUNT(*) AS orders_count
FROM orders
GROUP BY user_id
ORDER BY orders_count DESC
LIMIT 10;
```

Логика:

```text
GROUP BY
    ↓
COUNT
    ↓
ORDER BY DESC
    ↓
LIMIT 10
```

---

# 🔹 GROUP BY после JOIN

Очень распространённый backend-сценарий.

Есть:

```text
users
orders
```

Нужно получить количество заказов каждого пользователя:

```python
SELECT
    u.id,
    u.name,
    COUNT(o.id) AS orders_count
FROM users AS u
LEFT JOIN orders AS o
    ON u.id = o.user_id
GROUP BY u.id, u.name;
```

Результат:

```text
id | name   | orders_count
---+--------+-------------
1  | Илья   | 2
2  | Анна   | 1
3  | Максим | 0
```

Здесь `GROUP BY` объединяет строки после `JOIN`.

---

# ⚠️ Почему COUNT(o.id), а не COUNT(*)

После `LEFT JOIN` пользователь без заказов всё равно представлен строкой:

```text
user_id | order_id
--------+---------
3       | NULL
```

Поэтому:

```python
COUNT(*)
```

может дать:

```text
1
```

а:

```python
COUNT(o.id)
```

даст:

```text
0
```

потому что `COUNT(column)` игнорирует `NULL`.

Это важный практический нюанс.

---

# 🔹 GROUP BY и NULL

`NULL`-значения группируются вместе.

Например:

```text
city
-----------
Москва
NULL
Москва
NULL
Казань
```

Запрос:

```python
SELECT
    city,
    COUNT(*)
FROM users
GROUP BY city;
```

даст концептуально:

```text
city    | count
--------+------
Москва  | 2
Казань  | 1
NULL    | 2
```

То есть все строки с `NULL` попадают в одну группу.

---

# 🔹 GROUP BY и вычисляемые выражения

Можно группировать по выражению.

Например:

```python
SELECT
    EXTRACT(YEAR FROM created_at) AS year,
    COUNT(*) AS orders_count
FROM orders
GROUP BY EXTRACT(YEAR FROM created_at);
```

Получим:

```text
year | orders_count
-----+-------------
2024 | 150
2025 | 320
2026 | 470
```

---

# 🧠 Логический порядок выполнения

Для запроса:

```python
SELECT
    user_id,
    COUNT(*) AS orders_count
FROM orders
WHERE amount > 100
GROUP BY user_id
HAVING COUNT(*) >= 5
ORDER BY orders_count DESC
LIMIT 10;
```

Упрощённый логический порядок:

```text
1. FROM
2. JOIN
3. WHERE
4. GROUP BY
5. HAVING
6. SELECT
7. ORDER BY
8. LIMIT
```

То есть:

```text
WHERE
   ↓
GROUP BY
   ↓
агрегация
   ↓
HAVING
```

---

# 🔹 GROUP BY и агрегатные функции

Агрегатная функция работает **для каждой группы отдельно**.

Например:

```python
SELECT
    city,
    AVG(salary)
FROM employees
GROUP BY city;
```

SQL концептуально делает:

```text
Екатеринбург:
100000
120000
110000
    ↓
AVG = 110000

Москва:
150000
180000
    ↓
AVG = 165000
```

Результат:

```text
city         | avg
-------------+--------
Екатеринбург | 110000
Москва       | 165000
```

---

# 📊 GROUP BY vs DISTINCT

Их часто сравнивают.

### DISTINCT

Получить уникальные значения:

```python
SELECT DISTINCT city
FROM users;
```

### GROUP BY

Сформировать группы и выполнять агрегаты:

```python
SELECT
    city,
    COUNT(*)
FROM users
GROUP BY city;
```

Главная идея:

```text
DISTINCT
→ убрать дубликаты

GROUP BY
→ сформировать группы для анализа/агрегации
```

Хотя `GROUP BY` без агрегатных функций иногда может дать результат, похожий на `DISTINCT`.

---

# 🎤 Как ответить на собеседовании

**Вопрос: Что такое GROUP BY?**

> `GROUP BY` группирует строки по значениям указанных столбцов. После этого агрегатные функции, такие как `COUNT`, `SUM`, `AVG`, `MIN` и `MAX`, вычисляются отдельно для каждой группы.

Пример:

```python
SELECT
    user_id,
    COUNT(*) AS orders_count
FROM orders
GROUP BY user_id;
```

**Вопрос: Чем GROUP BY отличается от WHERE?**

> `WHERE` фильтрует отдельные строки до группировки, а `GROUP BY` объединяет оставшиеся строки в группы.

**Вопрос: Чем GROUP BY отличается от HAVING?**

> `GROUP BY` создаёт группы, а `HAVING` фильтрует уже сформированные группы.

**Вопрос: Можно ли выбрать столбец, которого нет в GROUP BY?**

> В обычном запросе он должен либо находиться в `GROUP BY`, либо быть аргументом агрегатной функции. Иначе SQL не знает, какое значение выбрать из нескольких строк группы.

---

# 🎯 Главное

```text
GROUP BY
→ группирует строки
→ одинаковые значения попадают в одну группу
→ агрегаты считаются отдельно для каждой группы
```

Классический шаблон:

```python
SELECT
    column,
    COUNT(*)
FROM table
GROUP BY column;
```

Связка:

```text
WHERE
→ фильтр строк

GROUP BY
→ создание групп

COUNT / SUM / AVG
→ расчёт по группам

HAVING
→ фильтр групп

ORDER BY
→ сортировка результата
```

И главный пример:

```python
SELECT
    user_id,
    COUNT(*) AS orders_count
FROM orders
WHERE amount > 100
GROUP BY user_id
HAVING COUNT(*) >= 5
ORDER BY orders_count DESC;
```

Логика:

```text
отфильтровали строки
        ↓
сгруппировали
        ↓
посчитали
        ↓
отфильтровали группы
        ↓
отсортировали
```
