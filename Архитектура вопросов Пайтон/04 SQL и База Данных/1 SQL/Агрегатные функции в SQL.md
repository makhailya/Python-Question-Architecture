# 🧮 Агрегатные функции в SQL

## 🎯 Ответ на собеседовании

**Агрегатные функции выполняют вычисления над набором строк и возвращают одно итоговое значение для всего набора или для каждой группы при использовании `GROUP BY`.**

Основные агрегатные функции:

* `COUNT()` — количество;
* `SUM()` — сумма;
* `AVG()` — среднее;
* `MIN()` — минимум;
* `MAX()` — максимум.

Например:

```python
SELECT
    user_id,
    COUNT(*) AS orders_count,
    SUM(amount) AS total_amount
FROM orders
GROUP BY user_id;
```

Здесь агрегатные функции выполняются **отдельно для каждого `user_id`**.

---

## 🎤 Суперкоротко

```text
Агрегатная функция
→ берёт множество строк
→ выполняет вычисление
→ возвращает одно значение
```

Например:

```text
100
200
300
 ↓
SUM()
 ↓
600
```

С `GROUP BY`:

```text
user_id = 1 → SUM() → 600
user_id = 2 → SUM() → 900
user_id = 3 → SUM() → 250
```

---

# 📌 Основные агрегатные функции

| Функция   | Назначение | Пример        |
| --------- | ---------- | ------------- |
| `COUNT()` | Количество | `COUNT(*)`    |
| `SUM()`   | Сумма      | `SUM(amount)` |
| `AVG()`   | Среднее    | `AVG(price)`  |
| `MIN()`   | Минимум    | `MIN(price)`  |
| `MAX()`   | Максимум   | `MAX(price)`  |

---

# 🔹 COUNT()

`COUNT()` считает количество.

### COUNT(*)

```python
SELECT COUNT(*)
FROM users;
```

Например:

```text
users = 1000 строк

COUNT(*)
→ 1000
```

`COUNT(*)` считает **строки**, независимо от того, есть ли в отдельных столбцах `NULL`.

---

## COUNT(column)

```python
SELECT COUNT(email)
FROM users;
```

Здесь `NULL` не учитывается.

Например:

```text
id | email
---+----------------
1  | a@test.ru
2  | b@test.ru
3  | NULL
4  | c@test.ru
```

```text
COUNT(*)      → 4
COUNT(email)  → 3
```

Главное:

```text
COUNT(*)
→ считает строки

COUNT(column)
→ считает ненулевые значения column
```

---

# 🔹 COUNT(DISTINCT)

Можно посчитать количество **уникальных значений**:

```python
SELECT COUNT(DISTINCT city)
FROM users;
```

Если:

```text
Москва
Москва
Казань
Екатеринбург
Екатеринбург
```

получим:

```text
3
```

То есть:

```text
COUNT(DISTINCT column)
→ количество уникальных ненулевых значений
```

---

# 🔹 SUM()

`SUM()` вычисляет сумму значений.

```python
SELECT SUM(amount)
FROM orders;
```

Например:

```text
amount
------
500
700
300
```

Результат:

```text
1500
```

---

## SUM() + GROUP BY

```python
SELECT
    user_id,
    SUM(amount) AS total_amount
FROM orders
GROUP BY user_id;
```

Например:

```text
user_id | total_amount
--------+-------------
1       | 1200
2       | 700
3       | 900
```

То есть сумма считается **отдельно для каждой группы**.

---

# 🔹 AVG()

`AVG()` вычисляет среднее арифметическое.

```python
SELECT AVG(amount)
FROM orders;
```

Для:

```text
500
700
300
```

получим:

```text
500
```

Потому что:

```text
(500 + 700 + 300) / 3 = 500
```

---

## AVG() и NULL

`AVG()` игнорирует `NULL`.

Например:

```text
500
700
NULL
```

Тогда:

```text
AVG()
→ (500 + 700) / 2
→ 600
```

`NULL` не считается как `0`.

Это важный момент.

---

# 🔹 MIN()

Возвращает минимальное значение:

```python
SELECT MIN(price)
FROM products;
```

Например:

```text
100
500
250
800
```

Результат:

```text
100
```

---

# 🔹 MAX()

Возвращает максимальное значение:

```python
SELECT MAX(price)
FROM products;
```

Для:

```text
100
500
250
800
```

получим:

```text
800
```

---

# 🔹 Агрегатные функции + GROUP BY

Именно здесь агрегатные функции особенно полезны.

Допустим:

```text
orders

user_id | amount
--------+-------
1       | 500
1       | 700
2       | 300
2       | 400
3       | 900
```

