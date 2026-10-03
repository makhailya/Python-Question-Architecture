# 🔐 Unique Constraint

## 🎯 Ответ на собеседовании

**Unique Constraint** — ограничение базы данных, которое гарантирует, что значение или комбинация значений не повторяется в таблице.

Например:

```python
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE
);
```

База данных не позволит создать двух пользователей с одинаковым `email`.

В контексте **идемпотентности и дедупликации** `UNIQUE` часто используется для гарантии, что одна операция или событие не будут зарегистрированы дважды.

> **UNIQUE — это гарантия уникальности на уровне БД, а не проверка в коде приложения.**

---

## 🎤 Суперкоротко

```text
UNIQUE
  ↓
значение не может повторяться
  ↓
БД гарантирует уникальность
```

Для дедупликации:

```text
event_id = 123
      ↓
UNIQUE(event_id)
      ↓
одно событие → одна запись
```

---

# 🔹 Зачем нужен Unique Constraint

Допустим, нам нужно хранить обработанные события:

```python
CREATE TABLE processed_events (
    id SERIAL PRIMARY KEY,
    event_id VARCHAR(100) UNIQUE
);
```

Первый раз:

```text
event_id = event-123
        ↓
INSERT
        ↓
✅ успешно
```

Второй раз:

```text
event_id = event-123
        ↓
INSERT
        ↓
❌ duplicate key
```

База сама гарантирует:

```text
event-123 → только одна запись
```

---

# 🔹 UNIQUE vs проверка в Python

Плохой вариант:

```python
if not user_exists(email):
    create_user(email)
```

Проблема — **race condition**.

Два запроса могут прийти одновременно:

```text
Request A ──→ user_exists? → нет
Request B ──→ user_exists? → нет

Request A ──→ create
Request B ──→ create
```

В результате могут появиться дубли.

---

## ✅ Правильнее

Создать ограничение:

```python
email VARCHAR(255) UNIQUE
```

Теперь даже если два запроса одновременно попытаются создать пользователя:

```text
Request A ──┐
            ├──→ Database
Request B ──┘
```

База гарантирует:

```text
Request A → INSERT ✅
Request B → UNIQUE violation ❌
```

---

# 🔒 UNIQUE — гарантия на уровне БД

Это важный принцип:

```text
Application
     ↓
Database
     ↓
UNIQUE
     ↓
гарантия целостности
```

Нельзя полностью полагаться на:

```python
if ...
```

если требование уникальности является **инвариантом данных**.

База должна сама гарантировать этот инвариант.

---

# 🔹 Уникальность одного поля

Например:

```python
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(100),
    email VARCHAR(255) UNIQUE
);
```

Получаем:

```text
user 1 → test@example.com ✅
user 2 → admin@example.com ✅
user 3 → test@example.com ❌
```

---

# 🔹 Составной UNIQUE

Иногда уникальным должно быть не одно поле, а **комбинация полей**.

Например:

```python
CREATE TABLE memberships (
    id SERIAL PRIMARY KEY,
    user_id INTEGER,
    team_id INTEGER,
    UNIQUE(user_id, team_id)
);
```

Теперь один пользователь не может дважды вступить в одну команду:

```text
user=1, team=10 → ✅
user=1, team=20 → ✅
user=1, team=10 → ❌
```

Но:

```text
user=2, team=10 → ✅
```

потому что комбинация уже другая.

---

# 🔄 UNIQUE и Idempotency Key

Очень распространённый паттерн:

```text
Client
  ↓
Idempotency-Key
  ↓
API
  ↓
Database
  ↓
UNIQUE(idempotency_key)
```

Например:

```python
CREATE TABLE idempotency_keys (
    id SERIAL PRIMARY KEY,
    key VARCHAR(255) UNIQUE,
    response JSONB
);
```

Первый запрос:

```text
key = abc-123
      ↓
INSERT
      ↓
✅
```

Retry:

```text
key = abc-123
      ↓
INSERT
      ↓
❌ duplicate
```

Приложение понимает:

> Эта операция уже была зарегистрирована.

---

# 📨 UNIQUE и Deduplication

Для событий:

```python
CREATE TABLE processed_events (
    event_id VARCHAR(255) PRIMARY KEY,
    processed_at TIMESTAMP
);
```

Здесь `PRIMARY KEY` также гарантирует уникальность `event_id`.

Схема:

```text
Event
  ↓
event_id
  ↓
INSERT
  ↓
┌───────────────┐
│ уже существует│
└───────────────┘
      ↓
   duplicate
      ↓
    skip
```

---

# ⚡ UNIQUE + INSERT

Один из важных подходов — не делать отдельно:

```python
SELECT
```

а затем:

```python
INSERT
```

Вместо этого можно попытаться сразу выполнить вставку и обработать конфликт.

В PostgreSQL:

```python
INSERT INTO processed_events (event_id)
VALUES ('event-123')
ON CONFLICT (event_id) DO NOTHING;
```

Получается:

```text
новый event_id
     ↓
INSERT
     ↓
обработать
```

Если уже существует:

```text
старый event_id
     ↓
ON CONFLICT
     ↓
DO NOTHING
```

Это хорошо подходит для дедупликации.

---

# 🔄 UNIQUE + Upsert

**Upsert** = `INSERT` или `UPDATE` при конфликте.

В PostgreSQL:

```python
INSERT INTO users (email, name)
VALUES ('test@example.com', 'Ilya')
ON CONFLICT (email)
DO UPDATE SET name = EXCLUDED.name;
```

Логика:

```text
email отсутствует
    ↓
INSERT

email существует
    ↓
UPDATE
```

---

# 🧩 UNIQUE Constraint vs PRIMARY KEY

Оба обеспечивают уникальность, но назначение отличается.

|                       | PRIMARY KEY    | UNIQUE                      |
| --------------------- | -------------- | --------------------------- |
| Уникальность          | ✅              | ✅                           |
| Идентифицирует строку | ✅              | Не обязательно              |
| NULL                  | Не допускается | Зависит от СУБД/правил      |
| На таблицу            | Обычно один PK | Может быть несколько UNIQUE |
| Типичный пример       | `id`           | `email`                     |

Например:

```python
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE
);
```

Здесь:

```text
id    → PRIMARY KEY
email → UNIQUE
```

---

# ⚠️ UNIQUE и NULL

Здесь есть важный нюанс.

В PostgreSQL обычный `UNIQUE` допускает несколько `NULL`, потому что `NULL` не считается равным другому `NULL`.

Например:

```python
email VARCHAR(255) UNIQUE
```

может позволить несколько строк:

```text
NULL
NULL
NULL
```

Если нужно запретить отсутствие значения:

```python
email VARCHAR(255) NOT NULL UNIQUE
```

Получаем:

```text
email
 ↓
NOT NULL
 +
UNIQUE
```

---

# 🔐 UNIQUE и Idempotency

Для идемпотентности часто используется комбинация:

```text
Idempotency-Key
       +
Client/User
       ↓
UNIQUE
```

Например, ключ может быть уникальным **в рамках клиента**:

```python
UNIQUE(client_id, idempotency_key)
```

Тогда:

```text
client=1, key=abc → ✅
client=1, key=abc → ❌

client=2, key=abc → ✅
```

Это полезно, если разные клиенты могут независимо использовать одинаковые ключи.

---

# ⚠️ UNIQUE не решает всю задачу идемпотентности

Важно понимать:

```text
UNIQUE
```

защищает от дублирования **записи**, но сама по себе не делает бизнес-операцию идемпотентной.

Например:

```text
Payment создан
      ↓
UNIQUE key сохранён
      ↓
но внешний платёжный сервис ещё не обработан
```

Нужна правильная транзакционная архитектура.

`UNIQUE` — один из строительных блоков, а не вся реализация идемпотентности.

---

# 🔄 UNIQUE + Transaction

Для критичных операций часто требуется:

```text
BEGIN
   ↓
INSERT idempotency_key
   ↓
выполнить бизнес-изменение
   ↓
COMMIT
```

Если произошла ошибка:

```text
ROLLBACK
```

и запись о ключе тоже откатывается.

Это позволяет избежать ситуации:

```text
Idempotency Key сохранён
       +
Payment не создан
```

---

# 🏗️ Пример для платежей

Таблица:

```python
CREATE TABLE payments (
    id SERIAL PRIMARY KEY,
    idempotency_key VARCHAR(255) UNIQUE,
    order_id INTEGER NOT NULL,
    amount INTEGER NOT NULL
);
```

Первый запрос:

```text
key = abc-123
       ↓
создать Payment #500
```

Retry:

```text
key = abc-123
       ↓
UNIQUE conflict
       ↓
Payment #500 уже существует
       ↓
вернуть существующий результат
```

Второго платежа нет.

---

# 🧠 Главное

**Unique Constraint**:

```text
UNIQUE
  ↓
гарантия уникальности
  ↓
на уровне БД
```

В распределённых системах:

```text
Retry
  ↓
Duplicate
  ↓
Deduplication
  ↓
UNIQUE(event_id)
  ↓
защита от повторной обработки
```

Для API:

```text
Retry
  ↓
Idempotency-Key
  ↓
UNIQUE(idempotency_key)
  ↓
не создать операцию повторно
```

### Формула для собеседования

> **Unique Constraint — ограничение БД, гарантирующее уникальность значения или комбинации значений. В идемпотентности и дедупликации его используют, чтобы на уровне базы гарантировать отсутствие дублей и защититься от race condition, которую нельзя надёжно устранить одной проверкой в коде.**
