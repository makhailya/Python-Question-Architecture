# 🧮 Expression Index

## 🎤 Короткий ответ

**Expression Index** — это индекс не по исходному значению колонки, а по **результату выражения или функции**, вычисляемого из одной или нескольких колонок.

Например:

```sql
CREATE INDEX idx_users_lower_email
ON users (LOWER(email));
```

Теперь PostgreSQL может эффективно выполнять запрос:

```sql
SELECT *
FROM users
WHERE LOWER(email) = 'user@example.com';
```

Вместо того чтобы каждый раз вычислять `LOWER(email)` для большого количества строк, PostgreSQL может использовать заранее проиндексированные результаты выражения.

Главная идея:

```text
обычный индекс
column → index

Expression Index
expression(column) → index
```

---

## 🗣️ Ответ на собеседовании

Expression Index — это индекс, построенный не непосредственно по значению колонки, а по результату выражения.

Например, если приложение ищет пользователей без учёта регистра:

```sql
WHERE LOWER(email) = 'user@example.com'
```

можно создать:

```sql
CREATE INDEX idx_users_lower_email
ON users (LOWER(email));
```

PostgreSQL хранит в индексе результаты `LOWER(email)` и может использовать их для поиска.

Это отличается от обычного индекса:

```sql
CREATE INDEX idx_users_email
ON users(email);
```

Потому что обычный индекс индексирует исходное значение `email`, а запрос использует результат функции `LOWER(email)`.

Expression Index особенно полезен для часто используемых вычислений и нормализации данных при поиске: например, `LOWER()`, арифметических выражений, функций над датами и других подходящих выражений.

---

## 🧭 Где я нахожусь

```text
04 SQL и База Данных
└── 02 Индексы и Query Planner
    ├── Индексы
    │   ├── B-Tree
    │   ├── Composite Index
    │   ├── Covering Index
    │   ├── Partial Index
    │   └── Expression Index ← Я здесь
    ├── Selectivity
    ├── Query Planner
    │   ├── EXPLAIN
    │   ├── Scan types
    │   └── JOIN algorithms
    └── Статистика
```

---

# 📚 Разбор поглубже

## 1. Обычный индекс

Допустим:

```sql
CREATE INDEX idx_users_email
ON users(email);
```

Индекс построен непосредственно по:

```text
email
```

Например:

```text
email
──────────────────────
alice@example.com
bob@example.com
john@example.com
```

Если запрос:

```sql
SELECT *
FROM users
WHERE email = 'bob@example.com';
```

PostgreSQL может использовать индекс.

---

# 2. Проблема с выражением

Теперь приложение хочет искать email без учёта регистра:

```sql
SELECT *
FROM users
WHERE LOWER(email) = 'bob@example.com';
```

Есть обычный индекс:

```sql
CREATE INDEX idx_users_email
ON users(email);
```

Но он построен по:

```text
email
```

а запрос работает с:

```text
LOWER(email)
```

Это не одно и то же.

Условно:

```text
обычный индекс:

email
  ↓
"Bob@Example.com"


запрос:

LOWER(email)
  ↓
"bob@example.com"
```

Поэтому обычный индекс не решает задачу так, как нужен planner'у.

---

# 3. Создаём Expression Index

Создаём индекс:

```sql
CREATE INDEX idx_users_lower_email
ON users (LOWER(email));
```

Теперь индекс построен по:

```text
LOWER(email)
```

Условно:

```text
email                  LOWER(email)
──────────────────     ──────────────────
Bob@Example.com   →    bob@example.com
ALICE@EXAMPLE.COM →    alice@example.com
John@Example.com  →    john@example.com
```

Запрос:

```sql
SELECT *
FROM users
WHERE LOWER(email) = 'bob@example.com';
```

может использовать этот индекс.

---

# 4. Главная идея

Обычный индекс:

```sql
CREATE INDEX idx
ON users(email);
```

```text
email → index
```

Expression Index:

```sql
CREATE INDEX idx
ON users(LOWER(email));
```

```text
LOWER(email) → index
```

Именно поэтому название **Expression Index**.

---

# 5. Пример с арифметическим выражением

Expression Index может использовать арифметику.

Допустим:

```text
products
────────────────────
price
tax
```

Можно создать:

```sql
CREATE INDEX idx_products_total_price
ON products (price * 1.2);
```

Теперь запрос:

```sql
SELECT *
FROM products
WHERE price * 1.2 > 1000;
```

может использовать этот индекс.

Здесь индекс строится по выражению:

```text
price * 1.2
```

а не просто по:

```text
price
```

---

# 6. Несколько колонок в выражении

Expression может использовать несколько колонок:

```sql
CREATE INDEX idx_full_name
ON users ((first_name || ' ' || last_name));
```

Запрос:

```sql
SELECT *
FROM users
WHERE first_name || ' ' || last_name = 'Ivan Ivanov';
```

Индекс построен по результату:

```text
first_name || ' ' || last_name
```

---

# 7. Функции

Один из наиболее распространённых вариантов:

```sql
CREATE INDEX idx_users_lower_email
ON users (LOWER(email));
```

Другие возможные примеры:

```sql
CREATE INDEX idx_users_lower_name
ON users (LOWER(name));
```

или выражение с датой:

```sql
CREATE INDEX idx_orders_created_date
ON orders ((created_at::date));
```

Здесь индекс строится по:

```text
created_at::date
```

---

# 8. Почему это ускоряет запрос

Без Expression Index PostgreSQL потенциально должен вычислить:

```text
LOWER(email)
```

для большого количества строк.

Условно:

```text
1 000 000 rows
       ↓
LOWER(email)
       ↓
сравнение
```

С Expression Index:

```text
1 000 000 rows
       ↓
Expression Index
       ↓
найти нужное значение
```

Само выражение всё равно должно быть поддержано при изменении данных, но при чтении PostgreSQL получает возможность использовать индекс для поиска.

---

# 9. Expression Index не означает «вычислить один раз навсегда»

Важно понимать механику.

Если:

```sql
CREATE INDEX idx_users_lower_email
ON users (LOWER(email));
```

и пользователь изменил:

```text
email
```

PostgreSQL должен обновить соответствующую запись индекса.

То есть при:

```sql
UPDATE users
SET email = 'New@Example.com'
WHERE id = 1;
```

результат:

```text
LOWER(email)
```

изменился.

Следовательно, индекс тоже должен быть обновлён.

---

# 10. Стоимость Expression Index

Expression Index ускоряет чтение, но не бесплатен.

При `INSERT` или изменении участвующих колонок PostgreSQL должен:

1. вычислить выражение;
2. обновить индекс.

Например:

```text
INSERT
  ↓
вычислить LOWER(email)
  ↓
добавить результат в index
```

Поэтому не стоит создавать Expression Index на каждое возможное выражение.

Нужен реальный запросный сценарий.

---

# 11. Expression Index и EXPLAIN

Проверить использование:

```sql
EXPLAIN
SELECT *
FROM users
WHERE LOWER(email) = 'bob@example.com';
```

Можно увидеть:

```text
Index Scan using idx_users_lower_email on users
```

Это означает, что planner выбрал созданный Expression Index.

Для фактического выполнения:

```sql
EXPLAIN ANALYZE
SELECT *
FROM users
WHERE LOWER(email) = 'bob@example.com';
```

---

# 12. Важное требование: выражение должно соответствовать запросу

Индекс:

```sql
CREATE INDEX idx_users_lower_email
ON users (LOWER(email));
```

Запрос:

```sql
WHERE LOWER(email) = 'bob@example.com'
```

логически соответствует индексу.

А запрос:

```sql
WHERE UPPER(email) = 'BOB@EXAMPLE.COM'
```

использует другое выражение:

```text
UPPER(email)
```

и созданный индекс `LOWER(email)` для этого выражения не является тем же самым индексом.

Поэтому при проектировании нужно смотреть на реальные формы запросов.

---

# 13. Expression Index vs обычный индекс

|                      | Обычный индекс              | Expression Index                                |
| -------------------- | --------------------------- | ----------------------------------------------- |
| Что индексируется    | Значение колонки            | Результат выражения                             |
| Пример               | `email`                     | `LOWER(email)`                                  |
| Запрос               | `email = ...`               | `LOWER(email) = ...`                            |
| Вычисление выражения | Нет                         | Да                                              |
| Стоимость записи     | Обычная                     | Есть вычисление выражения                       |
| Основной сценарий    | Поиск по исходному значению | Поиск по вычисленному/нормализованному значению |

