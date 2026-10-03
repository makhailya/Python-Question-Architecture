# 🔗 JOIN Algorithms

## 🎤 Короткий ответ

**JOIN algorithm** — это внутренний алгоритм, которым PostgreSQL физически выполняет `JOIN` двух таблиц.

Основные алгоритмы:

1. **Nested Loop Join** — для каждой строки одной таблицы ищем совпадения в другой.
2. **Hash Join** — строим хеш-таблицу по одной стороне и ищем в ней строки другой стороны.
3. **Merge Join** — сортируем обе стороны по ключу и затем последовательно сопоставляем строки.

Важно различать:

```text
SQL JOIN
   ↓
логическая операция

JOIN algorithm
   ↓
физический способ выполнения
```

Например, один и тот же:

```sql
SELECT *
FROM orders o
JOIN users u ON u.id = o.user_id;
```

PostgreSQL может выполнить через `Nested Loop`, `Hash Join` или `Merge Join` — решение принимает **Query Planner** на основании статистики, стоимости операций, индексов, размеров таблиц и условий запроса.

---

## 🗣️ Ответ на собеседовании

В PostgreSQL `JOIN` — это логическая операция, а Nested Loop, Hash Join и Merge Join — физические алгоритмы, которыми planner может её реализовать.

**Nested Loop** берёт строки одной таблицы и для каждой ищет совпадения в другой. Он особенно эффективен, когда внешняя таблица небольшая, а по внутренней стороне есть подходящий индекс.

**Hash Join** строит hash table по одной из таблиц по ключу соединения, после чего для строк второй таблицы выполняет hash lookup. Он хорошо подходит для больших таблиц при equi-join, когда нет необходимости использовать порядок строк.

**Merge Join** работает с двумя потоками строк, отсортированными по ключу JOIN, и последовательно сопоставляет их. Если данные уже отсортированы или есть подходящие индексы, сортировка может быть дешёвой или вообще не понадобиться.

Конкретный алгоритм выбирает PostgreSQL Query Planner. Поэтому нельзя сказать, что один алгоритм всегда лучше другого — всё зависит от конкретного плана и статистики.

---

## 🧭 Где я нахожусь

```text
04 SQL и База Данных
└── 02 Индексы и Query Planner
    ├── Индексы
    ├── Selectivity
    ├── EXPLAIN
    ├── Query Planner
    │   └── JOIN algorithms ← Я здесь
    │       ├── Nested Loop Join
    │       ├── Hash Join
    │       └── Merge Join
    └── Scan types
        ├── Sequential Scan
        └── Index Scan
```

---

# 📚 Разбор поглубже

## 1. JOIN — логика, алгоритм — реализация

Возьмём:

```sql
SELECT
    u.name,
    o.amount
FROM users u
JOIN orders o
    ON o.user_id = u.id;
```

Логически нам нужно:

```text
users
   +
orders
   ↓
сопоставить:
users.id = orders.user_id
```

Но физически PostgreSQL должен решить:

> Как именно найти совпадающие строки?

Например:

```text
             JOIN
              │
       Query Planner
        /     |      \
       /      |       \
Nested Loop  Hash    Merge
```

---

# 2. Nested Loop Join

## Идея

Самый простой алгоритм:

```text
для каждой строки A
    найти подходящие строки B
```

Условно:

```text
A:
1
2
3

B:
1
1
2
3
3
```

Алгоритм:

```text
A=1 → ищем 1 в B
A=2 → ищем 2 в B
A=3 → ищем 3 в B
```

---

## 2.1. Наивный вариант

Если внутреннюю таблицу каждый раз приходится полностью сканировать:

```text
A rows × B rows
```

Приблизительная сложность:

```text
O(N × M)
```

Например:

```text
1000 × 100000
```

может означать огромное количество сравнений.

Но PostgreSQL может использовать индекс на внутренней стороне.

---

## 2.2. Nested Loop + Index Scan

Допустим:

```sql
CREATE INDEX idx_orders_user_id
ON orders(user_id);
```

Запрос:

```sql
SELECT *
FROM users u
JOIN orders o
    ON o.user_id = u.id;
```

Тогда PostgreSQL может сделать:

```text
users
  ↓
Nested Loop
  ↓
для каждого user.id
  ↓
Index Scan по orders.user_id
```

Например:

```text
user 10
   ↓
index lookup orders.user_id = 10

user 20
   ↓
index lookup orders.user_id = 20

user 30
   ↓
index lookup orders.user_id = 30
```

Если пользователей после фильтрации мало, это может быть очень эффективно.

---

## 2.3. Когда Nested Loop подходит

Типичный случай:

```text
маленькая внешняя таблица
          +
индекс на внутренней таблице
          ↓
Nested Loop
```

Например:

```sql
SELECT *
FROM users u
JOIN orders o
    ON o.user_id = u.id
WHERE u.id = 42;
```

Если найден один пользователь, PostgreSQL может сделать один индексный поиск заказов.

---

# 3. Hash Join

## Идея

Hash Join использует hash table.

Допустим:

```text
users
id
---
1
2
3
```

и:

```text
orders
user_id
-------
2
3
1
3
```

PostgreSQL может:

```text
users
   ↓
создать hash table
   ↓
hash(id)
   ↓
┌───────────────┐
│ 1 → user A    │
│ 2 → user B    │
│ 3 → user C    │
└───────────────┘
```

После этого читает `orders`:

```text
order.user_id = 2
       ↓
hash(2)
       ↓
нашли user 2
```

---

# 4. Фазы Hash Join

Упрощённо:

```text
        Hash Join
            │
      ┌─────┴─────┐
      ↓           ↓
   Build        Probe
      ↓           ↓
создать hash   искать в hash
 table           table
```

### Build phase

Одна сторона используется для построения hash table.

### Probe phase

Вторая сторона читается и для каждой строки выполняется поиск в hash table.

---

# 5. Пример Hash Join

```sql
SELECT *
FROM users u
JOIN orders o
    ON u.id = o.user_id;
```

Условно:

```text
users
 ↓
Hash
 ↓
Hash Table

orders
 ↓
для каждой строки
 ↓
Hash(user_id)
 ↓
lookup
```

---

# 6. Когда Hash Join подходит

Hash Join особенно естественен для:

```sql
A.key = B.key
```

То есть для **equi-join**.

Например:

```sql
ON users.id = orders.user_id
```

Он может быть эффективен при больших объёмах данных, когда нет преимущества от индексного Nested Loop.

---

# 7. Hash Join и память

Hash table должна помещаться в доступную память.

PostgreSQL использует параметры памяти, в частности:

```text
work_mem
```

Если hash table не помещается в память, PostgreSQL может использовать специальные стратегии с разделением данных на batches и временным хранением.

Поэтому:

> Hash Join не означает, что вся таблица всегда полностью находится в RAM.

---

# 8. Merge Join

## Идея

Merge Join требует входные данные, упорядоченные по ключу JOIN.

Например:

```text
A:
1
2
3
4

B:
1
2
3
5
```

Далее PostgreSQL двигается по двум последовательностям:

```text
A → 1
B → 1
   ↓
match

A → 2
B → 2
   ↓
match

A → 3
B → 3
   ↓
match
```

Если значения не совпадают, продвигается соответствующая сторона.

---

# 9. Почему Merge Join эффективен

После сортировки обеих сторон можно пройти по данным последовательно.

Упрощённо:

```text
Sort A
  ↓
1 2 3 4

Sort B
  ↓
1 2 3 5

      ↓
Merge Join
      ↓
1 2 3
```

После сортировки этап merge может работать примерно линейно:

```text
O(N + M)
```

Но сортировка сама стоит ресурсов.

Поэтому полная стоимость:

```text
Sort(A) + Sort(B) + Merge
```

Если данные уже отсортированы или planner может получить их в нужном порядке через индекс, Merge Join становится интереснее.

---

# 10. Сравнение алгоритмов

| Алгоритм        | Основная идея                    | Когда может быть эффективен                   |
| --------------- | -------------------------------- | --------------------------------------------- |
| **Nested Loop** | Для каждой строки A ищем B       | Маленькая внешняя сторона, индекс на B        |
| **Hash Join**   | Hash table + lookup              | Большие equi-join                             |
| **Merge Join**  | Сортированные последовательности | Данные уже отсортированы / выгодна сортировка |

Важно:

> Это не правило «маленькая таблица → Nested Loop, большая → Hash Join».

Planner учитывает множество факторов.

---

# 11. Как PostgreSQL выбирает алгоритм

Planner оценивает стоимость различных планов.

Учитываются:

* количество строк;
* статистика таблиц;
* selectivity условий;
* наличие индексов;
* стоимость sequential scan;
* стоимость index scan;
* необходимость сортировки;
* доступная память;
* стоимость hash table;
* порядок соединения таблиц;
* предполагаемое количество строк после фильтрации.

Поэтому:

```text
один SQL-запрос
       ↓
разные данные / статистика
       ↓
может получить
другой JOIN algorithm
```

---

# 12. EXPLAIN

Чтобы увидеть выбранный алгоритм:

```sql
EXPLAIN
SELECT *
FROM users u
JOIN orders o
    ON u.id = o.user_id;
```

Например, можно увидеть:

```text
Hash Join
  Hash Cond: (o.user_id = u.id)
```

Это означает, что planner выбрал Hash Join.

Другой план:

```text
Nested Loop
  -> Index Scan ...
  -> Index Scan ...
```

или:

```text
Merge Join
```

---

# 13. EXPLAIN ANALYZE

Для реального выполнения:

```sql
EXPLAIN ANALYZE
SELECT *
FROM users u
JOIN orders o
    ON u.id = o.user_id;
```

Здесь PostgreSQL **действительно выполняет запрос** и показывает фактические показатели.

Например:

```text
Hash Join
  (cost=...)
  (actual time=... rows=...)
```

Особенно полезно сравнивать:

```text
estimated rows
        vs
actual rows
```

Если оценки сильно отличаются, planner может выбрать неоптимальный план.

---

# 14. JOIN algorithm и индексы

Важно понимать:

> Индекс не означает автоматически, что PostgreSQL выберет Nested Loop.

Например, индекс существует:

```sql
CREATE INDEX idx_orders_user_id
ON orders(user_id);
```

Но если PostgreSQL считает, что нужно прочитать большую часть `orders`, ему может оказаться дешевле использовать:

```text
Hash Join
```

вместо тысяч или миллионов отдельных index lookup.

---

# 15. Nested Loop + Index Scan

Очень важная комбинация:

```text
Nested Loop
    ↓
Index Scan
```

Например:

```text
10 users
   ↓
10 index lookups
   ↓
orders
```

Если после фильтрации осталось мало строк, это может быть выгоднее полного сканирования большой таблицы.

---

# 16. Hash Join + Sequential Scan

Другой типичный вариант:

```text
Hash Join
   ├── Seq Scan users
   │
   └── Seq Scan orders
```

Это не обязательно плохо.

Если нужно обработать большую часть таблиц, sequential scan может быть дешевле множества случайных обращений через индекс.

---

# 17. Merge Join + Sort

Типичный план:

```text
Merge Join
├── Sort
│   └── Seq Scan A
└── Sort
    └── Seq Scan B
```

Если данные уже отсортированы:

```text
Merge Join
├── Index Scan A
└── Index Scan B
```

и отдельная сортировка может оказаться не нужна.

---

# 18. JOIN algorithm vs JOIN type

Это принципиально разные вещи.

### JOIN type

Определяет **какие строки логически должны попасть в результат**:

```sql
INNER JOIN
LEFT JOIN
RIGHT JOIN
FULL JOIN
```

### JOIN algorithm

Определяет **как физически найти эти строки**:

```text
Nested Loop
Hash Join
Merge Join
```

Например:

```sql
LEFT JOIN
```

может физически выполняться через Nested Loop.

А может использовать Hash Join или другой подход в зависимости от запроса и плана.

---

# 19. JOIN order

При нескольких таблицах:

```sql
SELECT *
FROM A
JOIN B ON ...
JOIN C ON ...;
```

planner также выбирает порядок соединения.

Например:

```text
(A JOIN B) JOIN C
```

или:

```text
A JOIN (B JOIN C)
```

Это важно, потому что промежуточный результат может иметь очень разный размер.

Например:

```text
A = 1 000 000 rows
B = 100 rows
C = 1 000 000 rows
```

Порядок JOIN сильно влияет на стоимость плана.

---

# 20. Почему статистика важна

PostgreSQL собирает статистику таблиц с помощью:

```sql
ANALYZE users;
```

или:

```sql
ANALYZE;
```

Planner использует статистику для оценки:

```text
сколько строк будет найдено
        ↓
какая стоимость операции
        ↓
какой JOIN выбрать
```

Если статистика устарела или оценки сильно ошибаются, planner может выбрать неудачный план.

---

# 21. Типичный пример для собеседования

Есть:

```text
users: 10 000 000 rows
orders: 50 000 000 rows
```

Запрос:

```sql
SELECT *
FROM users u
JOIN orders o
    ON o.user_id = u.id;
```

Нельзя просто сказать:

> Здесь будет Hash Join.

Правильный ответ:

> PostgreSQL сам выберет алгоритм на основании стоимости плана. Для большого equi-join Hash Join может оказаться подходящим, но конкретный выбор нужно смотреть через `EXPLAIN` и учитывать статистику, индексы, selectivity и стоимость сортировки.

---

# 22. Как отвечать на вопрос «Какой JOIN самый быстрый?»

Правильная формулировка:

