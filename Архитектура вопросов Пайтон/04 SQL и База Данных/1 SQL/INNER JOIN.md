# 🔗 INNER JOIN в SQL

## 🎯 Ответ на собеседовании

**`INNER JOIN` объединяет две таблицы и возвращает только те строки, для которых выполняется условие соединения `ON`.**

Если соответствующей строки в одной из таблиц нет — она **не попадает в результат**.

```python
SELECT users.name, orders.amount
FROM users
INNER JOIN orders
    ON users.id = orders.user_id;
```

Здесь получим только пользователей, у которых есть соответствующий заказ.

---

## 🎤 Суперкоротко

```text
INNER JOIN = только совпадения между таблицами
```

Или:

```text
A ∩ B
```

Условие совпадения задаётся через:

```python
ON ...
```

---

# 📌 Пример

Есть таблица `users`:

```text
id | name
---+--------
1  | Илья
2  | Анна
3  | Максим
```

И таблица `orders`:

```text
id | user_id | amount
---+---------+-------
10 | 1       | 500
11 | 1       | 700
12 | 2       | 300
```

Связь:

```text
users.id = orders.user_id
```

---

# 🔹 Простой INNER JOIN

```python
SELECT users.name, orders.amount
FROM users
INNER JOIN orders
    ON users.id = orders.user_id;
```

Результат:

```text
name | amount
-----+-------
Илья | 500
Илья | 700
Анна | 300
```

Максим отсутствует.

Почему?

У Максима:

```text
users.id = 3
```

Но в `orders` нет:

```text
orders.user_id = 3
```

Поэтому совпадения нет → строка не попадает в результат.

---

# 🔹 Как работает INNER JOIN

Условно PostgreSQL делает:

```text
users                 orders

id=1 ───────────────→ user_id=1
id=1 ───────────────→ user_id=1
id=2 ───────────────→ user_id=2
id=3                 ✗
```

Получаются только пары, для которых:

```text
users.id = orders.user_id
```

---

# 🔹 INNER JOIN может вернуть несколько строк

Это важный момент.

У одного пользователя может быть несколько заказов:

```text
users

1 | Илья
```

```text
orders

10 | 1 | 500
11 | 1 | 700
12 | 1 | 900
```

После `INNER JOIN`:

```text
Илья | 500
Илья | 700
Илья | 900
```

То есть одна строка `users` может соединиться с **несколькими строками `orders`**.

Поэтому `JOIN` не гарантирует сохранение количества строк исходной таблицы.

---

# 🔹 INNER JOIN и отношения 1:N

Очень распространённый случай:

```text
User 1 ──────── N Orders
```

Например:

```text
users
  │
  ├── order 1
  ├── order 2
  └── order 3
```

При `INNER JOIN` пользователь появится три раза — по одному разу для каждого совпавшего заказа.

---

# 🔹 INNER JOIN и WHERE

Можно дополнительно фильтровать результат:

```python
SELECT users.name, orders.amount
FROM users
INNER JOIN orders
    ON users.id = orders.user_id
WHERE orders.amount > 500;
```

Сначала соединяются таблицы:

```text
Илья | 500
Илья | 700
Анна | 300
```

Затем `WHERE` оставляет:

```text
Илья | 700
```

---

# 🔹 INNER JOIN с несколькими условиями

Условие `ON` может содержать несколько частей:

```python
SELECT users.name, orders.amount
FROM users
INNER JOIN orders
    ON users.id = orders.user_id
    AND orders.status = 'paid';
```

Теперь соединяем пользователя только с его оплаченными заказами.

---

# 🔹 INNER JOIN нескольких таблиц

Можно объединять больше двух таблиц:

```python
SELECT
    users.name,
    orders.id,
    payments.status
FROM users
INNER JOIN orders
    ON users.id = orders.user_id
INNER JOIN payments
    ON orders.id = payments.order_id;
```

Получается:

```text
users
  │
  │ users.id = orders.user_id
  ▼
orders
  │
  │ orders.id = payments.order_id
  ▼
payments
```

При `INNER JOIN` на каждом этапе остаются только строки, для которых найдено соответствие.

---

# 🔹 INNER JOIN и NULL

`INNER JOIN` не возвращает строки, если условие:

```python
users.id = orders.user_id
```

