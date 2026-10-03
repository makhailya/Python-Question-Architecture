# 🫱 LEFT JOIN в SQL

## 🎯 Ответ на собеседовании

**`LEFT JOIN` возвращает все строки из левой таблицы и подходящие строки из правой таблицы.**

Если для строки левой таблицы соответствия справа нет, SQL всё равно оставляет эту строку, а столбцы правой таблицы заполняет `NULL`.

```python
SELECT users.name, orders.amount
FROM users
LEFT JOIN orders
    ON users.id = orders.user_id;
```

---

## 🎤 Суперкоротко

```text
LEFT JOIN
→ сохранить ВСЕ строки слева
→ справа взять совпадения
→ если совпадения нет → NULL
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
```

Соединяем:

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

**Максим остаётся**, несмотря на отсутствие заказа.

---

# 🔹 Почему появляется `NULL`

SQL берёт каждую строку **левой** таблицы:

```text
Илья
Анна
Максим
```

и ищет соответствие в `orders`.

```text
Илья   → найдено → 500
Илья   → найдено → 700
Анна   → найдено → 300
Максим → не найдено → NULL
```

То есть:

```text
LEFT JOIN
    ↓
левая строка сохраняется
    ↓
справа нет соответствия
    ↓
NULL
```

---

# 🔹 Главное отличие от INNER JOIN

Пусть:

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

Можно запомнить:

```text
INNER JOIN → только совпадения

LEFT JOIN  → всё слева + совпадения справа
```

---

# 🔹 Почему LEFT JOIN особенно полезен

Очень частая задача:

> Найти пользователей, у которых **нет заказов**.

Используем:

```python
SELECT users.*
FROM users
LEFT JOIN orders
    ON users.id = orders.user_id
WHERE orders.id IS NULL;
```

Результат:

```text
3 | Максим
```

Логика:

```text
users
  ↓
LEFT JOIN orders
  ↓
пользователь сохраняется
  ↓
заказ не найден
  ↓
orders.id = NULL
  ↓
WHERE orders.id IS NULL
```

Это называется паттерном **anti-join через `LEFT JOIN`**.

---

# 🔹 LEFT JOIN и `WHERE` — важный нюанс ⚠️

Рассмотрим:

```python
SELECT users.name, orders.amount
FROM users
LEFT JOIN orders
    ON users.id = orders.user_id
WHERE orders.amount > 500;
```

На первый взгляд кажется, что это всё ещё обычный `LEFT JOIN`.

Но `WHERE` отбрасывает:

```text
orders.amount = NULL
```

Поэтому пользователи без заказов исчезают.

Фактически мы получаем поведение, похожее на:

```text
INNER JOIN
```

---

# 🔹 Условие в `ON` и `WHERE` — не одно и то же

Допустим, нам нужны пользователи и **только заказы дороже 500**.

### Вариант 1 — условие в `WHERE`

```python
SELECT users.name, orders.amount
FROM users
LEFT JOIN orders
    ON users.id = orders.user_id
WHERE orders.amount > 500;
```

Пользователь без заказа будет отброшен.

---

### Вариант 2 — условие в `ON`

```python
SELECT users.name, orders.amount
FROM users
LEFT JOIN orders
    ON users.id = orders.user_id
    AND orders.amount > 500;
```

Теперь пользователь остаётся:

```text
Илья   | 700
Анна   | NULL
Максим | NULL
```

Потому что условие определяет, **какие строки справа присоединять**, но не удаляет строки левой таблицы.

### Это важно запомнить:

```text
ON
→ определяет совпадение при JOIN

WHERE
→ фильтрует итоговый результат
```

---

# 🔹 LEFT JOIN и отношения 1:N

Одна строка слева может соответствовать нескольким строкам справа.

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

После:

```python
SELECT users.name, orders.amount
FROM users
LEFT JOIN orders
    ON users.id = orders.user_id;
```

получим:

```text
Илья | 500
Илья | 700
Илья | 900
```

То есть `JOIN` может **увеличить количество строк**.

Если справа нет ни одной строки:

```text
Максим | NULL
```

Если справа 3 строки:

```text
Илья | 500
Илья | 700
Илья | 900
```

---

# 🔹 LEFT JOIN нескольких таблиц

Можно строить цепочку:

```python
SELECT
    users.name,
    orders.id,
    payments.status
FROM users
LEFT JOIN orders
    ON users.id = orders.user_id
LEFT JOIN payments
    ON orders.id = payments.order_id;
```

Смысл:

```text
users
  │
  ├── orders
  │      │
  │      └── payments
  │
  └── пользователь всё равно сохраняется
```

Если у пользователя нет заказа:

```text
name   | order_id | payment
-------+----------+--------
Максим | NULL     | NULL
```

---

# 🔹 LEFT JOIN и `COUNT`

Очень распространённый backend/SQL сценарий — посчитать количество заказов каждого пользователя.

```python
SELECT
    users.id,
    users.name,
    COUNT(orders.id) AS orders_count
FROM users
LEFT JOIN orders
    ON users.id = orders.user_id
GROUP BY users.id, users.name;
```

Результат:

```text
id | name   | orders_count
---+--------+-------------
1  | Илья   | 2
2  | Анна   | 1
3  | Максим | 0
```

Почему используется:

```text
COUNT(orders.id)
```

а не:

```text
COUNT(*)
```

Для Максима после `LEFT JOIN` существует строка результата, но:

```text
orders.id = NULL
```

`COUNT(orders.id)` не считает `NULL`.

Поэтому получается:

```text
Максим → 0
```

---

# 🔹 LEFT JOIN и NULL

`NULL` означает не:

```text
0
```

и не:

```text
''
```

а отсутствие значения.

Например:

```text
Максим | NULL
```

означает:

> Для Максима соответствующей строки в правой таблице не найдено.

Проверять `NULL` нужно через:

```python
WHERE orders.id IS NULL
```

а не:

```python
WHERE orders.id = NULL
```

Неправильно:

```python
WHERE orders.id = NULL
```

Правильно:

```python
WHERE orders.id IS NULL
```

---

# 🔹 LEFT JOIN и алиасы

Чтобы запрос был читаемее:

```python
SELECT
    u.name,
    o.amount
FROM users AS u
LEFT JOIN orders AS o
    ON u.id = o.user_id;
```

Здесь:

```text
u → users
o → orders
```

В больших запросах алиасы практически обязательны для читаемости.

---

# 🔹 LEFT JOIN не меняет исходные таблицы

`LEFT JOIN` — это операция формирования **результата запроса**.

Он не:

* изменяет `users`;
* изменяет `orders`;
* создаёт новые строки физически;
* удаляет данные.

Он только формирует результирующий набор:

```text
users + orders
       ↓
    LEFT JOIN
       ↓
результат SELECT
```

---

# 🔹 LEFT JOIN и внешний ключ

Типичный случай:

```python
CREATE TABLE orders (
    id INTEGER PRIMARY KEY,
    user_id INTEGER REFERENCES users(id)
);
```

После этого можно:

```python
SELECT *
FROM users AS u
LEFT JOIN orders AS o
    ON u.id = o.user_id;
```

Но сам `LEFT JOIN` **не требует** `FOREIGN KEY`.

`FOREIGN KEY` отвечает за целостность данных.

`JOIN` отвечает за получение связанных данных.

---

# 📊 INNER JOIN vs LEFT JOIN

| Ситуация                        | INNER JOIN | LEFT JOIN |
| ------------------------------- | ---------: | --------: |
| Есть совпадение                 |          ✅ |         ✅ |
| Нет строки справа               |          ❌ |         ✅ |
| Строка слева сохраняется        |          ❌ |         ✅ |
| Справа при отсутствии → `NULL`  |          — |         ✅ |
| Найти пользователей без заказов |   неудобно |         ✅ |

---

# 🧠 Ментальная модель

Представь:

```text
ЛЕВАЯ ТАБЛИЦА
      │
      │ сохранить ВСЕ
      ↓
   LEFT JOIN
      │
      │ ищем совпадения
      ↓
ПРАВАЯ ТАБЛИЦА
```

Например:

```text
users
  │
  ├── Илья ─────→ orders
  │
  ├── Анна ─────→ orders
  │
  └── Максим ──X→ orders
                  ↓
                NULL
```

---

# 🎯 Главное

```text
LEFT JOIN
→ ВСЕ строки левой таблицы
→ совпадения из правой
→ если совпадения нет → NULL
```

Основной шаблон:

```python
SELECT ...
FROM table_a
LEFT JOIN table_b
    ON table_a.id = table_b.a_id;
```

Очень важный практический паттерн:

```python
SELECT a.*
FROM a
LEFT JOIN b
    ON a.id = b.a_id
WHERE b.id IS NULL;
```

Это:

```text
→ найти строки A,
→ для которых нет соответствующих строк B.
```

И главное различие:

```text
INNER JOIN
→ только совпадения

LEFT JOIN
→ всё слева + совпадения справа
```
