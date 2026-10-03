# 🗝️ Key-Value

## 🎤 Короткий ответ

**Key-Value** — это модель хранения данных, в которой каждому уникальному **ключу (`key`) соответствует значение (`value`)**.

Простейшая аналогия — Python `dict`:

```python
data = {
    "name": "Ilya",
    "age": 31,
}
```

Здесь:

```text
key → value

name → Ilya
age  → 31
```

Такая модель используется, например, в **Redis**, а также в других NoSQL-хранилищах.

---

## 🎯 Формула для собеседования

> **Key-Value = уникальный ключ → значение → быстрый доступ по ключу.**

```python id="q8v2mk"
Key
 ↓
Value
```

Например:

```python id="m5k9rx"
"user:42" → {"name": "Ilya", "age": 31}
```

---

# 🧠 Что такое Key-Value модель

Вместо таблиц и строк, как в реляционной БД:

```text
users
--------------------------------
id | name | age
1  | Ilya | 31
```

Key-Value хранилище концептуально работает так:

```text
"user:1" → {"name": "Ilya", "age": 31}
```

Основная операция:

```text
найти Value по Key
```

---

# 🔑 Key

**Key** — уникальный идентификатор значения.

Например:

```python id="h4p7zc"
"user:42"
"session:abc123"
"cache:product:100"
"rate_limit:192.0.2.10"
```

Хороший ключ должен быть:

* однозначным;
* предсказуемым;
* удобным для поиска;
* желательно иметь понятную структуру.

---

# 📦 Value

**Value** — данные, связанные с ключом.

В зависимости от конкретного хранилища это может быть:

```text
String
Number
JSON
Binary data
List
Set
Hash
```

Например:

```python id="y6m3vx"
"user:42" → "Ilya"
```

или:

```python id="p8q2ns"
"user:42" → {
    "name": "Ilya",
    "age": 31
}
```

В Redis значение может быть не только простой строкой — Redis поддерживает несколько структур данных.

---

# ⚡ Почему Key-Value быстро

Основной сценарий:

```text
GET(key)
   ↓
найти значение
   ↓
return value
```

Не нужно выполнять сложный SQL `JOIN`, группировку или поиск по множеству условий.

Например:

```python id="w3n7km"
GET user:42
```

Получаем:

```text
Ilya
```

---

# 🐍 Аналогия с Python dict

Самая простая аналогия:

```python id="x9c4pv"
users = {
    1: "Ilya",
    2: "Anna",
    3: "Bob",
}

print(users[1])
```

Результат:

```text
Ilya
```

Концептуально:

```text
Key 1 → Value "Ilya"
```

Но важно:

> **Key-Value database ≠ Python `dict`.**

`dict` находится в памяти конкретного процесса Python, а Key-Value хранилище обычно является отдельным сервисом, доступным по сети.

---

# 🗄️ Redis как Key-Value

Redis — один из самых известных примеров Key-Value storage.

```python id="k7m2qx"
SET user:42 "Ilya"
```

Получение:

```python id="c5v8nz"
GET user:42
```

Результат:

```text
Ilya
```

Можно задать TTL:

```python id="a4q9mw"
SET session:abc123 "user:42" EX 3600
```

Через 3600 секунд ключ автоматически истечёт.

---

# 🧩 Структура ключей

В Redis часто используют соглашение:

```text
entity:id
```

Например:

```python id="f3x7kp"
user:42
product:100
order:500
```

Для кэша:

```python id="n8q2vz"
cache:user:42
cache:product:100
```

Для сессии:

```python id="m6w4rs"
session:abc123
```

Это помогает организовать большое количество ключей.

---

# 🧱 Key-Value vs Relational Database

| Key-Value                            | Relational DB                      |
| ------------------------------------ | ---------------------------------- |
| Key → Value                          | Таблицы → строки → столбцы         |
| Быстрый доступ по ключу              | SQL-запросы                        |
| Простая модель                       | Богатая структура                  |
| Обычно нет JOIN как основной модели  | JOIN                               |
| Часто используется для cache/session | Часто используется как основная БД |
| Redis, DynamoDB                      | PostgreSQL, MySQL                  |

Пример:

### PostgreSQL

```python id="q5k8mp"
SELECT *
FROM users
WHERE id = 42;
```

### Redis

```python id="z7n3vx"
GET user:42
```

---

# 🆚 Key-Value vs Document Database

Они похожи, но модель отличается.

### Key-Value

```text
key → value
```

Например:

```text
user:42 → {...}
```

Основной доступ — по ключу.

### Document Database

Хранит документы и обычно предоставляет возможность искать по полям документа.

Например:

```python id="c8m4qy"
{
    "id": 42,
    "name": "Ilya",
    "age": 31
}
```

