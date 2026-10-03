# 📦 Covering Index

## 🎤 Короткий ответ

**Covering Index** — это индекс, который содержит **все данные, необходимые для выполнения конкретного запроса**, поэтому PostgreSQL в некоторых случаях может получить результат непосредственно из индекса, не читая строки таблицы.

Например:

```sql
CREATE INDEX idx_users_email_name
ON users(email)
INCLUDE (name);
```

Запрос:

```sql
SELECT name
FROM users
WHERE email = 'alice@example.com';
```

Индекс содержит:

```text
email → поиск
name  → данные для SELECT
```

Поэтому PostgreSQL потенциально может выполнить **Index Only Scan** вместо обычного `Index Scan`.

Главная идея:

```text
Обычный Index Scan:

Index
  ↓
найти row
  ↓
Table
  ↓
получить данные


Covering Index:

Index
  ↓
найти row
  ↓
получить все нужные данные
```

---

## 🗣️ Ответ на собеседовании

Covering Index — это индекс, который содержит все колонки, необходимые конкретному запросу: как для поиска, так и для формирования результата.

В PostgreSQL часто используется `INCLUDE`:

```sql
CREATE INDEX idx_users_email
ON users(email)
INCLUDE (name);
```

Если запрос:

```sql
SELECT name
FROM users
WHERE email = 'alice@example.com';
```

то `email` используется для поиска, а `name` уже находится в индексе.

В подходящих условиях PostgreSQL может использовать **Index Only Scan**, то есть не обращаться к heap-страницам таблицы за самой строкой.

Но важный нюанс: наличие Covering Index не гарантирует Index Only Scan. PostgreSQL должен учитывать visibility map и стоимость плана. Если видимость строк нельзя проверить только по visibility map, обращения к таблице всё равно могут понадобиться.

---

## 🧭 Где я нахожусь

```text
04 SQL и База Данных
└── 02 Индексы и Query Planner
    ├── Индексы
    │   ├── B-Tree
    │   ├── Composite Index
    │   ├── Partial Index
    │   ├── Expression Index
    │   └── Covering Index ← Я здесь
    ├── Selectivity
    ├── Query Planner
    │   ├── EXPLAIN
    │   ├── Scan types
    │   │   ├── Index Scan
    │   │   └── Index Only Scan
    │   └── JOIN algorithms
    └── Статистика
```

---

# 📚 Разбор поглубже

## 1. Какую проблему решает Covering Index

Рассмотрим таблицу:

```text
users
────────────────────
id
email
name
age
created_at
```

Есть индекс:

```sql
CREATE INDEX idx_users_email
ON users(email);
```

Запрос:

```sql
SELECT name
FROM users
WHERE email = 'alice@example.com';
```

Индекс позволяет быстро найти строку по `email`.

Но `name` в обычном индексе отсутствует.

Поэтому PostgreSQL должен:

```text
Index
  ↓
найти нужную запись
  ↓
получить ссылку на heap
  ↓
обратиться к таблице
  ↓
прочитать name
```

Это называется **Index Scan**.

---

# 2. Что меняет Covering Index

Создаём:

```sql
CREATE INDEX idx_users_email_name
ON users(email)
INCLUDE (name);
```

Теперь индекс содержит:

```text
email
name
```

Условно:

```text
┌────────────────────────────┐
│ email          │ name      │
├────────────────────────────┤
│ alice@...      │ Alice     │
│ bob@...        │ Bob       │
│ john@...       │ John      │
└────────────────────────────┘
```

Запрос:

```sql
SELECT name
FROM users
WHERE email = 'alice@example.com';
```

может получить и условие поиска, и результат непосредственно из индекса.

---

# 3. Почему используется термин Covering

Индекс **покрывает запрос**, если в нём присутствуют все необходимые данные.

Например:

```sql
SELECT name
FROM users
WHERE email = 'alice@example.com';
```

Нужны:

```text
WHERE:
    email

SELECT:
    name
```

Индекс:

```sql
CREATE INDEX idx_users_email_name
ON users(email)
INCLUDE(name);
```

