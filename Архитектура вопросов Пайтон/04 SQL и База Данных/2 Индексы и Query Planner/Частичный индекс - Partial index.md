# 🧩 Partial Index

## 🎤 Короткий ответ

**Partial Index** — это индекс, который содержит не все строки таблицы, а только строки, удовлетворяющие определённому условию `WHERE`.

Например:

```sql
CREATE INDEX idx_orders_active_user
ON orders(user_id)
WHERE status = 'active';
```

Такой индекс содержит только активные заказы.

Это позволяет:

* уменьшить размер индекса;
* уменьшить стоимость его обновления;
* ускорить запросы, которые работают с соответствующим подмножеством данных.

Но PostgreSQL сможет использовать partial index только тогда, когда он может доказать, что условие запроса соответствует предикату индекса.

---

## 🗣️ Ответ на собеседовании

Partial Index — это индекс не на всю таблицу, а только на строки, удовлетворяющие условию `WHERE`.

Например, если в `orders` миллионы заказов, но большинство запросов работают только с активными:

```sql
CREATE INDEX idx_orders_active_user
ON orders(user_id)
WHERE status = 'active';
```

Индекс будет содержать только активные заказы.

Это уменьшает размер индекса и количество записей, которые PostgreSQL должен поддерживать при изменениях данных.

При запросе:

```sql
SELECT *
FROM orders
WHERE status = 'active'
  AND user_id = 42;
```

PostgreSQL может использовать этот индекс.

Но важно, чтобы условие запроса логически гарантировало предикат индекса. Если запрос не содержит соответствующего условия, PostgreSQL не может просто использовать partial index для произвольных строк.

---

## 🧭 Где я нахожусь

```text
04 SQL и База Данных
└── 02 Индексы и Query Planner
    ├── Индексы
    │   ├── B-Tree
    │   ├── Composite Index
    │   ├── Covering Index
    │   └── Partial Index ← Я здесь
    ├── Selectivity
    ├── Query Planner
    │   ├── EXPLAIN
    │   ├── Scan types
    │   └── JOIN algorithms
    └── Статистика
```

---

# 📚 Разбор поглубже

## 1. Что такое обычный индекс

Допустим, есть таблица:

```text
orders
────────────────────
id
user_id
status
created_at
```

И создаём обычный индекс:

```sql
CREATE INDEX idx_orders_user_id
ON orders(user_id);
```

Упрощённо:

```text
orders
  ↓
все строки
  ↓
B-Tree index
```

Индекс содержит записи для всех строк, для которых индекс применим.

---

# 2. Что меняет Partial Index

Теперь:

```sql
CREATE INDEX idx_orders_active_user
ON orders(user_id)
WHERE status = 'active';
```

Получается:

```text
orders
│
├── status = active
│       ↓
│    INDEX
│
└── status != active
        ↓
    не попадает
```

То есть индекс представляет собой индекс **подмножества таблицы**.

---

# 3. Пример

Предположим:

```text
orders = 10 000 000 строк
```

Из них:

```text
active   → 100 000
completed → 8 000 000
cancelled → 1 900 000
```

Создаём:

```sql
CREATE INDEX idx_orders_active_user
ON orders(user_id)
WHERE status = 'active';
```

Индекс будет строиться только для:

```text
100 000 active orders
```

а не для всех:

```text
10 000 000 orders
```

Поэтому он потенциально значительно меньше полного индекса.

---

# 4. Как выглядит запрос

Индекс:

```sql
CREATE INDEX idx_orders_active_user
ON orders(user_id)
WHERE status = 'active';
```

Запрос:

```sql
SELECT *
FROM orders
WHERE status = 'active'
  AND user_id = 42;
```

Логика:

```text
Query
 ↓
status = active
 ↓
совпадает с predicate index
 ↓
idx_orders_active_user
 ↓
ищем user_id = 42
```

---

# 5. Почему PostgreSQL может использовать этот индекс

В индексе отсутствуют строки:

```text
status != 'active'
```

Поэтому PostgreSQL должен быть уверен, что запрос не требует таких строк.

В запросе:

```sql
WHERE status = 'active'
  AND user_id = 42
```

это очевидно.

А в запросе:

```sql
WHERE user_id = 42
```

условие `status = 'active'` отсутствует.

PostgreSQL не может использовать partial index как полноценный источник результата, потому что в таблице могут существовать:

```text
user_id = 42
status = completed
```

а их в индексе нет.

---

# 6. Predicate

Условие:

```sql
WHERE status = 'active'
```

при создании partial index называется **predicate** индекса.

Например:

```sql
CREATE INDEX idx_orders_active
ON orders(user_id)
WHERE status = 'active';
```

Здесь:

```text
index predicate
        ↓
status = 'active'
```

Это важный термин для собеседования.

---

# 7. Partial Index ≠ индекс по колонке WHERE

Важное различие.

В:

```sql
CREATE INDEX idx_orders_active
ON orders(user_id)
WHERE status = 'active';
```

`user_id` — **индексируемая колонка**.

```text
ON orders(user_id)
           ↑
       index key
```

А:

```sql
WHERE status = 'active'
```

— **predicate**, определяющий, какие строки попадут в индекс.

```text
WHERE status = 'active'
       ↑
    predicate
```

То есть:

```text
Partial Index
├── key       → user_id
└── predicate → status = 'active'
```

---

# 8. Partial Index может индексировать несколько колонок

Например:

```sql
CREATE INDEX idx_active_orders_user_date
ON orders(user_id, created_at)
WHERE status = 'active';
```

Получаем:

```text
key:
    user_id
    created_at

predicate:
    status = 'active'
```

Это уже комбинация **composite + partial index**.

---

# 9. Частый практический сценарий

Представим таблицу задач:

```text
tasks
────────────────────
id
user_id
status
created_at
```

Большинство задач:

```text
completed
```

Но приложение постоянно ищет только:

```text
pending
```

Можно создать:

```sql
CREATE INDEX idx_tasks_pending_user
ON tasks(user_id)
WHERE status = 'pending';
```

Запрос:

```sql
SELECT *
FROM tasks
WHERE user_id = 42
  AND status = 'pending';
```

Получает специализированный индекс для наиболее важного подмножества данных.

---

# 10. Почему Partial Index может быть меньше

Обычный индекс:

```text
10 000 000 rows
        ↓
10 000 000 index entries
```

Partial:

```text
10 000 000 rows
        ↓
100 000 matching rows
        ↓
100 000 index entries
```

Следовательно, потенциально:

* меньше дискового пространства;
* меньше страниц индекса;
* меньше I/O;
* быстрее обслуживание индекса.

Но конкретный выигрыш зависит от данных и структуры индекса.

---

# 11. Partial Index и INSERT

Допустим:

```sql
CREATE INDEX idx_active_orders
ON orders(user_id)
WHERE status = 'active';
```

Вставляем:

```sql
INSERT INTO orders (user_id, status)
VALUES (42, 'completed');
```

Строка не удовлетворяет predicate:

```text
status = 'active'
```

поэтому соответствующая запись в partial index не создаётся.

Если позже:

```sql
UPDATE orders
SET status = 'active'
WHERE id = 100;
```

строка начинает удовлетворять predicate и должна попасть в индекс.

Обратная ситуация также возможна:

```sql
UPDATE orders
SET status = 'completed'
WHERE id = 100;
```

Теперь строка перестаёт соответствовать predicate и должна быть удалена из индекса.

---

# 12. Partial Index и UPDATE

Это важный нюанс.

Partial index уменьшает работу по поддержанию индекса **для строк, которые не входят в его predicate**.

Но если строка изменяет состояние так, что она:

```text
не входит
   ↓
входит
```

или:

```text
входит
   ↓
не входит
```

PostgreSQL должен изменить индекс.

Например:

```text
pending
   ↓
completed
```

при индексе:

```sql
WHERE status = 'pending'
```

строка должна исчезнуть из partial index.

---

# 13. Partial Index и Selectivity

Partial Index особенно интересен, когда predicate выбирает относительно небольшую часть таблицы.

Например:

```text
10 000 000 rows
       ↓
status = active
       ↓
50 000 rows
```

Тогда partial index может быть существенно меньше.

Если же:

```text
10 000 000 rows
       ↓
status = active
       ↓
9 500 000 rows
```

выигрыш от partial index по размеру уже значительно меньше.

Поэтому здесь важна **selectivity** условия.

---

# 14. Partial Index и Query Planner

PostgreSQL сам решает, использовать ли индекс.

Проверяем:

```sql
EXPLAIN
SELECT *
FROM orders
WHERE status = 'active'
  AND user_id = 42;
```

