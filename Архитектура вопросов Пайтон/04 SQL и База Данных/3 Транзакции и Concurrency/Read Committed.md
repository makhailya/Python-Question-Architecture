# 🔐 READ COMMITTED

## 🎯 Ответ на собеседовании

**READ COMMITTED** — уровень изоляции транзакций, при котором запрос видит только данные, **зафиксированные (`COMMIT`) до начала этого конкретного запроса**.

В PostgreSQL это **уровень изоляции по умолчанию**.

Главная особенность: **каждый SQL-запрос внутри транзакции получает собственный snapshot**. Поэтому два одинаковых `SELECT` в одной транзакции могут увидеть разные данные, если между ними другая транзакция сделала `COMMIT`.

---

## 🎤 Суперкоротко

> **READ COMMITTED** гарантирует, что запрос не прочитает незакоммиченные данные другой транзакции. Но разные запросы одной транзакции могут увидеть разные состояния БД.

Главный эффект:

**Dirty Read ❌**
**Non-repeatable Read ✅ возможен**
**Phantom Read ✅ возможен**

---

## 📌 Как работает

Представим две транзакции:

```python
# Транзакция A
BEGIN;

SELECT balance FROM accounts WHERE id = 1;
-- 1000
```

Параллельно:

```python
# Транзакция B
BEGIN;

UPDATE accounts
SET balance = 500
WHERE id = 1;

COMMIT;
```

Теперь транзакция A выполняет тот же запрос ещё раз:

```python
SELECT balance FROM accounts WHERE id = 1;
-- 500
```

Почему?

Потому что второй `SELECT` получил **новый snapshot**, который уже видит изменения транзакции B после её `COMMIT`.

---

# 🧠 Snapshot в READ COMMITTED

В PostgreSQL:

```python
BEGIN;

SELECT ...;  # Snapshot №1

SELECT ...;  # Snapshot №2

COMMIT;
```

Каждый оператор получает свой snapshot.

Условно:

```python
Transaction A

SELECT ──► Snapshot 1 ──► видит состояние БД на этот момент

             ↓

Transaction B
UPDATE + COMMIT

             ↓

SELECT ──► Snapshot 2 ──► уже видит изменение B
```

Поэтому:

> **READ COMMITTED — изоляция на уровне отдельных операторов, а не всей транзакции.**

---

# 🚫 Dirty Read невозможен

Другой процесс может изменить данные:

```python
BEGIN;

UPDATE accounts
SET balance = 0
WHERE id = 1;
```

Но пока нет:

```python
COMMIT;
```

другая транзакция в PostgreSQL не увидит это изменение.

Если первая транзакция сделает:

```python
ROLLBACK;
```

то другая транзакция вообще не увидит промежуточное состояние.

Поэтому:

**Dirty Read → ❌**

---

# ⚠️ Non-repeatable Read

Это ситуация, когда один и тот же `SELECT` внутри одной транзакции возвращает разные данные.

Например:

```python
# Transaction A
BEGIN;

SELECT balance FROM accounts WHERE id = 1;
-- 1000
```

Другая транзакция:

```python
# Transaction B
UPDATE accounts
SET balance = 500
WHERE id = 1;

COMMIT;
```

И снова:

```python
# Transaction A

SELECT balance FROM accounts WHERE id = 1;
-- 500
```

Получается:

```python
SELECT → 1000

другая транзакция COMMIT

SELECT → 500
```

Это называется:

**Non-repeatable Read** — повторное чтение той же строки даёт другое значение.

---

# 👻 Phantom Read

READ COMMITTED также допускает появление новых строк между запросами.

Например:

```python
# Transaction A

BEGIN;

SELECT COUNT(*)
FROM orders
WHERE user_id = 10;

-- 5
```

Другая транзакция:

```python
# Transaction B

INSERT INTO orders (user_id)
VALUES (10);

COMMIT;
```

Снова:

```python
# Transaction A

SELECT COUNT(*)
FROM orders
WHERE user_id = 10;

-- 6
```

Появилась новая строка, удовлетворяющая условию.

Это:

**Phantom Read** 👻

---

# 🔒 READ COMMITTED и блокировки

