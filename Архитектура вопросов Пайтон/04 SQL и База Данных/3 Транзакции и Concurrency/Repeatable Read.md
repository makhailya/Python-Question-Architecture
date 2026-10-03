# 🔐 REPEATABLE READ

## 🎯 Ответ на собеседовании

**REPEATABLE READ** — уровень изоляции транзакций, при котором все обычные запросы внутри одной транзакции видят **согласованный snapshot, созданный в начале транзакции**.

В PostgreSQL это означает, что повторное чтение тех же данных в рамках одной транзакции не увидит изменения, которые были закоммичены другими транзакциями после создания snapshot.

Главная идея:

> **Одна транзакция → один snapshot → стабильное представление данных.**

---

## 🎤 Суперкоротко

> **REPEATABLE READ** гарантирует стабильный snapshot на всю транзакцию. Dirty Read и Non-repeatable Read невозможны, а обычные Phantom Read в PostgreSQL также предотвращаются. При конфликте конкурентных изменений транзакция может получить ошибку сериализации и её потребуется повторить.

---

# 🧠 Как работает

Представим транзакцию A:

```python id="7k3mqp"
BEGIN;

SELECT balance
FROM accounts
WHERE id = 1;

-- 1000
```

В этот момент PostgreSQL создаёт snapshot транзакции.

Затем транзакция B изменяет данные:

```python id="9n4vtx"
BEGIN;

UPDATE accounts
SET balance = 500
WHERE id = 1;

COMMIT;
```

Транзакция A снова выполняет запрос:

```python id="c8rj2w"
SELECT balance
FROM accounts
WHERE id = 1;

-- 1000
```

Несмотря на то что B уже сделала `COMMIT`, A продолжает видеть своё исходное согласованное состояние.

---

# 📸 Snapshot на всю транзакцию

Сравним с READ COMMITTED.

### READ COMMITTED

```python id="2qf7sa"
BEGIN;

SELECT ...;  # Snapshot 1

SELECT ...;  # Snapshot 2

COMMIT;
```

### REPEATABLE READ

```python id="5m1xkd"
BEGIN;

SELECT ...;  # Snapshot 1

SELECT ...;  # Snapshot 1

SELECT ...;  # Snapshot 1

COMMIT;
```

Именно поэтому название:

**Repeatable Read → повторное чтение даёт стабильный результат.**

---

# 🚫 Dirty Read

Как и READ COMMITTED, REPEATABLE READ не позволяет прочитать незакоммиченные изменения.

```python id="x8f3na"
# Transaction B

BEGIN;

UPDATE accounts
SET balance = 500
WHERE id = 1;

# COMMIT ещё не выполнен
```

Transaction A:

```python id="w4j2kc"
SELECT balance
FROM accounts
WHERE id = 1;
```

Не увидит `500`.

**Dirty Read → ❌**

---

# 🚫 Non-repeatable Read

Повторное чтение строки внутри одной транзакции остаётся стабильным.

```python id="p3s6ye"
BEGIN;

SELECT balance
FROM accounts
WHERE id = 1;

-- 1000
```

Другая транзакция:

```python id="a9v5nd"
UPDATE accounts
SET balance = 500
WHERE id = 1;

COMMIT;
```

Первая транзакция:

```python id="j7c2qm"
SELECT balance
FROM accounts
WHERE id = 1;

-- 1000
```

Изменение другой транзакции не попадает в snapshot текущей транзакции.

**Non-repeatable Read → ❌**

---

# 👻 Phantom Read

В PostgreSQL `REPEATABLE READ` также обеспечивает стабильное представление набора строк.

Например:

```python id="h2x8qw"
BEGIN;

SELECT COUNT(*)
FROM orders
WHERE user_id = 10;

-- 5
```

Другая транзакция добавляет заказ:

```python id="m6k1zr"
INSERT INTO orders (user_id)
VALUES (10);

COMMIT;
```

Первая транзакция:

```python id="r4d9pt"
SELECT COUNT(*)
FROM orders
WHERE user_id = 10;

-- 5
```

Новая строка не появляется в snapshot текущей транзакции.

**Phantom Read → ❌ в PostgreSQL**

---

# ⚠️ Но REPEATABLE READ не означает «никаких конфликтов»

Очень важный момент для собеседования.

REPEATABLE READ сохраняет snapshot, но если две транзакции одновременно пытаются изменить одну и ту же строку, PostgreSQL может завершить одну из них ошибкой.

Например:

```python id="u6p2ab"
ERROR: could not serialize access due to concurrent update
```

Приложение в таком случае должно обработать ошибку и, как правило, **повторить транзакцию**.

---

# 🔒 REPEATABLE READ и блокировки

REPEATABLE READ не отменяет блокировки.

Например:

```python id="k9x4vd"
SELECT *
FROM accounts
WHERE id = 1
FOR UPDATE;
```

`FOR UPDATE` блокирует выбранную строку для конфликтующих операций.

