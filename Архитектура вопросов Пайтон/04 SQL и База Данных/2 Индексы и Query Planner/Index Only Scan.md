# Index Only Scan в PostgreSQL 🔎

## 🎯 Ответ на собеседовании

**Index Only Scan** — способ выполнения запроса, при котором PostgreSQL получает необходимые для результата данные **из индекса**, не читая соответствующие строки heap-таблицы в обычном режиме.

Это возможно, когда индекс содержит все необходимые столбцы запроса и PostgreSQL может проверить **видимость строк** через visibility map.

Главное преимущество — меньше обращений к heap, поэтому такой план может быть быстрее обычного `Index Scan`.

---

## 🎤 Суперкоротко

```python
Index Scan:

Index
  ↓
Heap
  ↓
Result
```

```python
Index Only Scan:

Index
  ↓
Result
```

Но важная оговорка:

> **Index Only Scan не гарантирует, что heap вообще не будет прочитан.** Если PostgreSQL не может определить видимость страницы через visibility map, он может выполнить дополнительные обращения к heap.

---

# 1. Простой пример

Есть таблица:

```python
CREATE TABLE users (
    id BIGSERIAL PRIMARY KEY,
    email TEXT,
    name TEXT
);
```

Создадим индекс:

```python
CREATE INDEX idx_users_email
ON users(email);
```

Запрос:

```python
EXPLAIN ANALYZE
SELECT email
FROM users
WHERE email = 'test@example.com';
```

PostgreSQL может выбрать:

```python
Index Only Scan using idx_users_email on users
  Index Cond: (email = 'test@example.com')
```

Почему?

Потому что запросу нужны только:

```python
email
```

а `email` уже находится в индексе.

---

# 2. Сравнение с Index Scan

### Index Scan

Запрос:

```python
SELECT *
FROM users
WHERE email = 'test@example.com';
```

Индекс содержит:

```python
email
```

Но запросу нужны:

```python
id
email
name
```

Поэтому PostgreSQL обычно должен обратиться к heap:

```python
Index
  ↓
найти строку
  ↓
Heap
  ↓
получить id, email, name
```

---

### Index Only Scan

Запрос:

```python
SELECT email
FROM users
WHERE email = 'test@example.com';
```

Индекс уже содержит `email`.

Поэтому потенциально:

```python
Index
  ↓
email
  ↓
Result
```

---

# 3. Почему он называется `Index Only`

Название буквально означает:

> PostgreSQL может получить необходимые данные **только из индекса**.

Но это относится именно к **данным**, необходимым запросу.

Есть ещё вопрос:

```python
Видима ли эта строка текущей транзакции?
```

Для этого PostgreSQL использует информацию о видимости.

---

# 4. Visibility Map

PostgreSQL хранит специальную **visibility map**.

Она содержит информацию о страницах heap-таблицы.

В частности, PostgreSQL может знать, что страница является **all-visible** — все строки на ней видимы всем текущим транзакциям, которым они должны быть видимы.

Тогда PostgreSQL может доверять информации индекса и не идти в heap для проверки видимости каждой строки.

Упрощённо:

```python
Index
  ↓
найти запись
  ↓
Visibility Map
  ↓
страница all-visible?
  ↓
да
  ↓
не нужно читать heap
```

---

# 5. Что происходит, если страница не all-visible

Например:

```python
Index Only Scan
```

но нужная heap-страница не отмечена как `all-visible`.

Тогда PostgreSQL может сделать дополнительное обращение к heap:

```python
Index
  ↓
найти запись
  ↓
Visibility Map
  ↓
не all-visible
  ↓
Heap
  ↓
проверить видимость
```

Поэтому:

> **Index Only Scan может обращаться к heap.**

Именно поэтому название не следует понимать как абсолютное «таблица никогда не читается».

---

# 6. `Heap Fetches`

В `EXPLAIN ANALYZE` для Index Only Scan можно увидеть:

```python
Heap Fetches: 0
```

Это очень хороший показатель.

Он означает, что для данного выполнения PostgreSQL не потребовалось получать строки из heap.

Например:

```python
Index Only Scan using idx_users_email on users
  Index Cond: (email = 'test@example.com')
  Heap Fetches: 0
```

---

Если:

```python
Heap Fetches: 5000
```