содержит оба значения.

```text
email → WHERE
name  → SELECT
```

Следовательно:

```text
Index covers query
```

---

# 4. `INCLUDE`

В PostgreSQL для covering index часто используется:

```sql
CREATE INDEX idx_users_email
ON users(email)
INCLUDE(name);
```

Здесь есть две разные категории:

```text
ON users(email)
       ↑
ключ индекса

INCLUDE(name)
       ↑
дополнительные данные
```

Это принципиально важно.

---

# 5. Key columns и included columns

Допустим:

```sql
CREATE INDEX idx_orders_user
ON orders(user_id)
INCLUDE(status, created_at);
```

Получаем:

```text
Index
├── key column
│   └── user_id
│
└── included columns
    ├── status
    └── created_at
```

### `user_id`

Используется непосредственно как ключ индекса:

```sql
WHERE user_id = 42
```

### `status`, `created_at`

Хранятся в индексе для покрытия запроса.

Например:

```sql
SELECT status, created_at
FROM orders
WHERE user_id = 42;
```

---

# 6. `INCLUDE` не то же самое, что составной индекс

Сравним.

### Composite Index

```sql
CREATE INDEX idx_orders
ON orders(user_id, created_at);
```

Здесь:

```text
user_id
created_at
```

— части **ключа индекса**.

Порядок колонок имеет значение для возможностей поиска и сортировки.

---

### Covering Index с INCLUDE

```sql
CREATE INDEX idx_orders
ON orders(user_id)
INCLUDE(created_at);
```

Здесь:

```text
user_id
→ index key

created_at
→ included payload
```

`created_at` находится в индексе, но не является частью его ключа.

---

# 7. Пример с API

Представим Backend endpoint:

```text
GET /users/{email}
```

Запрос:

```sql
SELECT id, name, created_at
FROM users
WHERE email = $1;
```

Можно создать:

```sql
CREATE INDEX idx_users_email_covering
ON users(email)
INCLUDE(id, name, created_at);
```

Теперь индекс содержит всё необходимое:

```text
email       → WHERE
id          → SELECT
name        → SELECT
created_at  → SELECT
```

В подходящих условиях PostgreSQL может выполнить:

```text
Index Only Scan
```

и не читать heap для получения самих значений.

---

# 8. Index Scan vs Index Only Scan

Это одна из самых важных частей темы.

## Index Scan

```text
Index
  ↓
найти нужную запись
  ↓
Heap/Table
  ↓
получить данные
```

## Index Only Scan

```text
Index
  ↓
найти нужную запись
  ↓
получить данные из Index
```

Упрощённо:

```text
Index Scan
= Index + Table

Index Only Scan
= Index
```

Но есть важный нюанс PostgreSQL: даже при `Index Only Scan` может потребоваться обращение к heap для проверки видимости некоторых строк.

---

# 9. Почему Index Only Scan не гарантирован

Допустим:

```sql
CREATE INDEX idx_users_email
ON users(email)
INCLUDE(name);
```

Есть запрос:

```sql
SELECT name
FROM users
WHERE email = 'alice@example.com';
```

Логически индекс покрывает запрос.

Но PostgreSQL использует **MVCC**.

Индексная запись сама по себе не содержит всей необходимой информации о том, видна ли строка текущей транзакции.

Для этого PostgreSQL использует **visibility map**.

Упрощённо:

```text
Index
 ↓
нашли tuple
 ↓
visibility map
 ↓
строка точно видима?
 ├── да → можно не читать heap
 └── нет → нужно обратиться к heap
```

Поэтому:

> Covering Index создаёт возможность для Index Only Scan, но не гарантирует его.

---

# 10. Visibility Map

PostgreSQL хранит информацию о видимости страниц таблицы в **visibility map**.

Если PostgreSQL знает, что все tuples на странице видимы для всех текущих транзакций, ему не нужно идти в heap для проверки каждой строки.

Тогда:

```text
Index
  ↓
Visibility Map
  ↓
страница all-visible
  ↓
данные можно получить без heap
```