Можно увидеть:

```text
Index Scan using idx_orders_active_user on orders
```

Если planner считает использование индекса выгодным.

---

# 15. EXPLAIN ANALYZE

Для фактической проверки:

```sql
EXPLAIN ANALYZE
SELECT *
FROM orders
WHERE status = 'active'
  AND user_id = 42;
```

Нужно смотреть:

```text
Index Scan
Index Cond
Filter
actual rows
actual time
```

и сравнивать с альтернативным планом.

---

# 16. Partial Index vs обычный индекс

| Характеристика       | Обычный индекс               | Partial Index                               |
| -------------------- | ---------------------------- | ------------------------------------------- |
| Строки               | Все применимые               | Только predicate                            |
| Размер               | Обычно больше                | Может быть значительно меньше               |
| Универсальность      | Выше                         | Ниже                                        |
| Обслуживание         | Для всех индексируемых строк | Только для входящих в predicate             |
| Специализация        | Нет                          | Да                                          |
| Требование к запросу | Нет специального predicate   | Запрос должен позволять доказать predicate  |
| Хороший сценарий     | Общие запросы                | Частое обращение к конкретному подмножеству |

---

# 17. Partial Index vs Composite Index

Это совершенно разные идеи.

### Composite Index

```sql
CREATE INDEX idx_orders_user_status
ON orders(user_id, status);
```

Индексирует комбинацию:

```text
user_id + status
```

### Partial Index

```sql
CREATE INDEX idx_orders_active_user
ON orders(user_id)
WHERE status = 'active';
```

Индексирует:

```text
user_id
```

но только для:

```text
status = active
```

Можно объединить:

```sql
CREATE INDEX idx_orders_active_user_date
ON orders(user_id, created_at)
WHERE status = 'active';
```

Здесь одновременно:

```text
Composite
    +
Partial
```

---

# 18. Очень полезный пример

Представим систему заказов.

```text
orders = 100 млн строк
```

Приложение часто выполняет:

```sql
SELECT *
FROM orders
WHERE user_id = 42
  AND status = 'pending';
```

Но `pending` составляет только:

```text
0.1% всех заказов
```

Можно создать:

```sql
CREATE INDEX idx_orders_pending_user
ON orders(user_id)
WHERE status = 'pending';
```

Получаем:

```text
100 млн orders
       ↓
  predicate
       ↓
pending
       ↓
100 тыс. rows
       ↓
small index
```

Для такого сценария partial index может дать большой выигрыш.

---

# 19. Partial Index для soft delete

Очень распространённый Backend-сценарий.

Таблица:

```text
users
────────────────
id
email
deleted_at
```

Удаление логическое:

```text
deleted_at IS NULL
```

Большинство запросов:

```sql
SELECT *
FROM users
WHERE email = 'user@example.com'
  AND deleted_at IS NULL;
```

Можно создать:

```sql
CREATE INDEX idx_users_active_email
ON users(email)
WHERE deleted_at IS NULL;
```

Индекс содержит только активные записи.

---

# 20. Partial UNIQUE Index

Partial index может быть не только обычным индексом.

Можно создать **частично уникальный индекс**:

```sql
CREATE UNIQUE INDEX idx_users_active_email
ON users(email)
WHERE deleted_at IS NULL;
```

Это означает:

> email должен быть уникальным среди активных пользователей.

Но один и тот же email может существовать у нескольких soft-deleted пользователей.

Например:

```text
email            deleted_at
────────────────────────────
a@example.com    NULL
a@example.com    2026-01-01
a@example.com    2026-05-01
```

Такой сценарий очень полезен для soft delete.

---

# 21. Ограничение Partial Index

Нельзя считать partial index универсальной заменой обычному индексу.

Например:

```sql
CREATE INDEX idx_active_orders
ON orders(user_id)
WHERE status = 'active';
```

Для запроса:

```sql
SELECT *
FROM orders
WHERE user_id = 42;
```

этот индекс не содержит все необходимые строки.

Поэтому нужен другой индекс или другой план.

---

# 22. Важный нюанс с параметризованными запросами

Predicate partial index должен быть доказуемым planner'ом.

Например:

```sql
WHERE status = 'active'
```

понятен непосредственно.

Но с параметризованным условием:

```sql
WHERE status = $1
```

planner не всегда может доказать, что условие соответствует:

```sql
WHERE status = 'active'
```

