# 💉 Dependency Injection — DI

## 🎯 Ответ на собеседовании

**Dependency Injection (DI)** — это паттерн внедрения зависимостей, при котором объект получает необходимые ему зависимости **извне**, вместо того чтобы создавать их самостоятельно.

Главная идея:

> **Класс не создаёт свои зависимости — ему их передают.**

Это уменьшает связанность между компонентами, упрощает замену реализаций и делает код удобнее для тестирования.

Например, вместо:

```python
class UserService:
    def __init__(self):
        self.repository = PostgreSQLRepository()
```

делаем:

```python
class UserService:
    def __init__(self, repository):
        self.repository = repository
```

Теперь `UserService` не знает, как создаётся `repository` и какая конкретно реализация используется.

---

## 🎤 Суперкоротко

```text
Без DI:

UserService → сам создаёт → PostgreSQLRepository


С DI:

Внешний код → создаёт → PostgreSQLRepository
                         ↓
                    UserService
```

**DI = зависимость передаётся объекту извне.**

---

# 📦 Что такое зависимость

Зависимость — это объект, который нужен классу для выполнения своей работы.

Например:

```python
class UserService:
    def __init__(self, repository):
        self.repository = repository
```

Здесь `repository` — зависимость `UserService`.

---

# ❌ Без Dependency Injection

Класс сам создаёт необходимый объект:

```python
class UserService:
    def __init__(self):
        self.repository = PostgreSQLRepository()

    def get_user(self, user_id):
        return self.repository.get_user(user_id)
```

Получается сильная связанность:

```text
UserService
    ↓
PostgreSQLRepository
```

`UserService` жёстко привязан к PostgreSQL.

Если понадобится `MongoRepository`, придётся изменять сам `UserService`.

---

# ✅ С Dependency Injection

Зависимость передаётся через конструктор:

```python
class UserService:
    def __init__(self, repository):
        self.repository = repository

    def get_user(self, user_id):
        return self.repository.get_user(user_id)
```

Теперь внешний код решает, какую зависимость использовать:

```python
repository = PostgreSQLRepository()
service = UserService(repository)
```

Зависимость:

```text
PostgreSQLRepository
        ↓
    передаётся
        ↓
   UserService
```

---

# 🏗️ Основные способы Dependency Injection

## 1. Constructor Injection

Самый распространённый и предпочтительный вариант.

Зависимость передаётся через `__init__`:

```python
class UserService:
    def __init__(self, repository):
        self.repository = repository
```

Преимущества:

* зависимость явно видна;
* объект нельзя случайно создать без обязательной зависимости;
* удобно тестировать;
* легко использовать type hints.

---

## 2. Setter Injection

Зависимость передаётся после создания объекта:

```python
class UserService:
    def set_repository(self, repository):
        self.repository = repository
```

Использование:

```python
service = UserService()
service.set_repository(PostgreSQLRepository())
```

Минус — объект может некоторое время находиться в некорректном состоянии без зависимости.

Поэтому для обязательных зависимостей обычно лучше использовать **constructor injection**.

---

## 3. Method Injection

Зависимость передаётся непосредственно в метод:

```python
class UserService:
    def save_user(self, repository, user):
        repository.save(user)
```

Зависимость нужна только конкретному методу, поэтому нет смысла хранить её в объекте.

---

# 🧩 DI + абстракция

Особенно полезен DI вместе с абстракциями.

Например:

```python
from abc import ABC, abstractmethod


class UserRepository(ABC):
    @abstractmethod
    def get_user(self, user_id):
        pass
```

Конкретная реализация:

```python
class PostgreSQLRepository(UserRepository):
    def get_user(self, user_id):
        return f"User {user_id}"
```

Сервис:

```python
class UserService:
    def __init__(self, repository: UserRepository):
        self.repository = repository

    def get_user(self, user_id):
        return self.repository.get_user(user_id)
```

Теперь можно передать любую реализацию `UserRepository`:

```python
repository = PostgreSQLRepository()
service = UserService(repository)
```

---

# 🧪 DI и тестирование

Одно из главных преимуществ DI — удобное тестирование.

Вместо реальной базы данных можно передать mock или fake:

```python
class FakeRepository:
    def get_user(self, user_id):
        return "Test User"
```

Используем:

```python
service = UserService(FakeRepository())

print(service.get_user(1))
```

Сервису всё равно, откуда пришёл repository.

```text
Production:

UserService
     ↓
PostgreSQLRepository


Tests:

UserService
     ↓
FakeRepository
```

---

# 🌐 Dependency Injection в FastAPI

В FastAPI DI является частью самого фреймворка.

Например:

```python
from fastapi import Depends, FastAPI

app = FastAPI()


def get_repository():
    return PostgreSQLRepository()


@app.get("/users/{user_id}")
def get_user(
    user_id: int,
    repository=Depends(get_repository),
):
    return repository.get_user(user_id)
```

FastAPI:

1. вызывает `get_repository()`;
2. получает зависимость;
3. передаёт её в обработчик;
4. обработчик использует готовый объект.

Схематично:

```text
HTTP Request
     ↓
FastAPI
     ↓
Depends(get_repository)
     ↓
PostgreSQLRepository
     ↓
endpoint
```

---

# 🔄 DI и DIP

Эти понятия часто путают.

### Dependency Inversion Principle

**DIP** — принцип проектирования:

> Высокоуровневые модули должны зависеть от абстракций, а не от конкретных реализаций.

### Dependency Injection

**DI** — техника:

> Передавать зависимости объекту извне.

```text
DIP
↓
Архитектурный принцип


DI
↓
Способ организовать зависимости
```

DI часто используется для реализации DIP, но это **не одно и то же**.

---

# 🆚 Без DI и с DI

| Без DI                        | С DI                         |
| ----------------------------- | ---------------------------- |
| Класс создаёт зависимость сам | Зависимость передаётся извне |
| Сильная связанность           | Слабая связанность           |
| Сложнее заменить реализацию   | Легко заменить реализацию    |
| Сложнее тестировать           | Удобно тестировать           |
| Скрытые зависимости           | Явные зависимости            |

---

# 💡 Главная идея

Плохой подход:

```python
class Service:
    def __init__(self):
        self.db = PostgreSQLDatabase()
```

Хороший подход:

```python
class Service:
    def __init__(self, db):
        self.db = db
```

Разница в одной вещи:

```text
Кто создаёт зависимость?
```

**Без DI:**

```text
Service → создаёт зависимость
```

**С DI:**

```text
Внешний код → создаёт зависимость → Service
```

---

# 🧠 Главное для собеседования

**Dependency Injection** — это способ передачи зависимостей объекту извне вместо их создания внутри объекта.

Наиболее распространённый вариант — **constructor injection**.

DI:

* уменьшает связанность;
* делает зависимости явными;
* упрощает тестирование;
* позволяет легко заменять реализации;
* часто используется вместе с DIP.

### Формула

```text
DI = не создавать зависимость внутри,
     а передать её извне
```