значит PostgreSQL был вынужден выполнить обращения к heap для проверки видимости.

---

# 7. Пример

```python
EXPLAIN (ANALYZE, BUFFERS)
SELECT email
FROM users
WHERE email = 'test@example.com';
```

Условный результат:

```python
Index Only Scan using idx_users_email on users
  (cost=0.42..8.44 rows=1 width=32)
  (actual time=0.030..0.035 rows=1 loops=1)
  Index Cond: (email = 'test@example.com')
  Heap Fetches: 0
```

Разбираем:

```python
Index Only Scan
```

→ PostgreSQL использовал только индекс для получения необходимых данных.

```python
Index Cond
```

→ индекс используется для поиска.

```python
actual rows=1
```

→ реально получена одна строка.

```python
Heap Fetches: 0
```

→ обращаться к heap не пришлось.

---

# 8. Почему Index Only Scan может быть быстрее

При обычном:

```python
Index Scan
```

получается:

```python
Index
 ↓
Heap
```

То есть есть дополнительные обращения к таблице.

При:

```python
Index Only Scan
```

может быть:

```python
Index
 ↓
Result
```

Меньше обращений к heap:

```python
Index Only Scan
    ↓
меньше random I/O
    ↓
меньше обращений к таблице
    ↓
потенциально быстрее
```

Особенно это может быть заметно на больших таблицах.

---

# 9. Покрывающий индекс

Чтобы Index Only Scan был возможен, индекс должен содержать все данные, необходимые запросу.

Такой индекс часто называют **covering index** — покрывающий индекс.

Например:

```python
CREATE INDEX idx_users_email_name
ON users(email, name);
```

Запрос:

```python
SELECT email, name
FROM users
WHERE email = 'test@example.com';
```

Индекс содержит:

```python
email
name
```

Поэтому PostgreSQL потенциально может выполнить:

```python
Index Only Scan
```

---

# 10. `INCLUDE`

В PostgreSQL для покрывающих индексов можно использовать `INCLUDE`.

Например:

```python
CREATE INDEX idx_users_email
ON users(email)
INCLUDE (name);
```

Здесь:

```python
email
```

является ключом индекса, а:

```python
name
```

добавлен как дополнительное значение.

Запрос:

```python
SELECT email, name
FROM users
WHERE email = 'test@example.com';
```

может использовать:

```python
Index Only Scan
```

---

# 11. Почему не стоит добавлять всё в индекс

Можно подумать:

> «Давайте добавим все столбцы в индекс, чтобы всегда получать Index Only Scan».

Это плохая идея.

Большие индексы:

* занимают больше дискового пространства;
* увеличивают стоимость `INSERT`;
* увеличивают стоимость `UPDATE`;
* увеличивают стоимость `DELETE`;
* требуют больше памяти;
* могут ухудшать производительность записи.

Индекс должен создаваться под реальные запросы.

---

# 12. Index Only Scan и SELECT *

Очень важный момент.

Допустим:

```python
CREATE INDEX idx_users_email
ON users(email);
```

Запрос:

```python
SELECT *
FROM users
WHERE email = 'test@example.com';
```

Индекс содержит только:

```python
email
```

а `SELECT *` требует:

```python
id
email
name
created_at
...
```

Поэтому такой индекс обычно не позволяет получить весь результат только из индекса.

Чаще будет:

```python
Index Scan
```

а не:

```python
Index Only Scan
```

---

# 13. Index Only Scan и COUNT

Это один из интересных практических случаев.

Например:

```python
SELECT COUNT(*)
FROM users;
```

PostgreSQL в некоторых ситуациях может использовать индекс или другой план, если это выгодно.

При наличии подходящего индекса возможен:

```python
Index Only Scan
```

поскольку для `COUNT(*)` не обязательно получать значения всех столбцов.

Но конкретный план зависит от размера таблицы, индексов, статистики и других факторов.

---

# 14. Index Only Scan и обновления

Если таблица активно изменяется:

```python
UPDATE
INSERT
DELETE
```

visibility map может чаще требовать обращения к heap.

Поэтому на сильно изменяемой таблице:

```python
Heap Fetches
```

может быть заметно больше.

После `VACUUM` страницы могут снова получить состояние `all-visible`, когда это допустимо.

Поэтому обслуживание таблицы влияет на эффективность Index Only Scan.

---

