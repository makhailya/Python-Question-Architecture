# Bitmap Heap Scan в PostgreSQL 🔎

## 🎯 Ответ на собеседовании

**Bitmap Heap Scan** — оператор PostgreSQL, который использует bitmap, построенный предыдущим `Bitmap Index Scan`, чтобы определить, **какие страницы heap-таблицы нужно прочитать**, а затем извлекает из них подходящие строки.

Типичная схема:

```python
Bitmap Index Scan
        ↓
      Bitmap
        ↓
Bitmap Heap Scan
        ↓
      Heap
        ↓
     Result
```

Главная идея — сначала собрать информацию о множестве совпадений, а затем эффективно прочитать нужные страницы таблицы.

---

## 🎤 Суперкоротко

```python
Bitmap Index Scan
    → ищет совпадения в индексе
    → строит bitmap

Bitmap Heap Scan
    → получает bitmap
    → читает нужные страницы таблицы
    → проверяет строки
    → возвращает результат
```

> **Bitmap Index Scan ищет через индекс, а Bitmap Heap Scan получает реальные строки из heap.**

---

# 1. Простой пример

Есть индекс:

```python
CREATE INDEX idx_users_age
ON users(age);
```

Запрос:

```python
EXPLAIN ANALYZE
SELECT *
FROM users
WHERE age BETWEEN 20 AND 30;
```

PostgreSQL может построить такой план:

```python
Bitmap Heap Scan on users
  Recheck Cond: ((age >= 20) AND (age <= 30))
  -> Bitmap Index Scan on idx_users_age
       Index Cond: ((age >= 20) AND (age <= 30))
```

Читаем снизу вверх:

```python
Bitmap Index Scan
        ↓
найти подходящие записи
        ↓
создать bitmap
        ↓
Bitmap Heap Scan
        ↓
прочитать нужные страницы heap
        ↓
проверить строки
        ↓
Result
```

---

# 2. Что такое Heap

**Heap** в PostgreSQL — основное физическое хранение строк таблицы.

Упрощённо:

```python
users
├── Page 1
├── Page 2
├── Page 3
├── Page 4
└── ...
```

Каждая страница содержит несколько строк.

`Bitmap Heap Scan` работает именно с этими страницами.

---

# 3. Что передаёт Bitmap Index Scan

Допустим, индекс нашёл подходящие строки:

```python
Row A → Page 10
Row B → Page 10
Row C → Page 25
Row D → Page 25
Row E → Page 100
```

Bitmap позволяет представить это примерно так:

```python
Page 10  → нужны строки
Page 25  → нужны строки
Page 100 → нужна строка
Page 200 → не нужна
Page 201 → не нужна
```

После этого:

```python
Bitmap Heap Scan
```

читает нужные страницы heap.

---

# 4. Почему не сделать обычный Index Scan?

При `Index Scan` можно получить последовательность обращений:

```python
Index
 ↓
Heap Page 10
 ↓
Heap Page 25
 ↓
Heap Page 10
 ↓
Heap Page 100
 ↓
Heap Page 25
 ↓
...
```

При большом количестве найденных строк это может означать много обращений к разным страницам.

Bitmap-подход сначала собирает информацию:

```python
Index
 ↓
Bitmap
 ↓
Pages: 10, 25, 100
 ↓
прочитать страницы
```

Это позволяет эффективнее организовать чтение heap.

---

# 5. Bitmap Heap Scan работает со страницами

Очень важно:

> `Bitmap Heap Scan` не просто получает список строк и идёт за каждой строкой отдельно.

Он использует bitmap для определения **страниц heap**, которые нужно прочитать.

Упрощённо:

```python
Bitmap
   ↓
Page 10 ───┐
Page 25 ───┼→ Heap
Page 100 ──┘
```

На каждой странице PostgreSQL затем проверяет нужные строки.

---

# 6. `Recheck Cond`

В плане часто встречается:

```python
Bitmap Heap Scan on users
  Recheck Cond: (age > 30)
```

`Recheck Cond` означает, что PostgreSQL повторно проверяет условие при чтении строк из heap.

Почему это необходимо?

Потому что bitmap в некоторых случаях может быть **lossy**.