Если страница не помечена как all-visible:

```text
Index
  ↓
Visibility Map
  ↓
не all-visible
  ↓
Heap
```

---

# 11. Почему VACUUM важен

`VACUUM` помогает поддерживать visibility map.

Поэтому на хорошо обслуживаемой таблице PostgreSQL имеет больше возможностей эффективно выполнять Index Only Scan.

Но:

> `VACUUM` не превращает любой запрос автоматически в Index Only Scan.

Planner всё равно выбирает план на основании стоимости и других факторов.

---

# 12. EXPLAIN

Проверяем план:

```sql
EXPLAIN
SELECT name
FROM users
WHERE email = 'alice@example.com';
```

Если PostgreSQL использует covering index, можно увидеть:

```text
Index Only Scan using idx_users_email_name on users
```

Для фактических показателей:

```sql
EXPLAIN ANALYZE
SELECT name
FROM users
WHERE email = 'alice@example.com';
```

Особенно интересно смотреть:

```text
Heap Fetches
```

Например:

```text
Index Only Scan using idx_users_email_name on users
Heap Fetches: 0
```

Это означает, что для этих строк PostgreSQL не потребовалось обращаться к heap для проверки видимости.

---

# 13. `Heap Fetches`

Например:

```text
Index Only Scan using idx_users_email_name on users
Heap Fetches: 0
```

Хороший признак для index-only сценария.

А:

```text
Heap Fetches: 10000
```

означает, что PostgreSQL всё равно обращался к heap для проверки видимости соответствующих tuples.

Поэтому при анализе Covering Index полезно смотреть не только на:

```text
Index Only Scan
```

но и на:

```text
Heap Fetches
```

---

# 14. Covering Index не всегда уменьшает размер

Важно не думать:

> `INCLUDE` бесплатный.

Если добавить:

```sql
INCLUDE(name, email, phone, address, created_at)
```

индекс станет значительно больше.

Получаем:

```text
меньше обращений к table
        ↓
но
        ↓
больше index
        ↓
больше disk
        ↓
больше I/O при поддержании
        ↓
дороже INSERT/UPDATE
```

Поэтому covering index нужно проектировать под конкретные запросы.

---

# 15. UPDATE и стоимость Covering Index

Допустим:

```sql
CREATE INDEX idx_users_email
ON users(email)
INCLUDE(name);
```

Если меняется:

```sql
UPDATE users
SET name = 'Bob'
WHERE id = 1;
```

изменяется included column.

Следовательно, индекс также может потребовать обновления.

То есть `INCLUDE` уменьшает обращения к таблице при чтении, но может увеличивать стоимость записи.

---

# 16. Когда Covering Index особенно полезен

Типичный сценарий:

```text
частые SELECT
+
небольшой набор возвращаемых колонок
+
стабильные данные
```

Например:

```sql
SELECT id, name
FROM users
WHERE email = $1;
```

Индекс:

```sql
CREATE INDEX idx_users_email
ON users(email)
INCLUDE(id, name);
```

---

# 17. Covering Index для часто выполняемых запросов

Представим API:

```text
GET /orders?user_id=42
```

Запрос:

```sql
SELECT id, status, created_at
FROM orders
WHERE user_id = 42
ORDER BY created_at DESC;
```

Можно рассмотреть:

```sql
CREATE INDEX idx_orders_user_created
ON orders(user_id, created_at DESC)
INCLUDE(status, id);
```

Здесь:

```text
user_id
    ↓
фильтрация

created_at
    ↓
сортировка

status, id
    ↓
данные для SELECT
```

Это пример индекса, одновременно учитывающего:

* filtering;
* ordering;
* covering.

---

# 18. Covering Index и ORDER BY

Очень важный нюанс.

Если колонка является **key column**, она может участвовать в порядке индекса:

```sql
CREATE INDEX idx_orders
ON orders(user_id, created_at DESC)
INCLUDE(status);
```

Тогда индекс может помочь:

```sql
SELECT status, created_at
FROM orders
WHERE user_id = 42
ORDER BY created_at DESC;
```

