# 📊 HAVING в SQL

## 🎯 Ответ на собеседовании

**`HAVING` фильтрует группы, которые были сформированы с помощью `GROUP BY`.**

Главное отличие:

* [[WHERE]] фильтрует **отдельные строки до группировки**;
* [[HAVING]] фильтрует **группы после группировки**.

`HAVING` особенно часто используется с агрегатными функциями:

```python
COUNT()
SUM()
AVG()
MIN()
MAX()
```

Например:

```python
SELECT
    user_id,
    COUNT(*) AS orders_count
FROM orders
GROUP BY user_id
HAVING COUNT(*) > 5;
```

Запрос вернёт только тех пользователей, у которых **больше 5 заказов**.

---

## 🎤 Суперкоротко

```text
WHERE
→ фильтрует строки

GROUP BY
→ формирует группы

HAVING
→ фильтрует группы
```

Ключевой пример:

```python
SELECT
    user_id,
    COUNT(*) AS orders_count
FROM orders
GROUP BY user_id
HAVING COUNT(*) > 5;
```

---

# 📌 Зачем нужен HAVING

Представим таблицу:

```text
orders

id | user_id | amount
---+---------+-------
1  | 10      | 500
2  | 10      | 700
3  | 10      | 300
4  | 20      | 100
5  | 30      | 900
6  | 30      | 800
```

Нужно:

> Найти пользователей, у которых больше одного заказа.

Сначала группируем:

```python
SELECT
    user_id,
    COUNT(*) AS orders_count
FROM orders
GROUP BY user_id;
```

Получаем:

```text
user_id | orders_count
--------+-------------
10      | 3
20      | 1
30      | 2
```

Теперь нужно отфильтровать **группы**:

```python
SELECT
    user_id,
    COUNT(*) AS orders_count
FROM orders
GROUP BY user_id
HAVING COUNT(*) > 1;
```

Результат:

```text
user_id | orders_count
--------+-------------
10      | 3
30      | 2
```

---

# 🔹 Почему здесь нельзя использовать WHERE

Неправильно:

```python
SELECT
    user_id,
    COUNT(*) AS orders_count
FROM orders
WHERE COUNT(*) > 1
GROUP BY user_id;
```

Почему?

Потому что `WHERE` работает **до `GROUP BY`**.

На этапе `WHERE` ещё нет групп:

```text
orders
  ↓
WHERE       ← групп ещё нет
  ↓
GROUP BY
  ↓
COUNT()
```

А нам нужно фильтровать результат:

```text
GROUP BY
  ↓
COUNT()
  ↓
HAVING
```

Поэтому:

```python
HAVING COUNT(*) > 1
```

---

# 🔹 WHERE + GROUP BY + HAVING

Их часто используют вместе.

Например:

> Найти города, где больше 10 совершеннолетних пользователей.

```python
SELECT
    city,
    COUNT(*) AS users_count
FROM users
WHERE age >= 18
GROUP BY city
HAVING COUNT(*) > 10;
```

Логика:

```text
users
  ↓
WHERE age >= 18
  ↓
остались только взрослые
  ↓
GROUP BY city
  ↓
получили группы городов
  ↓
COUNT(*)
  ↓
HAVING COUNT(*) > 10
  ↓
остались города с > 10 пользователей
```

Это очень хороший пример для собеседования.

---

# 📊 WHERE vs HAVING

|                                 | `WHERE`       | `HAVING`        |
| ------------------------------- | ------------- | --------------- |
| Фильтрует                       | Строки        | Группы          |
| До `GROUP BY`                   | ✅             | ❌               |
| После `GROUP BY`                | ❌             | ✅               |
| Часто используется с агрегатами | ❌             | ✅               |
| `COUNT()` / `SUM()` в условии   | Обычно нельзя | ✅               |
| Пример                          | `age > 18`    | `COUNT(*) > 10` |

Мнемоника:

```text
WHERE
→ WHERE is this row valid?

HAVING
→ HAVING this group enough?
```

---

# 🔹 HAVING и агрегатные функции

Самое частое применение:

### COUNT

```python
SELECT
    user_id,
    COUNT(*) AS orders_count
FROM orders
GROUP BY user_id
HAVING COUNT(*) >= 10;
```

Пользователи с минимум 10 заказами.

---

### SUM

```python
SELECT
    user_id,
    SUM(amount) AS total_amount
FROM orders
GROUP BY user_id
HAVING SUM(amount) > 100000;
```

Пользователи, совершившие покупок более чем на `100000`.

---

### AVG

```python
SELECT
    category_id,
    AVG(price) AS avg_price
FROM products
GROUP BY category_id
HAVING AVG(price) > 5000;
```

Категории со средней ценой товара выше `5000`.

---

### MAX

```python
SELECT
    user_id,
    MAX(amount) AS max_order
FROM orders
GROUP BY user_id
HAVING MAX(amount) > 10000;
```

Пользователи, у которых есть заказ дороже `10000`.

---

### MIN

```python
SELECT
    category_id,
    MIN(price) AS min_price
FROM products
GROUP BY category_id
HAVING MIN(price) < 100;
```

Категории, в которых есть товар дешевле `100`.

---

# 🔹 HAVING без GROUP BY

В SQL `HAVING` может использоваться и без явного `GROUP BY`.

Например:

```python
SELECT COUNT(*) AS users_count
FROM users
HAVING COUNT(*) > 100;
```

В таком случае результат агрегирования рассматривается как одна группа.

Если пользователей больше `100`:

```text
users_count
-----------
150
```

Если пользователей `100` или меньше — запрос не вернёт строку.

Но в прикладном SQL наиболее распространённый сценарий:

```text
GROUP BY
+
HAVING
```

---

# 🔹 HAVING и алиас

Можно встретить:

```python
SELECT
    user_id,
    COUNT(*) AS orders_count
FROM orders
GROUP BY user_id
HAVING orders_count > 5;
```

Но переносимость такого варианта между СУБД хуже.

Для PostgreSQL надёжный и понятный вариант:

```python
SELECT
    user_id,
    COUNT(*) AS orders_count
FROM orders
GROUP BY user_id
HAVING COUNT(*) > 5;
```

То есть лучше повторить агрегатное выражение в `HAVING`.

---

# 🔹 HAVING с несколькими условиями

Можно использовать `AND` и `OR`:

```python
SELECT
    user_id,
    COUNT(*) AS orders_count,
    SUM(amount) AS total_amount
FROM orders
GROUP BY user_id
HAVING COUNT(*) >= 5
   AND SUM(amount) > 10000;
```

Теперь группа должна удовлетворять **обоим условиям**:

```text
заказов >= 5
AND
общая сумма > 10000
```

---

# 🔹 HAVING и JOIN

`HAVING` часто применяется после `JOIN`.

Например:

```python
SELECT
    u.id,
    u.name,
    COUNT(o.id) AS orders_count
FROM users AS u
LEFT JOIN orders AS o
    ON u.id = o.user_id
GROUP BY u.id, u.name
HAVING COUNT(o.id) >= 5;
```

Логика:

```text
users
  ↓
LEFT JOIN orders
  ↓
GROUP BY user
  ↓
COUNT(orders)
  ↓
HAVING COUNT >= 5
```

Получаем пользователей минимум с пятью заказами.

---

# 🔹 Почему здесь LEFT JOIN

Можно было бы использовать `INNER JOIN`:

```python
FROM users AS u
INNER JOIN orders AS o
    ON u.id = o.user_id
```

Но `LEFT JOI
