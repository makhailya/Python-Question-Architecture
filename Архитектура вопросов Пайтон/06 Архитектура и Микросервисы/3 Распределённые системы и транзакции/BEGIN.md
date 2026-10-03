# BEGIN в SQL 🟢

## 🎯 Ответ на собеседовании

`BEGIN` начинает **транзакцию** в базе данных.

После `BEGIN` несколько SQL-операций выполняются в рамках одной транзакции и могут быть либо все зафиксированы через `COMMIT`, либо отменены через `ROLLBACK`.

```python
BEGIN;

UPDATE accounts
SET balance = balance - 100
WHERE id = 1;

UPDATE accounts
SET balance = balance + 100
WHERE id = 2;

COMMIT;
```

Если одна из операций завершилась ошибкой:

```python
BEGIN;

UPDATE ...;
UPDATE ...;

ROLLBACK;
```

Все изменения текущей транзакции будут отменены.

---

## 🎤 Суперкоротко

> `BEGIN` начинает транзакцию. Все последующие операции выполняются в рамках этой транзакции до `COMMIT` или `ROLLBACK`.

---

# 1. Зачем нужен `BEGIN`?

Представим перевод денег:

```text
Счёт A: 1000 ₽
Счёт B: 500 ₽
```

Нужно перевести 200 ₽.

Операция состоит из двух изменений:

```text
1. Списать 200 ₽ с A
2. Зачислить 200 ₽ на B
```

Нужно объединить их в одну транзакцию:

```python
BEGIN;

UPDATE accounts
SET balance = balance - 200
WHERE id = 1;

UPDATE accounts
SET balance = balance + 200
WHERE id = 2;

COMMIT;
```

Теперь база воспринимает эти изменения как одну логическую операцию.

---

# 2. Что происходит после `BEGIN`?

После:

```python
BEGIN;
```

открывается транзакция.

Далее:

```text
BEGIN
  ↓
SQL
  ↓
SQL
  ↓
SQL
  ↓
COMMIT / ROLLBACK
```

Изменения не становятся окончательно зафиксированными до `COMMIT`.

---

# 3. Пример

```python
BEGIN;

INSERT INTO users (name)
VALUES ('Ilya');

UPDATE statistics
SET users_count = users_count + 1;

COMMIT;
```

Здесь две операции:

```text
INSERT
UPDATE
```

относятся к одной транзакции.

Если выполнить:

```python
ROLLBACK;
```

обе операции будут отменены.

---

# 4. `BEGIN` и `COMMIT`

Полный жизненный цикл:

```text
BEGIN
  ↓
начало транзакции
  ↓
INSERT
  ↓
UPDATE
  ↓
DELETE
  ↓
COMMIT
  ↓
изменения зафиксированы
```

`BEGIN` отвечает за **начало**, а `COMMIT` — за **фиксацию**.

---

# 5. `BEGIN` и `ROLLBACK`

При ошибке:

```text
BEGIN
  ↓
UPDATE A
  ↓
UPDATE B
  ↓
ошибка
  ↓
ROLLBACK
```

Получаем:

```text
изменения после BEGIN
        ↓
      отменены
```

Это позволяет не оставить базу в промежуточном состоянии.

---

# 6. Что происходит без `BEGIN`?

Зависит от настроек и режима работы драйвера.

В PostgreSQL часто используется транзакционный режим, где отдельные SQL-команды могут выполняться в отдельных транзакциях, если приложение не объединило их явно.

Например, концептуально:

```text
UPDATE A → COMMIT

UPDATE B → COMMIT
```

Если второе обновление завершится ошибкой, первое уже может остаться сохранённым.

При явной транзакции:

```text
BEGIN

UPDATE A
UPDATE B

COMMIT
```

обе операции объединяются.

---

# 7. `BEGIN` в PostgreSQL

Стандартный вариант:

```python
BEGIN;
```

Также можно встретить:

```python
START TRANSACTION;
```

В PostgreSQL это альтернативный способ начать транзакцию.

---

# 8. `BEGIN` не сохраняет изменения

Это важный момент.

`BEGIN` означает:

> «Начать транзакцию».

Он **не означает**:

> «Сохранить изменения».

Сохранение выполняется:

```python
COMMIT;
```

А отмена:

```python
ROLLBACK;
```

Поэтому:

```text
BEGIN
  ↓
изменения
  ↓
COMMIT
```

или:

```text
BEGIN
  ↓
изменения
  ↓
ROLLBACK
```

---

# 9. `BEGIN` и ACID

`BEGIN` сам по себе не является отдельным свойством ACID.

Он обозначает начало транзакционного блока, внутри которого СУБД обеспечивает транзакционную семантику.

```text
BEGIN
 ↓
Transaction
 ├── Atomicity
 ├── Consistency
 ├── Isolation
 └── Durability
 ↓
COMMIT / ROLLBACK
```

---

# 10. `BEGIN` в Python

Например, при работе с PostgreSQL через DB-API драйвер:

```python
connection.begin()
```

или непосредственно SQL:

```python
cursor.execute("BEGIN")
```

Однако конкретный способ зависит от библиотеки и её управления транзакциями.

Например, приложение может использовать контекстный менеджер:

```python
with connection:
    cursor.execute(
        "UPDATE accounts SET balance = balance - 100 WHERE id = 1"
    )

    cursor.execute(
        "UPDATE accounts SET balance = balance + 100 WHERE id = 2"
    )
```

Конкретная семантика зависит от используемого драйвера.

---

# 11. `BEGIN` в Django

В Django обычно не пишут SQL:

```python
BEGIN;
```

Вместо этого используют:

```python
from django.db import transaction


with transaction.atomic():
    account_a.balance -= 100
    account_a.save()

    account_b.balance += 100
    account_b.save()
```

`transaction.atomic()` используется для управления транзакционным блоком.

Концептуально:

```text
atomic()
   ↓
BEGIN
   ↓
SQL operations
   ↓
COMMIT
```

При исключении:

```text
atomic()
   ↓
BEGIN
   ↓
SQL
   ↓
Exception
   ↓
ROLLBACK
```

---

# 12. `BEGIN` и вложенные транзакции

Важно: в большинстве реляционных СУБД настоящая вложенная транзакция — не то же самое, что просто написать:

```python
BEGIN;

BEGIN;
```

Для частичного отката обычно используют **SAVEPOINT**:

```python
BEGIN;

UPDATE accounts
SET balance = balance - 100
WHERE id = 1;

SAVEPOINT point1;

UPDATE accounts
SET balance = balance + 100
WHERE id = 2;

ROLLBACK TO SAVEPOINT point1;

COMMIT;
```

Схема:

```text
BEGIN
 ↓
операция A
 ↓
SAVEPOINT
 ↓
операция B
 ↓
ROLLBACK TO SAVEPOINT
 ↓
операция A остаётся
 ↓
COMMIT
```

---

# 13. Важное отличие

Не путать:

```text
BEGIN
```

и:

```text
SAVEPOINT
```

`BEGIN`:

> начинает всю транзакцию.

`SAVEPOINT`:

> создаёт точку внутри уже существующей транзакции, к которой можно откатиться.

---

# 14. Типичный вопрос на собеседовании

### ❓ Что делает `BEGIN`?

> `BEGIN` начинает транзакцию. Последующие операции выполняются в рамках этой транзакции до её завершения через `COMMIT` или `ROLLBACK`.

### ❓ Когда изменения становятся постоянными?

> После `COMMIT`.

### ❓ Что произойдёт после `BEGIN`, если запрос завершится ошибкой?

> Транзакция может перейти в состояние ошибки, и в PostgreSQL обычно необходимо выполнить `ROLLBACK`, прежде чем продолжать работу с этой транзакцией.

### ❓ Можно ли отменить изменения после `BEGIN`?

> Да, если они ещё не были зафиксированы, можно выполнить `ROLLBACK`.

---

# 15. Схема

```text
              BEGIN
                │
                ▼
         ┌──────────────┐
         │ TRANSACTION  │
         └──────┬───────┘
                │
       ┌────────┴────────┐
       ▼                 ▼
   операции           ошибка
       │                 │
       ▼                 ▼
    COMMIT            ROLLBACK
       │                 │
       ▼                 ▼
  сохранить           отменить
  изменения           изменения
```

---

# 16. Главное

```text
BEGIN
 ↓
начать транзакцию

COMMIT
 ↓
зафиксировать транзакцию

ROLLBACK
 ↓
отменить транзакцию
```

Главная связка:

```text
BEGIN → SQL operations → COMMIT
```

или при ошибке:

```text
BEGIN → SQL operations → ROLLBACK
```

> **`BEGIN` — это точка начала транзакции, а не команда сохранения данных.**
