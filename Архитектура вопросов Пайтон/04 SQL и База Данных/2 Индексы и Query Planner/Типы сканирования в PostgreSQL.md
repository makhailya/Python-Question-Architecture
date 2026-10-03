# Типы сканирования в PostgreSQL 🔎

## 🎯 Ответ на собеседовании

**Тип сканирования** показывает, каким способом PostgreSQL читает данные из таблицы или индекса.

Основные варианты:

* **[[Последовательное сканирование - Sequential Scan]]** — последовательное чтение таблицы;
* **[[Index Scan]]** — поиск через индекс с последующим обращением к таблице;
* **[[Index Only Scan]]** — данные можно получить непосредственно из индекса;
* **[[Bitmap Index Scan]] + [[Bitmap Heap Scan]]** — сначала строится bitmap подходящих строк/страниц по индексу, затем читаются нужные страницы таблицы;
* **[[TID Scan]]** — поиск строки непосредственно по её физическому `ctid`.

Выбор делает **query planner** на основе статистики, стоимости операций и предполагаемого количества строк.

---

## 🎤 Суперкоротко

```python
Seq Scan
    → читаем таблицу последовательно

Index Scan
    → индекс → нужные строки → таблица

Index Only Scan
    → индекс → результат

Bitmap Scan
    → индекс → bitmap → страницы таблицы

TID Scan
    → физический адрес строки (ctid) → строка
```

**Важно:** `Seq Scan` не означает автоматически «плохо», а `Index Scan` — автоматически «хорошо».

---

# 1. Seq Scan

```python
Seq Scan on users
```

**Sequential Scan** — PostgreSQL последовательно читает таблицу и проверяет строки.

Условно:

```python
Таблица:

1
2
3
4
5
...
1 000 000
```

PostgreSQL проходит по страницам таблицы и проверяет строки на соответствие условию.

Например:

```python
EXPLAIN ANALYZE
SELECT *
FROM users
WHERE age > 30;
```

Может получиться:

```python
Seq Scan on users
  Filter: (age > 30)
```

### Когда Seq Scan нормален

Если нужно вернуть большую часть таблицы:

```python
SELECT *
FROM users;
```

или условие имеет низкую селективность:

```python
WHERE is_active = true
```

и почти все пользователи активны.

В таком случае последовательное чтение может оказаться дешевле обращения к индексу.

---

# 2. Index Scan

```python
Index Scan using users_email_idx on users
```

PostgreSQL сначала использует индекс, чтобы найти подходящие записи, а затем обращается к таблице за самими данными.

Схематично:

```python
Index
  ↓
найдены TID
  ↓
Heap / Table
  ↓
получены строки
```

Например:

```python
EXPLAIN ANALYZE
SELECT *
FROM users
WHERE email = 'test@example.com';
```

План:

```python
Index Scan using users_email_idx on users
  Index Cond: (email = 'test@example.com')
```

Если условие очень селективное и подходит индекс, `Index Scan` обычно эффективен.

---

# 3. Почему Index Scan обращается к таблице

Индекс обычно содержит:

```python
значение → ссылка на строку
```

Например:

```python
email
  ↓
TID
  ↓
строка таблицы
```

Если запрос делает:

```python
SELECT *
```

индекс сам по себе обычно не содержит все необходимые данные.

Поэтому PostgreSQL:

```python
Index
  ↓
Table
```

---

# 4. Index Only Scan

```python
Index Only Scan using users_email_idx on users
```

Здесь PostgreSQL может получить необходимые данные **из самого индекса**, не читая соответствующие строки таблицы в обычном режиме.

Например:

```python
CREATE INDEX users_email_idx
ON users(email);
```

Запрос:

```python
SELECT email
FROM users
WHERE email = 'test@example.com';
```

Поскольку нужный столбец `email` находится в индексе, PostgreSQL потенциально может использовать:

```python
Index Only Scan
```

Схема:

```python
Index
  ↓
Result
```

вместо:

```python
Index
  ↓
Table
  ↓
Result
```

---

# 5. Почему Index Only Scan не всегда полностью обходится без таблицы

В PostgreSQL есть **visibility map**.

Она помогает определить, можно ли считать страницы таблицы видимыми для всех необходимых транзакций.

