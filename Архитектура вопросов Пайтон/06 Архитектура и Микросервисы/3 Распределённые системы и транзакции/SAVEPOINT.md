# 🎯 SAVEPOINT

## 🎯 Ответ на собеседовании

**SAVEPOINT** — это точка внутри транзакции, к которой можно вернуться, не отменяя всю транзакцию.

Он позволяет выполнить **частичный ROLLBACK**.

```text
BEGIN
  ↓
Operation A
  ↓
SAVEPOINT sp1
  ↓
Operation B
  ↓
Operation C
  ↓
ROLLBACK TO sp1
  ↓
B и C отменены
  ↓
A остаётся
  ↓
COMMIT
```

> **SAVEPOINT позволяет частично откатить транзакцию и продолжить её выполнение.**

---

## 🎤 Суперкоротко

> **SAVEPOINT — контрольная точка внутри транзакции, к которой можно откатиться через `ROLLBACK TO SAVEPOINT`.**

```text
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
A ✅
  ↓
COMMIT
```

---

# 🔹 Зачем нужен SAVEPOINT

Без `SAVEPOINT`:

```text
BEGIN
  ↓
A
  ↓
B
  ↓
C ❌
  ↓
ROLLBACK
```

Отменяется вся транзакция:

```text
A ❌
B ❌
C ❌
```

С `SAVEPOINT`:

```text
BEGIN
  ↓
A
  ↓
SAVEPOINT
  ↓
B
  ↓
C ❌
  ↓
ROLLBACK TO SAVEPOINT
```

Получаем:

```text
A ✅
B ❌
C ❌
```

И транзакцию можно продолжить.

---

# 🔹 Основные команды

Создать точку:

```python
SAVEPOINT payment_step;
```

Откатиться к ней:

```python
ROLLBACK TO SAVEPOINT payment_step;
```

Удалить точку:

```python
RELEASE SAVEPOINT payment_step;
```

---

# 🧩 Полный пример

```python
BEGIN;

INSERT INTO orders (id, status)
VALUES (100, 'created');

SAVEPOINT before_payment;

INSERT INTO payments (order_id, amount)
VALUES (100, 1000);

ROLLBACK TO SAVEPOINT before_payment;

UPDATE orders
SET status = 'payment_failed'
WHERE id = 100;

COMMIT;
```

Что произошло:

```text
Создали Order
      ↓
SAVEPOINT
      ↓
Попытались создать Payment
      ↓
ROLLBACK TO SAVEPOINT
      ↓
Payment отменён
      ↓
изменили Order
      ↓
COMMIT
```

В итоге:

```text
Order → payment_failed
Payment → изменения отменены
```

---

# 🔄 SAVEPOINT vs ROLLBACK

|                            | `ROLLBACK`     | `ROLLBACK TO SAVEPOINT` |
| -------------------------- | -------------- | ----------------------- |
| Что отменяет               | Всю транзакцию | Часть транзакции        |
| Транзакция продолжается    | ❌              | ✅                       |
| Использует SAVEPOINT       | Нет            | Да                      |
| Можно сделать COMMIT после | Нет            | Да                      |

```text
ROLLBACK
    ↓
Transaction завершена
```

```text
ROLLBACK TO SAVEPOINT
    ↓
Transaction продолжается
```

---

# 🔹 Несколько SAVEPOINT

Можно создавать несколько точек:

```python
BEGIN;

Operation A;

SAVEPOINT sp1;

Operation B;

SAVEPOINT sp2;

Operation C;

ROLLBACK TO SAVEPOINT sp2;

Operation D;

COMMIT;
```

Схематично:

```text
BEGIN
  ↓
A
  ↓
SP1
  ↓
B
  ↓
SP2
  ↓
C
  ↓
ROLLBACK TO SP2
  ↓
C ❌
  ↓
D
  ↓
COMMIT
```

`A`, `B` и `D` остаются частью успешно завершённой транзакции.

---

# ⚠️ SAVEPOINT не фиксирует изменения

Очень важный момент.

```python
SAVEPOINT sp1;
```

**не является `COMMIT`.**

После:

```python
ROLLBACK TO SAVEPOINT sp1;
```

можно продолжить работу.

Но если затем выполнить:

```python
ROLLBACK;
```

будет отменена **вся транзакция**, включая изменения, сделанные до `SAVEPOINT`.

---

# 🔒 SAVEPOINT и ACID