READ COMMITTED **не означает отсутствие блокировок**.

Например:

```python
SELECT *
FROM accounts
WHERE id = 1
FOR UPDATE;
```

Такой запрос устанавливает блокировку строки.

Другие транзакции могут быть вынуждены ждать завершения текущей транзакции.

То есть:

> **MVCC позволяет обычным чтениям не блокировать записи, но блокировки всё равно существуют и используются для координации конкурентных операций.**

---

# 🆚 READ COMMITTED vs REPEATABLE READ

| Характеристика      | READ COMMITTED      | REPEATABLE READ |
| ------------------- | ------------------- | --------------- |
| Dirty Read          | ❌                   | ❌               |
| Non-repeatable Read | ✅                   | ❌               |
| Phantom Read        | ✅                   | ❌*              |
| Snapshot            | На каждый statement | На транзакцию   |
| PostgreSQL default  | ✅                   | ❌               |
| Изоляция            | Ниже                | Выше            |

* В PostgreSQL `REPEATABLE READ` предотвращает обычные phantom reads благодаря единому snapshot.

---

# ⚙️ Как установить READ COMMITTED

Для конкретной транзакции:

```python
BEGIN;

SET TRANSACTION ISOLATION LEVEL READ COMMITTED;

SELECT * FROM users;

COMMIT;
```

Посмотреть текущий уровень:

```python
SHOW transaction_isolation;
```

В PostgreSQL обычно получим:

```python
read committed
```

---

# 🐍 Пример с PostgreSQL

Условный Python-код:

```python
conn = psycopg2.connect(...)
conn.set_isolation_level(
    psycopg2.extensions.ISOLATION_LEVEL_READ_COMMITTED
)

cursor = conn.cursor()

cursor.execute(
    "SELECT balance FROM accounts WHERE id = %s",
    (1,)
)

balance = cursor.fetchone()[0]

conn.commit()
```

На практике `READ COMMITTED` обычно можно вообще не указывать, потому что это default PostgreSQL.

---

# ⚠️ Важная проблема READ COMMITTED

Предположим, два процесса одновременно проверяют баланс:

```python
SELECT balance
FROM accounts
WHERE id = 1;
```

Оба получают:

```python
1000
```

После этого оба решают:

```python
balance >= 800
```

и выполняют списание.

Можно получить некорректный результат.

Поэтому для критичных конкурентных операций используют дополнительные механизмы:

### 1. `SELECT FOR UPDATE`

```python
SELECT balance
FROM accounts
WHERE id = 1
FOR UPDATE;
```

### 2. Атомарный `UPDATE`

```python
UPDATE accounts
SET balance = balance - 800
WHERE id = 1
  AND balance >= 800;
```

### 3. Более высокий уровень изоляции

Например:

```python
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
```

Выбор зависит от бизнес-логики.

---

# 🔥 Главное отличие от REPEATABLE READ

Запомнить можно одной фразой:

> **READ COMMITTED — новый snapshot на каждый запрос.**

А:

> **REPEATABLE READ — один snapshot на всю транзакцию.**

Схематично:

```python
READ COMMITTED:

BEGIN
  │
  ├── SELECT → Snapshot 1
  │
  ├── другая транзакция COMMIT
  │
  └── SELECT → Snapshot 2
COMMIT
```

```python
REPEATABLE READ:

BEGIN
  │
  ├── SELECT → Snapshot 1
  │
  ├── другая транзакция COMMIT
  │
  └── SELECT → Snapshot 1
COMMIT
```

---

# 🎯 Формулировка для собеседования

> **READ COMMITTED — уровень изоляции по умолчанию в PostgreSQL. Каждый SQL-оператор получает собственный snapshot и видит только данные, закоммиченные до начала этого оператора. Поэтому dirty read невозможен, но возможны non-repeatable и phantom reads. Если нужно сохранить единое состояние данных на всю транзакцию, используют REPEATABLE READ или более высокий уровень.**

---

## 📌 Формула

```python
READ COMMITTED
=
Committed data
+
Snapshot per statement
+
No Dirty Read
+
Possible Non-repeatable Read
+
Possible Phantom Read
```