Если PostgreSQL не может получить необходимую информацию о видимости из visibility map, ему может понадобиться обратиться к таблице.

Поэтому `Index Only Scan` не означает буквально:

> «Таблица вообще никогда не читается».

Важно понимать концепцию:

> PostgreSQL старается получить результат только из индекса, когда это возможно.

---

# 6. Bitmap Index Scan

```python
Bitmap Index Scan on users_age_idx
```

Это часть двухэтапного bitmap-плана.

Сначала PostgreSQL использует индекс и строит **bitmap** подходящих страниц/строк.

Например:

```python
Bitmap Index Scan
        ↓
      Bitmap
        ↓
Bitmap Heap Scan
```

Сам `Bitmap Index Scan` обычно не возвращает итоговые строки пользователю.

Он формирует информацию о том, **какие страницы таблицы нужно прочитать**.

---

# 7. Bitmap Heap Scan

После:

```python
Bitmap Index Scan
```

может выполняться:

```python
Bitmap Heap Scan
```

Он читает необходимые страницы таблицы.

Полная схема:

```python
          Index
            ↓
    Bitmap Index Scan
            ↓
         Bitmap
            ↓
    Bitmap Heap Scan
            ↓
        Table pages
            ↓
         Result
```

Это может быть выгодно, когда подходящих строк уже достаточно много и обычный `Index Scan` потребовал бы слишком большого количества случайных обращений к таблице.

---

# 8. Index Scan vs Bitmap Scan

Представим таблицу:

```python
1 000 000 строк
```

Запрос возвращает:

```python
5 строк
```

Часто выгоден:

```python
Index Scan
```

Потому что найдено очень мало строк.

Но если запрос возвращает:

```python
200 000 строк
```

PostgreSQL может выбрать:

```python
Bitmap Index Scan
        ↓
Bitmap Heap Scan
```

Потому что эффективнее сначала определить нужные страницы, а затем обработать их более организованно.

Это **не жёсткое правило** — окончательное решение принимает planner.

---

# 9. TID Scan

```python
Tid Scan on users
```

`TID` — **Tuple Identifier**.

В PostgreSQL физическое расположение строки можно представить через:

```python
ctid
```

Например:

```python
SELECT *
FROM users
WHERE ctid = '(10,5)';
```

PostgreSQL может использовать:

```python
Tid Scan
```

Схема:

```python
ctid
 ↓
физическое расположение строки
 ↓
строка
```

Это специализированный способ доступа.

`ctid` не следует использовать как обычный постоянный идентификатор строки: он может измениться после обновления строки.

---

# 10. Seq Scan vs Index Scan

|                                     | Seq Scan        | Index Scan             |
| ----------------------------------- | --------------- | ---------------------- |
| Использует индекс                   | ❌               | ✅                      |
| Читает таблицу                      | Последовательно | Через найденные строки |
| Хорош для большого количества строк | Часто да        | Часто нет              |
| Хорош для малого количества строк   | Не всегда       | Часто да               |
| Зависит от селективности            | ✅               | ✅                      |
| Всегда быстрее                      | ❌               | ❌                      |

---

# 11. Index Scan vs Index Only Scan

### Index Scan

```python
Index
  ↓
Table
```

Индекс используется для поиска, но нужные данные приходится получать из таблицы.

### Index Only Scan

```python
Index
  ↓
Result
```

Необходимые данные находятся в индексе, и PostgreSQL может избежать обычного чтения таблицы.

---

# 12. Index Scan vs Bitmap Scan

### Index Scan

```python
Index
 ↓
Row
 ↓
Row
 ↓
Row
```

Подходит, когда найдено относительно небольшое количество строк.

### Bitmap Scan

```python
Index
 ↓
Bitmap
 ↓
Pages
 ↓
Rows
```

Может быть выгоднее при большом количестве совпадений.

---

# 13. Что такое селективность

**Селективность** показывает, насколько сильно условие сокращает количество строк.

Например:

```python
WHERE id = 123
```

Если `id` уникален:

```python
1 000 000 строк
        ↓
      1 строка
```

Очень высокая селективность.

Индекс здесь обычно полезен.

---

Другой пример:

```python
WHERE gender = 'M'
```

Если половина таблицы соответствует условию:

```python
1 000 000 строк
        ↓
500 000 строк
```

Селективность значительно ниже.