# 15. Index Only Scan vs Index Scan

|                                         | Index Scan                   | Index Only Scan   |
| --------------------------------------- | ---------------------------- | ----------------- |
| Использует индекс                       | ✅                            | ✅                 |
| Может читать heap                       | ✅                            | ⚠️ Да             |
| Может получить данные только из индекса | ❌                            | ✅                 |
| Требует покрывающий индекс              | Нет                          | Обычно да         |
| Использует visibility map               | Не для основной идеи доступа | ✅                 |
| `Heap Fetches`                          | Не основной показатель       | Важный показатель |
| Потенциально меньше I/O                 | ❌                            | ✅                 |

---

# 16. Index Only Scan vs Seq Scan

```python
Seq Scan
```

читает таблицу:

```python
Table
 ↓
Pages
 ↓
Rows
```

А:

```python
Index Only Scan
```

может читать только индекс:

```python
Index
 ↓
Result
```

Но это не означает, что Index Only Scan всегда быстрее.

Например, если запрос возвращает огромную часть таблицы, последовательное чтение может быть выгоднее.

Planner выбирает план на основе стоимости.

---

# 17. Как проверить Index Only Scan

Используем:

```python
EXPLAIN (ANALYZE, BUFFERS)
SELECT email
FROM users
WHERE email = 'test@example.com';
```

Ищем:

```python
Index Only Scan
```

и:

```python
Heap Fetches
```

Например:

```python
Index Only Scan using idx_users_email on users
  Index Cond: (email = 'test@example.com')
  Heap Fetches: 0
```

Это хороший сценарий.

---

# 18. Типичная схема

### Index Scan

```python
             Index
               ↓
        найти TID строк
               ↓
             Heap
               ↓
          получить данные
               ↓
            Result
```

### Index Only Scan

```python
             Index
               ↓
       получить данные
               ↓
     проверить visibility
               ↓
            Result
```

Если visibility map не позволяет доверять странице:

```python
             Index
               ↓
       найти необходимые данные
               ↓
       Visibility Map
               ↓
        не all-visible
               ↓
             Heap
               ↓
       проверить видимость
```

---

# 19. Как объяснить на собеседовании

Вопрос:

> **«Что такое Index Only Scan?»**

Хороший ответ:

> «Index Only Scan — это план PostgreSQL, при котором все необходимые для запроса данные находятся в индексе, поэтому PostgreSQL может избежать обычного чтения heap-таблицы. Для проверки видимости строк используется visibility map. Если соответствующие страницы отмечены как all-visible, heap fetches могут быть равны нулю. Это потенциально уменьшает I/O и ускоряет запрос.»

---

# 20. Типичная ошибка на собеседовании

❌ Неправильно:

> «Index Only Scan вообще никогда не обращается к таблице.»

Правильно:

> «Index Only Scan позволяет получить необходимые данные из индекса, но PostgreSQL всё ещё может обращаться к heap для проверки видимости, если visibility map не позволяет этого избежать.»

---

# 21. Связь с другими типами сканирования

```python
Seq Scan
    ↓
Table → Rows
```

```python
Index Scan
    ↓
Index → Heap → Rows
```

```python
Index Only Scan
    ↓
Index → Rows
```

```python
Bitmap Scan
    ↓
Index
  ↓
Bitmap
  ↓
Heap Pages
  ↓
Rows
```

---

## 🧠 Главное

```python
Index Only Scan
        ↓
используем индекс
        ↓
все необходимые данные есть в индексе
        ↓
проверяем visibility
        ↓
можем не обращаться к heap
```

Ключевые понятия:

* **Covering Index** — индекс содержит необходимые запросу данные.
* **Visibility Map** — помогает определить, можно ли не читать heap для проверки видимости.
* **Heap Fetches** — показывает количество обращений к heap при Index Only Scan.
* `Heap Fetches: 0` — хороший сценарий.
* `INCLUDE` — способ добавить дополнительные данные в индекс для покрытия запросов.

### Три основных типа рядом:

```python
Seq Scan
→ Table

Index Scan
→ Index → Heap

Index Only Scan
→ Index → Result
```

> **Index Only Scan — это оптимизация доступа, при которой PostgreSQL старается выполнить запрос, используя только индекс, что позволяет сократить обращения к heap и потенциально уменьшить I/O.**
