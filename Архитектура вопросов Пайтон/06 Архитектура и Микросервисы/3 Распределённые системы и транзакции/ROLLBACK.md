# ↩️ ROLLBACK

## 🎯 Ответ на собеседовании

**ROLLBACK** — SQL-команда, которая отменяет изменения текущей транзакции, выполненные после её начала или последнего [[SAVEPOINT]].

Если транзакция завершилась ошибкой, выполняется `ROLLBACK`, и изменения этой транзакции не фиксируются в базе.

```text id="7x3mqa"
BEGIN
  ↓
INSERT
UPDATE
DELETE
  ↓
ERROR
  ↓
ROLLBACK
  ↓
изменения отменены
```

> **COMMIT фиксирует транзакцию, ROLLBACK отменяет её незакоммиченные изменения.**

---

## 🎤 Суперкоротко

> **ROLLBACK отменяет изменения текущей транзакции, которые ещё не были зафиксированы через COMMIT.**

```text id="9n4vcz"
BEGIN
  ↓
операции
  ↓
ROLLBACK
  ↓
отмена изменений
```

---

# 🔹 Простой пример

Начальное состояние:

```text id="j5q8xm"
balance = 1000
```

Выполняем:

```python id="2r8m1k"
BEGIN;

UPDATE accounts
SET balance = 500
WHERE id = 1;

ROLLBACK;
```

После `ROLLBACK`:

```text id="p7c4wd"
balance = 1000
```

Изменение:

```text id="k2m9sx"
1000 → 500
```

было отменено.

---

# 🔄 COMMIT vs ROLLBACK

| Команда    | Что делает                         |
| ---------- | ---------------------------------- |
| `COMMIT`   | Фиксирует изменения                |
| `ROLLBACK` | Отменяет незакоммиченные изменения |

```text id="q8f2vn"
             BEGIN
               │
          SQL operations
               │
        ┌──────┴──────┐
        ↓             ↓
     COMMIT        ROLLBACK
        ↓             ↓
     сохранить     отменить
```

---

# ⚛️ ROLLBACK и Atomicity

Рассмотрим перевод денег:

```text id="m4z7pq"
Account A: -1000
Account B: +1000
```

Обе операции выполняются внутри одной транзакции:

```python id="d8v3kc"
BEGIN;

UPDATE accounts
SET balance = balance - 1000
WHERE id = 1;

UPDATE accounts
SET balance = balance + 1000
WHERE id = 2;

COMMIT;
```

Если вторая операция завершилась ошибкой:

```text id="f6w2ra"
UPDATE A → ✅
UPDATE B → ❌
        ↓
     ROLLBACK
```

Изменение счёта A также отменяется.

Получаем принцип:

```text id="x3n9vb"
всё успешно → COMMIT
ошибка       → ROLLBACK
```

Это и есть **Atomicity — "всё или ничего"**.

---

# 🔥 ROLLBACK после ошибки

Типичный сценарий:

```python id="k7p4md"
try:
    cursor.execute(...)
    cursor.execute(...)
    connection.commit()
except Exception:
    connection.rollback()
    raise
```

Логика:

```text id="z6m2qa"
операции
   ↓
успех → COMMIT
   ↓
ошибка → ROLLBACK
```

После `ROLLBACK` можно:

* вернуть ошибку клиенту;
* выполнить retry;
* обработать исключение;
* завершить операцию.

---

# 🔒 ROLLBACK и блокировки

Если транзакция держит блокировку:

```python id="a4v8nx"
BEGIN;

SELECT *
FROM accounts
WHERE id = 1
FOR UPDATE;
```

после:

```python id="p6m3wc"
ROLLBACK;
```

транзакция завершается и её блокировки освобождаются.

```text id="j8q2rv"
BEGIN
  ↓
LOCK
  ↓
операции
  ↓
ROLLBACK
  ↓
UNLOCK
```

---

# 🧩 ROLLBACK и Race Condition

Транзакция может обнаружить конфликт конкурентных операций.

Например:

```text id="v5c8nz"
Transaction A
     ↓
изменяет данные
     ↓
конфликт
     ↓
ROLLBACK
```

После этого приложение может повторить транзакцию:

```text id="m7x2kp"
ROLLBACK
   ↓
Retry
   ↓
BEGIN
   ↓
операции
   ↓
COMMIT
```

Особенно актуально при `SERIALIZABLE`, где транзакция может завершиться ошибкой сериализации.

---

# 🔄 ROLLBACK и Retry

Важно понимать:

> **Retry транзакции обычно означает сначала откатить неудачную попытку, а затем выполнить новую транзакцию.**

```text id="u9r3fk"
BEGIN
  ↓
операции
  ↓
Serialization Error
  ↓
ROLLBACK
  ↓
Retry
  ↓
BEGIN
  ↓
операции
  ↓
COMMIT
```

Нельзя просто продолжать использовать транзакцию после ошибки, если она уже находится в состоянии, требующем rollback.

---

# 🧩 SAVEPOINT

`ROLLBACK` может отменять не обязательно всю транзакцию.

Для частичного отката используют **SAVEPOINT**.

```python id="n5w8qc"
BEGIN;

INSERT INTO orders (...);

SAVEPOINT before_payment;

INSERT INTO payments (...);

ROLLBACK TO SAVEPOINT before_payment;

COMMIT;
```

Получается:

```text id="x4r7mb"
BEGIN
  ↓
создать Order
  ↓
SAVEPOINT
  ↓
создать Payment ❌
  ↓
ROLLBACK TO SAVEPOINT
  ↓
Payment отменён
  ↓
COMMIT
  ↓
Order сохранён
```

