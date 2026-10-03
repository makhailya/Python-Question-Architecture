# Index Scan в PostgreSQL 🔎

## 🎯 Ответ на собеседовании

**Index Scan** — способ выполнения запроса, при котором PostgreSQL использует индекс, чтобы найти подходящие записи, а затем обращается к таблице (**heap**) для получения самих данных.

Упрощённо:

```python
Индекс
  ↓
найти ссылки на строки
  ↓
Table / Heap
  ↓
получить данные
```

`Index Scan` обычно эффективен, когда запрос выбирает **небольшую часть таблицы** и условие хорошо поддерживается индексом.

---

## 🎤 Суперкоротко

```python
Index Scan
    ↓
индекс находит нужные строки
    ↓
PostgreSQL обращается к таблице
    ↓
получает данные
```

Главное отличие:

```python
Index Scan
    → Index + Table

Index Only Scan
    → Index
```

---

# 1. Простой пример

Есть таблица:

```python
CREATE TABLE users (
    id BIGSERIAL PRIMARY KEY,
    email TEXT,
    name TEXT
);
```

Создадим индекс:

```python
CREATE INDEX idx_users_email
ON users(email);
```

Запрос:

```python
EXPLAIN ANALYZE
SELECT *
FROM users
WHERE email = 'test@example.com';
```

PostgreSQL может выбрать:

```python
Index Scan using idx_users_email on users
  Index Cond: (email = 'test@example.com')
```

Это означает:

```python
idx_users_email
       ↓
найти подходящую запись
       ↓
получить ссылку на строку
       ↓
обратиться к таблице
       ↓
получить id, email, name
```

---

# 2. Что находится в индексе

Упрощённо можно представить индекс так:

```python
email                  → ссылка на строку
------------------------------------------
a@example.com          → TID
b@example.com          → TID
test@example.com       → TID
z@example.com          → TID
```

Индекс позволяет быстро определить, **где находится нужная строка**.

После этого PostgreSQL обращается к таблице за остальными данными.

---

# 3. Что такое Heap

В PostgreSQL основная таблица хранится в так называемом **heap**.

Упрощённо:

```python
Index
  ↓
TID
  ↓
Heap
  ↓
Row
```

Например, индекс знает:

```python
email = 'test@example.com'
        ↓
       TID
```

А PostgreSQL по этому TID находит физическую строку в таблице.

Поэтому `Index Scan` — это не просто «читаем индекс».

Это:

> **поиск через индекс + получение соответствующих строк из heap.**

---

# 4. `Index Cond`

В плане можно увидеть:

```python
Index Scan using idx_users_email on users
  Index Cond: (email = 'test@example.com')
```

`Index Cond` означает, что условие используется самим индексом для поиска.

То есть:

```python
email = 'test@example.com'
```

не просто фильтруется после чтения таблицы — индекс помогает сразу найти подходящие записи.

---

# 5. Index Scan с диапазоном

Индекс может использоваться не только для `=`.

Например, B-tree:

```python
CREATE INDEX idx_users_age
ON users(age);
```

Запрос:

```python
EXPLAIN ANALYZE
SELECT *
FROM users
WHERE age BETWEEN 20 AND 30;
```

может использовать:

```python
Index Scan using idx_users_age on users
  Index Cond: ((age >= 20) AND (age <= 30))
```

Это одно из преимуществ B-tree.

Он поддерживает:

```python
=
<
>
<=
>=
BETWEEN
```

и другие подходящие операции.

---

# 6. Почему Index Scan может быть быстрым

Представим:

```python
10 000 000 строк
```

и запрос:

```python
WHERE id = 123456
```

Результат:

```python
1 строка
```

Без индекса:

```python
Seq Scan
    ↓
10 000 000 строк
    ↓
найти нужную
```

С индексом:

```python
Index Scan
    ↓
найти id
    ↓
получить одну строку
```

Для такого запроса индекс обычно даёт большое преимущество.

---

# 7. Index Scan не всегда быстрее Seq Scan

Допустим:

```python
10 000 000 строк
```

и запрос возвращает:

```python
8 000 000 строк
```

Тогда:

```python
Index Scan
    ↓
миллионы обращений к таблице
```

может оказаться дороже, чем:

```python
Seq Scan
    ↓
последовательно прочитать таблицу
```

Поэтому PostgreSQL может сознательно выбрать `Seq Scan`.

---

# 8. Селективность

Ключевое понятие для понимания Index Scan — **селективность**.

Высокая селективность:

```python
WHERE id = 123
```

Например:

```python
10 000 000
      ↓
      1
```

Индекс очень полезен.

Низкая селективность:

```python
WHERE status = 'active'
```

Например:

```python
10 000 000
      ↓
  8 000 000
```

Индекс может быть менее эффективен.

---

# 9. Index Scan vs Seq Scan

|                                 | Seq Scan  | Index Scan |
| ------------------------------- | --------- | ---------- |
| Использует индекс               | ❌         | ✅          |
| Основной источник поиска        | Таблица   | Индекс     |
| Получает данные из heap         | Да        | Обычно да  |
| Хорош для небольшого результата | Не всегда | Часто      |
| Хорош для большого результата   | Часто     | Не всегда  |
| Зависит от селективности        | ✅         | ✅          |
| Всегда лучше                    | ❌         | ❌          |

---

# 10. Index Scan vs Index Only Scan

Это один из самых популярных вопросов.

### Index Scan

```python
Index
  ↓
Heap
  ↓
Result
```

Индекс используется для поиска, но PostgreSQL обращается к таблице за данными.

### Index Only Scan

```python
Index
  ↓
Result
```

Все необходимые для запроса значения находятся в индексе, поэтому PostgreSQL может избежать обычного чтения heap.

Например:

```python
CREATE INDEX idx_users_email
ON users(email);
```

Запрос:

```python
SELECT email
FROM users
WHERE email = 'test@example.com';
```

может позволить:

```python
Index Only Scan
```

А:

```python
SELECT *
FROM users
WHERE email = 'test@example.com';
```

скорее требует данных, которых в индексе нет, поэтому может использовать:

```python
Index Scan
```

---

# 11. Index Scan vs Bitmap Scan

При небольшом количестве найденных строк часто эффективен:

```python
Index Scan
```

При большем количестве совпадений PostgreSQL может выбрать:

```python
Bitmap Index Scan
        ↓
Bitmap Heap Scan
```

Упрощённо:

```python
мало строк
    ↓
Index Scan
```

```python
много строк
    ↓
Bitmap Scan
```

Но это не жёсткое правило.

Решение принимает planner на основе стоимости конкретного плана.

---

# 12. Index Scan и составной индекс

Допустим:

```python
CREATE INDEX idx_users_name_age
ON users(name, age);
```

Запрос:

```python
SELECT *
FROM users
WHERE name = 'Ivan'
  AND age = 30;
```

может использовать:

```python
Index Scan
```

Составной индекс:

```python
(name, age)
```

имеет определённый порядок столбцов.

Упрощённо важен принцип **leading/leftmost column**:

```python
(name, age)
 ↑
первый столбец
```

Индекс особенно хорошо подходит для условий, начинающихся с `name`.

Например:

```python
WHERE name = 'Ivan'
```

или:

```python
WHERE name = 'Ivan'
AND age = 30
```

А запрос только:

```python
WHERE age = 30
```

не обязательно сможет эффективно использовать такой индекс.

---

# 13. Index Scan и ORDER BY

B-tree хранит ключи в упорядоченном виде.

Поэтому индекс может помочь не только с:

```python
WHERE
```

но и с:

```python
ORDER BY
```

Например:

```python
CREATE INDEX idx_users_created_at
ON users(created_at);
```

Запрос:

```python
SELECT *
FROM users
ORDER BY created_at;
```

может использовать порядок индекса и избежать отдельной сортировки в подходящем плане.

Особенно полезен сценарий:

```python
ORDER BY created_at DESC
LIMIT 10;
```

Индекс может позволить быстро получить первые нужные строки.

---

# 14. Index Scan и LIMIT

Очень хороший сценарий:

```python
SELECT *
FROM users
ORDER BY created_at DESC
LIMIT 10;
```

Если есть подходящий индекс:

```python
CREATE INDEX idx_users_created_at
ON users(created_at);
```

PostgreSQL потенциально может быстро получить первые нужные записи из индекса.

Это может быть значительно эффективнее:

```python
Seq Scan
    ↓
прочитать всю таблицу
    ↓
Sort
    ↓
LIMIT 10
```

---

# 15. Почему индекс не гарантирует Index Scan

Допустим:

```python
CREATE INDEX idx_users_age
ON users(age);
```

Но PostgreSQL показывает:

```python
Seq Scan on users
```

Это нормально.

Причины могут быть:

* таблица маленькая;
* условие возвращает много строк;
* низкая селективность;
* статистика;
* стоимость random access;
* planner считает Seq Scan дешевле.

Поэтому нельзя заставлять PostgreSQL использовать индекс без понимания причины.

---

# 16. Как проверить Index Scan

Используем:

```python
EXPLAIN (ANALYZE, BUFFERS)
SELECT *
FROM users
WHERE email = 'test@example.com';
```

Например:

```python
Index Scan using idx_users_email on users
  (cost=0.42..8.44 rows=1 width=64)
  (actual time=0.030..0.035 rows=1 loops=1)
  Index Cond: (email = 'test@example.com')
```

Смотрим:

```python
Index Scan
```

→ индекс используется.

```python
Index Cond
```

→ условие поиска выполняется через индекс.

```python
actual rows=1
```

→ фактически найдена одна строка.

---

# 17. Что такое `ctid`

Внутри PostgreSQL строка имеет физический идентификатор:

```python
ctid
```

Например:

```python
(10,5)
```

Это можно представить как:

```python
(page, position)
```

Индекс содержит ссылки, позволяющие PostgreSQL найти соответствующие строки.

Но `ctid`:

* не является логическим ID;
* может измениться при обновлении строки;
* не должен использоваться как постоянный идентификатор записи.

---

# 18. Почему Index Scan может стать дорогим

Индекс позволяет быстро найти строки, но после этого PostgreSQL может совершить много обращений к heap.

Например:

```python
Index
 ↓
Row → Page 10
Row → Page 500
Row → Page 20
Row → Page 900
...
```

Если найдено огромное количество строк, такие обращения могут стать дорогими.

В такой ситуации planner может предпочесть:

```python
Bitmap Heap Scan
```

или:

```python
Seq Scan
```

---

# 19. Практический пример

Есть:

```python
orders
```

с:

```python
10 000 000 строк
```

Индекс:

```python
CREATE INDEX idx_orders_user_id
ON orders(user_id);
```

Запрос:

```python
EXPLAIN ANALYZE
SELECT *
FROM orders
WHERE user_id = 123;
```

Если у пользователя:

```python
5 заказов
```

вероятно, выгоден:

```python
Index Scan
```

Если у пользователя:

```python
2 000 000 заказов
```

planner уже может выбрать другой план.

---

# 20. Как читать Index Scan в EXPLAIN ANALYZE

Например:

```python
Index Scan using idx_orders_user_id on orders
  (cost=0.43..50.00 rows=10 width=100)
  (actual time=0.020..0.100 rows=8 loops=1)
  Index Cond: (user_id = 123)
```

Разбираем:

```python
Index Scan
```

→ используется индекс.

```python
using idx_orders_user_id
```

→ имя индекса.

```python
rows=10
```

→ planner ожидал 10 строк.

```python
actual rows=8
```

→ реально получили 8.

```python
loops=1
```

→ узел выполнился один раз.

```python
Index Cond
```

→ условие поиска по индексу.

---

## 🧠 Главное

```python
Index Scan
    ↓
используем индекс
    ↓
находим ссылки на строки
    ↓
обращаемся к heap
    ↓
получаем данные
```

### Три важных варианта:

```python
Seq Scan
→ читаем таблицу последовательно
```

```python
Index Scan
→ Index → Heap
```

```python
Index Only Scan
→ Index → Result
```

### И ключевая логика:

```python
мало подходящих строк
        ↓
Index Scan часто выгоден

много подходящих строк
        ↓
Bitmap Scan или Seq Scan может быть выгоднее

все необходимые данные есть в индексе
        ↓
Index Only Scan может быть выгоднее
```

> **Index Scan — это не просто «использование индекса». PostgreSQL сначала использует индекс для поиска подходящих записей, а затем обычно обращается к heap-таблице за самими данными.**
