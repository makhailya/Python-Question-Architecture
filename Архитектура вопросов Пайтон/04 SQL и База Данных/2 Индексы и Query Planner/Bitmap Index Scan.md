# Bitmap Index Scan в PostgreSQL 🔎

## 🎯 Ответ на собеседовании

**Bitmap Index Scan** — это этап выполнения плана PostgreSQL, при котором индекс используется для поиска подходящих записей, после чего PostgreSQL строит **bitmap** — структуру, описывающую, какие строки или страницы таблицы потенциально нужно прочитать.

Сам по себе `Bitmap Index Scan` обычно является частью пары:

```python
Bitmap Index Scan
        ↓
Bitmap Heap Scan
```

Первый этап работает с **индексом**, второй — с **heap-таблицей**.

---

## 🎤 Суперкоротко

```python
Bitmap Index Scan
        ↓
используем индекс
        ↓
находим подходящие записи
        ↓
строим bitmap
        ↓
Bitmap Heap Scan
        ↓
читаем нужные страницы таблицы
```

Главная идея:

> **Bitmap Index Scan сначала определяет, какие данные нужны, а Bitmap Heap Scan затем получает их из таблицы.**

---

# 1. Простой пример

Допустим, есть:

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

PostgreSQL может выбрать:

```python
Bitmap Heap Scan on users
  Recheck Cond: ((age >= 20) AND (age <= 30))
  -> Bitmap Index Scan on idx_users_age
       Index Cond: ((age >= 20) AND (age <= 30))
```

Здесь:

```python
Bitmap Index Scan
```

использует индекс и строит bitmap.

Затем:

```python
Bitmap Heap Scan
```

использует этот bitmap для чтения нужных страниц таблицы.

---

# 2. Почему используется Bitmap

Представим:

```python
1 000 000 строк
```

Запрос нашёл:

```python
100 000 строк
```

Обычный `Index Scan` может привести к большому количеству обращений к heap:

```python
Index
 ↓
Row
 ↓
Heap
 ↓
Row
 ↓
Heap
 ↓
Row
 ↓
...
```

При большом количестве найденных строк это может быть неэффективно.

Bitmap-подход:

```python
Index
 ↓
Bitmap
 ↓
сгруппировать нужные страницы
 ↓
прочитать страницы таблицы
```

позволяет PostgreSQL организовать доступ к heap более эффективно.

---

# 3. Что такое Bitmap

Упрощённо bitmap можно представить как набор отметок:

```python
Page 1   → нет
Page 2   → да
Page 3   → нет
Page 4   → да
Page 5   → да
Page 6   → нет
```

То есть PostgreSQL получает информацию:

> «Вот эти страницы таблицы могут содержать подходящие строки».

Затем `Bitmap Heap Scan` читает необходимые страницы.

---

# 4. Важное отличие от Index Scan

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
найти строку
  ↓
Heap
  ↓
получить строку
```

### Bitmap Scan

```python
Index
  ↓
Bitmap Index Scan
  ↓
Bitmap
  ↓
Bitmap Heap Scan
  ↓
нужные страницы
  ↓
строки
```

Bitmap позволяет сначала собрать информацию о множестве совпадений, а затем более эффективно обратиться к heap.

---

# 5. Bitmap Index Scan не возвращает результат пользователю

Это важный момент.

`Bitmap Index Scan` — **не конечный оператор получения строк**.

Он выполняет подготовительную работу:

```python
Bitmap Index Scan
        ↓
создать bitmap
        ↓
Bitmap Heap Scan
        ↓
получить строки
```

Поэтому в плане обычно можно увидеть их вместе:

```python
Bitmap Heap Scan on users
  -> Bitmap Index Scan on idx_users_age
```

---

# 6. Bitmap Index Scan vs Bitmap Heap Scan

| Оператор            | Что делает                                     |
| ------------------- | ---------------------------------------------- |
| `Bitmap Index Scan` | использует индекс и строит bitmap              |
| `Bitmap Heap Scan`  | использует bitmap и читает heap                |
| `Index Scan`        | индекс → обращение к heap по найденным строкам |
| `Index Only Scan`   | старается получить данные из одного индекса    |

Схема:

```python
Bitmap Index Scan
        ↓
      Bitmap
        ↓