Поэтому:

> **MVCC + snapshot ≠ отсутствие locks.**

PostgreSQL использует MVCC для видимости версий, а блокировки — для координации конфликтующих операций.

---

# ⚙️ Как установить REPEATABLE READ

Для конкретной транзакции:

```python id="n5r8yc"
BEGIN;

SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;

SELECT * FROM users;

COMMIT;
```

Или:

```python id="e3q7mw"
BEGIN ISOLATION LEVEL REPEATABLE READ;

SELECT * FROM users;

COMMIT;
```

Проверить текущий уровень:

```python id="t8v2lf"
SHOW transaction_isolation;
```

Результат:

```python id="b4k9xs"
repeatable read
```

---

# 🆚 READ COMMITTED vs REPEATABLE READ

| Характеристика                    | READ COMMITTED           | REPEATABLE READ                        |
| --------------------------------- | ------------------------ | -------------------------------------- |
| Dirty Read                        | ❌                        | ❌                                      |
| Non-repeatable Read               | ✅                        | ❌                                      |
| Phantom Read в PostgreSQL         | ✅                        | ❌                                      |
| Snapshot                          | На каждый statement      | На всю транзакцию                      |
| Изменения после начала транзакции | Видны следующим запросам | Не видны обычным запросам              |
| Конфликты записи                  | Возможны                 | Возможны, возможна ошибка сериализации |
| Default PostgreSQL                | ✅                        | ❌                                      |

---

# 🎯 Когда использовать

REPEATABLE READ полезен, когда нескольким запросам внутри одной транзакции нужно работать с **одним и тем же согласованным состоянием данных**.

Например:

* сложные отчёты;
* финансовые расчёты;
* чтение нескольких связанных наборов данных;
* операции, где важно стабильное состояние данных на протяжении транзакции.

Но не стоит автоматически использовать его везде.

Более высокий уровень изоляции может увеличить количество конфликтов и необходимость повторных транзакций.

---

# 🧩 REPEATABLE READ + MVCC

В PostgreSQL это тесно связано с MVCC.

У строки может существовать несколько версий:

```python id="c5m8za"
accounts

id=1
│
├── version 1 → balance=1000
│
└── version 2 → balance=500
```

Snapshot транзакции определяет, какую версию она может видеть.

Поэтому одна транзакция может продолжать видеть старую версию строки, даже когда другая транзакция уже закоммитила новую.

---

# ⚠️ Долгие транзакции

Длительная транзакция с REPEATABLE READ удерживает старый snapshot.

Например:

```python id="v7n3pq"
BEGIN;

-- snapshot создан

SELECT ...;

-- приложение работает 30 минут

SELECT ...;

COMMIT;
```

За это время другие транзакции могут создавать новые версии строк.

Старый snapshot может мешать PostgreSQL окончательно очистить некоторые старые версии, поэтому **долгие транзакции могут способствовать bloat**.

---

# 🆚 REPEATABLE READ vs SERIALIZABLE

Это частый вопрос на собеседовании.

### REPEATABLE READ

Гарантирует стабильный snapshot:

```python id="z2m6rx"
Transaction
    │
    └── один согласованный snapshot
```

### SERIALIZABLE

Добавляет более строгий контроль конкурентных операций и гарантирует поведение, эквивалентное некоторому последовательному выполнению транзакций.

При конфликте PostgreSQL может отменить транзакцию:

```python id="f8q1nd"
ERROR: could not serialize access
```

Приложение должно повторить транзакцию.

Упрощённо:

```python id="s4w9kc"
READ COMMITTED
    ↓
REPEATABLE READ
    ↓
SERIALIZABLE
```

Чем выше уровень, тем строже требования к конкурентному выполнению, но потенциально больше конфликтов и retry.

---

# 📌 Логика уровней

```python id="r8v3mk"
READ UNCOMMITTED
    │
    │ PostgreSQL фактически = READ COMMITTED
    ↓
READ COMMITTED
    │
    │ snapshot на каждый statement
    ↓
REPEATABLE READ
    │
    │ snapshot на всю транзакцию
    ↓
SERIALIZABLE
    │
    │ поведение как при последовательном выполнении
    ↓
самая строгая изоляция
```

---

# 🎤 Формулировка для собеседования

> **REPEATABLE READ — уровень изоляции, при котором транзакция работает с согласованным snapshot на протяжении всей транзакции. В PostgreSQL это предотвращает dirty reads, non-repeatable reads и обычные phantom reads. При конкурентных изменениях возможны конфликты, из-за которых PostgreSQL может завершить транзакцию ошибкой сериализации, поэтому приложение должно быть готово повторять транзакцию.**

---

## 📌 Формула

```python id="n7x4qa"
REPEATABLE READ
=
1 transaction
+
1 consistent snapshot
+
No Dirty Read
+
No Non-repeatable Read
+
No Phantom Read in PostgreSQL
+
Possible serialization conflicts
```