---

# 14. Expression Index vs Partial Index

Это особенно важно после предыдущей темы.

### Expression Index

Определяет **что индексировать**:

```sql
CREATE INDEX idx_users_lower_email
ON users (LOWER(email));
```

```text
LOWER(email)
      ↓
   index
```

### Partial Index

Определяет **какие строки индексировать**:

```sql
CREATE INDEX idx_active_users
ON users(email)
WHERE deleted_at IS NULL;
```

```text
email
  ↑
только строки
deleted_at IS NULL
```

Их можно комбинировать.

Например:

```sql
CREATE INDEX idx_active_lower_email
ON users (LOWER(email))
WHERE deleted_at IS NULL;
```

Получаем одновременно:

```text
Expression
    +
Partial
```

То есть:

```text
что индексируем:
    LOWER(email)

какие строки:
    deleted_at IS NULL
```

---

# 15. Expression Index + UNIQUE

Expression Index можно сделать уникальным:

```sql
CREATE UNIQUE INDEX idx_users_unique_lower_email
ON users (LOWER(email));
```

Теперь PostgreSQL не позволит существовать одновременно:

```text
Alice@example.com
alice@example.com
```

потому что:

```text
LOWER("Alice@example.com")
    =
LOWER("alice@example.com")
```

Оба значения превращаются в:

```text
alice@example.com
```

и нарушают уникальность.

Это очень практический пример.

---

# 16. Expression Index и case-insensitive поиск

Один из классических сценариев:

```sql
CREATE UNIQUE INDEX idx_users_email_lower
ON users (LOWER(email));
```

Запрос:

```sql
SELECT *
FROM users
WHERE LOWER(email) = LOWER('Alice@Example.com');
```

Получаем поиск без учёта регистра.

Но на практике выбор между `LOWER()`, типом `citext` и другими вариантами зависит от требований приложения и семантики сравнения.

---

# 17. Expression Index и функции

Не любую произвольную функцию можно использовать без ограничений.

Для индексируемого выражения PostgreSQL должен иметь возможность корректно полагаться на результат функции.

Особенно важно понятие **IMMUTABLE**.

Упрощённо:

```text
IMMUTABLE
→ одинаковые аргументы
→ одинаковый результат
```

Например, функция должна быть предсказуемой с точки зрения индекса.

PostgreSQL поэтому ограничивает использование функций с неподходящей volatility.

Проверить свойства функций можно через системные каталоги или документацию PostgreSQL.

---

# 18. Почему `NOW()` — плохой пример

Нельзя просто построить индекс на выражении, зависящем от текущего времени:

```sql
CREATE INDEX idx_test
ON orders (NOW());
```

Проблема очевидна:

```text
NOW()
```

меняется со временем.

Индекс должен содержать значение, которое корректно поддерживать при изменениях данных.

Поэтому PostgreSQL не позволяет использовать произвольные volatile expressions в индексах.

---

# 19. Expression Index и SELECTIVITY

Как и для обычных индексов, полезность зависит от того, насколько хорошо выражение помогает отфильтровать данные.

Например:

```sql
CREATE INDEX idx_users_lower_email
ON users (LOWER(email));
```

Для уникальных email:

```text
LOWER(email)
    ↓
высокая селективность
```

индекс особенно естественен.

А индекс по выражению с очень небольшим количеством возможных значений может давать меньший выигрыш для некоторых запросов.

---

# 20. Expression Index и Query Planner

Общая цепочка:

```text
SQL Query
    ↓
Query Planner
    ↓
видит WHERE LOWER(email) = ...
    ↓
сравнивает доступные планы
    ↓
Expression Index?
    ↓
стоимость выгодна?
    ↓
Index Scan
```

Поэтому само наличие Expression Index не гарантирует его использования.

---

# 21. Типичная Backend-задача

Есть API:

```text
GET /users?email=Alice@Example.com
```

В базе:

```text
email
```

Хранимый регистр может отличаться:

```text
Alice@example.com
ALICE@EXAMPLE.COM
alice@example.com
```

Запрос:

```sql
SELECT *
FROM users
WHERE LOWER(email) = LOWER($1);
```

