# 🔄 Dependency Inversion Principle — DIP

## 🎯 Ответ на собеседовании

**Dependency Inversion Principle (DIP)** — принцип инверсии зависимостей.

Он говорит:

1. **Модули высокого уровня не должны зависеть от модулей низкого уровня. Оба должны зависеть от абстракций.**
2. **Абстракции не должны зависеть от деталей. Детали должны зависеть от абстракций.**

То есть вместо того, чтобы бизнес-логика напрямую зависела от конкретной реализации, мы вводим абстракцию и передаём конкретную реализацию извне.

Главная идея:

> **Зависеть нужно от абстракции, а не от конкретной реализации.**

---

## 🎤 Суперкоротко

```text
Плохо:

UserService → PostgreSQLRepository

Хорошо:

UserService → Repository ← PostgreSQLRepository
```

`UserService` знает только об интерфейсе `Repository`, а не о PostgreSQL.

---

## 📌 Что такое зависимость

**Зависимость** — это объект или компонент, который нужен другому объекту для работы.

Например:

```python
class UserService:
    def __init__(self):
        self.repository = PostgreSQLRepository()
```

Здесь `UserService` напрямую зависит от `PostgreSQLRepository`.

Если мы захотим заменить PostgreSQL на Redis или MongoDB, придётся менять `UserService`.

---

# ❌ Нарушение DIP

Представим сервис пользователей:

```python
class PostgreSQLRepository:
    def get_user(self, user_id):
        return f"User {user_id}"


class UserService:
    def __init__(self):
        self.repository = PostgreSQLRepository()

    def get_user(self, user_id):
        return self.repository.get_user(user_id)
```

Проблема:

```text
UserService
     ↓
PostgreSQLRepository
```

`UserService` жёстко связан с конкретной реализацией.

Если понадобится:

```text
PostgreSQL
    ↓
MongoDB
    ↓
Redis
    ↓
API другого сервиса
```

придётся изменять `UserService`.

---

# ✅ Соблюдение DIP

Создаём абстракцию:

```python
from abc import ABC, abstractmethod


class UserRepository(ABC):
    @abstractmethod
    def get_user(self, user_id):
        pass
```

Конкретная реализация зависит от этой абстракции:

```python
class PostgreSQLRepository(UserRepository):
    def get_user(self, user_id):
        return f"User {user_id}"
```

А сервис работает с абстракцией:

```python
class UserService:
    def __init__(self, repository: UserRepository):
        self.repository = repository

    def get_user(self, user_id):
        return self.repository.get_user(user_id)
```

Теперь зависимости выглядят так:

```text
              UserRepository
             ↑              ↑
             │              │
     UserService    PostgreSQLRepository
```

`UserService` не знает, какая именно реализация используется.

---

# 💉 Dependency Injection

Здесь появляется **Dependency Injection (DI)** — внедрение зависимостей.

Зависимость не создаётся внутри класса:

```python
class UserService:
    def __init__(self):
        self.repository = PostgreSQLRepository()
```

Она передаётся извне:

```python
class UserService:
    def __init__(self, repository):
        self.repository = repository
```

Использование:

```python
repository = PostgreSQLRepository()
service = UserService(repository)
```

То есть:

```text
Внешний код
    ↓
создаёт PostgreSQLRepository
    ↓
передаёт его
    ↓
UserService
```

---

# ⚠️ DIP ≠ DI

Это важно на собеседовании.

**DIP** — архитектурный принцип.

**DI** — техника реализации, которая часто используется для соблюдения DIP.

```text
DIP
↓
Зависимость от абстракции

DI
↓
Передача зависимости извне
```

Можно сказать:

> **Dependency Injection — один из способов реализовать Dependency Inversion Principle.**

---

# 🧩 Пример с Protocol

В Python для абстракции необязательно использовать `ABC`.

Можно использовать `Protocol`:

```python
from typing import Protocol


class UserRepository(Protocol):
    def get_user(self, user_id: int) -> str:
        ...


class PostgreSQLRepository:
    def get_user(self, user_id: int) -> str:
        return f"User {user_id}"


class UserService:
    def __init__(self, repository: UserRepository):
        self.repository = repository

    def get_user(self, user_id: int):
        return self.repository.get_user(user_id)
```

Создание:

```python
repository = PostgreSQLRepository()
service = UserService(repository)

print(service.get_user(1))
```

`UserService` зависит от контракта `UserRepository`, а не от PostgreSQL.

---

# 🧪 DIP и тестирование

DIP сильно упрощает тестирование.

Можно вместо настоящей базы передать тестовую реализацию:

```python
class FakeRepository:
    def get_user(self, user_id):
        return "Test User"
```

Используем её:

```python
repository = FakeRepository()
service = UserService(repository)

print(service.get_user(1))
```

Схема:

```text
Production:

UserService → PostgreSQLRepository


Tests:

UserService → FakeRepository
```

Бизнес-логику не нужно менять ради тестов.

---

# 🌐 DIP в Backend

Например, FastAPI-приложение:

```text
API
 ↓
Service
 ↓
Repository
 ↓
Database
```

Без DIP сервис может напрямую создавать конкретный репозиторий:

```python
class UserService:
    def __init__(self):
        self.repository = PostgreSQLRepository()
```

С DIP:

```python
class UserService:
    def __init__(self, repository: UserRepository):
        self.repository = repository
```

А FastAPI или другой внешний слой решает, какую реализацию передать:

```text
FastAPI
   ↓
создаёт зависимости
   ↓
UserService
   ↓
UserRepository
   ↑
PostgreSQLRepository
```

---

# 🔗 Связь с другими принципами SOLID

### SRP

**Single Responsibility Principle**

Класс должен иметь одну ответственность.

```text
UserService → бизнес-логика
Repository  → работа с БД
```

---

### OCP

**Open/Closed Principle**

Код должен быть открыт для расширения, но закрыт для изменения.

DIP помогает добавлять новые реализации:

```text
UserRepository
    ├── PostgreSQLRepository
    ├── MongoRepository
    └── FakeRepository
```

Не изменяя `UserService`.

---

### DIP

**Dependency Inversion Principle**

Высокоуровневая логика зависит от абстракции:

```text
UserService
     ↓
  абстракция
     ↑
конкретная реализация
```

---

# 🆚 DIP и обычная зависимость

| Без DIP                           | С DIP                         |
| --------------------------------- | ----------------------------- |
| Зависимость от конкретного класса | Зависимость от абстракции     |
| Сильная связанность               | Слабая связанность            |
| Сложнее заменить реализацию       | Реализацию легко заменить     |
| Сложнее тестировать               | Удобнее тестировать           |
| Бизнес-логика знает детали        | Бизнес-логика не знает детали |

---

# ⚠️ Важный момент

DIP **не означает**, что абсолютно каждый класс нужно превращать в интерфейс.

Плохо:

```text
10 классов
↓
10 интерфейсов
↓
огромное количество абстракций
```

Абстракция нужна там, где есть смысл отделить высокоуровневую логику от конкретной реализации.

Например, для:

```text
БД
внешних API
очередей
хранилищ
платёжных систем
email-сервисов
```

это часто особенно полезно.

---

# 🧠 Главное

```text
Конкретная реализация
        ↓
   не должна быть
   центром зависимости
        ↓
      Абстракция
        ↑
        │
Высокоуровневая логика
```

**DIP:**

> Высокоуровневый код не должен зависеть от деталей. Оба должны зависеть от абстракции.

**DI:**

> Зависимость передаётся объекту извне, а не создаётся внутри него.

### Формула для собеседования

```text
DIP = зависеть от абстракций, а не от деталей

DI = передавать зависимости извне
```

# Связанные темы:

[[SOLID|SOLID]]