---

# 7. Lossy Bitmap

Bitmap может хранить информацию с разной точностью.

### Точный вариант

Условно:

```python
Page 10:
    Row 2  → подходит
    Row 5  → подходит
    Row 8  → подходит
```

### Lossy-вариант

```python
Page 10:
    возможно есть подходящие строки
```

То есть PostgreSQL знает:

> «На этой странице есть потенциальные совпадения».

Но не знает точно, какие строки подходят.

Поэтому:

```python
Bitmap Heap Scan
        ↓
прочитать страницу
        ↓
Recheck Cond
        ↓
проверить строки
```

---

# 8. `Rows Removed by Index Recheck`

В плане можно увидеть:

```python
Rows Removed by Index Recheck: 1000
```

Это означает, что после повторной проверки некоторые строки не соответствовали условию.

Упрощённо:

```python
Bitmap
 ↓
кандидатные строки
 ↓
Heap
 ↓
Recheck Cond
 ↓
часть строк отброшена
 ↓
Result
```

Это особенно характерно для lossy bitmap.

---

# 9. Bitmap Heap Scan + Bitmap Index Scan

Эти два оператора обычно рассматриваются вместе.

```python
Bitmap Heap Scan
    ↓
    └── Bitmap Index Scan
```

Например:

```python
Bitmap Heap Scan on orders
  Recheck Cond: (user_id = 100)
  -> Bitmap Index Scan on idx_orders_user_id
       Index Cond: (user_id = 100)
```

### Bitmap Index Scan

Отвечает на вопрос:

> Где находятся подходящие данные?

### Bitmap Heap Scan

Отвечает на вопрос:

> Как получить реальные строки из таблицы?

---

# 10. Сравнение с Index Scan

### Index Scan

```python
Index
  ↓
найти строку
  ↓
Heap
  ↓
получить строку

Index
  ↓
найти следующую строку
  ↓
Heap
```

### Bitmap Heap Scan

```python
Index
  ↓
Bitmap
  ↓
собрать нужные страницы
  ↓
Heap
  ↓
прочитать страницы
  ↓
проверить строки
```

Поэтому Bitmap Heap Scan особенно полезен, когда совпадений **достаточно много**, но читать всю таблицу ещё невыгодно.

---

# 11. Сравнение трёх вариантов

| План               | Схема                        | Типичный сценарий          |
| ------------------ | ---------------------------- | -------------------------- |
| `Seq Scan`         | Table → Rows                 | большая часть таблицы      |
| `Index Scan`       | Index → Heap → Rows          | небольшое количество строк |
| `Bitmap Heap Scan` | Index → Bitmap → Heap → Rows | заметное количество строк  |

Но это только общая логика.

Фактический выбор всегда делает PostgreSQL Planner.

---

# 12. Bitmap Heap Scan может использовать несколько индексов

Например:

```python
SELECT *
FROM users
WHERE age = 30
  AND city = 'Moscow';
```

Есть:

```python
CREATE INDEX idx_users_age
ON users(age);

CREATE INDEX idx_users_city
ON users(city);
```

План может выглядеть примерно так:

```python
Bitmap Heap Scan on users
  Recheck Cond: ((age = 30) AND (city = 'Moscow'))
  -> BitmapAnd
       -> Bitmap Index Scan on idx_users_age
       -> Bitmap Index Scan on idx_users_city
```

Логика:

```python
age = 30
    ↓
Bitmap A

city = Moscow
    ↓
Bitmap B

A AND B
    ↓
Bitmap Heap Scan
    ↓
Heap
```

Это одно из преимуществ bitmap-подхода.

---

# 13. `BitmapOr`

Для `OR` возможен другой вариант:

```python
SELECT *
FROM users
WHERE age = 20
   OR age = 30;
```

Например:

```python
Bitmap Heap Scan on users
  -> BitmapOr
       -> Bitmap Index Scan on idx_users_age
       -> Bitmap Index Scan on idx_users_age
```

Логика:

```python
age = 20
    ↓
Bitmap A

age = 30
    ↓
Bitmap B

A OR B
    ↓
Bitmap Heap Scan
```

---