То есть `SAVEPOINT` позволяет вернуться к определённой точке внутри транзакции.

---

# 🔹 ROLLBACK vs ROLLBACK TO SAVEPOINT

### `ROLLBACK`

Отменяет всю текущую транзакцию:

```text id="q3m7vx"
BEGIN
  ↓
A
  ↓
B
  ↓
C
  ↓
ROLLBACK
  ↓
A ❌
B ❌
C ❌
```

### `ROLLBACK TO SAVEPOINT`

Откатывает только часть операций после точки сохранения:

```text id="f8k2nr"
BEGIN
  ↓
A
  ↓
SAVEPOINT
  ↓
B
  ↓
C
  ↓
ROLLBACK TO SAVEPOINT
  ↓
B ❌
C ❌
  ↓
A остаётся
```

После этого транзакцию можно продолжить.

---

# ⚠️ ROLLBACK не отменяет COMMIT

Очень важный момент.

```python id="h6v3mz"
BEGIN;

UPDATE accounts
SET balance = 500
WHERE id = 1;

COMMIT;
```

После `COMMIT`:

```text id="p8x2qa"
balance = 500
```

Если затем выполнить:

```python id="w4n7kc"
ROLLBACK;
```

это **не вернёт** баланс к `1000`.

```text id="r9m3vd"
COMMIT
  ↓
изменения зафиксированы
  ↓
ROLLBACK
  ↓
уже не отменяет COMMIT
```

Чтобы изменить данные обратно, нужна **новая операция/транзакция**.

---

# 🔗 ROLLBACK + Idempotency

Представим:

```text id="t5k8pz"
BEGIN
  ↓
создать Payment
  ↓
сохранить Idempotency Key
  ↓
ошибка
  ↓
ROLLBACK
```

Обе операции откатываются.

После retry:

```text id="y7m2qx"
BEGIN
  ↓
создать Payment
  ↓
сохранить Idempotency Key
  ↓
COMMIT
```

Получаем согласованное состояние:

```text id="b4x9nr"
Payment существует
+
Idempotency Key существует
```

---

# 📨 ROLLBACK + Deduplication

Например, Consumer обрабатывает:

```text id="k2w6mc"
event_id = 123
```

Транзакция:

```text id="f9v3qa"
BEGIN
  ↓
зарегистрировать event_id
  ↓
изменить Order
  ↓
ошибка
  ↓
ROLLBACK
```

После `ROLLBACK` запись `event_id` тоже отменяется.

Поэтому сообщение может быть обработано повторно:

```text id="s6m1xz"
event_id = 123
   ↓
Retry
   ↓
обработать заново
```

Это правильное поведение, если первая попытка не была успешно зафиксирована.

---

# 🗄️ ROLLBACK и состояние соединения

После ошибки некоторые драйверы БД оставляют текущую транзакцию в состоянии ошибки.

Например:

```text id="e7q3mx"
SQL ERROR
   ↓
transaction aborted
   ↓
ROLLBACK
   ↓
соединение снова готово
```

Поэтому после ошибки важно корректно завершить транзакцию.

---

# 🧠 Типичный паттерн

```text id="v3q8km"
BEGIN
  ↓
Operation 1
  ↓
Operation 2
  ↓
Operation 3
  ↓
  ├── Success → COMMIT
  │
  └── Error   → ROLLBACK
```

---

# 🐍 Django

В Django:

```python id="m8q4vz"
from django.db import transaction

with transaction.atomic():
    account.balance -= 1000
    account.save()

    target.balance += 1000
    target.save()
```

Если внутри блока возникает исключение, Django откатывает транзакцию.

Концептуально:

```text id="x6r2pw"
atomic()
   ↓
operations
   ↓
Exception
   ↓
ROLLBACK
```

---

# 🐘 PostgreSQL

Пример:

```python id="j4n8qs"
BEGIN;

INSERT INTO users (email)
VALUES ('test@example.com');

INSERT INTO users (email)
VALUES ('test@example.com');

ROLLBACK;
```

Если второй `INSERT` нарушает `UNIQUE`, транзакция может потребовать rollback.

После него изменения первой операции также не будут зафиксированы.

---

# ⚠️ ROLLBACK и внешние системы

Транзакция БД может откатить:

```text id="w8m4qc"
PostgreSQL
   ↓
INSERT
UPDATE
DELETE
```

Но она не может автоматически откатить уже выполненный внешний HTTP-запрос:

```text id="k5r2vx"
BEGIN
  ↓
UPDATE PostgreSQL
  ↓
HTTP → Payment Service
  ↓
Payment created
  ↓
ROLLBACK PostgreSQL
```

`ROLLBACK` отменит изменения PostgreSQL, но **не отменит автоматически платеж во внешнем сервисе**.

Для таких сценариев используются:

```text id="z3p7mn"
Saga
Compensation
Outbox
```

---

# 🧠 Главное

```text id="m7x4qc"
BEGIN
  ↓
операции
  ↓
ошибка
  ↓
ROLLBACK
  ↓
отмена незакоммиченных изменений
```

### Связь с COMMIT

```text id="p9r3vx"
Успех
  ↓
COMMIT
  ↓
сохранить
```

```text id="f6k2wm"
Ошибка
  ↓
ROLLBACK
  ↓
отменить
```

### Формула для собеседования

> **ROLLBACK отменяет незакоммиченные изменения текущей транзакции. Он используется при ошибках, чтобы сохранить атомарность операции. После COMMIT обычный ROLLBACK уже не может отменить зафиксированные изменения. Для частичного отката внутри транзакции используются SAVEPOINT.**
