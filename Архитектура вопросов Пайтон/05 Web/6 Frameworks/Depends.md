# 💉 `Depends` в FastAPI

## 🎯 Ответ на собеседовании

`Depends` в **FastAPI** — механизм для реализации **Dependency Injection (DI)**.

С его помощью FastAPI автоматически получает и передаёт в endpoint необходимые зависимости.

Вместо того чтобы создавать зависимость внутри обработчика:

```python
@app.get("/users")
def get_users():
    repository = UserRepository()
    return repository.get_users()
```

мы объявляем её параметром:

```python
@app.get("/users")
def get_users(repository=Depends(get_repository)):
    return repository.get_users()
```

FastAPI сам вызывает `get_repository()`, получает результат и передаёт его в `repository`.

То есть:

```text
Depends
   ↓
FastAPI вызывает dependency
   ↓
получает результат
   ↓
передаёт его в endpoint
```

**`Depends` — это инструмент FastAPI для Dependency Injection.**

---

## 🎤 Суперкоротко

```text
Depends = "FastAPI, сам получи эту зависимость и передай её сюда"
```

Например:

```python
def get_db():
    return db


@app.get("/users")
def get_users(db=Depends(get_db)):
    ...
```

FastAPI сам вызовет `get_db()`.

---

# 🔌 Простейший пример

Создадим dependency:

```python
def get_repository():
    return UserRepository()
```

Используем её:

```python
from fastapi import Depends, FastAPI

app = FastAPI()


@app.get("/users")
def get_users(repository=Depends(get_repository)):
    return repository.get_users()
```

Что происходит:

```text
HTTP Request
     ↓
FastAPI
     ↓
get_repository()
     ↓
UserRepository
     ↓
get_users(repository)
     ↓
Response
```

---

# 🧩 Что такое Dependency

Dependency — это функция или callable, результат которого нужен другому компоненту.

Например:

```python
def get_current_user():
    return current_user
```

Она может быть dependency:

```python
@app.get("/profile")
def profile(user=Depends(get_current_user)):
    return user
```

Здесь:

```text
get_current_user
       ↓
     user
       ↓
    profile()
```

---

# 💉 Почему это Dependency Injection

Без DI:

```python
class UserService:
    def __init__(self):
        self.repository = UserRepository()
```

Класс сам создаёт зависимость.

С DI:

```python
class UserService:
    def __init__(self, repository):
        self.repository = repository
```

Зависимость передаётся извне.

В FastAPI:

```python
@app.get("/users")
def get_users(
    repository=Depends(get_repository),
):
    ...
```

FastAPI выступает как механизм управления зависимостями.

---

# 🗄️ Типичный пример с базой данных

Dependency:

```python
def get_db():
    db = SessionLocal()

    try:
        yield db
    finally:
        db.close()
```

Endpoint:

```python
@app.get("/users")
def get_users(db=Depends(get_db)):
    return db.query(User).all()
```

Схема:

```text
Request
   ↓
FastAPI
   ↓
get_db()
   ↓
создание DB Session
   ↓
endpoint
   ↓
работа с БД
   ↓
finally
   ↓
закрытие Session
```

Это особенно удобно для управления ресурсами.

---

# 🔐 `Depends` для авторизации

Например, получение текущего пользователя:

```python
def get_current_user():
    return User(id=1, name="Ilya")
```

Используем:

```python
@app.get("/profile")
def get_profile(
    user=Depends(get_current_user),
):
    return user
```

Теперь каждый endpoint может использовать одну и ту же dependency:

```python
@app.get("/profile")
def profile(user=Depends(get_current_user)):
    return user


@app.get("/orders")
def orders(user=Depends(get_current_user)):
    return user.orders
```

Общая логика не дублируется.

---

# 🌳 Цепочка зависимостей

Dependency сама может иметь dependency.

Например:

```python
def get_db():
    return db


def get_user(
    db=Depends(get_db),
):
    return get_user_from_db(db)


@app.get("/profile")
def profile(
    user=Depends(get_user),
):
    return user
```

Получается дерево:

```text
profile
   ↓
get_user
   ↓
get_db
```

FastAPI анализирует эту цепочку и разрешает зависимости.

---

# 🔄 `Depends` и `async`

Dependency может быть обычной функцией:

```python
def get_repository():
    return UserRepository()
```

А может быть асинхронной:

```python
async def get_repository():
    return UserRepository()
```

И endpoint тоже может быть асинхронным:

```python
@app.get("/users")
async def get_users(
    repository=Depends(get_repository),
):
    return await repository.get_users()
```

FastAPI учитывает, является ли dependency синхронной или асинхронной.

---

# 🧪 `Depends` и тестирование

Одна из сильных сторон DI — возможность заменить dependency при тестировании.

Например, production dependency:

```python
def get_repository():
    return PostgreSQLRepository()
```

В тесте можно подставить fake:

```python
def get_fake_repository():
    return FakeRepository()
```

FastAPI позволяет переопределять зависимости.

Концептуально:

```text
Production:

Depends(get_repository)
        ↓
PostgreSQLRepository


Tests:

Depends(get_repository)
        ↓
FakeRepository
```

Endpoint при этом менять не нужно.

---

# 🏗️ `Depends` и SOLID

`Depends` хорошо сочетается с **Dependency Inversion Principle**.

Например:

```python
class UserService:
    def __init__(self, repository):
        self.repository = repository
```

Сервис зависит не от конкретного PostgreSQL-класса, а от переданной зависимости.

FastAPI затем связывает конкретные реализации:

```text
UserService
     ↓
Repository abstraction
     ↑
PostgreSQLRepository
```

`Depends` помогает организовать эту передачу зависимостей на уровне приложения.

---

# 🔄 IoC → DI → Depends

Это важно понимать как одну цепочку.

### IoC

**Inversion of Control**:

```text
управление передаётся извне
```

### DI

**Dependency Injection**:

```text
зависимость передаётся извне
```

### `Depends`

Конкретный механизм FastAPI:

```text
FastAPI автоматически
разрешает и передаёт dependency
```

Получается:

```text
IoC
 ↓
DI
 ↓
Depends
 ↓
FastAPI Dependency Injection
```

---

# ⚠️ Важный момент

`Depends` **не является самой зависимостью**.

Например:

```python
Depends(get_repository)
```

Здесь:

```text
get_repository
    ↓
сама dependency

Depends(...)
    ↓
указание FastAPI,
что эту dependency нужно внедрить
```

То есть:

> `get_repository` — dependency, а `Depends(get_repository)` — декларация зависимости для FastAPI.

---

# 🆚 Обычный вызов vs `Depends`

### Обычный вызов

```python
@app.get("/users")
def get_users():
    repository = get_repository()
    return repository.get_users()
```

Endpoint сам управляет получением зависимости.

### `Depends`

```python
@app.get("/users")
def get_users(
    repository=Depends(get_repository),
):
    return repository.get_users()
```

Получением зависимости занимается FastAPI.

---

# 🎯 Главное

```text
Depends
   ↓
FastAPI видит dependency
   ↓
вызывает её
   ↓
получает результат
   ↓
передаёт результат
   ↓
в endpoint
```

Запомнить:

```text
Dependency Injection
        ↓
передача зависимостей извне

Depends
        ↓
механизм DI в FastAPI
```

### Формула для собеседования

> **`Depends` — это механизм FastAPI для Dependency Injection, который позволяет объявлять зависимости endpoint'а, а FastAPI автоматически вызывает их, разрешает цепочку зависимостей и передаёт полученные значения в обработчик.**

```text
IoC → передали контроль
DI  → передали зависимость
Depends → FastAPI автоматически внедряет зависимость
```