Запрос:

```python
SELECT
    user_id,
    COUNT(*) AS orders_count,
    SUM(amount) AS total_amount,
    AVG(amount) AS avg_amount,
    MIN(amount) AS min_amount,
    MAX(amount) AS max_amount
FROM orders
GROUP BY user_id;
```

Получим:

```text
user_id | orders_count | total_amount | avg_amount | min_amount | max_amount
--------+--------------+--------------+------------+------------+-----------
1       | 2            | 1200         | 600        | 500        | 700
2       | 2            | 700          | 350        | 300        | 400
3       | 1            | 900          | 900        | 900        | 900
```

Одна группа:

```text
user_id = 1
```

превращается в одну результирующую строку.

---

# 🔹 Агрегат без GROUP BY

Можно использовать агрегатную функцию вообще без `GROUP BY`:

```python
SELECT
    COUNT(*),
    SUM(amount),
    AVG(amount),
    MIN(amount),
    MAX(amount)
FROM orders;
```

В этом случае вся таблица рассматривается как **один набор строк**.

Результат:

```text
count | sum  | avg | min | max
------+------|-----|-----|----
5     | 2800 | 560 | 300 | 900
```

Ментальная модель:

```text
Все строки
    ↓
агрегатная функция
    ↓
одно значение
```

---

# 🔹 Агрегат с GROUP BY

С `GROUP BY`:

```python
SELECT
    user_id,
    SUM(amount)
FROM orders
GROUP BY user_id;
```

Теперь:

```text
Все строки
    ↓
GROUP BY user_id
    ↓
несколько групп
    ↓
SUM() для каждой группы
    ↓
несколько результатов
```

---

# 🔹 Агрегатные функции и HAVING

`HAVING` особенно часто используется с агрегатами.

Например:

> Найти пользователей, у которых больше 5 заказов.

```python
SELECT
    user_id,
    COUNT(*) AS orders_count
FROM orders
GROUP BY user_id
HAVING COUNT(*) > 5;
```

Здесь:

```text
COUNT()
→ считает заказы

HAVING
→ фильтрует группы по результату COUNT()
```

---

# 🔹 WHERE нельзя использовать для фильтрации агрегата

Неправильно:

```python
SELECT
    user_id,
    COUNT(*)
FROM orders
WHERE COUNT(*) > 5
GROUP BY user_id;
```

Потому что `WHERE` работает до группировки и агрегации.

Правильно:

```python
SELECT
    user_id,
    COUNT(*)
FROM orders
GROUP BY user_id
HAVING COUNT(*) > 5;
```

---

# 🔥 WHERE + GROUP BY + HAVING

Очень важная связка:

```python
SELECT
    user_id,
    COUNT(*) AS orders_count,
    SUM(amount) AS total_amount
FROM orders
WHERE amount > 100
GROUP BY user_id
HAVING COUNT(*) >= 5;
```

Логика:

```text
orders
   ↓
WHERE amount > 100
   ↓
фильтруем строки
   ↓
GROUP BY user_id
   ↓
формируем группы
   ↓
COUNT / SUM
   ↓
HAVING COUNT(*) >= 5
   ↓
фильтруем группы
```

---

# 🔹 NULL и агрегатные функции

Большинство агрегатных функций игнорируют `NULL`.

Например:

```text
amount
------
100
200
NULL
300
```

Тогда:

```text
SUM(amount) → 600
AVG(amount) → 200
MIN(amount) → 100
MAX(amount) → 300
COUNT(amount) → 3
COUNT(*) → 4
```

Главное исключение по смыслу:

```text
COUNT(*)
→ считает строки
```

а не значения конкретного столбца.

---

# ⚠️ А что если все значения NULL?

Например:

```text
amount
------
NULL
NULL
NULL
```

Тогда:

```python
SELECT SUM(amount)
FROM orders;
```

результат будет:

```text
NULL
```

А:

```python
SELECT COUNT(amount)
FROM orders;
```

вернёт:

```text
0
```

При необходимости `NULL` можно заменить через `COALESCE`:

```python
SELECT COALESCE(SUM(amount), 0)
FROM orders;
```

Теперь результат:

```text
0
```

---

# 🔹 Агрегаты после JOIN

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

Получим:

```text
id | name   | orders_count
---+--------+-------------
1  | Илья   | 2
2  | Анна   | 1
3  | Максим | 0
```

Здесь одновременно используются:

```text
LEFT JOIN
GROUP BY
COUNT()
```

Это очень типичный SQL-паттерн.

---

# 🔹 Почему COUNT(o.id), а не COUNT(*)