`SAVEPOINT` помогает управлять **Atomicity** внутри одной транзакции.

Обычная атомарность:

```text
вся транзакция
      ↓
COMMIT или ROLLBACK
```

С `SAVEPOINT` появляется дополнительный уровень:

```text
транзакция
   ↓
частичные точки отката
   ↓
продолжение транзакции
   ↓
финальный COMMIT
```

---

# 🐍 SAVEPOINT в Django

Django использует savepoints внутри некоторых вложенных `transaction.atomic()`.

Например:

```python
from django.db import transaction

with transaction.atomic():
    create_order()

    try:
        with transaction.atomic():
            create_payment()
    except Exception:
        pass

    update_order()
```

Концептуально:

```text
Outer transaction
      ↓
create_order()
      ↓
SAVEPOINT
      ↓
create_payment()
      ↓
ошибка
      ↓
ROLLBACK TO SAVEPOINT
      ↓
update_order()
      ↓
COMMIT
```

То есть внутренняя `atomic()` может использовать savepoint, позволяя откатить внутреннюю часть, не откатывая внешнюю транзакцию.

---

# 💾 SAVEPOINT и PostgreSQL

PostgreSQL поддерживает стандартные команды:

```python
SAVEPOINT sp1;
ROLLBACK TO SAVEPOINT sp1;
RELEASE SAVEPOINT sp1;
```

Например:

```python
BEGIN;

INSERT INTO users (name)
VALUES ('Ilya');

SAVEPOINT user_step;

INSERT INTO users (name)
VALUES ('Alex');

ROLLBACK TO SAVEPOINT user_step;

COMMIT;
```

После `COMMIT`:

```text
Ilya → сохранён
Alex → отменён
```

---

# ⚠️ SAVEPOINT vs отдельная транзакция

SAVEPOINT **не создаёт новую независимую транзакцию**.

```text
Transaction
   │
   ├── Operation A
   ├── SAVEPOINT
   ├── Operation B
   └── Operation C
```

Все операции остаются частью одной транзакции.

Поэтому:

```text
ROLLBACK
```

отменит всё.

А:

```text
ROLLBACK TO SAVEPOINT
```

отменит только изменения после точки.

---

# 🔗 SAVEPOINT и Race Condition

SAVEPOINT не является механизмом синхронизации.

Он не заменяет:

```text
UNIQUE
LOCK
SELECT FOR UPDATE
Isolation Level
```

Его задача другая:

```text
SAVEPOINT
   ↓
частичный откат
```

а не:

```text
SAVEPOINT
   ↓
защита от конкурентного доступа
```

---

# 🔗 SAVEPOINT и Retry

Можно использовать savepoint, если отдельная часть транзакции может завершиться ошибкой:

```text
BEGIN
  ↓
основная операция
  ↓
SAVEPOINT
  ↓
дополнительная операция
  ↓
ошибка
  ↓
ROLLBACK TO SAVEPOINT
  ↓
продолжить
  ↓
COMMIT
```

Но для полноценного retry транзакции часто проще откатить всю транзакцию и начать её заново.

---

# 🧠 COMMIT → ROLLBACK → SAVEPOINT

Полезно держать эти три понятия вместе:

```text
BEGIN
  ↓
SQL operations
  ↓
┌─────────────────────────┐
│ SAVEPOINT               │
│ ↓                       │
│ частичный ROLLBACK      │
│ ↓                       │
│ продолжение транзакции  │
└─────────────────────────┘
  ↓
COMMIT
```

Или при критической ошибке:

```text
BEGIN
  ↓
операции
  ↓
ROLLBACK
  ↓
вся транзакция отменена
```

---

# 🧠 Главное

```text
SAVEPOINT
    ↓
точка внутри транзакции
    ↓
ROLLBACK TO SAVEPOINT
    ↓
отменить часть изменений
    ↓
продолжить транзакцию
    ↓
COMMIT
```

### Формула для собеседования

> **SAVEPOINT — это точка сохранения внутри транзакции. С помощью `ROLLBACK TO SAVEPOINT` можно отменить изменения, выполненные после этой точки, и продолжить транзакцию. В отличие от обычного `ROLLBACK`, транзакция при этом не завершается.**

### Три команды

```python
SAVEPOINT sp1;

ROLLBACK TO SAVEPOINT sp1;

RELEASE SAVEPOINT sp1;
```