Можно иметь запросы по содержимому документа, в зависимости от конкретной БД.

---

# 🧮 Типичные операции

Классические операции:

```text
SET
GET
DELETE
EXISTS
```

В Redis:

```python id="v2x7nk"
SET user:42 "Ilya"
GET user:42
EXISTS user:42
DEL user:42
```

Можно представить API так:

```text
PUT(key, value)
GET(key)
DELETE(key)
```

---

# 🚀 Где используется Key-Value

## 1. Cache

```text
cache:user:42
       ↓
данные пользователя
```

Вместо обращения к PostgreSQL при каждом запросе.

---

## 2. Sessions

```text
session:abc123
       ↓
user_id = 42
```

---

## 3. Rate Limiting

```text
rate:user:42
       ↓
87 requests
```

---

## 4. Counters

```text
views:article:100
       ↓
1524
```

Например:

```python id="e6w3mp"
INCR views:article:100
```

---

## 5. Временные данные

Например, код подтверждения:

```text
verification:42
       ↓
"583921"
```

с TTL:

```text
5 минут
```

---

# ⚠️ Ограничения Key-Value

Главное ограничение:

> **Модель оптимизирована прежде всего под доступ по ключу.**

Например, у нас:

```text
user:1
user:2
user:3
...
```

И мы хотим:

> Найти всех пользователей старше 30 лет.

В реляционной БД:

```python id="m2q7vx"
SELECT *
FROM users
WHERE age > 30;
```

В простом Key-Value подходе такой запрос не является основным сценарием.

Для сложных запросов, связей и аналитики реляционная БД обычно подходит лучше.

---

# 🔄 Key-Value в архитектуре Backend

Очень распространённая схема:

```text
             Client
                ↓
             FastAPI
                ↓
          ┌─────┴─────┐
          ↓           ↓
       Redis       PostgreSQL
          ↓           ↓
       Cache      Source of Truth
```

Redis:

```text
Key → Cached Value
```

PostgreSQL:

```text
Tables → Relations → Business Data
```

---

# 🧠 Key-Value и кэш

Например, запрос пользователя:

```text
GET /users/42
```

Backend:

```text
FastAPI
   ↓
Redis
   ↓
GET user:42
```

### Cache Hit

```text
Redis
 ↓
user:42 найден
 ↓
Response
```

### Cache Miss

```text
Redis
 ↓
user:42 отсутствует
 ↓
PostgreSQL
 ↓
получили User
 ↓
Redis SET user:42
 ↓
Response
```

---

# ⚠️ Key-Value не означает обязательно Redis

Redis — только один пример.

К Key-Value хранилищам относятся различные системы, например:

* Redis;
* Amazon DynamoDB;
* Amazon ElastiCache for Redis;
* Memcached — в основном cache-oriented key-value store.

При этом конкретные возможности, гарантии консистентности, persistence и модель масштабирования у них отличаются.

---

# 🎤 Как рассказать на собеседовании

> **Key-Value — это модель хранения данных, где каждому ключу соответствует значение. Основная операция — быстрый доступ к значению по ключу. Например, Redis использует такую модель и позволяет хранить строки, Hash, List, Set и другие структуры. Key-Value хорошо подходит для кэша, сессий, счётчиков и временных данных, но для сложных связей и запросов по множеству полей обычно лучше использовать реляционную БД.**

---

# ❓ Частые вопросы

### Что такое Key-Value?

Модель хранения:

```text
Key → Value
```

где значение получается по уникальному ключу.

### Пример Key-Value?

```text
"user:42" → "Ilya"
```

или:

```text
"session:abc" → {"user_id": 42}
```

### Redis — Key-Value база?

Да, Redis относится к in-memory data stores с моделью key-value и дополнительными структурами данных.

### Почему Key-Value быстро?

Потому что основной сценарий — прямой доступ к данным по ключу без сложных реляционных операций.

### Где использовать Key-Value?

* Cache;
* Sessions;
* Rate Limiting;
* Counters;
* временные данные;
* некоторые очереди/событийные сценарии.

### Можно ли заменить PostgreSQL на Key-Value?

Зависит от задачи, но **не стоит рассматривать Key-Value как универсальную замену реляционной БД**. Если нужны сложные запросы, JOIN, связи и транзакционная модель реляционной БД — PostgreSQL обычно естественнее.

---

## 🔑 Главное

```python id="u7m3qx"
Key-Value

       Key
        ↓
      Value

"user:42"
     ↓
"Ilya"
```

**Key-Value = простой способ хранения и быстрого получения значения по ключу.**

В контексте Backend:

```python id="p4k8nz"
Redis
  ↓
Key → Value
  ↓
Cache / Sessions / Counters / TTL
```