После `LEFT JOIN` пользователь без заказов всё равно присутствует:

```text
user_id | order_id
--------+---------
3       | NULL
```

Поэтому:

```python
COUNT(*)
```

считает строку.

А:

```python
COUNT(o.id)
```

игнорирует `NULL`.

Поэтому:

```text
COUNT(*)
→ 1

COUNT(o.id)
→ 0
```

Для подсчёта связанных объектов после `LEFT JOIN` обычно нужен:

```python
COUNT(o.id)
```

---

# 🔹 Агрегатные функции не обязательно возвращают INTEGER

Например:

```python
AVG(price)
```

обычно возвращает числовой результат с дробной частью.

Например:

```text
500
700
```

Среднее:

```text
600
```

А если:

```text
500
701
```

результат будет:

```text
600.5
```

Точный тип результата зависит от типа исходного столбца и конкретной СУБД.

---

# 🔹 Можно использовать выражения внутри агрегатов

Например:

```python
SELECT SUM(price * quantity)
FROM order_items;
```

Здесь сначала для каждой строки вычисляется:

```text
price × quantity
```

а затем результаты суммируются.

Можно использовать `CASE`:

```python
SELECT
    SUM(
        CASE
            WHEN status = 'paid' THEN amount
            ELSE 0
        END
    ) AS paid_amount
FROM orders;
```

Так можно считать агрегаты только для определённого типа строк.

---

# 🔹 Несколько агрегатов одновременно

Это нормально:

```python
SELECT
    COUNT(*) AS orders_count,
    SUM(amount) AS total_amount,
    AVG(amount) AS avg_amount,
    MIN(amount) AS min_amount,
    MAX(amount) AS max_amount
FROM orders;
```

Одна выборка может одновременно вернуть несколько агрегатных показателей.

---

# 📊 Основные функции — ещё раз

```text
COUNT()
→ сколько

SUM()
→ сумма

AVG()
→ среднее

MIN()
→ минимум

MAX()
→ максимум
```

Можно запомнить:

```text
COUNT → количество
SUM   → сумма
AVG   → среднее
MIN   → минимум
MAX   → максимум
```

---

# 🧠 Ментальная модель

Без `GROUP BY`:

```text
100
200
300
400
 ↓
SUM()
 ↓
1000
```

С `GROUP BY`:

```text
user_id = 1
100
200
 ↓
SUM() = 300

user_id = 2
300
400
 ↓
SUM() = 700
```

То есть:

```text
Агрегат
→ сворачивает множество строк в одно значение
```

А `GROUP BY` определяет:

```text
по каким группам выполнять это сворачивание
```

---

# 🎤 Как ответить на собеседовании

**Вопрос: Что такое агрегатные функции?**

> Агрегатные функции выполняют вычисления над набором строк и возвращают одно значение для всего набора либо одно значение для каждой группы при использовании `GROUP BY`. Основные функции — `COUNT`, `SUM`, `AVG`, `MIN` и `MAX`.

**Вопрос: В чём разница между `COUNT(*)` и `COUNT(column)`?**

> `COUNT(*)` считает строки, а `COUNT(column)` считает только ненулевые значения этого столбца.

**Вопрос: Как получить пользователей с более чем пятью заказами?**

```python
SELECT
    user_id,
    COUNT(*) AS orders_count
FROM orders
GROUP BY user_id
HAVING COUNT(*) > 5;
```

**Вопрос: Что происходит с NULL в агрегатах?**

> Большинство агрегатных функций игнорируют `NULL`. Например, `AVG`, `SUM`, `MIN`, `MAX` и `COUNT(column)` не учитывают `NULL`. `COUNT(*)` считает строки независимо от `NULL`.

---

# 🎯 Главное

```text
Агрегатная функция
→ множество строк
→ одно вычисляемое значение
```

Основные:

```text
COUNT → количество
SUM   → сумма
AVG   → среднее
MIN   → минимум
MAX   → максимум
```

Без `GROUP BY`:

```text
вся выборка
    ↓
агрегат
    ↓
одно значение
```

С `GROUP BY`:

```text
выборка
    ↓
GROUP BY
    ↓
несколько групп
    ↓
агрегат для каждой группы
```

Связка:

```text
WHERE
→ фильтрует строки

GROUP BY
→ создаёт группы

COUNT / SUM / AVG / MIN / MAX
→ вычисляют значения по группам

HAVING
→ фильтрует группы по агрегатам
```

И особенно запомнить:

```text
COUNT(*)
→ считает строки

COUNT(column)
→ считает НЕ-NULL значения column
```