Создаём:

```sql
CREATE INDEX idx_users_lower_email
ON users (LOWER(email));
```

Теперь база имеет специализированный индекс под форму поиска API.

---

# 22. Типичные ошибки

### ❌ «Expression Index — это индекс на несколько колонок»

Нет.

Несколько колонок — это **Composite Index**.

Expression Index — индекс по выражению:

```sql
LOWER(email)
```

Хотя выражение действительно может использовать несколько колонок.

---

### ❌ «Обычный индекс автоматически оптимизирует `LOWER(column)`»

Не обязательно.

Индекс:

```sql
ON users(email)
```

и выражение:

```sql
LOWER(email)
```

— разные индексируемые значения.

---

### ❌ «Expression Index всегда быстрее»

Нет.

Он добавляет стоимость обслуживания при изменении данных и полезен только для соответствующих запросов.

---

### ❌ «PostgreSQL всегда использует Expression Index»

Нет.

Решение принимает Query Planner.

Проверять:

```sql
EXPLAIN
```

или:

```sql
EXPLAIN ANALYZE
```

---

# 23. Сравнение четырёх индексов

```text
Обычный:
email
  ↓
Index


Composite:
(email, created_at)
  ↓
Index


Partial:
email
  ↓
только WHERE deleted_at IS NULL
  ↓
Index


Expression:
LOWER(email)
  ↓
Index
```

Можно комбинировать характеристики:

```text
Expression
    +
Composite
    +
Partial
```

Например:

```sql
CREATE INDEX idx_active_user_name
ON users (LOWER(first_name), LOWER(last_name))
WHERE deleted_at IS NULL;
```

Здесь:

* expression index — `LOWER(...)`;
* composite index — две индексируемые части;
* partial index — только `deleted_at IS NULL`.

---

## 🎤 Вопросы на собеседовании

### Что такое Expression Index?

Индекс, построенный по результату выражения или функции, а не непосредственно по исходному значению колонки.

### Пример?

```sql
CREATE INDEX idx_users_lower_email
ON users (LOWER(email));
```

### Зачем он нужен?

Чтобы эффективно выполнять запросы, использующие то же выражение:

```sql
WHERE LOWER(email) = 'alice@example.com'
```

### Чем Expression Index отличается от обычного?

Обычный индекс:

```text
email → index
```

Expression:

```text
LOWER(email) → index
```

### Чем Expression Index отличается от Partial Index?

Expression Index определяет **значение, которое индексируется**.

Partial Index определяет **подмножество строк, которые попадут в индекс**.

### Можно ли объединить Expression и Partial Index?

Да:

```sql
CREATE INDEX idx_active_lower_email
ON users (LOWER(email))
WHERE deleted_at IS NULL;
```

### Можно ли сделать Expression Index уникальным?

Да:

```sql
CREATE UNIQUE INDEX idx_users_email
ON users (LOWER(email));
```

Это позволяет обеспечить уникальность с учётом нормализованного выражения.

### Всегда ли Expression Index используется?

Нет. PostgreSQL Query Planner сам выбирает план на основании стоимости.

### Как проверить использование?

```sql
EXPLAIN
SELECT *
FROM users
WHERE LOWER(email) = 'alice@example.com';
```

### Почему нельзя использовать любую функцию?

Потому что результат выражения должен иметь подходящие свойства для поддержания индекса. В частности, PostgreSQL ограничивает функции с неподходящей volatility; изменяющийся результат нельзя надёжно использовать как индексируемое значение.

### Что такое `IMMUTABLE`?

Функция с одинаковыми аргументами должна возвращать одинаковый результат независимо от контекста и времени. Это свойство важно для функций, используемых в индексируемых выражениях.

---

## 🎯 Формула для собеседования

```text
Expression Index
        ↓
индексируем не column,
а expression
        ↓
LOWER(email)
price * 1.2
created_at::date
        ↓
запрос использует то же выражение
        ↓
Query Planner
        ↓
может использовать Expression Index
```

Пример:

```sql
CREATE INDEX idx_users_lower_email
ON users (LOWER(email));
```

Ключевая фраза:

> **Expression Index — это индекс по результату выражения над данными, который позволяет ускорить запросы, использующие это выражение.**
