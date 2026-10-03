# 📦 Selection Load

## 🎯 Ответ на собеседовании

**Selection Load** — это стратегия eager loading в ORM, при которой связанные данные загружаются **отдельным SQL-запросом**, но сразу для всех полученных объектов.

Вместо N отдельных запросов ORM делает один запрос с `WHERE ... IN (...)`.

Таким образом, вместо **N+1** запросов обычно получается **2 запроса**.

---

## 🎤 Суперкоротко

```text
N+1:

SELECT users;

SELECT orders WHERE user_id = 1;
SELECT orders WHERE user_id = 2;
SELECT orders WHERE user_id = 3;
...

→ 1 + N запросов
```

Selection Load:

```text
SELECT users;

SELECT orders
WHERE user_id IN (1, 2, 3, ...);

→ 2 запроса
```

**Главная идея:**

> Не JOIN'ить связанные данные, а загрузить их отдельным запросом сразу для всех объектов.

---

# 1. 🐌 Проблема N+1

Допустим, получили список пользователей:

```python
SELECT *
FROM users;
```

Получили:

```text
User 1
User 2
User 3
User 4
```

Теперь нам нужны их заказы.

При обычном lazy loading ORM может выполнить:

```python
SELECT *
FROM orders
WHERE user_id = 1;

SELECT *
FROM orders
WHERE user_id = 2;

SELECT *
FROM orders
WHERE user_id = 3;

SELECT *
FROM orders
WHERE user_id = 4;
```

Итого:

```text
1 + 4 = 5 запросов
```

При N пользователях:

```text
1 + N
```

---

# 2. 📦 Как работает Selection Load

Сначала ORM получает пользователей:

```python
SELECT *
FROM users;
```

Допустим, их ID:

```text
1, 2, 3, 4
```

Затем ORM собирает эти ID и делает **один дополнительный запрос**:

```python
SELECT *
FROM orders
WHERE user_id IN (1, 2, 3, 4);
```

Получаем все необходимые заказы сразу.

После этого ORM связывает их с соответствующими пользователями в памяти.

```text
Database
   │
   ├── SELECT users
   │
   └── SELECT orders
       WHERE user_id IN (...)
             │
             ▼
            ORM
             │
             ▼
       User → Orders
```

---

# 3. 🎯 Почему это решает N+1

Вместо:

```text
1 + N
```

получаем:

```text
2
```

Например:

```text
100 пользователей

N+1:
101 SQL-запрос

Selection Load:
2 SQL-запроса
```

Количество запросов перестаёт расти линейно вместе с количеством пользователей.

---

# 4. 🔗 Selection Load без JOIN

Ключевая особенность — **связанные данные не объединяются с основной таблицей через JOIN**.

Есть два отдельных запроса:

```python
SELECT *
FROM users;
```

и:

```python
SELECT *
FROM orders
WHERE user_id IN (...);
```

В отличие от JOIN Load:

```python
SELECT *
FROM users
LEFT JOIN orders
    ON orders.user_id = users.id;
```

---

# 5. ⚖️ Главное преимущество перед JOIN Load

Selection Load не создаёт размножение строк основной выборки из-за `one-to-many`.

Например:

```text
1 User
100 Orders
```

При JOIN:

```text
User
 ↓
100 строк результата
```

При Selection Load:

```text
SELECT users;
→ 1 строка

SELECT orders WHERE user_id IN (...);
→ 100 строк
```

ORM затем связывает их:

```text
User
 ├── Order 1
 ├── Order 2
 ├── ...
 └── Order 100
```

То есть связанные коллекции не перемножаются внутри одного JOIN-результата.

---

# 6. 💥 Особенно полезен для One-to-Many

Например:

```text
User
 ↓
Orders
```

или:

```text
Post
 ↓
Comments
```

или:

```text
Category
 ↓
Products
```

Если у каждого родительского объекта может быть много дочерних объектов, Selection Load часто является хорошим вариантом.

---

# 7. 📊 JOIN Load vs Selection Load

|                     | JOIN Load             | Selection Load       |
| ------------------- | --------------------- | -------------------- |
| SQL-запросы         | Обычно 1              | Обычно 2             |
| `JOIN`              | ✅                     | ❌                    |
| N+1                 | Решает                | Решает               |
| `WHERE IN`          | ❌                     | ✅                    |
| Row explosion       | Возможен              | Нет из-за JOIN       |
| One-to-many         | ⚠️ Может быть тяжёлым | ✅ Часто удобно       |
| Many-to-one         | ✅ Хорошо              | ✅ Можно              |
| Данные загружаются  | Одной выборкой        | Отдельными выборками |
| Связывание объектов | ORM                   | ORM                  |

