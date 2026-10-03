# 🔗 JOIN в SQL

## 🎯 Ответ на собеседовании

**JOIN** — оператор SQL для объединения строк из нескольких таблиц по связанному условию.

Основные виды:

* [[INNER JOIN]] — только совпавшие строки;
* [[LEFT JOIN]] — все строки левой таблицы + совпадения справа;
* [[RIGHT JOIN]] — все строки правой таблицы + совпадения слева;
* [[FULL JOIN]] — все строки обеих таблиц;
* [[CROSS JOIN]] — декартово произведение.

Чаще всего в прикладном коде используются **[[INNER JOIN]] и [[LEFT JOIN]]**.

---

## 🎤 Суперкоротко

```python
SELECT *
FROM users
JOIN orders ON users.id = orders.user_id;
```

Здесь SQL соединяет:

```text
users.id
   ↓
orders.user_id
```

То есть находит заказы, принадлежащие пользователям.

---

# 📌 Пример таблиц

### users

```text
id | name
---+------
1  | Илья
2  | Анна
3  | Максим
```

### orders

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

# 🔹 INNER JOIN

Возвращает **только строки, для которых есть совпадение в обеих таблицах**.

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

Максим не попал в результат, потому что у него нет заказа.

### Схема

```text
users             orders

Илья  ─────────── 500
Илья  ─────────── 700
Анна  ─────────── 300
Максим            ✗
```

**INNER JOIN = только пересечение.**

---

# 🔹 LEFT JOIN

Возвращает **все строки левой таблицы**, даже если соответствующей строки справа нет.

```python
SELECT users.name, orders.amount
FROM users
LEFT JOIN orders
    ON users.id = orders.user_id;
```

Результат:

```text
name   | amount
-------+-------
Илья   | 500
Илья   | 700
Анна   | 300
Максим | NULL
```

Для Максима нет заказа, поэтому:

```text
amount = NULL
```

### Схема

```text
users                    orders

Илья  ────────────────→  500
Илья  ────────────────→  700
Анна  ────────────────→  300
Максим ────────────────→  NULL
```

**LEFT JOIN = сохранить всё из левой таблицы.**

---

# 🔹 RIGHT JOIN

Зеркальный вариант `LEFT JOIN`.

```python
SELECT users.name, orders.amount
FROM users
RIGHT JOIN orders
    ON users.id = orders.user_id;
```

Возвращает **все строки правой таблицы**.

На практике `RIGHT JOIN` используется реже, потому что его обычно можно переписать через `LEFT JOIN`, поменяв таблицы местами.

Например:

```python
FROM orders
LEFT JOIN users
    ON users.id = orders.user_id
```

---

# 🔹 FULL OUTER JOIN

Возвращает:

* совпавшие строки;
* строки только из левой таблицы;
* строки только из правой таблицы.

```python
SELECT *
FROM users
FULL OUTER JOIN orders
    ON users.id = orders.user_id;
```

Концептуально:

```text
LEFT ONLY
    +
INNER
    +
RIGHT ONLY
```

В результате сохраняются данные **обеих таблиц полностью**.

---

# 🔹 CROSS JOIN

Создаёт **декартово произведение** двух таблиц.

```python
SELECT *
FROM users
CROSS JOIN orders;
```

Если:

```text
users  = 3 строки
orders = 3 строки
```

то результат:

```text
3 × 3 = 9 строк
```

Каждая строка первой таблицы соединяется с каждой строкой второй.

⚠️ Поэтому `CROSS JOIN` нужно использовать осознанно — количество строк может очень быстро увеличиться.

---

# 📊 Сравнение JOIN

| JOIN              | Что возвращает                |
| ----------------- | ----------------------------- |
| `INNER JOIN`      | Только совпадения             |
| `LEFT JOIN`       | Всё слева + совпадения справа |
| `RIGHT JOIN`      | Всё справа + совпадения слева |
| `FULL OUTER JOIN` | Всё из обеих таблиц           |
| `CROSS JOIN`      | Все комбинации строк          |

---

# 🔹 Условие `ON`

Условие соединения обычно находится после `ON`.

```python
SELECT *
FROM users
JOIN orders
    ON users.id = orders.user_id;
```

`ON` определяет:

> Какие строки таблиц считаются связанными.

Например:

```python
ON users.id = orders.user_id
```

означает:

```text
ID пользователя
        =
ID пользователя в заказе
```

---

# 🔹 JOIN по нескольким условиям

Можно использовать несколько условий:

```python
SELECT *
FROM users
JOIN orders
    ON users.id = orders.user_id
    AND orders.status = 'paid';
```

Теперь соединяются только оплаченные заказы.

---

# 🔹 `ON` vs `WHERE`

Это важный вопрос на собеседовании.

Рассмотрим:

```python
SELECT *
FROM users
LEFT JOIN orders
    ON users.id = orders.user_id
WHERE orders.amount > 500;
```

`LEFT JOIN` сначала сохраняет пользователей без заказов, но затем:

```python
WHERE orders.amount > 500
```

отбрасывает строки, где:

```text
orders.amount = NULL
```

В результате такой `LEFT JOIN` может фактически вести себя как `INNER JOIN` относительно этого условия.

Если условие относится именно к присоединяемой таблице и нужно сохранить левую таблицу:

```python
SELECT *
FROM users
LEFT JOIN orders
    ON users.id = orders.user_id
    AND orders.amount > 500;
```

Здесь пользователь без подходящего заказа всё равно останется:

```text
Максим | NULL
```

---

# 🔹 JOIN и NULL

При `LEFT JOIN`, если соответствующей строки справа нет, SQL подставляет:

```text
NULL
```

Например:

```text
users

id | name
---+------
1  | Илья
2  | Анна
3  | Максим
```

```text
orders

user_id | amount
--------+-------
1       | 500
2       | 300
```

После `LEFT JOIN`:

```text
Илья   | 500
Анна   | 300
Максим | NULL
```

Поэтому для проверки отсутствия связанной записи часто используют:

```python
SELECT users.*
FROM users
LEFT JOIN orders
    ON users.id = orders.user_id
WHERE orders.id IS NULL;
```

Это означает:

> Найти пользователей, у которых нет заказов.

---

# 🔹 JOIN одной таблицы с несколькими

Можно соединять несколько таблиц:

```python
SELECT
    users.name,
    orders.id,
    payments.status
FROM users
JOIN orders
    ON users.id = orders.user_id
JOIN payments
    ON orders.id = payments.order_id;
```

Получается цепочка:

```text
users
  │
  │ user_id
  ↓
orders
  │
  │ order_id
  ↓
payments
```

Это очень распространённый сценарий в backend-разработке.

---

# 🔹 JOIN и производительность

`JOIN` может быть дорогой операцией на больших таблицах.

Особенно важно наличие подходящих индексов.

Например:

```python
CREATE INDEX idx_orders_user_id
ON orders(user_id);
```

Если часто выполняется:

```python
JOIN orders
    ON users.id = orders.user_id
```

индекс на:

```text
orders.user_id
```

может значительно помочь.

Но индекс не гарантирует использование: PostgreSQL выбирает план выполнения на основе статистики и стоимости операций.

---

# 🔹 Как PostgreSQL выполняет JOIN

PostgreSQL может использовать разные алгоритмы соединения.

### 1. Nested Loop

```text
для каждой строки A
    найти подходящие строки B
```

Хорошо работает, например, когда одна таблица маленькая или есть эффективный индекс.

### 2. Hash Join

```text
построить hash table
        ↓
сопоставлять строки по ключу
```

Часто эффективен для больших объёмов данных при соединении по равенству.

### 3. Merge Join

```text
отсортировать обе стороны
        ↓
идти по ним одновременно
```

Эффективен, когда данные уже отсортированы или сортировка выгодна.

Посмотреть выбранный PostgreSQL план можно через:

```python
EXPLAIN ANALYZE
SELECT *
FROM users
JOIN orders
    ON users.id = orders.user_id;
```

---

# 🔹 JOIN не обязательно означает связь через FK

На практике часто соединяют таблицы по логически связанным значениям:

```python
JOIN users
    ON users.email = orders.customer_email
```

Но для нормализованной реляционной модели обычно предпочтительнее связь через идентификаторы:

```python
users.id = orders.user_id
```

и внешний ключ:

```python
FOREIGN KEY (user_id)
REFERENCES users(id)
```

---

# 🧠 JOIN в голове

Представляй:

```text
       JOIN
        │
        ▼
┌─────────────┐
│ Таблица A   │
└──────┬──────┘
       │
       │ ON условие
       │
┌──────▼──────┐
│ Таблица B   │
└─────────────┘
```

Главный вопрос:

> **Какие строки A и B нужно считать связанными?**

Ответ находится в:

```python
ON ...
```

А вопрос:

> **Какие строки оставить после соединения?**

может дополнительно решаться через:

```python
WHERE ...
```

---

# 🎯 Главное для собеседования

```text
INNER JOIN
→ только совпавшие

LEFT JOIN
→ всё слева + совпадения справа

RIGHT JOIN
→ всё справа + совпадения слева

FULL OUTER JOIN
→ всё с обеих сторон

CROSS JOIN
→ каждая строка × каждая строка
```

Ключевая конструкция:

```python
SELECT ...
FROM table_a
JOIN table_b
    ON table_a.id = table_b.a_id;
```

И главное различие:

```text
ON
→ определяет условие соединения

WHERE
→ фильтрует уже полученный результат
```
