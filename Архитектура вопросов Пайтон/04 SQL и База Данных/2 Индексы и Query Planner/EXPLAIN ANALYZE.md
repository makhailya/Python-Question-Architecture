# EXPLAIN ANALYZE в PostgreSQL 🔬

## 🎯 Ответ на собеседовании

**EXPLAIN ANALYZE** — команда PostgreSQL, которая **реально выполняет SQL-запрос** и показывает фактическую статистику его выполнения вместе с планом.

Она позволяет сравнить:

* что PostgreSQL **ожидал** получить;
* что произошло **фактически**;
* сколько времени заняло выполнение;
* сколько строк было обработано;
* сколько раз выполнялся каждый узел плана.

Используется для поиска **узких мест (bottlenecks)** и оптимизации SQL-запросов.

---

## 🎤 Суперкоротко

```python
EXPLAIN
    → только показывает предполагаемый план

EXPLAIN ANALYZE
    → выполняет запрос
    → показывает план
    → показывает actual time
    → показывает actual rows
    → показывает loops
```

Главное:

> **EXPLAIN ANALYZE позволяет сравнить оценку планировщика с реальным выполнением запроса.**

---

# 1. Базовый пример

```python
EXPLAIN ANALYZE
SELECT *
FROM users
WHERE id = 100;
```

Условный результат:

```python
Index Scan using users_pkey on users
  (cost=0.29..8.30 rows=1 width=64)
  (actual time=0.020..0.025 rows=1 loops=1)
  Index Cond: (id = 100)
Planning Time: 0.100 ms
Execution Time: 0.040 ms
```

Здесь PostgreSQL:

1. построил план;
2. реально выполнил `SELECT`;
3. собрал фактическую статистику;
4. показал результат.

---

# 2. `cost` vs `actual time`

Это одно из самых важных различий.

```python
cost=0.29..8.30
```

Это **оценочная стоимость**, которую использует планировщик.

Это **не миллисекунды**.

А:

```python
actual time=0.020..0.025
```

это уже фактическое время выполнения узла в **миллисекундах**.

### Поэтому:

```python
cost
```

→ внутренняя оценка стоимости.

```python
actual time
```

→ фактическое время.

---

# 3. `rows` vs `actual rows`

Например:

```python
rows=100
actual rows=95
```

Планировщик ожидал:

```python
100 строк
```

Фактически получил:

```python
95 строк
```

Оценка достаточно близкая.

---

Но может быть:

```python
rows=100
actual rows=100000
```

Это уже серьёзное расхождение.

Планировщик ожидал:

```python
100
```

а реально получил:

```python
100 000
```

Такое расхождение может привести к выбору неоптимального плана.

---

# 4. `loops`

Например:

```python
(actual time=0.010..0.020 rows=5 loops=100)
```

`loops=100` означает, что этот узел выполнялся **100 раз**.

Это особенно важно при:

```python
Nested Loop
```

Например:

```python
Nested Loop
    ↓
    Index Scan
```

Внутренний `Index Scan` может выполняться много раз — по одному разу для каждой строки внешнего узла.

---

# 5. Время и `loops`

Если узел имеет:

```python
actual time=1.0..2.0 loops=10
```

не нужно механически воспринимать `2 ms` как суммарное время всех десяти запусков.

`actual time` в строке узла показывается **для одного запуска**, а `loops` показывает количество запусков.

Поэтому при анализе нужно учитывать оба значения.

---

# 6. `Planning Time`

В конце плана можно увидеть:

```python
Planning Time: 0.150 ms
```

Это время, которое PostgreSQL потратил на:

* анализ запроса;
* выбор плана;
* подготовку к выполнению.

---

# 7. `Execution Time`

Например:

```python
Execution Time: 25.400 ms
```

Это фактическое время выполнения запроса.

При оптимизации запроса обычно сравнивают:

```python
до:
Execution Time: 250 ms

после:
Execution Time: 20 ms
```

---

# 8. EXPLAIN ANALYZE и индексы

Допустим, есть индекс:

```python
CREATE INDEX idx_users_email
ON users(email);
```

Проверяем:

```python
EXPLAIN ANALYZE
SELECT *
FROM users
WHERE email = 'test@example.com';
```

