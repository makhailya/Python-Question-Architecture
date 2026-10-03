# 🔄 FULL OUTER JOIN в SQL

## 🎯 Ответ на собеседовании

**`FULL OUTER JOIN` возвращает все строки из обеих таблиц: и совпавшие, и те, для которых соответствия в другой таблице нет.**

Если совпадения нет:

* отсутствующие значения слева становятся `NULL`;
* отсутствующие значения справа становятся `NULL`.

```python
SELECT users.name, orders.amount
FROM users
FULL OUTER JOIN orders
    ON users.id = orders.user_id;
```

---

## 🎤 Суперкоротко

```text
FULL OUTER JOIN
→ ВСЁ из левой таблицы
→ ВСЁ из правой таблицы
→ совпадения объединяем
→ где пары нет → NULL
```

Условно:

```text
A ∪ B
```

---

# 📌 Пример

### users

```text
id | name
---+--------
1  | Илья
2  | Анна
3  | Максим
```

### orders

```text
id | user_id | amount
---+---------+-------
10 | 1       | 500
11 | 2       | 300
12 | 4       | 900
```

Обрати внимание:

* `1` есть в обеих таблицах;
* `2` есть в обеих;
* `3` есть только в `users`;
* `4` есть только в `orders`.

---

# 🔹 Выполняем FULL OUTER JOIN

```python
SELECT
    users.id,
    users.name,
    orders.id AS order_id,
    orders.amount
FROM users
FULL OUTER JOIN orders
    ON users.id = orders.user_id;
```

Результат:

```text
user_id | name   | order_id | amount
--------+--------+----------+-------
1       | Илья   | 10       | 500
2       | Анна   | 11       | 300
3       | Максим | NULL     | NULL
NULL    | NULL   | 12       | 900
```

---

# 🔹 Что произошло

### 1. Илья

Есть:

```text
users.id = 1
orders.user_id = 1
```

→ соединяем:

```text
Илья | 10 | 500
```

### 2. Анна

Есть совпадение:

```text
users.id = 2
orders.user_id = 2
```

→

```text
Анна | 11 | 300
```

### 3. Максим

Есть только в `users`:

```text
users.id = 3
```

→ сохраняем его:

```text
Максим | NULL | NULL
```

### 4. Заказ №12

Есть только в `orders`:

```text
orders.user_id = 4
```

→ сохраняем заказ:

```text
NULL | NULL | 12 | 900
```

---

# 🧠 Визуально

```text
          users                  orders

        ┌────────┐            ┌────────┐
        │ Илья   │───────────→│ 500    │
        │ Анна   │───────────→│ 300    │
        └────────┘            └────────┘
        │ Максим │
        └────────┘
                                 │ 900
                                 └───────
```

`FULL OUTER JOIN` сохраняет **обе стороны целиком**:

```text
users ONLY
    +
INNER JOIN
    +
orders ONLY
```

---

# 📊 Сравнение всех основных JOIN

| JOIN              | Что возвращает                |
| ----------------- | ----------------------------- |
| `INNER JOIN`      | Только совпадения             |
| `LEFT JOIN`       | Всё слева + совпадения справа |
| `RIGHT JOIN`      | Всё справа + совпадения слева |
| `FULL OUTER JOIN` | **Всё слева + всё справа**    |

Мнемоника:

```text
INNER → пересечение

LEFT → всё LEFT

RIGHT → всё RIGHT

FULL → всё с обеих сторон
```

---

# 🔹 FULL OUTER JOIN и NULL

`NULL` появляется с той стороны, где не было соответствующей строки.

### Только слева

```text
users:
3 | Максим

orders:
нет user_id = 3
```

Получаем:

```text
3 | Максим | NULL | NULL
```

### Только справа

```text
orders:
12 | user_id=4 | 900

users:
нет id = 4
```

Получаем:

```text
NULL | NULL | 12 | 900
```

---

# 🔹 Найти строки, которые есть только в одной таблице

Это один из интересных практических сценариев.

Допустим, нужно найти:

> записи, которые есть только в одной из двух таблиц.

Можно использовать:

```python
SELECT
    users.id,
    users.name,
    orders.id AS order_id
FROM users
FULL OUTER JOIN orders
    ON users.id = orders.user_id
WHERE users.id IS NULL
   OR orders.id IS NULL;
```

Результат:

```text
Максим | NULL
NULL   | order 12
```

То есть нашли **несовпадения с обеих сторон**.

---

# 🔹 FULL OUTER JOIN vs UNION

Иногда спрашивают, можно ли заменить `FULL OUTER JOIN`.

Концептуально `FULL OUTER JOIN` можно представить как:

```text
LEFT JOIN
+
RIGHT ONLY
```

Но просто заменять его на `UNION` нельзя без изменения логики запроса.

`JOIN` сопоставляет строки по условию:

```python
ON users.id = orders.user_id
```

а `UNION` объединяет результаты двух `SELECT`.

Это разные операции.

---

# 🔹 FULL OUTER JOIN и WHERE ⚠️

Как и с `LEFT JOIN`, нужно внимательно относиться к `WHERE`.

Например:

```python
SELECT *
FROM users
FULL OUTER JOIN orders
    ON users.id = orders.user_id
WHERE users.id IS NOT NULL;
```

Мы удалим строки, которые существовали только справа.

Поэтому после внешнего JOIN `WHERE` может изменить смысл результата.

---

# 🔹 FULL OUTER JOIN с несколькими таблицами

Можно использовать несколько соединений:

```python
SELECT *
FROM users
FULL OUTER JOIN orders
    ON users.id = orders.user_id
FULL OUTER JOIN payments
    ON orders.id = payments.order_id;
```

Но такие запросы быстро становятся сложными.

На практике `FULL OUTER JOIN` используется значительно реже, чем:

```text
INNER JOIN
LEFT JOIN
```

---

# 🔹 Где применяется FULL OUTER JOIN

Типичные сценарии:

### Сверка двух наборов данных

Например:

```text
Система A
    ↕
FULL OUTER JOIN
    ↕
Система B
```

Можно найти:

* записи, которые есть только в A;
* записи, которые есть только в B;
* записи, которые есть в обеих системах.

### Аудит данных

Например, сравнить:

```text
CRM
vs
ERP
```

### Миграция данных

Проверить:

```text
старая БД
vs
новая БД
```

и найти расхождения.

---

# 🔹 FULL OUTER JOIN в PostgreSQL

PostgreSQL поддерживает:

```python
FULL OUTER JOIN
```

Также можно написать сокращённо:

```python
FULL JOIN
```

То есть:

```python
FROM users
FULL JOIN orders
    ON users.id = orders.user_id;
```

и:

```python
FROM users
FULL OUTER JOIN orders
    ON users.id = orders.user_id;
```

имеют одинаковый смысл.

---

# 🎤 Как ответить на собеседовании

Если спросят:

> Чем `FULL OUTER JOIN` отличается от `LEFT JOIN`?

Хороший ответ:

> `LEFT JOIN` гарантированно сохраняет все строки только левой таблицы, а `FULL OUTER JOIN` сохраняет все строки обеих таблиц. Если соответствия нет с одной из сторон, значения другой стороны будут `NULL`.

Если спросят:

> Чем отличается от `INNER JOIN`?

Ответ:

> `INNER JOIN` возвращает только совпавшие строки, а `FULL OUTER JOIN` возвращает и совпадения, и строки без соответствия с обеих сторон.

---

# 🎯 Главное

```text
FULL OUTER JOIN
        ↓
┌─────────────────────┐
│ ВСЯ левая таблица   │
│         +           │
│ ВСЯ правая таблица  │
└─────────────────────┘
```

Основной шаблон:

```python
SELECT ...
FROM table_a
FULL OUTER JOIN table_b
    ON table_a.id = table_b.a_id;
```

Логика:

```text
Есть слева + есть справа
        ↓
      JOIN

Есть только слева
        ↓
   справа NULL

Есть только справа
        ↓
   слева NULL
```

Итого:

```text
INNER JOIN
→ только пересечение

LEFT JOIN
→ всё слева

RIGHT JOIN
→ всё справа

FULL OUTER JOIN
→ всё с обеих сторон
```