не выполняется.

Например:

```text
users

3 | Максим
```

и:

```text
orders

нет user_id = 3
```

Максим не попадёт в результат.

Это главное отличие от:

```text
LEFT JOIN
```

где Максим попал бы в результат с `NULL` справа.

---

# 📊 INNER JOIN vs LEFT JOIN

|                                         | INNER JOIN | LEFT JOIN |
| --------------------------------------- | ---------- | --------- |
| Совпадения                              | ✅          | ✅         |
| Строки без совпадения слева             | ❌          | ✅         |
| Строки без совпадения справа            | ❌          | ❌         |
| `NULL` справа при отсутствии совпадения | ❌          | ✅         |

Пример:

```text
users:
Илья
Анна
Максим

orders:
Илья
Анна
```

### INNER JOIN

```text
Илья
Анна
```

### LEFT JOIN

```text
Илья
Анна
Максим | NULL
```

---

# 🔹 INNER JOIN без слова INNER

В SQL можно написать:

```python
SELECT *
FROM users
JOIN orders
    ON users.id = orders.user_id;
```

Это то же самое, что:

```python
SELECT *
FROM users
INNER JOIN orders
    ON users.id = orders.user_id;
```

`INNER` является необязательным.

На практике часто пишут просто:

```python
JOIN
```

---

# 🔹 JOIN по неравенству

`INNER JOIN` не обязательно должен использовать только `=`.

Например:

```python
SELECT *
FROM products
INNER JOIN discounts
    ON products.price >= discounts.min_price;
```

Условие может использовать:

```text
=
>
<
>=
<=
<>
```

а также более сложные логические выражения.

Но наиболее распространённый вариант — соединение по ключам:

```python
ON users.id = orders.user_id
```

---

# 🔹 INNER JOIN и внешние ключи

Типичный backend-сценарий:

```python
CREATE TABLE users (
    id INTEGER PRIMARY KEY
);
```

```python
CREATE TABLE orders (
    id INTEGER PRIMARY KEY,
    user_id INTEGER REFERENCES users(id)
);
```

Тогда:

```python
SELECT users.id, orders.id
FROM users
INNER JOIN orders
    ON users.id = orders.user_id;
```

соединяет пользователя с его заказами.

Важно:

**`INNER JOIN` не требует наличия `FOREIGN KEY`.**

SQL может соединять любые столбцы, если условие `ON` это позволяет.

Но внешний ключ обеспечивает **целостность данных**, а `JOIN` выполняет **выборку связанных данных**.

---

# 🔹 INNER JOIN и производительность

На больших таблицах JOIN может быть дорогим.

Например:

```python
SELECT *
FROM users
INNER JOIN orders
    ON users.id = orders.user_id;
```

Часто полезен индекс:

```python
CREATE INDEX idx_orders_user_id
ON orders(user_id);
```

Но PostgreSQL сам выбирает алгоритм выполнения запроса.

Можно посмотреть план:

```python
EXPLAIN ANALYZE
SELECT *
FROM users
INNER JOIN orders
    ON users.id = orders.user_id;
```

PostgreSQL может использовать, например:

```text
Nested Loop
Hash Join
Merge Join
```

---

# 🧠 Как объяснить INNER JOIN на собеседовании

Хорошая формулировка:

> `INNER JOIN` соединяет строки двух таблиц по условию `ON` и возвращает только те комбинации строк, для которых условие соединения истинно. Если соответствующей строки нет, она не попадает в результат.

Пример:

```python
SELECT
    u.name,
    o.amount
FROM users AS u
INNER JOIN orders AS o
    ON u.id = o.user_id;
```

Здесь:

```text
u.id
 ↓
o.user_id
```

и результат содержит только пользователей, у которых есть заказ.

---

# 🎯 Главное

```text
INNER JOIN
      ↓
только совпавшие строки
```

Основная конструкция:

```python
SELECT ...
FROM table_a
INNER JOIN table_b
    ON table_a.id = table_b.a_id;
```

Причём:

```python
JOIN
```

и:

```python
INNER JOIN
```

— одно и то же.

Главное отличие от `LEFT JOIN`:

```text
INNER JOIN
→ без совпадения строка исчезает

LEFT JOIN
→ строка остаётся, справа будет NULL
```