# 14. Почему Bitmap Heap Scan может быть выгоднее Index Scan

Представим:

```python
10 000 000 строк
```

Найдено:

```python
500 000 строк
```

`Index Scan` может потребовать большого количества обращений к heap.

Bitmap-подход:

```python
Index
 ↓
Bitmap
 ↓
сгруппировать нужные страницы
 ↓
прочитать страницы
```

может эффективнее работать с большим количеством совпадений.

---

# 15. Но Bitmap Heap Scan не всегда лучше

Если найдено:

```python
1 строка
```

создавать bitmap может быть бессмысленно.

Проще:

```python
Index Scan
```

Если найдено:

```python
9 000 000 строк из 10 000 000
```

может оказаться выгоднее:

```python
Seq Scan
```

Поэтому:

```python
мало строк
    ↓
Index Scan

среднее/большое количество совпадений
    ↓
Bitmap Scan может быть выгоднее

огромная часть таблицы
    ↓
Seq Scan может быть выгоднее
```

Это не фиксированные пороги.

---

# 16. Как читать `EXPLAIN ANALYZE`

Пример:

```python
EXPLAIN (ANALYZE, BUFFERS)
SELECT *
FROM orders
WHERE user_id = 100;
```

Результат:

```python
Bitmap Heap Scan on orders
  (cost=100.00..2000.00 rows=50000 width=100)
  (actual time=1.000..15.000 rows=48000 loops=1)
  Recheck Cond: (user_id = 100)
  -> Bitmap Index Scan on idx_orders_user_id
       (cost=0.00..87.50 rows=50000 width=0)
       (actual time=0.500..0.500 rows=48000 loops=1)
       Index Cond: (user_id = 100)
```

Разбираем:

```python
Bitmap Heap Scan on orders
```

→ читаем heap-таблицу через bitmap.

```python
actual rows=48000
```

→ фактически получили 48 000 строк.

```python
Recheck Cond
```

→ условие проверяется повторно.

```python
Bitmap Index Scan
```

→ bitmap был построен на основе индекса.

```python
Index Cond
```

→ индекс использовался для поиска.

---

# 17. `Buffers`

Можно посмотреть:

```python
EXPLAIN (ANALYZE, BUFFERS)
```

Например:

```python
Bitmap Heap Scan on orders
  Buffers: shared hit=1500 read=200
```

Это показывает обращения к страницам PostgreSQL buffer cache:

```python
shared hit
    → страница уже была в памяти

shared read
    → страницу пришлось прочитать
```

Это полезно для анализа I/O.

---

# 18. Важная терминология

Не путать:

```python
Bitmap Index Scan
```

с:

```python
Bitmap Heap Scan
```

### Bitmap Index Scan

Работает с:

```python
INDEX
```

и создаёт:

```python
BITMAP
```

### Bitmap Heap Scan

Работает с:

```python
HEAP
```

и использует:

```python
BITMAP
```

для чтения нужных страниц.

---

# 19. Полная схема

```python
                 SQL
                  ↓
              Planner
                  ↓
        Bitmap Index Scan
                  ↓
        поиск через индекс
                  ↓
               Bitmap
                  ↓
         Bitmap Heap Scan
                  ↓
        нужные страницы Heap
                  ↓
            Recheck Cond
                  ↓
              Result
```

---

## 🧠 Главное

```python
Bitmap Index Scan
        ↓
использует индекс
        ↓
создаёт bitmap
        ↓
Bitmap Heap Scan
        ↓
читает нужные страницы heap
        ↓
проверяет строки
        ↓
возвращает результат
```

### Ключевое отличие:

> **Bitmap Index Scan работает на стороне индекса, а Bitmap Heap Scan — на стороне таблицы.**

И:

```python
Index Scan
    → Index → Heap

Bitmap Scan
    → Index → Bitmap → Heap
```

`Bitmap Heap Scan` часто появляется, когда нужно получить **достаточно много строк**, но последовательное чтение всей таблицы ещё не является самым дешёвым вариантом.

> **Bitmap Heap Scan — это оператор, который использует bitmap от индекса для эффективного чтения подходящих страниц heap-таблицы с последующей проверкой условий.**