Можем увидеть:

```python
Index Scan using idx_users_email on users
  (cost=0.29..8.30 rows=1 width=64)
  (actual time=0.020..0.025 rows=1 loops=1)
```

Это говорит о том, что PostgreSQL использовал индекс и фактически получил одну строку.

---

# 9. Но Seq Scan не всегда плохо

Например:

```python
EXPLAIN ANALYZE
SELECT *
FROM users;
```

Может быть:

```python
Seq Scan on users
  (cost=0.00..1000.00 rows=50000 width=64)
  (actual time=0.010..15.000 rows=50000 loops=1)
```

Это не означает автоматически, что нужно создавать индекс.

Если запросу нужны практически **все строки таблицы**, последовательное чтение может быть эффективнее.

---

# 10. `BUFFERS`

Для более глубокого анализа используют:

```python
EXPLAIN (ANALYZE, BUFFERS)
SELECT *
FROM users
WHERE email = 'test@example.com';
```

Например:

```python
Buffers: shared hit=10 read=2
```

### `shared hit`

Страница уже была найдена в PostgreSQL buffer cache.

### `shared read`

Страницу пришлось прочитать.

Упрощённо:

```python
shared hit
    ↓
данные уже в памяти PostgreSQL

shared read
    ↓
данные пришлось прочитать
```

`BUFFERS` особенно полезен при анализе проблем с I/O.

---

# 11. `EXPLAIN (ANALYZE, BUFFERS)`

На практике часто используют именно такой вариант:

```python
EXPLAIN (ANALYZE, BUFFERS)
SELECT *
FROM orders
WHERE user_id = 100;
```

Можно получить одновременно:

```python
план
+
фактическое выполнение
+
время
+
количество строк
+
loops
+
информация о чтении страниц
```

---

# 12. Как искать проблему

Допустим, запрос выполняется:

```python
Execution Time: 1500 ms
```

Смотрим план.

Обращаем внимание на:

### 1. Большой `actual time`

```python
actual time=0.010..1400.000
```

Возможный bottleneck.

### 2. Большой `actual rows`

```python
actual rows=1000000
```

Обрабатывается огромное количество данных.

### 3. Большой `loops`

```python
loops=100000
```

Операция выполняется огромное количество раз.

### 4. Большое расхождение `rows` и `actual rows`

```python
rows=10
actual rows=100000
```

Планировщик сильно ошибся в оценке.

### 5. Неожиданный `Seq Scan`

Например:

```python
Seq Scan on orders
```

при запросе, который должен выбирать очень небольшое количество строк.

Тогда стоит проверить:

* наличие индекса;
* селективность;
* статистику;
* условие запроса;
* актуальность статистики.

---

# 13. Пример с плохой оценкой

Допустим:

```python
Nested Loop
  (cost=...)
  (actual time=... rows=100000 loops=1)
```

Внутри:

```python
Index Scan
  (rows=1)
  (actual rows=100000)
```

Планировщик ожидал:

```python
1 строку
```

а получил:

```python
100000 строк
```

Это может привести к неэффективному плану.

Проверяем статистику:

```python
ANALYZE users;
```

После этого снова:

```python
EXPLAIN ANALYZE ...
```

---

# 14. EXPLAIN ANALYZE для JOIN

Запрос:

```python
EXPLAIN ANALYZE
SELECT *
FROM users u
JOIN orders o
    ON o.user_id = u.id;
```

PostgreSQL может выбрать:

```python
Hash Join
```

или:

```python
Nested Loop
```

или:

```python
Merge Join
```

`EXPLAIN ANALYZE` позволяет увидеть, какой алгоритм был выбран **и насколько эффективно он реально отработал**.

---

# 15. Важная особенность — запрос реально выполняется ⚠️

Это главное отличие от обычного `EXPLAIN`.

```python
EXPLAIN
UPDATE users
SET name = 'Alex'
WHERE id = 100;
```

Изменение данных не выполняется.

Но:

```python
EXPLAIN ANALYZE
UPDATE users
SET name = 'Alex'
WHERE id = 100;
```

**реально выполнит UPDATE.**

