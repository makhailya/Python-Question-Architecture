# 🔗 JOIN Load

## 🎯 Ответ на собеседовании

**JOIN Load** — это способ eager loading, при котором ORM загружает основную сущность и связанные данные **одним SQL-запросом через `JOIN`**.

Он помогает решить **N+1 проблему**, но при связи **«один ко многим»** может привести к **размножению строк (row explosion)**.

---

## 🎤 Суперкоротко

```text
N+1:

SELECT users;
SELECT orders WHERE user_id = 1;
SELECT orders WHERE user_id = 2;
SELECT orders WHERE user_id = 3;
...

JOIN Load:

SELECT users
JOIN orders
ON orders.user_id = users.id;

→ один SQL-запрос
```

---

## 1. 🐌 Как возникает N+1

Допустим, нужно получить пользователей и их заказы.

Сначала ORM получает пользователей:

```python
SELECT *
FROM users;
```

Получили:

```text
User 1
User 2
User 3
...
User N
```

При ленивой загрузке ORM затем делает отдельный запрос для каждого пользователя:

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
```

В итоге:

```text
1 + N запросов
```

Это **N+1 проблема**.

---

# 2. 🔗 Как работает JOIN Load

Вместо отдельных запросов ORM сразу объединяет данные:

```python
SELECT
    users.id,
    users.name,
    orders.id,
    orders.amount
FROM users
LEFT JOIN orders
    ON orders.user_id = users.id;
```

Теперь БД сама выполняет `JOIN` и возвращает результат одним запросом.

```text
ORM
 │
 │  один SQL-запрос
 ▼
Database
 │
 ├── users
 └── orders
```

После получения результата ORM собирает строки обратно в объекты:

```text
User
 ├── Order
 ├── Order
 └── Order
```

---

# 3. ⚠️ Главная проблема JOIN Load

При связи **one-to-many** одна строка основной таблицы может соответствовать нескольким строкам связанной таблицы.

Например:

```text
users

id | name
---+------
1  | Ivan
```

```text
orders

id | user_id | amount
---+---------+-------
10 | 1       | 100
11 | 1       | 200
12 | 1       | 300
```

После `JOIN`:

```text
user_id | name | order_id | amount
--------+------+----------+-------
1       | Ivan | 10       | 100
1       | Ivan | 11       | 200
1       | Ivan | 12       | 300
```

Один пользователь появился **три раза**.

ORM затем должна собрать это обратно:

```text
Ivan
 ├── Order 10
 ├── Order 11
 └── Order 12
```

---

# 4. 💥 Row Explosion

Если связанных записей много, результат JOIN может сильно увеличиться.

Например:

```text
100 пользователей
×
100 заказов
```

Потенциально:

```text
10 000 строк результата
```

А если одновременно присоединить ещё одну `one-to-many` связь:

```text
User
 ├── Orders
 └── Comments
```

Допустим:

```text
1 пользователь
3 заказа
4 комментария
```

JOIN может дать:

```text
3 × 4 = 12 строк
```

Хотя реальных объектов:

```text
1 User
3 Orders
4 Comments
```

Это называют **row explosion** или **мультипликацией строк**.

---

# 5. ✅ Когда JOIN Load подходит хорошо

JOIN Load особенно хорошо подходит для связей:

### `many-to-one`

```text
Order → User
```

У каждого заказа один пользователь.

```python
SELECT orders.*, users.*
FROM orders
JOIN users
    ON users.id = orders.user_id;
```

Здесь JOIN обычно не создаёт сильного размножения строк.

---

### `one-to-one`

Например:

```text
User → UserProfile
```

Один пользователь имеет один профиль.

JOIN также хорошо подходит:

```python
SELECT users.*, profiles.*
FROM users
JOIN profiles
    ON profiles.user_id = users.id;
```

---

# 6. ⚠️ Когда нужно быть осторожным

Осторожнее с:

```text
one-to-many
many-to-many
```

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
Tags
```

Если связанных объектов много, JOIN может вернуть огромное количество строк.

Особенно опасно делать несколько `one-to-many` JOIN одновременно:

```python
SELECT *
FROM users
LEFT JOIN orders
    ON orders.user_id = users.id
LEFT JOIN comments
    ON comments.user_id = users.id;
```

Здесь связанные коллекции могут перемножаться.

---

# 7. 📊 JOIN Load и количество запросов

Главное преимущество:

```text
N+1:

1 + N запросов
```

JOIN Load:

```text
1 запрос
```

Но:

```text
1 запрос ≠ автоматически быстрее
```

Один огромный JOIN может оказаться тяжелее двух небольших запросов.

Поэтому при оптимизации смотрят не только на **количество SQL-запросов**, но и на:

* количество возвращаемых строк;
* размер результата;
* количество JOIN;
* кардинальность связей;
* индексы;
* план выполнения;
* нагрузку на БД;
* сетевой трафик.

---

# 8. 🧠 JOIN Load vs обычный JOIN

Важно не путать понятия.

**SQL JOIN** — операция реляционной БД:

```python
SELECT *
FROM users
JOIN orders
    ON orders.user_id = users.id;
```

**JOIN Load** — стратегия ORM, которая использует такой `JOIN`, чтобы **загрузить связанные объекты заранее**.

То есть:

```text
SQL JOIN
   ↓
механизм БД

JOIN Load
   ↓
стратегия загрузки связанных объектов в ORM
```

---

# 9. 🔄 JOIN Load и Lazy Loading

### Lazy Loading

Связанные данные загружаются только при обращении:

```text
SELECT users

↓ обращаемся к orders

SELECT orders WHERE user_id = 1

↓ следующий user

SELECT orders WHERE user_id = 2

...
```

Получаем:

```text
N+1
```

### JOIN Load

Связанные данные загружаются сразу:

```text
SELECT users
JOIN orders
```

Получаем:

```text
1 запрос
```

---

# 10. 🎯 Главное

```text
JOIN Load
│
├── eager loading
│
├── загружает связанные данные через JOIN
│
├── помогает устранить N+1
│
├── обычно один SQL-запрос
│
├── хорошо подходит для one-to-one
│   и many-to-one
│
└── при one-to-many может возникнуть
    row explosion
```

## Формула для собеседования

> **JOIN Load — это стратегия eager loading в ORM, при которой связанные данные загружаются одним SQL-запросом через JOIN. Она устраняет N+1, но при one-to-many может привести к размножению строк и большому объёму результата.**

**Ключевая мысль:**

```text
JOIN Load
→ меньше SQL-запросов
→ но потенциально больше строк в результате
```