В такой ситуации PostgreSQL может предпочесть:

```python
Seq Scan
```

---

# 14. Почему PostgreSQL может выбрать Seq Scan вместо индекса

Даже если индекс существует:

```python
CREATE INDEX users_age_idx
ON users(age);
```

PostgreSQL может выбрать:

```python
Seq Scan
```

Причины:

* таблица маленькая;
* запрос возвращает большую часть строк;
* индекс недостаточно селективен;
* последовательное чтение дешевле;
* статистика говорит planner, что Seq Scan выгоднее;
* стоимость обращения к таблице через индекс слишком высокая.

Поэтому:

> **Наличие индекса не гарантирует Index Scan.**

---

# 15. Как посмотреть тип сканирования

Используем:

```python
EXPLAIN ANALYZE
SELECT *
FROM users
WHERE email = 'test@example.com';
```

Смотрим на первую строку узла:

```python
Index Scan using users_email_idx on users
```

или:

```python
Seq Scan on users
```

или:

```python
Index Only Scan using users_email_idx on users
```

или:

```python
Bitmap Heap Scan on users
  -> Bitmap Index Scan on users_email_idx
```

---

# 16. Типичная структура Bitmap Scan

Например:

```python
Bitmap Heap Scan on users
  (actual time=1.000..10.000 rows=50000 loops=1)
  Recheck Cond: (age > 30)
  -> Bitmap Index Scan on users_age_idx
       (actual time=0.500..0.500 rows=50000 loops=1)
       Index Cond: (age > 30)
```

Читаем снизу вверх:

```python
Bitmap Index Scan
        ↓
нашли подходящие страницы/строки
        ↓
Bitmap Heap Scan
        ↓
прочитали страницы таблицы
        ↓
получили строки
```

---

# 17. `Recheck Cond`

В bitmap-плане можно увидеть:

```python
Recheck Cond: (age > 30)
```

Это означает, что PostgreSQL дополнительно проверяет условие при чтении таблицы.

Это особенно связано с тем, что bitmap представляет информацию на уровне страниц/битов и в некоторых случаях может быть **lossy**.

Поэтому PostgreSQL может повторно проверить условие на строках.

---

# 18. Полная картина

Упрощённо основные варианты выглядят так:

```python
SEQ SCAN

Table
 ↓
читаем последовательно
 ↓
Filter
 ↓
Result
```

```python
INDEX SCAN

Index
 ↓
найти строки
 ↓
Table
 ↓
Result
```

```python
INDEX ONLY SCAN

Index
 ↓
Result
```

```python
BITMAP SCAN

Index
 ↓
Bitmap Index Scan
 ↓
Bitmap
 ↓
Bitmap Heap Scan
 ↓
Table pages
 ↓
Result
```

```python
TID SCAN

ctid
 ↓
физическая строка
 ↓
Result
```

---

# 19. Что важно сказать на собеседовании

Если спрашивают:

> **«Какие типы сканирования вы знаете в PostgreSQL?»**

Хороший ответ:

> «Основные — Sequential Scan, Index Scan, Index Only Scan и Bitmap Scan, который обычно состоит из Bitmap Index Scan и Bitmap Heap Scan. Также существует специализированный TID Scan. Выбор типа сканирования делает planner на основе статистики и стоимости операций. Seq Scan не является автоматически плохим — при небольших таблицах или низкой селективности он может быть быстрее индекса.»

---

## 🧠 Главное

```python
Seq Scan
→ вся таблица последовательно

Index Scan
→ индекс + обращение к таблице

Index Only Scan
→ данные можно получить из индекса

Bitmap Index Scan
→ строит bitmap по индексу

Bitmap Heap Scan
→ читает страницы таблицы по bitmap

TID Scan
→ обращение по физическому адресу строки
```

Главная логика:

```python
мало подходящих строк
        ↓
   Index Scan

много подходящих строк
        ↓
 Bitmap Scan может быть выгоднее

нужно читать большую часть таблицы
        ↓
    Seq Scan может быть выгоднее

все необходимые данные есть в индексе
        ↓
 Index Only Scan может быть выгоднее
```

> **Тип сканирования выбирает PostgreSQL Planner. Нельзя оценивать его изолированно: нужно смотреть на объём данных, селективность, статистику и весь план выполнения.**
