# 🟢 READ UNCOMMITTED

## 🎯 Ответ на собеседовании

**READ UNCOMMITTED** — самый слабый уровень изоляции транзакций. Теоретически он позволяет транзакции читать **незакоммиченные изменения** другой транзакции.

Главная проблема — **Dirty Read**: транзакция может увидеть данные, которые другая транзакция ещё не зафиксировала и впоследствии откатила.

Однако в **PostgreSQL** `READ UNCOMMITTED` фактически работает как `READ COMMITTED`, поэтому настоящего Dirty Read в PostgreSQL нет.

---

## 🎤 Суперкоротко

> **READ UNCOMMITTED** — самый низкий уровень изоляции, который в стандарте SQL допускает чтение незакоммиченных данных и, следовательно, Dirty Read. В PostgreSQL этот уровень фактически реализован как `READ COMMITTED`, поэтому Dirty Read невозможен.

---

# 📌 Что означает READ UNCOMMITTED

Название буквально означает:

```text
READ UNCOMMITTED
       ↓
читать даже незакоммиченные изменения
```

То есть одна транзакция потенциально может увидеть изменения другой транзакции **до `COMMIT`**.

---

# 💳 Пример Dirty Read

Начальное состояние:

```text
balance = 1000
```

### Транзакция A

```python
BEGIN;

UPDATE accounts
SET balance = 500
WHERE id = 1;
```

Изменение ещё не зафиксировано:

```text
balance = 500
```

### Транзакция B

При теоретическом `READ UNCOMMITTED`:

```python
BEGIN;

SELECT balance
FROM accounts
WHERE id = 1;
```

может увидеть:

```text
500
```

Но затем транзакция A выполняет:

```python
ROLLBACK;
```

И реальное состояние снова:

```text
balance = 1000
```

Получается:

```text
B прочитала 500
       ↓
A сделала ROLLBACK
       ↓
500 никогда не было зафиксировано
```

Это и есть:

> **Dirty Read — «грязное чтение».**

---

# 🧠 Почему Dirty Read опасен

Приложение может принять решение на основании данных, которых фактически никогда не существовало в зафиксированном состоянии базы.

Например:

```text
A:
баланс = 500  ← ещё не COMMIT

B:
читает баланс = 500
↓
разрешает операцию

A:
ROLLBACK
↓
баланс снова = 1000
```

Транзакция B работала с временным состоянием.

---

# 🐘 READ UNCOMMITTED в PostgreSQL

Это самый важный момент для собеседования.

В PostgreSQL можно написать:

```python
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
```

Но PostgreSQL **не предоставляет настоящего READ UNCOMMITTED**.

Фактически:

```text
READ UNCOMMITTED
        ↓
READ COMMITTED
```

Поэтому PostgreSQL не позволяет одной транзакции увидеть незакоммиченные изменения другой.

---

# 🔐 Почему PostgreSQL не допускает Dirty Read

PostgreSQL использует **MVCC**.

Упрощённо:

```text
Transaction A
     ↓
UPDATE
     ↓
новая версия строки
     ↓
ещё не COMMIT
     
Transaction B
     ↓
SELECT
     ↓
MVCC определяет видимость
     ↓
незакоммиченная версия A не видна
```

Поэтому B получает предыдущую допустимую версию данных, а не незакоммиченное изменение A.

---

# 🆚 READ UNCOMMITTED vs READ COMMITTED

|                     | READ UNCOMMITTED               | READ COMMITTED |
| ------------------- | ------------------------------ | -------------- |
| Уровень изоляции    | Самый слабый                   | Выше           |
| Dirty Read          | Теоретически возможен          | ❌              |
| Non-repeatable Read | Возможен                       | Возможен       |
| Phantom Read        | Возможен                       | Возможен       |
| PostgreSQL          | Реализуется как READ COMMITTED | ✅ По умолчанию |

---

# 📊 Где READ UNCOMMITTED вообще может быть полезен

Теоретически такой уровень может использоваться, когда:

* максимальная производительность важнее точности чтения;
* допустимы временно некорректные данные;
* приложение выполняет приблизительный анализ;
* нужна минимальная изоляция.

Но это серьёзный компромисс:

```text
меньше изоляции
      ↓
меньше гарантий
      ↓
больше потенциальных аномалий
```

На практике настоящий `READ UNCOMMITTED` используется редко.

---

# 🔗 READ UNCOMMITTED и другие аномалии

Упрощённо классическая модель выглядит так:

```text
READ UNCOMMITTED
      ↓
Dirty Read       ✅ возможно
Non-repeatable   ✅ возможно
Phantom Read     ✅ возможно
```

Следующий уровень:

```text
READ COMMITTED
      ↓
Dirty Read       ❌
Non-repeatable   ✅
Phantom Read     ✅
```

Далее:

```text
REPEATABLE READ
      ↓
Dirty Read       ❌
Non-repeatable   ❌
Phantom Read     зависит от СУБД;
                 в PostgreSQL не допускается
```

И:

```text
SERIALIZABLE
      ↓
максимальные гарантии
```

---

# 🧩 READ UNCOMMITTED и MVCC

Важно понимать:

**READ UNCOMMITTED — уровень изоляции.**

**MVCC — механизм управления версиями данных.**

Это разные понятия.

```text
Isolation Level
       ↓
определяет гарантии
       ↓
MVCC
       ↓
помогает реализовать эти гарантии
```

В PostgreSQL:

```text
READ UNCOMMITTED
       ↓
фактически READ COMMITTED
       ↓
MVCC
       ↓
Dirty Read невозможен
```

---

# ⚠️ Частая ошибка на собеседовании

Неправильно:

> «PostgreSQL поддерживает READ UNCOMMITTED и позволяет Dirty Read».

Правильно:

> «PostgreSQL принимает `READ UNCOMMITTED` как допустимое значение уровня изоляции, но фактически реализует его как `READ COMMITTED`, поэтому Dirty Read невозможен».

---

# 🎤 Частые вопросы на собеседовании

### Что такое READ UNCOMMITTED?

> Самый слабый уровень изоляции, который теоретически позволяет читать незакоммиченные изменения других транзакций.

### Что такое Dirty Read?

> Чтение транзакцией данных, которые другая транзакция изменила, но ещё не закоммитила.

### Есть ли настоящий READ UNCOMMITTED в PostgreSQL?

> Нет. PostgreSQL фактически реализует `READ UNCOMMITTED` как `READ COMMITTED`.

### Может ли PostgreSQL допустить Dirty Read?

> Нет, обычный механизм MVCC PostgreSQL не позволяет читать незакоммиченные изменения других транзакций.

### Как установить READ UNCOMMITTED?

```python
BEGIN;

SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;

SELECT * FROM users;

COMMIT;
```

Но в PostgreSQL это будет эквивалентно `READ COMMITTED`.

### Чем READ UNCOMMITTED отличается от READ COMMITTED?

> Главное отличие в стандартной модели — `READ UNCOMMITTED` допускает Dirty Read, а `READ COMMITTED` — нет. В PostgreSQL практической разницы между ними нет, потому что `READ UNCOMMITTED` фактически реализован как `READ COMMITTED`.

---

# 🧭 Ментальная модель

```text
              READ UNCOMMITTED
                     ↓
          "можно читать даже
       незакоммиченные изменения"
                     ↓
                Dirty Read
                     ↓
          ❌ опасно для корректности


             PostgreSQL
                     ↓
        READ UNCOMMITTED
                     ↓
          READ COMMITTED
                     ↓
          Dirty Read невозможен
```

---

## 🔑 Главное

```text
READ UNCOMMITTED
= самый слабый уровень изоляции.

В классической модели:
→ допускает Dirty Read;
→ допускает Non-repeatable Read;
→ допускает Phantom Read.

Dirty Read:
→ прочитали незакоммиченное изменение;
→ другая транзакция сделала ROLLBACK;
→ прочитанные данные фактически не были зафиксированы.

В PostgreSQL:

READ UNCOMMITTED
        ↓
фактически READ COMMITTED

Поэтому:
→ настоящего Dirty Read нет.

Главная фраза:

"READ UNCOMMITTED — самый слабый уровень изоляции,
который в стандартной модели допускает чтение
незакоммиченных данных. В PostgreSQL он фактически
эквивалентен READ COMMITTED, поэтому Dirty Read
невозможен."
```
