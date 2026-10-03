# 🔒 SERIALIZABLE

## 🎯 Ответ на собеседовании

**SERIALIZABLE** — самый строгий стандартный уровень изоляции транзакций, при котором результат конкурентного выполнения транзакций должен быть эквивалентен **какому-либо последовательному выполнению этих транзакций**.

В PostgreSQL `SERIALIZABLE` реализуется поверх механизмов MVCC и Serializable Snapshot Isolation (SSI).

Главная особенность:

> PostgreSQL не обязан физически выполнять транзакции строго по очереди. Он контролирует зависимости между ними и, если обнаруживает несовместимое конкурентное выполнение, **отменяет одну из транзакций**.

Приложение после такой ошибки должно **повторить транзакцию**.

---

## 🎤 Суперкоротко

> **SERIALIZABLE** гарантирует результат, эквивалентный последовательному выполнению транзакций. В PostgreSQL конкурентные транзакции могут выполняться параллельно, но при обнаружении опасного конфликта одна из них завершается ошибкой сериализации и должна быть повторена.

---

# 🧠 Что означает «сериализуемый»

Представим две транзакции:

```python id="p8m3qx"
Transaction A
Transaction B
```

Они выполняются одновременно:

```python id="v2k7na"
A ────────────────┐
                  ├── параллельное выполнение
B ────────────────┘
```

Но итог должен быть таким, будто они выполнились последовательно:

```python id="x5r9cw"
A → B
```

или:

```python id="m4t8zs"
B → A
```

То есть:

> **Concurrent execution → результат как при serial execution.**

---

# ⚠️ SERIALIZABLE не означает «транзакции выполняются по очереди»

Это очень важный момент.

Неправильное объяснение:

> «SERIALIZABLE просто блокирует все транзакции и выполняет их одну за другой».

В PostgreSQL это **не так**.

Транзакции могут выполняться одновременно:

```python id="h6q2wd"
Transaction A ────────┐
                      │
Transaction B ────────┤
                      │
Transaction C ────────┘
```

PostgreSQL отслеживает зависимости и предотвращает небезопасные комбинации.

Если безопасно сериализовать результат нельзя:

```python id="c9f4yk"
ERROR: could not serialize access due to ...
```

Одна транзакция прерывается.

---

# 🔥 Пример конкурентного конфликта

Допустим, есть ограниченный ресурс:

```python id="r7m2vp"
seats = 1
```

Два пользователя одновременно пытаются купить последнее место.

### Transaction A

```python id="k3w8qa"
BEGIN ISOLATION LEVEL SERIALIZABLE;

SELECT seats
FROM events
WHERE id = 1;

-- 1
```

### Transaction B

```python id="n6x4tz"
BEGIN ISOLATION LEVEL SERIALIZABLE;

SELECT seats
FROM events
WHERE id = 1;

-- 1
```

Обе транзакции видят свободное место и пытаются изменить состояние.

PostgreSQL может определить конфликт сериализации и отменить одну транзакцию:

```python id="z9c5lm"
ERROR: could not serialize access due to concurrent update
```

Приложение должно повторить транзакцию.

---

# 🔄 Retry — обязательная часть работы с SERIALIZABLE

Если приложение использует `SERIALIZABLE`, оно должно быть готово к serialization failure.

Условно:

```python id="q4v7ns"
for attempt in range(3):
    try:
        begin_transaction()

        do_business_operation()

        commit()

        break

    except SerializationFailure:
        rollback()

        retry()
```

Важно:

> **Retry должен повторять всю транзакцию целиком**, а не только последний SQL-запрос.

---

# 📸 Snapshot и SERIALIZABLE

В PostgreSQL `SERIALIZABLE` использует механизм **Serializable Snapshot Isolation (SSI)**.

Упрощённо:

```python id="w8n2kc"
Transaction
     │
     ▼
Snapshot
     │
     ▼
MVCC
     │
     ▼
Отслеживание опасных зависимостей
     │
     ├── всё безопасно → COMMIT
     │
     └── конфликт → serialization failure
```

Поэтому SERIALIZABLE не является просто «более сильным блокированием».

---

# 🚫 Какие аномалии предотвращает

В SERIALIZABLE результат не должен содержать аномалий, которые нарушают сериализуемость.

В классическом представлении:

| Аномалия              | SERIALIZABLE |
| --------------------- | -----------: |
| Dirty Read            |            ❌ |
| Non-repeatable Read   |            ❌ |
| Phantom Read          |            ❌ |
| Serialization anomaly |            ❌ |

Главное отличие:

> **SERIALIZABLE защищает не только отдельные чтения, но и корректность результата конкурентного выполнения транзакций.**

---

# 🆚 READ COMMITTED

```python id="f5x8mr"
READ COMMITTED
```

Каждый запрос получает свой snapshot.

Например:

```python id="a7k3qp"
BEGIN;

SELECT ...;  # Snapshot 1

-- другая транзакция COMMIT

SELECT ...;  # Snapshot 2

COMMIT;
```

Поэтому состояние между запросами может измениться.

---

# 🆚 REPEATABLE READ

```python id="u9d4ls"
REPEATABLE READ
```

Вся транзакция работает с согласованным snapshot:

```python id="c2m6wa"
BEGIN;

SELECT ...;  # Snapshot 1

-- другая транзакция COMMIT

SELECT ...;  # всё ещё Snapshot 1

COMMIT;
```

Но SERIALIZABLE идёт дальше:

```python id="e8q5tv"
SERIALIZABLE
```

Он должен обеспечить **сериализуемый результат конкурентного выполнения**.

Если обнаруживается опасная зависимость:

```python id="b3n7xy"
Transaction A ──┐
                ├── конфликт
Transaction B ──┘
       ↓
serialization failure
```

---

# 🆚 Три уровня

|                              | READ COMMITTED             | REPEATABLE READ     | SERIALIZABLE            |
| ---------------------------- | -------------------------- | ------------------- | ----------------------- |
| Snapshot                     | На каждый statement        | На транзакцию       | На транзакцию + SSI     |
| Dirty Read                   | ❌                          | ❌                   | ❌                       |
| Non-repeatable Read          | ✅                          | ❌                   | ❌                       |
| Phantom Read в PostgreSQL    | ✅                          | ❌                   | ❌                       |
| Serialization anomaly        | Возможна                   | Возможна            | ❌                       |
| Возможна ошибка сериализации | Редко/по другим конфликтам | Да                  | Да, ожидаемое поведение |
| Retry транзакции             | Обычно не требуется        | Может потребоваться | **Нужно предусмотреть** |
| Строгость                    | Низкая                     | Средняя             | Максимальная            |

---

# ⚙️ Как включить

Для конкретной транзакции:

```python id="j6w4ps"
BEGIN;

SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;

SELECT * FROM accounts;

COMMIT;
```

Или:

```python id="d8r2mk"
BEGIN ISOLATION LEVEL SERIALIZABLE;

SELECT * FROM accounts;

COMMIT;
```

Проверить текущий уровень:

```python id="s5k9qa"
SHOW transaction_isolation;
```

Результат:

```python id="t3m7vx"
serializable
```

---

# 💻 Практический пример

Представим перевод денег.

```python id="n4q8yc"
BEGIN ISOLATION LEVEL SERIALIZABLE;

SELECT balance
FROM accounts
WHERE id = 1;

SELECT balance
FROM accounts
WHERE id = 2;

UPDATE accounts
SET balance = balance - 500
WHERE id = 1;

UPDATE accounts
SET balance = balance + 500
WHERE id = 2;

COMMIT;
```

Если параллельная транзакция создаёт несовместимое состояние, PostgreSQL может отменить одну из транзакций.

Приложение получает ошибку и повторяет **всю операцию перевода**.

---

# ⚠️ Цена SERIALIZABLE

Более строгая изоляция имеет стоимость.

Возможны:

* больше serialization failures;
* дополнительные проверки зависимостей;
* необходимость retry;
* снижение эффективной пропускной способности при высокой конкуренции;
* более сложная обработка ошибок на уровне приложения.

Поэтому:

> **SERIALIZABLE не означает «лучше всегда».**

Нужно выбирать уровень изоляции исходя из требований бизнес-операции.

---

# 🆚 SERIALIZABLE vs SELECT FOR UPDATE

Это тоже важно различать.

### `SELECT FOR UPDATE`

Явно блокирует выбранные строки:

```python id="y6p3fw"
SELECT *
FROM accounts
WHERE id = 1
FOR UPDATE;
```

### `SERIALIZABLE`

Контролирует конкурентное выполнение транзакции в целом.

```python id="q8n4mz"
BEGIN ISOLATION LEVEL SERIALIZABLE;

-- несколько SELECT/UPDATE

COMMIT;
```

То есть:

> `FOR UPDATE` — конкретная блокировка строк.
> `SERIALIZABLE` — гарантия сериализуемого результата всей транзакции.

Они могут использоваться вместе, если это оправдано.

---

# 🔥 Частая ошибка на собеседовании

### ❌ Неправильно

> SERIALIZABLE блокирует всю таблицу и запускает транзакции последовательно.

### ✅ Правильно

> В PostgreSQL SERIALIZABLE позволяет транзакциям выполняться конкурентно, но отслеживает опасные зависимости. Если конкурентный результат нельзя считать эквивалентным последовательному выполнению, PostgreSQL прерывает одну из транзакций с serialization failure. Приложение должно повторить транзакцию.

---

# 🧩 Связь с MVCC

```python id="c7v5xn"
MVCC
 │
 ├── хранит версии строк
 │
 └── формирует snapshot
          │
          ▼
   REPEATABLE READ
          │
          ▼
      стабильный
       snapshot

SERIALIZABLE
      │
      ├── snapshot
      ├── MVCC
      └── контроль опасных зависимостей
                │
                ▼
       безопасный результат
                │
          ┌─────┴─────┐
          ▼           ▼
        COMMIT     ROLLBACK
                    +
                  RETRY
```

---

# 🎯 Когда использовать

SERIALIZABLE имеет смысл, когда **бизнес-правило сложно корректно обеспечить обычными блокировками или более слабой изоляцией**, а ошибка транзакции допустима и может быть обработана повтором.

Например:

* сложные финансовые операции;
* конкурирующие изменения ресурсов;
* операции с критическими инвариантами;
* ситуации, где потеря корректности недопустима.

Но для обычного CRUD:

```python id="m8q2vd"
READ COMMITTED
```

обычно достаточно.

---

# 🎤 Формулировка для собеседования

> **SERIALIZABLE — самый строгий уровень изоляции, при котором результат конкурентного выполнения транзакций должен быть эквивалентен некоторому последовательному выполнению. В PostgreSQL он реализован через Serializable Snapshot Isolation. Транзакции могут выполняться параллельно, но при обнаружении опасного конфликта PostgreSQL завершает одну из них с ошибкой serialization failure. Поэтому приложение должно уметь повторять всю транзакцию.**

---

## 📌 Формула

```python id="r5w8kc"
SERIALIZABLE
=
Concurrent execution
+
MVCC / Snapshot
+
Conflict detection
+
Serializable result
+
Possible serialization
```
