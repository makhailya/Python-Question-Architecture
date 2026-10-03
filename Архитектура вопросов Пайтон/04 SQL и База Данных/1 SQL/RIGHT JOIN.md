# 👉 RIGHT JOIN в SQL

## 🎯 Ответ на собеседовании

**`RIGHT JOIN` возвращает все строки из правой таблицы и подходящие строки из левой таблицы.**

Если соответствующей строки в левой таблице нет, строка правой таблицы всё равно попадёт в результат, а столбцы левой таблицы будут заполнены `NULL`.

```python
SELECT users.name, orders.amount
FROM users
RIGHT JOIN orders
    ON users.id = orders.user_id;
```

---

## 🎤 Суперкоротко

```text
RIGHT JOIN
→ сохранить ВСЕ строки справа
→ слева взять совпадения
→ если совпадения нет → NULL
```

Главное:

```text
LEFT JOIN  → сохраняем левую таблицу
RIGHT JOIN → сохраняем правую таблицу
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
11 | 1       | 700
12 | 2       | 300
13 | 4       | 900
```

Обрати внимание: заказ `13` с `user_id = 4` не имеет соответствующего пользователя.

---

# 🔹 Простой RIGHT JOIN

```python
SELECT users.name, orders.amount
FROM users
RIGHT JOIN orders
    ON users.id = orders.user_id;
```

Результат:

```text
name | amount
-----+-------
Илья | 500
Илья | 700
Анна | 300
NULL | 900
```

Почему появилась:

```text
NULL | 900
```

Потому что заказ существует:

```text
orders.user_id = 4
```

но пользователя:

```text
users.id = 4
```

нет.

**Правая таблица `orders` сохраняется полностью.**

---

# 🔹 Визуально

```text
users                         orders

Илья  ─────────────────────→  500
Илья  ─────────────────────→  700
Анна  ─────────────────────→  300
                              900
                               ↑
                         пользователя нет
                               ↓
                             NULL
```

То есть:

```text
RIGHT JOIN
     ↓
сохранить ВСЕ orders
     ↓
найти users
     ↓
не найден → NULL
```

---

# 🔹 RIGHT JOIN vs LEFT JOIN

Эти JOIN являются зеркальными.

### RIGHT JOIN

```python
SELECT *
FROM users
RIGHT JOIN orders
    ON users.id = orders.user_id;
```

Сохраняет:

```text
orders
```

### LEFT JOIN

Можно переписать тот же смысл:

```python
SELECT *
FROM orders
LEFT JOIN users
    ON users.id = orders.user_id;
```

Сохраняет:

```text
orders
```

Поэтому:

```text
users RIGHT JOIN orders
```

эквивалентен по смыслу:

```text
orders LEFT JOIN users
```

при соответствующей перестановке таблиц в `SELECT`.

---

# 🔹 Почему RIGHT JOIN используют редко

На практике разработчики часто предпочитают `LEFT JOIN`.

Например:

```python
SELECT *
FROM users
RIGHT JOIN orders
    ON users.id = orders.user_id;
```

можно записать понятнее:

```python
SELECT *
FROM orders
LEFT JOIN users
    ON users.id = orders.user_id;
```

При чтении второго запроса проще сразу понять:

> `orders` — главная таблица, все её строки нужно сохранить.

Поэтому `LEFT JOIN` обычно встречается значительно чаще.

---

# 🔹 RIGHT JOIN и INNER JOIN

Допустим:

```text
users:

1 | Илья
2 | Анна

orders:

10 | 1 | 500
11 | 3 | 900
```

### INNER JOIN

```python
SELECT users.name, orders.amount
FROM users
INNER JOIN orders
    ON users.id = orders.user_id;
```

Получим:

```text
Илья | 500
```

Заказ `900` исчезает, потому что пользователя `3` нет.

---

### RIGHT JOIN

```python
SELECT users.name, orders.amount
FROM users
RIGHT JOIN orders
    ON users.id = orders.user_id;
```

Получим:

```text
Илья | 500
NULL | 900
```

Потому что **все строки правой таблицы сохраняются**.

---

# 🔹 RIGHT JOIN и WHERE ⚠️

Как и с `LEFT JOIN`, важно понимать влияние `WHERE`.

Например:

```python
SELECT users.name, orders.amount
FROM users
RIGHT JOIN orders
    ON users.id = orders.user_id
WHERE users.name = 'Илья';
```

Строка:

```text
NULL | 900
```

будет удалена, потому что:

```text
users.name = NULL
```

и условие:

```python
users.name = 'Илья'
```

не выполняется.

То есть `WHERE` после внешнего JOIN может удалить строки, которые JOIN специально сохранял.

---

# 🔹 Условие в ON

Можно перенести фильтрацию в `ON`:

```python
SELECT users.name, orders.amount
FROM users
RIGHT JOIN orders
    ON users.id = orders.user_id
    AND users.name = 'Илья';
```

Теперь все `orders` сохраняются, но пользователь присоединяется только если его имя — `Илья`.

Результат концептуально:

```text
Илья | 500
NULL | 900
NULL | 300
```

То есть:

```text
ON
→ определяет, какую строку присоединить

WHERE
→ фильтрует итоговый результат
```

---

# 🔹 RIGHT JOIN и несколько строк

Как и любой обычный JOIN, `RIGHT JOIN` может размножать строки.

Например:

```text
orders

10 | user_id=1 | 500
11 | user_id=1 | 700
12 | user_id=1 | 900
```

Пользователь:

```text
1 | Илья
```

После JOIN:

```text
Илья | 500
Илья | 700
Илья | 900
```

Одна строка `users` соответствует трём строкам `orders`.

---

# 🔹 RIGHT JOIN нескольких таблиц

Можно использовать несколько JOIN:

```python
SELECT
    users.name,
    orders.id,
    payments.status
FROM users
RIGHT JOIN orders
    ON users.id = orders.user_id
RIGHT JOIN payments
    ON orders.id = payments.order_id;
```

Но такой запрос быстро становится менее читаемым.

Часто его можно переписать с использованием `LEFT JOIN`, выбрав главную таблицу первой:

```python
SELECT
    users.name,
    orders.id,
    payments.status
FROM payments
LEFT JOIN orders
    ON orders.id = payments.order_id
LEFT JOIN users
    ON users.id = orders.user_id;
```

---

# 📊 Сравнение JOIN

| JOIN              | Что сохраняется        |
| ----------------- | ---------------------- |
| `INNER JOIN`      | Только совпадения      |
| `LEFT JOIN`       | **Вся левая таблица**  |
| `RIGHT JOIN`      | **Вся правая таблица** |
| `FULL OUTER JOIN` | Обе таблицы            |
| `CROSS JOIN`      | Все комбинации         |

Мнемоника:

```text
LEFT  → сохраняем LEFT
RIGHT → сохраняем RIGHT
```

---

# 🧠 Практический пример

Допустим, нужно получить **все заказы**, даже если пользователь был удалён или данные о нём отсутствуют.

Можно написать:

```python
SELECT
    o.id,
    o.amount,
    u.name
FROM users AS u
RIGHT JOIN orders AS o
    ON u.id = o.user_id;
```

Получим:

```text
order_id | amount | name
---------+--------+------
10       | 500    | Илья
11       | 700    | Илья
12       | 300    | Анна
13       | 900    | NULL
```

Но более привычная запись:

```python
SELECT
    o.id,
    o.amount,
    u.name
FROM orders AS o
LEFT JOIN users AS u
    ON u.id = o.user_id;
```

То же самое по смыслу, но читается проще:

```text
Все orders
    ↓
LEFT JOIN users
    ↓
если user найден → имя
если нет → NULL
```

---

# 🎯 Главное

```text
RIGHT JOIN
→ сохраняет ВСЕ строки правой таблицы
→ добавляет совпадения из левой
→ если совпадения нет → NULL слева
```

Основной шаблон:

```python
SELECT ...
FROM table_a
RIGHT JOIN table_b
    ON table_a.id = table_b.a_id;
```

Запомнить можно одной фразой:

> **LEFT JOIN сохраняет левую таблицу, RIGHT JOIN — правую.**

И важный практический момент:

```text
A RIGHT JOIN B
```

обычно можно переписать как:

```text
B LEFT JOIN A
```

Поэтому в реальном backend-коде чаще используют **`LEFT JOIN`**, а `RIGHT JOIN` нужен значительно реже.