> Универсально самого быстрого JOIN algorithm нет. PostgreSQL выбирает Nested Loop, Hash Join или Merge Join в зависимости от конкретного запроса, размеров и статистики таблиц, индексов, selectivity и стоимости операций.

Это важнее, чем запоминание условных правил.

---

# 23. Короткая схема трёх алгоритмов

```text
Nested Loop
────────────────────
A
 ↓
для каждой строки
 ↓
ищем B


Hash Join
────────────────────
A
 ↓
Hash Table
 ↓
B → hash lookup


Merge Join
────────────────────
A → sorted ─┐
            ├→ Merge
B → sorted ─┘
```

---

# 24. Что смотреть в EXPLAIN

При анализе JOIN полезно обратить внимание на:

```text
Join type:
    Nested Loop
    Hash Join
    Merge Join

Access method:
    Seq Scan
    Index Scan
    Bitmap Heap Scan

Estimated rows
Actual rows

Cost
Actual time

Hash Cond
Merge Cond
Index Cond
```

Например:

```text
Hash Join
  Hash Cond: (orders.user_id = users.id)
  -> Seq Scan on orders
  -> Hash
       -> Seq Scan on users
```

Здесь можно восстановить структуру выполнения:

```text
users
 ↓
Seq Scan
 ↓
Hash
 ↓
Hash Table
       ↑
       │
orders
 ↓
Seq Scan
 ↓
lookup
 ↓
Hash Join
```

---

# 25. Главное различие трёх алгоритмов

### Nested Loop

```text
Для каждой строки A
    ищем B
```

Сильная сторона:

```text
маленький outer + хороший index
```

---

### Hash Join

```text
Построить hash table
        ↓
делать lookup
```

Сильная сторона:

```text
большие equi-join
```

---

### Merge Join

```text
Отсортировать
      ↓
последовательно объединить
```

Сильная сторона:

```text
данные уже упорядочены
или сортировка выгодна
```

---

## 🎤 Вопросы на собеседовании

### Какие основные JOIN algorithms есть в PostgreSQL?

* Nested Loop
* Hash Join
* Merge Join

### Чем JOIN algorithm отличается от `INNER JOIN` / `LEFT JOIN`?

`INNER JOIN` и `LEFT JOIN` определяют логическую семантику результата, а Nested Loop, Hash Join и Merge Join — физический способ выполнения соединения.

### Как работает Nested Loop?

Для каждой строки внешнего источника PostgreSQL ищет соответствующие строки внутреннего источника.

### Когда Nested Loop может быть эффективен?

Когда внешняя сторона небольшая, особенно если для внутренней стороны есть подходящий индекс.

### Как работает Hash Join?

PostgreSQL строит hash table по одной стороне JOIN, затем читает другую сторону и выполняет hash lookup по ключу.

### Для каких JOIN Hash Join особенно подходит?

Для **equi-join**, где условие содержит равенство, например:

```sql
ON a.id = b.a_id
```

### Как работает Merge Join?

Обе стороны должны быть упорядочены по ключу JOIN, после чего PostgreSQL последовательно проходит их и сопоставляет значения.

### Когда Merge Join может быть выгоден?

Когда данные уже отсортированы или сортировка относительно дёшева и выгодна по сравнению с альтернативами.

### Всегда ли наличие индекса приводит к Nested Loop?

Нет. Planner сравнивает стоимость разных планов.

### Как узнать, какой JOIN algorithm выбрал PostgreSQL?

```sql
EXPLAIN
SELECT ...;
```

или для фактического выполнения:

```sql
EXPLAIN ANALYZE
SELECT ...;
```

### Какой JOIN algorithm самый быстрый?

Универсального ответа нет. Выбор зависит от конкретных данных, статистики, индексов, selectivity и стоимости плана.

### Что такое `Hash Cond` в EXPLAIN?

Это условие, по которому Hash Join сопоставляет строки через hash table.

Например:

```text
Hash Cond: (o.user_id = u.id)
```

### Что такое `Merge Cond`?

Условие, по которому Merge Join сопоставляет отсортированные входные данные.

### Что может заставить PostgreSQL выбрать Hash Join вместо Nested Loop?

Например, большой объём данных и ситуация, когда множество отдельных index lookup оказывается дороже построения hash table и последовательного чтения данных.

### Что такое JOIN order?

Порядок, в котором planner соединяет несколько таблиц. При большом количестве таблиц он может существенно влиять на размер промежуточных результатов и общую стоимость запроса.

### Как связаны JOIN algorithms и Query Planner?

```text
SQL
 ↓
Query Planner
 ↓
оценка стоимости
 ↓
выбор плана
 ↓
Nested Loop / Hash Join / Merge Join
 ↓
execution
```
