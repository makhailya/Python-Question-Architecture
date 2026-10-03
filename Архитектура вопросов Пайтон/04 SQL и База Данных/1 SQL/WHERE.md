# 🔎 WHERE в SQL

## 🎯 Ответ на собеседовании

**`WHERE` — оператор фильтрации строк в SQL.** Он задаёт условие, которому должна соответствовать строка, чтобы попасть в результат запроса.

```python
SELECT *
FROM users
WHERE age >= 18;
```

В результат попадут только пользователи, для которых условие:

```text
age >= 18
```

истинно.

Важно: `WHERE` работает **до `GROUP BY` и агрегатных функций** и не используется для фильтрации результата агрегирования — для этого применяется `HAVING`.

---

## 🎤 Суперкоротко

```text
WHERE
→ фильтрует строки
→ оставляет только подходящие
```

Шаблон:

```python
SELECT ...
FROM table
WHERE condition;
```

Например:

```python
SELECT *
FROM users
WHERE age > 30;
```

---

# 📌 Простой пример

Таблица:

```text
id | name   | age
---+--------+----
1  | Илья   | 31
2  | Анна   | 25
3  | Максим | 17
```

Запрос:

```python
SELECT *
FROM users
WHERE age >= 18;
```

Результат:

```text
1 | Илья   | 31
2 | Анна   | 25
```

Максим исключён, потому что:

```text
17 >= 18 → FALSE
```

---

# 🔹 Операторы сравнения

В `WHERE` используются обычные операторы сравнения:

| Оператор | Значение         |
| -------- | ---------------- |
| `=`      | равно            |
| `<>`     | не равно         |
| `!=`     | не равно         |
| `>`      | больше           |
| `<`      | меньше           |
| `>=`     | больше или равно |
| `<=`     | меньше или равно |

Пример:

```python
SELECT *
FROM products
WHERE price > 1000;
```

---

# 🔹 AND

`AND` требует выполнения **всех условий**.

```python
SELECT *
FROM users
WHERE age >= 18
  AND city = 'Екатеринбург';
```

Логика:

```text
age >= 18
     AND
city = Екатеринбург
```

Оба условия должны быть `TRUE`.

---

# 🔹 OR

`OR` требует выполнения **хотя бы одного условия**.

```python
SELECT *
FROM users
WHERE city = 'Екатеринбург'
   OR city = 'Москва';
```

Пользователь подходит, если он находится:

```text
Екатеринбург
ИЛИ
Москва
```

---

# 🔹 NOT

Инвертирует условие.

```python
SELECT *
FROM users
WHERE NOT is_blocked;
```

Или:

```python
SELECT *
FROM users
WHERE NOT city = 'Москва';
```

---

# 🔹 BETWEEN

Проверяет попадание значения в диапазон.

```python
SELECT *
FROM products
WHERE price BETWEEN 1000 AND 5000;
```

Обычно границы диапазона включаются:

```text
1000 <= price <= 5000
```

---

# 🔹 IN

Проверяет наличие значения в списке.

Вместо:

```python
SELECT *
FROM users
WHERE city = 'Москва'
   OR city = 'Екатеринбург'
   OR city = 'Казань';
```

можно написать:

```python
SELECT *
FROM users
WHERE city IN ('Москва', 'Екатеринбург', 'Казань');
```

Это особенно удобно для нескольких возможных значений.

---

# 🔹 NOT IN

Обратная проверка:

```python
SELECT *
FROM users
WHERE city NOT IN ('Москва', 'Екатеринбург');
```

Оставляет пользователей из городов, которых нет в списке.

⚠️ При работе с `NULL` у `NOT IN` есть важные особенности из-за трёхзначной логики SQL.

---

# 🔹 LIKE

Используется для поиска по шаблону.

```python
SELECT *
FROM users
WHERE name LIKE 'Ил%';
```

`%` означает любое количество символов.

Например:

```text
Илья
Илона
Илларион
```

могут соответствовать шаблону:

```text
Ил%
```

---

# 🔹 `_` в LIKE

Символ `_` соответствует **одному символу**.

```python
SELECT *
FROM users
WHERE name LIKE 'Ил_';
```

Например, шаблон соответствует строкам длиной три символа, начинающимся с `Ил`.

---

# 🔹 IS NULL

Для `NULL` нельзя использовать:

```python
WHERE email = NULL;
```

Правильно:

```python
WHERE email IS NULL;
```

И:

```python
WHERE email IS NOT NULL;
```

Причина — SQL использует трёхзначную логику:

```text
TRUE
FALSE
UNKNOWN
```

---

# 🔹 WHERE после JOIN

`WHERE` часто используется вместе с `JOIN`.

```python
SELECT
    u.name,
    o.amount
FROM users AS u
INNER JOIN orders AS o
    ON u.id = o.user_id
WHERE o.amount > 1000;
```

Здесь:

```text
JOIN
→ определяет, какие строки связать

WHERE
→ фильтрует получившиеся строки
```

---

# ⚠️ WHERE и LEFT JOIN

Это один из важных вопросов на собеседовании.

Допустим:

```python
SELECT
    u.name,
    o.amount
FROM users AS u
LEFT JOIN orders AS o
    ON u.id = o.user_id
WHERE o.amount > 1000;
```

`LEFT JOIN` должен сохранить пользователя даже без заказа.

Но после него:

```python
WHERE o.amount > 1000
```

строка:

```text
o.amount = NULL
```

условию не соответствует.

В результате пользователь без заказа исчезает.

Если нужно сохранить пользователя, условие часто переносят в `ON`:

```python
SELECT
    u.name,
    o.amount
FROM users AS u
LEFT JOIN orders AS o
    ON u.id = o.user_id
   AND o.amount > 1000;
```

Теперь:

```text
JOIN
→ какие заказы присоединять

WHERE
→ какие итоговые строки удалить
```

Это очень важное различие.

---

# 🔹 WHERE и GROUP BY

`WHERE` фильтрует **строки до группировки**.

```python
SELECT
    city,
    COUNT(*) AS users_count
FROM users
WHERE age >= 18
GROUP BY city;
```

Логика:

```text
users
  ↓
WHERE age >= 18
  ↓
GROUP BY city
  ↓
COUNT(*)
```

То есть несовершеннолетние вообще не участвуют в группировке.

---

# 🔹 WHERE vs HAVING

Это классический вопрос.

### WHERE

Фильтрует **отдельные строки**:

```python
SELECT *
FROM users
WHERE age >= 18;
```

### HAVING

Фильтрует **группы после GROUP BY**:

```python
SELECT
    city,
    COUNT(*) AS users_count
FROM users
GROUP BY city
HAVING COUNT(*) > 10;
```

Здесь:

```text
WHERE
→ фильтрация строк

GROUP BY
→ группировка

HAVING
→ фильтрация групп
```

---

# 🔹 WHERE и ORDER BY

`ORDER BY` не фильтрует данные.

Например:

```python
SELECT *
FROM users
WHERE age >= 18
ORDER BY age DESC;
```

Логика:

```text
WHERE
→ оставить взрослых

ORDER BY
→ отсортировать их по возрасту
```

---

# 🔹 WHERE и LIMIT

`LIMIT` ограничивает количество возвращаемых строк:

```python
SELECT *
FROM users
WHERE age >= 18
ORDER BY age DESC
LIMIT 10;
```

Получается:

```text
WHERE
  ↓
ORDER BY
  ↓
LIMIT
```

Концептуально:

```text
отфильтровать
→ отсортировать
→ взять первые 10
```

---

# 🧠 Порядок выполнения SQL

Это важно для собеседования.

Запрос:

```python
SELECT city, COUNT(*)
FROM users
WHERE age >= 18
GROUP BY city
HAVING COUNT(*) > 10
ORDER BY city
LIMIT 5;
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

То есть `WHERE` логически выполняется **до группировки**.

---

# 🔹 WHERE и алиасы SELECT

В стандартном SQL нельзя обычно использовать алиас из `SELECT` в `WHERE`.

Например:

```python
SELECT
    price * quantity AS total
FROM orders
WHERE total > 1000;
```

Такой запрос обычно не сработает.

Почему?

Потому что логически:

```text
WHERE
```

обрабатывается раньше:

```text
SELECT
```

Можно написать выражение непосредственно:

```python
SELECT
    price * quantity AS total
FROM orders
WHERE price * quantity > 1000;
```

---

# 🔹 WHERE и EXISTS

`WHERE` часто используется с `EXISTS`.

Например, найти пользователей, у которых есть хотя бы один заказ:

```python
SELECT *
FROM users AS u
WHERE EXISTS (
    SELECT 1
    FROM orders AS o
    WHERE o.user_id = u.id
);
```

Логика:

```text
для каждого пользователя
        ↓
существует ли заказ?
        ↓
YES → оставить
NO  → убрать
```

Это особенно полезно для проверки существования связанных данных.

---

# 🔹 WHERE и подзапрос

Например, найти товары дороже средней цены:

```python
SELECT *
FROM products
WHERE price > (
    SELECT AVG(price)
    FROM products
);
```

Внешний `WHERE` сравнивает цену каждого товара со значением, полученным подзапросом.

---

# ⚠️ WHERE не изменяет данные

`WHERE` в `SELECT` только фильтрует результат.

Но `WHERE` также используется в `UPDATE` и `DELETE`, где уже критически важно, какие строки будут изменены.

Например:

```python
UPDATE users
SET is_active = FALSE
WHERE id = 10;
```

Изменится только пользователь:

```text
id = 10
```

А:

```python
UPDATE users
SET is_active = FALSE;
```

изменит **всех пользователей**.

То же самое с `DELETE`:

```python
DELETE FROM users
WHERE id = 10;
```

Удаляет конкретную строку.

Без `WHERE`:

```python
DELETE FROM users;
```

удалит все строки таблицы.

⚠️ Поэтому `WHERE` в `UPDATE`/`DELETE` — особенно критичная часть запроса.

---

# 🎤 Как ответить на собеседовании

**Вопрос: Что такое WHERE?**

> `WHERE` — условие фильтрации строк в SQL. Оно оставляет только те строки, для которых условие истинно. Логически `WHERE` выполняется после `FROM/JOIN`, но до `GROUP BY`.

**Вопрос: Чем WHERE отличается от HAVING?**

> `WHERE` фильтрует отдельные строки до группировки, а `HAVING` фильтрует группы после `GROUP BY`, поэтому `HAVING` обычно используется с агрегатными функциями.

**Вопрос: Как проверить NULL?**

> Через `IS NULL` или `IS NOT NULL`, потому что сравнение через `= NULL` даёт `UNKNOWN`.

**Вопрос: Что произойдёт с LEFT JOIN, если поставить условие правой таблицы в WHERE?**

> Строки с `NULL` справа будут отфильтрованы, поэтому внешний `LEFT JOIN` может фактически начать вести себя как `INNER JOIN` для этого условия.

---

# 🎯 Главное

```text
WHERE
→ фильтрует строк
```
