# 🔄 Inversion of Control — IoC

## 🎯 Ответ на собеседовании

**Inversion of Control (IoC)** — это принцип, при котором управление созданием объектов, их зависимостями или выполнением определённой логики передаётся **внешнему компоненту или фреймворку**.

Обычно объект сам контролирует свои зависимости:

```python
class UserService:
    def __init__(self):
        self.repository = PostgreSQLRepository()
```

При IoC объект больше не решает самостоятельно, какую зависимость создать:

```python
class UserService:
    def __init__(self, repository):
        self.repository = repository
```

Теперь управление созданием `repository` находится снаружи.

**Dependency Injection — один из наиболее распространённых способов реализации IoC.**

---

## 🎤 Суперкоротко

```text
IoC = передача управления извне

DI = один из способов реализовать IoC

DIP = принцип проектирования зависимостей
```

Главная идея:

> **Не объект управляет всем сам — управление передаётся внешнему коду или фреймворку.**

---

# 🔍 Что именно инвертируется?

Без IoC:

```text
UserService
    ↓
сам создаёт Repository
    ↓
сам управляет зависимостью
```

С IoC:

```text
Внешний код / Framework
        ↓
создаёт Repository
        ↓
передаёт его
        ↓
UserService
```

То есть меняется **контроль над созданием и связыванием объектов**.

---

# ❌ Без IoC

```python
class UserService:
    def __init__(self):
        self.repository = PostgreSQLRepository()
```

`UserService` самостоятельно решает:

* какую реализацию использовать;
* когда её создать;
* как связать её с сервисом.

Контроль находится внутри класса.

---

# ✅ С IoC

```python
class UserService:
    def __init__(self, repository):
        self.repository = repository
```

Теперь:

```python
repository = PostgreSQLRepository()
service = UserService(repository)
```

Контроль находится во внешнем коде.

```text
Внешний код
    │
    ├── создаёт repository
    │
    └── передаёт его
           ↓
      UserService
```

---

# 💉 IoC через Dependency Injection

Самый простой пример IoC — Dependency Injection.

```python
class EmailService:
    def send(self, message):
        print(message)


class UserService:
    def __init__(self, email_service):
        self.email_service = email_service

    def register(self):
        self.email_service.send("User registered")
```

Создание объектов происходит снаружи:

```python
email_service = EmailService()
user_service = UserService(email_service)
```

`UserService` не создаёт `EmailService`.

Это и есть **инверсия контроля**.

---

# 🌐 IoC в FastAPI

FastAPI активно использует механизм Dependency Injection.

```python
from fastapi import Depends, FastAPI

app = FastAPI()


def get_repository():
    return PostgreSQLRepository()


@app.get("/users")
def get_users(repository=Depends(get_repository)):
    return repository.get_users()
```

Здесь endpoint не создаёт `PostgreSQLRepository` самостоятельно.

FastAPI:

```text
HTTP Request
      ↓
   FastAPI
      ↓
Depends(get_repository)
      ↓
создание Repository
      ↓
endpoint
```

**Фреймворк управляет созданием и передачей зависимости.**

Это пример IoC.

---

# ⚙️ IoC шире, чем DI

Очень важный момент:

**IoC — более широкое понятие, чем Dependency Injection.**

DI:

```text
IoC
└── Dependency Injection
```

DI касается прежде всего **передачи зависимостей**.

IoC может означать передачу управления и в других формах.

---

# 🔁 IoC в callback

Например:

```python
def on_finished():
    print("Finished")


def process(callback):
    print("Processing...")
    callback()
```

Мы передаём функцию `on_finished` внутрь другого компонента.

`process()` сам управляет основным процессом, но момент вызова callback контролируется уже внутри него.

Это тоже форма инверсии контроля.

---

# 🧩 IoC и фреймворки

Фреймворки особенно хорошо демонстрируют IoC.

Без фреймворка:

```text
Наш код
 ↓
управляет программой
 ↓
вызывает функции
```

С фреймворком:

```text
Фреймворк
 ↓
управляет жизненным циклом
 ↓
вызывает наш код
```

Например, мы пишем endpoint:

```python
@app.get("/users")
def get_users():
    return {"users": []}
```

Мы не вызываем `get_users()` самостоятельно.

Когда приходит HTTP-запрос, **FastAPI сам решает, когда вызвать нашу функцию**.

Это классический пример IoC:

> **Framework calls your code, rather than your code calling the framework's application flow.**

---

# 🆚 IoC, DI и DIP

| Термин  | Что это                                   |
| ------- | ----------------------------------------- |
| **IoC** | Общий принцип передачи контроля наружу    |
| **DI**  | Способ внедрения зависимостей извне       |
| **DIP** | Принцип SOLID о зависимости от абстракций |

Связь:

```text
IoC
│
├── Dependency Injection
│
├── Callbacks
│
└── Framework-controlled lifecycle
```

А DIP — это отдельный архитектурный принцип:

```text
DIP
↓
зависеть от абстракций
```

---

# 🧠 Простой пример для запоминания

Представь ресторан.

### Без IoC

Ты сам:

```text
покупаешь продукты
↓
готовишь еду
↓
выбираешь посуду
↓
убираешь
```

Ты контролируешь весь процесс.

### С IoC

Ты говоришь ресторану:

```text
"Мне нужна паста"
```

А ресторан сам решает:

```text
какие продукты взять
как приготовить
кто приготовит
когда подать
```

Ты передал управление ресторану.

В программировании эту роль может выполнять **фреймворк или контейнер зависимостей**.

---

# 🎯 Главное

```text
IoC
↓
Инверсия управления

Объект не управляет всем самостоятельно
↓
управление передаётся наружу
```

**DI** — один из способов реализовать IoC.

**DIP** — принцип SOLID, который говорит зависеть от абстракций, а не от конкретных реализаций.

### Формула для собеседования

```text
IoC = кто управляет?

DI  = как передаются зависимости?

DIP = от чего должны зависеть модули?
```

Запомнить можно так:

```text
IoC → передали контроль
DI  → передали зависимость
DIP → перевернули направление зависимости
```