---

# 8. 🧠 Пример

Есть:

```text
Users:

1 — Иван
2 — Пётр
3 — Сергей
```

И:

```text
Orders:

101 — user_id=1
102 — user_id=1
103 — user_id=2
104 — user_id=3
105 — user_id=3
```

Selection Load:

### Запрос №1

```python
SELECT *
FROM users;
```

### Запрос №2

```python
SELECT *
FROM orders
WHERE user_id IN (1, 2, 3);
```

ORM получает:

```text
Иван
 ├── Order 101
 └── Order 102

Пётр
 └── Order 103

Сергей
 ├── Order 104
 └── Order 105
```

---

# 9. ⚠️ Недостатки Selection Load

Selection Load не означает:

> «Всегда быстрее JOIN Load».

У него тоже есть особенности.

### 1. Дополнительный SQL-запрос

Вместо одного JOIN-запроса выполняются два:

```text
SELECT users
SELECT orders WHERE user_id IN (...)
```

### 2. Большой `IN`

Если исходных объектов очень много, список ID может стать большим.

Например:

```python
SELECT *
FROM orders
WHERE user_id IN (1, 2, 3, ... 100000);
```

ORM обычно решает это батчингом или другими механизмами, но размер выборки всё равно нужно учитывать.

### 3. Данные нужно связать в памяти

ORM должна сопоставить:

```text
order.user_id
```

с:

```text
user.id
```

Это требует дополнительной работы на стороне приложения.

---

# 10. 🧩 Selection Load и Batch Loading

На практике Selection Load часто реализуется через **batch loading**.

Вместо:

```text
SELECT orders WHERE user_id = 1
SELECT orders WHERE user_id = 2
SELECT orders WHERE user_id = 3
...
```

ORM собирает ID:

```text
1, 2, 3, 4, 5
```

и выполняет:

```python
SELECT *
FROM orders
WHERE user_id IN (1, 2, 3, 4, 5);
```

Если объектов очень много, список может быть разбит на несколько batch'ей.

---

# 11. 🔄 Сравнение с Lazy Loading

### Lazy Loading

```text
SELECT users
      ↓
SELECT orders WHERE user_id = 1
SELECT orders WHERE user_id = 2
SELECT orders WHERE user_id = 3
...
```

```text
N+1
```

### Selection Load

```text
SELECT users
      ↓
SELECT orders WHERE user_id IN (...)
```

```text
2 запроса
```

### JOIN Load

```text
SELECT users
JOIN orders
```

```text
1 запрос
```

---

# 12. 🎯 Когда использовать Selection Load

Особенно полезен, когда:

* есть `one-to-many`;
* связанных объектов много;
* JOIN создаёт большое количество строк;
* нужно избежать N+1;
* связанные данные можно получить отдельной выборкой;
* нет необходимости объединять всё в один SQL-результат.

Типичный сценарий:

```text
User → Orders
Post → Comments
Author → Posts
Category → Products
```

---

# 13. ⚠️ Важный момент

Selection Load — это **не просто ручной `SELECT ... IN`**.

В контексте ORM это стратегия загрузки связанных объектов, при которой ORM сама:

```text
1. получает основные объекты
2. собирает их ID
3. выполняет дополнительный SELECT
4. получает связанные объекты
5. сопоставляет их с родительскими объектами
```

Для разработчика результат выглядит как обычные связанные ORM-объекты.

---

# 14. 🧠 Схема

```text
              N+1
               │
               ▼
        Lazy Loading
               │
        1 + N запросов
               │
        ┌──────┴──────┐
        ▼             ▼
   JOIN Load     Selection Load
        │             │
        ▼             ▼
      JOIN        SELECT ... IN
        │             │
        ▼             ▼
    1 запрос       2 запроса
        │             │
        ▼             ▼
 row explosion    нет JOIN explosion
```

---

## 🎯 Главное

```text
Selection Load
│
├── eager loading
│
├── не использует JOIN
│
├── сначала получает основные объекты
│
├── затем делает дополнительный SELECT
│   WHERE related_id IN (...)
│
├── решает N+1
│
└── особенно полезен для one-to-many
```

### Формула для собеседования

> **Selection Load — стратегия eager loading, при которой ORM сначала загружает основные объекты, а затем одним или несколькими batch-запросами загружает связанные данные через `WHERE IN`. В отличие от JOIN Load, он не размножает строки из-за JOIN, поэтому часто удобен для связей one-to-many.**

**Ключевая мысль:**

```text
JOIN Load
→ 1 запрос
→ JOIN
→ возможен row explosion

Selection Load
→ 2+ запроса
→ SELECT ... IN (...)
→ без JOIN-мультипликации
```