То же самое:

```python
EXPLAIN ANALYZE DELETE FROM users
WHERE id = 100;
```

реально удалит строку.

Поэтому `EXPLAIN ANALYZE` нужно осторожно использовать с:

```python
INSERT
UPDATE
DELETE
```

---

# 16. Как безопасно анализировать изменения

Если нужно исследовать изменяющий запрос, можно использовать транзакцию:

```python
BEGIN;

EXPLAIN ANALYZE
UPDATE users
SET name = 'Alex'
WHERE id = 100;

ROLLBACK;
```

Запрос всё равно будет выполнен внутри транзакции, но изменения будут отменены через `ROLLBACK`.

⚠️ Однако это не означает абсолютную безопасность: побочные эффекты, внешние вызовы и другие особенности конкретного запроса могут существовать.

---

# 17. `EXPLAIN ANALYZE` не оптимизирует запрос автоматически

Команда:

```python
EXPLAIN ANALYZE
```

не означает:

> «PostgreSQL сейчас оптимизирует мой SQL».

Она:

```python
строит план
      ↓
выполняет запрос
      ↓
собирает фактические данные
      ↓
показывает результат
```

Оптимизацию выполняет разработчик/DBA, анализируя этот результат.

Например:

```python
EXPLAIN ANALYZE
        ↓
обнаружили Seq Scan
        ↓
проверили причину
        ↓
создали индекс
        ↓
повторили EXPLAIN ANALYZE
        ↓
сравнили результат
```

---

# 18. Практический workflow

При медленном запросе:

```python
EXPLAIN (ANALYZE, BUFFERS)
SELECT ...
```

Затем:

```python
1. Смотрим Execution Time
       ↓
2. Находим дорогие узлы
       ↓
3. Смотрим actual time
       ↓
4. Сравниваем rows и actual rows
       ↓
5. Смотрим loops
       ↓
6. Проверяем Seq Scan / Index Scan
       ↓
7. Проверяем JOIN
       ↓
8. Проверяем Sort / Aggregate
       ↓
9. Смотрим BUFFERS
       ↓
10. Меняем запрос/индекс
       ↓
11. Повторяем EXPLAIN ANALYZE
```

---

# 19. Основные показатели

| Показатель       | Значение                             |
| ---------------- | ------------------------------------ |
| `cost`           | оценочная стоимость узла             |
| `rows`           | ожидаемое количество строк           |
| `width`          | ожидаемый средний размер строки      |
| `actual time`    | фактическое время выполнения узла    |
| `actual rows`    | фактическое количество строк         |
| `loops`          | количество выполнений узла           |
| `Planning Time`  | время построения плана               |
| `Execution Time` | фактическое время выполнения запроса |
| `Buffers`        | информация об обращениях к страницам |

---

# 20. EXPLAIN vs EXPLAIN ANALYZE

|                            |      `EXPLAIN` | `EXPLAIN ANALYZE` |
| -------------------------- | -------------: | ----------------: |
| Показывает план            |              ✅ |                 ✅ |
| Выполняет запрос           |              ❌ |                 ✅ |
| `cost`                     |              ✅ |                 ✅ |
| Estimated `rows`           |              ✅ |                 ✅ |
| Actual `rows`              |              ❌ |                 ✅ |
| Actual `time`              |              ❌ |                 ✅ |
| `loops`                    |              ❌ |                 ✅ |
| `BUFFERS`                  | Можно добавить |    Можно добавить |
| Опасен для `UPDATE/DELETE` |              ❌ |             ⚠️ Да |

---

## 🧠 Главное

```python
EXPLAIN
    → Что PostgreSQL собирается сделать?

EXPLAIN ANALYZE
    → Что PostgreSQL реально сделал?
```

Самая важная часть анализа:

```python
estimated rows
        VS
actual rows
```

и:

```python
actual time
loops
BUFFERS
```

Если оценки сильно расходятся:

```python
rows=10
actual rows=100000
```

это повод разобраться, почему планировщик неправильно оценил объём данных.

И главное:

> **EXPLAIN ANALYZE — основной инструмент PostgreSQL для проверки реального плана выполнения и поиска узких мест SQL-запроса.**