Но если сделать:

```sql
CREATE INDEX idx_orders
ON orders(user_id)
INCLUDE(created_at, status);
```

`created_at` будет доступен в индексе как included column, но не становится частью порядка ключей индекса.

То есть `INCLUDE` — не замена key columns для условий поиска и сортировки.

---

# 19. Covering Index vs Composite Index

### Composite Index

```sql
CREATE INDEX idx_orders
ON orders(user_id, created_at);
```

Обе колонки являются key columns.

Используются для структуры поиска и порядка индекса.

---

### Covering Index

```sql
CREATE INDEX idx_orders
ON orders(user_id)
INCLUDE(status, created_at);
```

`user_id` — key.

```text
status
created_at
```

— payload.

---

### Комбинация

Можно сделать:

```sql
CREATE INDEX idx_orders
ON orders(user_id, created_at)
INCLUDE(status);
```

Получаем:

```text
Key:
    user_id
    created_at

Included:
    status
```

---

# 20. Covering Index vs обычный Index

|                 | Обычный Index                | Covering Index                      |
| --------------- | ---------------------------- | ----------------------------------- |
| Индексирует     | Колонки для поиска           | Колонки для поиска + данные запроса |
| `INCLUDE`       | Нет                          | Может использоваться                |
| Index Only Scan | Возможен в некоторых случаях | Основной сценарий                   |
| Размер          | Обычно меньше                | Может быть больше                   |
| Запись          | Дешевле                      | Может быть дороже                   |
| Чтение          | Иногда нужен heap            | Может обойтись без heap             |

---

# 21. Covering Index vs Partial Index

Они решают разные задачи.

### Partial Index

Уменьшает **набор строк**:

```sql
CREATE INDEX idx_active_users
ON users(email)
WHERE deleted_at IS NULL;
```

```text
какие строки?
        ↓
только active
```

### Covering Index

Расширяет **данные внутри индекса**:

```sql
CREATE INDEX idx_users_email
ON users(email)
INCLUDE(name);
```

```text
какие данные?
        ↓
email + name
```

Их можно объединить:

```sql
CREATE INDEX idx_active_users
ON users(email)
INCLUDE(name)
WHERE deleted_at IS NULL;
```

Получаем:

```text
Partial
+
Covering
```

---

# 22. Covering Index vs Expression Index

Expression:

```sql
CREATE INDEX idx_users_lower_email
ON users(LOWER(email));
```

Определяет **что вычислять и индексировать**.

Covering:

```sql
CREATE INDEX idx_users_email
ON users(email)
INCLUDE(name);
```

Определяет **какие дополнительные данные хранить в индексе**.

Их также можно объединить:

```sql
CREATE INDEX idx_users_lower_email
ON users(LOWER(email))
INCLUDE(name);
```

---

# 23. Полная комбинация

PostgreSQL позволяет комбинировать идеи:

```sql
CREATE INDEX idx_active_lower_email
ON users(LOWER(email))
INCLUDE(name, created_at)
WHERE deleted_at IS NULL;
```

Здесь:

```text
Expression:
    LOWER(email)

Covering:
    name
    created_at

Partial:
    deleted_at IS NULL
```

Это мощный инструмент, но такой индекс нужно создавать только под обоснованный workload: он увеличивает стоимость записи и занимает место.

---

# 24. Типичная Backend-задача

Есть таблица:

```text
orders
────────────────────────
id
user_id
status
created_at
total
```

API постоянно выполняет:

```sql
SELECT id, status, total
FROM orders
WHERE user_id = 42
ORDER BY created_at DESC;
```

Можно рассмотреть:

```sql
CREATE INDEX idx_orders_user_created
ON orders(user_id, created_at DESC)
INCLUDE(id, status, total);
```

Логика:

```text
user_id
   ↓
WHERE

created_at DESC
   ↓
ORDER BY

id, status, total
   ↓
SELECT
```

При подходящем плане PostgreSQL потенциально сможет получить результат преимущественно из индекса.

---

# 25. Главная ловушка

Не нужно говорить:

> Covering Index всегда позволяет не обращаться к таблице.

Точнее:

> Covering Index содержит все данные, необходимые запросу, поэтому PostgreSQL получает возможность использовать Index Only Scan. Но фактическая реализация зависит от visibility map и выбранного planner'ом плана.

Это хороший ответ уровня Middle.

---

# 26. Формула для собеседования

```text
Covering Index
        ↓
индекс содержит всё нужное запросу
        ↓
key columns
    +
INCLUDE columns
        ↓
может быть Index Only Scan
        ↓
меньше обращений к heap
```

Пример:

```sql
CREATE INDEX idx_users_email_name
ON users(email)
INCLUDE(name);
```

Запрос:

```sql
SELECT name
FROM users
WHERE email = 'alice@example.com';
```

Цепочка:

```text
email
 ↓
поиск в index

name
 ↓
уже есть в index

visibility map
 ↓
можно ли избежать heap?

 ↓

Index Only Scan
```

---

## 🎤 Вопросы на собеседовании

### Что такое Covering Index?

Индекс, содержащий все данные, необходимые конкретному запросу, что позволяет PostgreSQL потенциально выполнить его через Index Only Scan без чтения heap для получения самих значений.

### Как создать Covering Index в PostgreSQL?

```sql
CREATE INDEX idx_users_email
ON users(email)
INCLUDE(name);
```

### Что такое `INCLUDE`?

`INCLUDE` добавляет дополнительные колонки в индекс как payload. Они не становятся частью ключа индекса.

### Чем `INCLUDE` отличается от обычных key columns?

Key columns участвуют в структуре ключа и могут использоваться для поиска и порядка индекса.

`INCLUDE`-колонки хранятся для покрытия запроса, но не являются частью ключа.

### Что такое Index Only Scan?

Тип сканирования, при котором PostgreSQL получает необходимые данные из индекса и может избежать чтения heap, когда visibility map позволяет не проверять соответствующие строки в таблице.

### Гарантирует ли Covering Index Index Only Scan?

Нет.

На выбор влияет Query Planner, а для полного обхода heap важна информация visibility map.

### Что такое `Heap Fetches`?

Количество обращений к heap, которые PostgreSQL всё же выполнил при `Index Only Scan` для проверки видимости tuples.

### Зачем нужен `VACUUM`?

В частности, для поддержания информации visibility map, что может повысить эффективность Index Only Scan.

### Может ли Covering Index быть больше обычного?

Да. `INCLUDE` увеличивает размер индекса и стоимость его поддержки.

### Увеличивает ли Covering Index стоимость `INSERT` и `UPDATE`?

Да. Дополнительные данные должны записываться и поддерживаться в индексе.

### Может ли `INCLUDE` помочь с `ORDER BY`?

Сами `INCLUDE`-колонки не являются частью ключевого порядка индекса. Для поиска и сортировки соответствующую колонку нужно делать key column.

Например:

```sql
CREATE INDEX idx_orders
ON orders(user_id, created_at DESC)
INCLUDE(status);
```

Здесь `created_at` может участвовать в порядке, а `status` — только покрывает запрос.

### Чем Covering Index отличается от Partial Index?

```text
Partial Index
→ уменьшает количество строк в индексе

Covering Index
→ добавляет в индекс данные, необходимые запросу
```

### Чем Covering Index отличается от Composite Index?

```text
Composite:
(user_id, created_at)
→ обе колонки являются ключами

Covering:
(user_id)
INCLUDE(created_at)
→ user_id — ключ
→ created_at — дополнительное значение
```

---

## 🎯 Главное

```text
Обычный Index Scan

Index
  ↓
найти row
  ↓
Heap
  ↓
получить данные


Covering Index

Index
  ↓
найти row
  ↓
данные уже в index
  ↓
Index Only Scan
```

**Ключевая фраза для собеседования:**

> **Covering Index — это индекс, который содержит все данные, необходимые запросу, благодаря чему PostgreSQL может использовать Index Only Scan и в подходящих условиях не обращаться к heap.**