Bitmap Heap Scan
        ↓
      Heap
```

---

# 7. Когда Bitmap Scan может быть выгоден

Типичный сценарий:

```python
таблица большая
+
условие возвращает заметное количество строк
+
есть подходящий индекс
```

Например:

```python
10 000 000 строк
        ↓
WHERE age BETWEEN 20 AND 40
        ↓
500 000 строк
```

Обычный `Index Scan` может потребовать много обращений к heap.

Planner может выбрать:

```python
Bitmap Index Scan
        ↓
Bitmap Heap Scan
```

---

# 8. Почему не использовать Bitmap всегда

Bitmap Scan тоже имеет стоимость.

PostgreSQL должен:

```python
1. пройти индекс
2. построить bitmap
3. обработать bitmap
4. прочитать heap
```

Если нужно получить всего одну строку:

```python
WHERE id = 123
```

часто проще:

```python
Index Scan
```

Поэтому:

```python
мало строк
    ↓
Index Scan
```

а при большем количестве совпадений:

```python
Bitmap Scan
```

может стать выгоднее.

Но это **не правило с фиксированным порогом** — решение принимает planner.

---

# 9. Bitmap Scan и селективность

Селективность здесь особенно важна.

### Очень высокая селективность

```python
WHERE id = 123
```

Получаем:

```python
1 строку
```

Часто:

```python
Index Scan
```

### Средняя селективность

```python
WHERE age BETWEEN 20 AND 40
```

Получаем:

```python
сотни тысяч строк
```

Возможен:

```python
Bitmap Index Scan
        ↓
Bitmap Heap Scan
```

### Очень низкая селективность

Например:

```python
WHERE is_active = true
```

и:

```python
95% строк = true
```

Planner может решить:

```python
Seq Scan
```

---

# 10. Bitmap может использовать несколько индексов

Интересный случай:

```python
SELECT *
FROM users
WHERE age = 30
  AND city = 'Moscow';
```

Есть два индекса:

```python
CREATE INDEX idx_users_age
ON users(age);

CREATE INDEX idx_users_city
ON users(city);
```

PostgreSQL может построить bitmap по обоим индексам:

```python
Bitmap Index Scan
    idx_users_age
          ↓
      Bitmap A

Bitmap Index Scan
    idx_users_city
          ↓
      Bitmap B

          ↓
     Bitmap AND

          ↓
 Bitmap Heap Scan
```

То есть bitmap позволяет комбинировать результаты нескольких индексов.

---

# 11. `BitmapAnd`

Например:

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

city = 'Moscow'
    ↓
Bitmap B

A AND B
    ↓
только страницы/строки,
которые подходят обоим условиям
```

---

# 12. `BitmapOr`

Есть запрос:

```python
SELECT *
FROM users
WHERE age = 20
   OR age = 30;
```

PostgreSQL может использовать:

```python
BitmapOr
```

Схема:

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

То есть bitmap может объединять результаты нескольких индексных условий через:

```python
AND
OR
```

---

# 13. Lossy Bitmap

Bitmap имеет ограниченный объём памяти.

Если bitmap становится слишком большим, PostgreSQL может использовать **lossy representation**.

Вместо точного указания отдельных строк может храниться информация на уровне страниц:

```python
точно:
Page 100 → строки 2, 5, 8

lossy:
Page 100 → возможно есть подходящие строки
```

Поэтому PostgreSQL затем выполняет дополнительную проверку:

```python
Recheck Cond
```

---

# 14. `Recheck Cond`

В плане можно увидеть:

```python
Bitmap Heap Scan on users
  Recheck Cond: (age > 30)
```

Это означает, что условие дополнительно проверяется при чтении heap.

Особенно это актуально для lossy bitmap.

Упрощённо:

```python
Bitmap
  ↓
кандидатные страницы
  ↓
Heap
  ↓
Recheck Cond
  ↓
точные строки
```

---

# 15. `Rows Removed by Index Recheck`

В некоторых планах можно увидеть:

```python
Rows Removed by Index Recheck
```

Это означает, что после повторной проверки часть строк не подошла условию.

Такое особенно характерно для случаев с lossy bitmap.

---

# 16. `work_mem`

Размер bitmap связан с доступной рабочей памятью PostgreSQL.

Один из параметров:

```python
work_mem
```

определяет объём памяти, который может использоваться операциями вроде:

* Sort;
* Hash;
* некоторых bitmap-операций.

Если bitmap не помещается эффективно в доступную память, PostgreSQL может использовать более грубое представление.

Но **увеличивать `work_mem` только ради Bitmap Scan без анализа не следует**.

---

# 17. Как читать Bitmap-план

Например:

```python
Bitmap Heap Scan on orders
  (cost=100.00..5000.00 rows=50000 width=100)
  (actual time=2.000..20.000 rows=48000 loops=1)
  Recheck Cond: (user_id = 100)
  -> Bitmap Index Scan on idx_orders_user_id
       (cost=0.00..87.50 rows=50000 width=0)
       (actual time=1.000..1.000 rows=48000 loops=1)
       Index Cond: (user_id = 100)
```

Читаем снизу вверх:

```python
Bitmap Index Scan
        ↓
использовали индекс
        ↓
создали bitmap
        ↓
Bitmap Heap Scan
        ↓
прочитали нужные страницы heap
        ↓
Recheck Cond
        ↓
получили результат
```

---

# 18. Bitmap Index Scan vs Index Scan

|                                           | Index Scan  | Bitmap Index Scan    |
| ----------------------------------------- | ----------- | -------------------- |
| Использует индекс                         | ✅           | ✅                    |
| Строит bitmap                             | ❌           | ✅                    |
| Сам получает строки таблицы               | Через heap  | Нет                  |
| Следующий этап                            | Heap fetch  | Bitmap Heap Scan     |
| Хорош для малого числа строк              | Часто       | Не всегда            |
| Хорош для большого числа совпадений       | Не всегда   | Часто                |
| Может комбинироваться с другими индексами | Ограниченно | `BitmapAnd/BitmapOr` |

---

# 19. Связь с типами индексов

Bitmap Scan — это **не отдельный тип индекса**.

Это способ выполнения запроса.

Например:

```python
B-tree
  ↓
Bitmap Index Scan
```

или другой индекс, поддерживающий нужную операцию:

```python
GIN
  ↓
Bitmap Index Scan
```

То есть не нужно говорить:

> «Bitmap Index — это тип индекса».

Правильно:

> **Bitmap Index Scan — это оператор плана выполнения PostgreSQL, который использует индекс для построения bitmap.**

---

# 20. Как проверить Bitmap Index Scan

Используем:

```python
EXPLAIN (ANALYZE, BUFFERS)
SELECT *
FROM orders
WHERE user_id = 100;
```

Ищем:

```python
Bitmap Heap Scan
  -> Bitmap Index Scan
```

Например:

```python
Bitmap Heap Scan on orders
  -> Bitmap Index Scan on idx_orders_user_id
```

Это означает:

```python
индекс
  ↓
bitmap
  ↓
heap
```

---

## 🧠 Главное

```python
Bitmap Index Scan
        ↓
использует индекс
        ↓
находит подходящие записи
        ↓
строит bitmap
        ↓
Bitmap Heap Scan
        ↓
читает нужные страницы таблицы
```

### Основное отличие:

```python
Index Scan
    ↓
Index → Heap
```

```python
Bitmap Scan
    ↓
Index → Bitmap → Heap
```

### Bitmap особенно полезен, когда:

```python
большая таблица
+
подходит заметное количество строк
+
прямой Index Scan был бы слишком дорогим
```

И ещё один важный момент:

```python
Bitmap Index Scan
```

может участвовать в:

```python
BitmapAnd
BitmapOr
```

что позволяет PostgreSQL комбинировать результаты нескольких индексов.

> **Bitmap Index Scan не является типом индекса. Это оператор плана PostgreSQL, который использует индекс для построения bitmap перед чтением соответствующих страниц таблицы через [[Bitmap Heap Scan]].**
