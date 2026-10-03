# 🔁 SELF JOIN в SQL

## 🎯 Ответ на собеседовании

**`SELF JOIN` — это соединение таблицы с самой собой.**

Физически таблица одна, но в запросе мы обращаемся к ней через **разные алиасы**, чтобы сравнить или связать её строки между собой.

Например, если в таблице сотрудников хранится `manager_id`:

```python
SELECT
    e.name AS employee,
    m.name AS manager
FROM employees AS e
LEFT JOIN employees AS m
    ON e.manager_id = m.id;
```

Здесь `employees` используется дважды:

* `e` — сотрудник;
* `m` — его руководитель.

---

## 🎤 Суперкоротко

```text
SELF JOIN
→ таблица JOIN сама с собой
→ используются разные алиасы
→ позволяет связать строки одной таблицы между собой
```

Важно:

> `SELF JOIN` — это не отдельный оператор SQL. Это обычный `JOIN`, в котором одна и та же таблица участвует с двух сторон.

---

# 📌 Главный пример — сотрудники и руководители

Таблица:

```text
employees

id | name   | manager_id
---+--------+-----------
1  | Илья   | NULL
2  | Анна   | 1
3  | Максим | 1
4  | Олег   | 2
```

Здесь:

```text
Илья
├── Анна
│   └── Олег
└── Максим
```

`manager_id` содержит `id` руководителя.

---

# 🔹 Получить сотрудника и его руководителя

Используем `SELF JOIN`:

```python
SELECT
    e.name AS employee,
    m.name AS manager
FROM employees AS e
LEFT JOIN employees AS m
    ON e.manager_id = m.id;
```

Результат:

```text
employee | manager
---------+--------
Илья     | NULL
Анна     | Илья
Максим   | Илья
Олег     | Анна
```

Обрати внимание:

```text
employees AS e
```

и:

```text
employees AS m
```

— это **одна и та же таблица**, но SQL рассматривает их как две роли:

```text
e → employee
m → manager
```

---

# 🔹 Почему нужны алиасы

Нельзя просто написать:

```python
SELECT employees.name, employees.name
FROM employees
JOIN employees
    ON employees.manager_id = employees.id;
```

SQL не поймёт, какую роль играет каждый экземпляр таблицы.

Поэтому пишем:

```python
FROM employees AS e
JOIN employees AS m
```

Теперь однозначно:

```text
e.id
→ ID сотрудника

e.manager_id
→ ID руководителя

m.id
→ ID руководителя

m.name
→ имя руководителя
```

---

# 🔹 Как работает условие JOIN

Главное условие:

```python
ON e.manager_id = m.id
```

Читаем буквально:

> Найди в таблице `employees` человека `m`, чей `id` совпадает с `manager_id` сотрудника `e`.

Например:

```text
Анна:

e.id = 2
e.manager_id = 1
```

Ищем:

```text
m.id = 1
```

Находим:

```text
m.name = Илья
```

Получаем:

```text
Анна → Илья
```

---

# 🔹 Почему здесь LEFT JOIN

Мы использовали:

```python
LEFT JOIN
```

а не:

```python
INNER JOIN
```

Потому что у руководителя верхнего уровня:

```text
Илья | manager_id = NULL
```

нет руководителя.

`LEFT JOIN` сохранит Илью:

```text
Илья | NULL
```

Если использовать:

```python
INNER JOIN
```

то Илья исчезнет из результата.

---

# 🔹 SELF JOIN с INNER JOIN

Если нужны только сотрудники, у которых есть руководитель:

```python
SELECT
    e.name AS employee,
    m.name AS manager
FROM employees AS e
INNER JOIN employees AS m
    ON e.manager_id = m.id;
```

Результат:

```text
employee | manager
---------+--------
Анна     | Илья
Максим   | Илья
Олег     | Анна
```

Илья не попадёт, потому что:

```text
manager_id = NULL
```

---

# 🔹 Найти сотрудников, работающих под одним руководителем

Можно снова использовать `SELF JOIN`.

Например, найти пары сотрудников с одним менеджером:

```python
SELECT
    e1.name AS employee_1,
    e2.name AS employee_2
FROM employees AS e1
JOIN employees AS e2
    ON e1.manager_id = e2.manager_id
WHERE e1.id < e2.id;
```

Результат:

```text
employee_1 | employee_2
-----------+-----------
Анна       | Максим
```

Почему:

```text
Анна   → manager_id = 1
Максим → manager_id = 1
```

Они имеют одного руководителя.

Условие:

```python
e1.id < e2.id
```

нужно, чтобы не получить одновременно:

```text
Анна → Максим
Максим → Анна
```

---

# 🔹 Найти сотрудников без руководителя

Здесь тоже можно использовать `SELF JOIN`:

```python
SELECT
    e.name
FROM employees AS e
LEFT JOIN employees AS m
    ON e.manager_id = m.id
WHERE m.id IS NULL;
```

Получим:

```text
Илья
```

Логика:

```text
employee
    ↓
manager
    ↓
не найден
    ↓
m.id IS NULL
```

---

# 🔹 SELF JOIN не обязательно означает `LEFT JOIN`

Можно использовать любой подходящий тип JOIN:

```python
INNER JOIN
LEFT JOIN
RIGHT JOIN
FULL OUTER JOIN
```

Например:

```python
SELECT ...
FROM employees AS e
INNER JOIN employees AS m
    ON ...
```