Особенно это важно для prepared statements и generic plans.

Поэтому partial index требует учитывать не только структуру индекса, но и то, **как именно формируется запрос**.

---

# 23. Типичные ошибки

### ❌ Ошибка 1

> Partial Index индексирует только колонку из WHERE.

Нет.

```sql
ON orders(user_id)
WHERE status = 'active'
```

Здесь:

```text
user_id → index key
status  → predicate
```

---

### ❌ Ошибка 2

> Partial Index всегда быстрее обычного.

Нет.

Он может быть выгоднее, если predicate существенно уменьшает индекс и запросы действительно соответствуют этому подмножеству.

---

### ❌ Ошибка 3

> Если есть Partial Index, PostgreSQL обязательно его использует.

Нет.

Решение принимает Query Planner.

---

### ❌ Ошибка 4

> Partial Index подходит для любого запроса по индексируемой колонке.

Нет.

Он содержит только строки, удовлетворяющие predicate.

---

### ❌ Ошибка 5

> Partial Index — это отдельный тип B-Tree.

Не совсем.

Partial Index — это характеристика **какие строки входят в индекс**.

При этом сам индекс может использовать определённый метод доступа, например B-tree.

---

# 24. Схема для запоминания

```text
Обычный индекс

Table
 │
 ├── row 1 ──→ Index
 ├── row 2 ──→ Index
 ├── row 3 ──→ Index
 └── row 4 ──→ Index


Partial Index

Table
 │
 ├── row 1 ──→ Index
 ├── row 2     ✕
 ├── row 3 ──→ Index
 └── row 4     ✕
       ↑
   predicate
```

---

# 25. Формула для собеседования

```text
Partial Index
=
индекс подмножества строк
+
WHERE predicate
+
меньше индекс
+
меньше обслуживание
+
нужен соответствующий запрос
```

Пример:

```sql
CREATE INDEX idx_orders_active_user
ON orders(user_id)
WHERE status = 'active';
```

Ответ одной фразой:

> Partial Index — это индекс только на строки, удовлетворяющие заданному predicate; он позволяет уменьшить размер и стоимость обслуживания индекса для запросов, работающих с конкретным подмножеством данных.

---

## 🎤 Вопросы на собеседовании

### Что такое Partial Index?

Индекс, содержащий только строки, удовлетворяющие условию `WHERE`.

### Зачем он нужен?

Чтобы не индексировать ненужное подмножество таблицы и получить меньший специализированный индекс.

### Что такое predicate Partial Index?

Условие `WHERE`, определяющее, какие строки входят в индекс.

### В чём разница?

```sql
CREATE INDEX idx
ON orders(user_id)
WHERE status = 'active';
```

```text
user_id
→ индексируемая колонка

status = 'active'
→ predicate
```

### Когда Partial Index особенно полезен?

Когда запросы регулярно работают с небольшим и хорошо определённым подмножеством таблицы: например, активными, необработанными или не удалёнными логически записями.

### Можно ли сделать Partial UNIQUE Index?

Да:

```sql
CREATE UNIQUE INDEX idx_users_active_email
ON users(email)
WHERE deleted_at IS NULL;
```

### Использует ли PostgreSQL Partial Index автоматически?

Не обязательно. Query Planner выбирает план на основании стоимости и возможности доказать, что запрос соответствует predicate.

### Чем Partial Index отличается от Composite Index?

Composite Index определяет несколько индексируемых колонок:

```sql
ON orders(user_id, created_at)
```

Partial Index ограничивает **набор строк**, например:

```sql
WHERE status = 'active'
```

Они могут использоваться вместе.

### Как проверить, используется ли Partial Index?

```sql
EXPLAIN
SELECT *
FROM orders
WHERE status = 'active'
  AND user_id = 42;
```

или:

```sql
EXPLAIN ANALYZE
SELECT *
FROM orders
WHERE status = 'active'
  AND user_id = 42;
```

### Может ли Partial Index быть B-tree?

Да. Partial — это не отдельный алгоритм индексирования, а условие отбора строк, входящих в индекс. Например, можно создать partial B-tree index.

### Главная мысль

```text
Не индексируем всю таблицу
          ↓
выбираем нужное подмножество
          ↓
WHERE predicate
          ↓
Partial Index
          ↓
меньше данных в индексе
          ↓
потенциально дешевле поиск и обслуживание
```