Это всё равно `SELF JOIN`.

Определяющий признак:

```text
одна таблица используется
несколько раз в одном JOIN
```

---

# 🔹 SELF JOIN и иерархические данные

Это одно из главных применений.

Например:

```text
Категории товаров

id | name        | parent_id
---+-------------+----------
1  | Электроника | NULL
2  | Телефоны    | 1
3  | Ноутбуки    | 1
4  | iPhone      | 2
```

Связь:

```text
Электроника
├── Телефоны
│   └── iPhone
└── Ноутбуки
```

Получить категорию и её родителя:

```python
SELECT
    c.name AS category,
    p.name AS parent_category
FROM categories AS c
LEFT JOIN categories AS p
    ON c.parent_id = p.id;
```

Результат:

```text
category    | parent_category
------------+----------------
Электроника | NULL
Телефоны    | Электроника
Ноутбуки    | Электроника
iPhone      | Телефоны
```

---

# 🔹 SELF JOIN и временные данные

SELF JOIN также может использоваться для сравнения строк одной таблицы.

Например, найти товары с одинаковой ценой:

```python
SELECT
    p1.name AS product_1,
    p2.name AS product_2
FROM products AS p1
JOIN products AS p2
    ON p1.price = p2.price
WHERE p1.id < p2.id;
```

Например:

```text
product_1 | product_2
----------+----------
Phone A   | Phone B
Laptop A  | Laptop C
```

---

# 🔹 SELF JOIN для сравнения соседних/связанных записей

Например, таблица:

```text
employees

id | name   | salary
---+--------+-------
1  | Илья   | 100000
2  | Анна   | 80000
3  | Максим | 120000
```

Можно сравнивать сотрудников между собой:

```python
SELECT
    e1.name AS employee,
    e2.name AS other_employee
FROM employees AS e1
JOIN employees AS e2
    ON e1.salary < e2.salary
WHERE e1.id <> e2.id;
```

Но здесь результат может содержать много комбинаций, поэтому условие JOIN нужно проектировать внимательно.

---

# ⚠️ SELF JOIN может сильно увеличить количество строк

Например:

```text
employees = 10 000 строк
```

Если условие позволяет соединить много строк между собой, результат может стать огромным.

Особенно опасны условия вроде:

```python
ON e1.salary < e2.salary
```

потому что одна строка может совпасть с большим количеством других.

---

# 🔹 SELF JOIN vs обычный JOIN

Обычный JOIN:

```text
users
  ↓
orders
```

Разные таблицы:

```python
FROM users AS u
JOIN orders AS o
```

SELF JOIN:

```text
employees
   ↙    ↘
employee manager
```

Одна таблица:

```python
FROM employees AS e
JOIN employees AS m
```

Разница только в том, что одна и та же таблица используется с двух сторон.

---

# 📊 Сравнение

|                     | Обычный JOIN | SELF JOIN                        |
| ------------------- | ------------ | -------------------------------- |
| Таблицы             | Разные       | Одна и та же                     |
| Алиасы              | Желательны   | Практически обязательны          |
| `ON`                | Обычно есть  | Обычно есть                      |
| Основное применение | Связь таблиц | Связь строк внутри одной таблицы |
| Иерархии            | Возможно     | Очень удобно                     |
| Сравнение строк     | Возможно     | Частый сценарий                  |

---

# 🧠 Ментальная модель

Представь, что SQL временно создаёт две роли одной таблицы:

```text
             employees
              /      \
             /        \
            ↓          ↓
     employees       employees
         e                m
     employee          manager
```

И соединяет их:

```python
ON e.manager_id = m.id
```

То есть:

```text
строка сотрудника
       ↓
manager_id
       ↓
строка руководителя
       ↓
тот же employees
```

---

# 🎤 Как ответить на собеседовании

**Вопрос: Что такое SELF JOIN?**

> `SELF JOIN` — это соединение таблицы самой с собой. Обычно используются разные алиасы, чтобы одна копия таблицы представляла одну роль, а другая — другую. Например, в таблице сотрудников можно соединить сотрудника с его руководителем через `manager_id`.

Пример:

```python
SELECT
    e.name AS employee,
    m.name AS manager
FROM employees AS e
LEFT JOIN employees AS m
    ON e.manager_id = m.id;
```

**Вопрос: Это отдельный тип JOIN?**

> Нет. `SELF JOIN` — это не отдельный оператор SQL. Это обычный `JOIN`, в котором одна и та же таблица участвует несколько раз.

**Вопрос: Где применяется?**

> Чаще всего для иерархических данных: сотрудники и руководители, категории и родительские категории, комментарии и родительские комментарии. Также его используют для сравнения строк одной таблицы.

---

# 🎯 Главное

```text
SELF JOIN
    ↓
таблица соединяется сама с собой
```

Главный шаблон:

```python
SELECT ...
FROM table AS a
JOIN table AS b
    ON a.some_id = b.other_id;
```

Для иерархии:

```python
SELECT
    e.name AS employee,
    m.name AS manager
FROM employees AS e
LEFT JOIN employees AS m
    ON e.manager_id = m.id;
```

Запомнить:

```text
SELF JOIN ≠ отдельный оператор

SELF JOIN
→ обычный JOIN
→ одна и та же таблица
→ разные алиасы
→ связываем строки этой таблицы между собой
```
