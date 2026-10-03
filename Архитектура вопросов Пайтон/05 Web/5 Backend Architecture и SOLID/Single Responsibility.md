# 🅢 Single Responsibility Principle — принцип единственной ответственности

## 🎯 Ответ на собеседовании

**Single Responsibility Principle (SRP)** — принцип единственной ответственности.

Он говорит, что **класс должен иметь одну ответственность и одну причину для изменения**.

Важно: SRP **не означает, что в классе должен быть только один метод**. Речь идёт о том, что класс должен отвечать за одну логически связанную область поведения.

Например, класс `UserService` может содержать несколько методов, связанных с бизнес-логикой пользователей, но не должен одновременно отвечать за работу с БД, отправку email и генерацию отчётов.

---

## 🎤 Суперкоротко

```text
SRP
 ↓
Одна ответственность
 ↓
Одна причина для изменения
```

Запомнить:

> **Один класс — одна ответственность.**

---

# 🧩 Что такое ответственность?

Ответственность — это **область обязанностей класса**.

Например:

```text
UserRepository
→ работа с БД

EmailService
→ отправка email

ReportGenerator
→ создание отчётов

UserService
→ бизнес-логика пользователей
```

У каждого класса своя область ответственности.

---

# ❌ Нарушение SRP

Допустим, есть такой класс:

```python
class User:
    def create_user(self):
        pass

    def save_to_database(self):
        pass

    def send_email(self):
        pass

    def generate_report(self):
        pass
```

Здесь `User` делает слишком много разных вещей:

```text
User
 ├── создание пользователя
 ├── работа с БД
 ├── отправка email
 └── генерация отчётов
```

У него несколько причин для изменения.

Например:

```text
Изменилась БД
    ↓
нужно менять User

Изменился способ отправки email
    ↓
нужно менять User

Изменился формат отчёта
    ↓
нужно менять User
```

Это нарушение SRP.

---

# ✅ Соблюдение SRP

Разделяем ответственности:

```python
class UserService:
    def create_user(self):
        pass


class UserRepository:
    def save(self, user):
        pass


class EmailService:
    def send(self, email):
        pass


class ReportGenerator:
    def generate(self, user):
        pass
```

Теперь:

```text
UserService
    ↓
бизнес-логика

UserRepository
    ↓
БД

EmailService
    ↓
email

ReportGenerator
    ↓
отчёты
```

Каждый класс имеет свою причину для изменения.

---

# 🎯 «Одна ответственность» ≠ «один метод»

Это очень частая ошибка.

Плохо понимать SRP так:

```python
class UserService:

    def create_user(self):
        pass
```

и считать, что именно один метод автоматически означает соблюдение SRP.

Класс вполне может иметь много методов:

```python
class UserService:

    def create_user(self):
        pass

    def update_user(self):
        pass

    def delete_user(self):
        pass

    def get_user(self):
        pass
```

Все эти методы могут относиться к **одной ответственности — бизнес-логике пользователей**.

Поэтому правильный вопрос:

> **«Есть ли у класса несколько независимых причин для изменения?»**

---

# 🔄 Как определить нарушение SRP

Полезный практический вопрос:

> **«Если изменится X, придётся ли менять этот класс?»**

Например:

```text
Изменилась база данных
        ↓
UserRepository изменится
```

Это нормально.

```text
Изменился SMTP-сервер
        ↓
EmailService изменится
```

Это нормально.

Но если:

```text
Изменилась БД
        ↓
User изменился

Изменился email
        ↓
User изменился

Изменился отчёт
        ↓
User изменился
```

то у `User` слишком много ответственности.

---

# 🧠 SRP и связность

SRP помогает повысить **cohesion (связность внутри компонента)** и уменьшить ненужную связанность между компонентами.

Хорошая структура:

```text
UserService
     │
     ├── UserRepository
     │
     └── EmailService
```

Каждый компонент занимается своей задачей.

Вместо:

```text
┌─────────────────────────┐
│         User            │
│                         │
│ Business Logic          │
│ Database                │
│ Email                   │
│ Reports                 │
└─────────────────────────┘
```

получаем несколько специализированных компонентов.

---

# 🐍 Пример из backend

Допустим, есть регистрация пользователя.

Не стоит складывать всё в один класс:

```python
class RegistrationService:

    def validate_user(self):
        pass

    def save_to_postgres(self):
        pass

    def send_email(self):
        pass

    def generate_report(self):
        pass
```

Можно разделить:

```python
class UserValidator:
    def validate(self, user):
        pass


class UserRepository:
    def save(self, user):
        pass


class EmailService:
    def send_welcome_email(self, user):
        pass
```

А бизнес-логика может координировать эти компоненты:

```python
class RegistrationService:

    def __init__(self, validator, repository, email_service):
        self.validator = validator
        self.repository = repository
        self.email_service = email_service

    def register(self, user):
        self.validator.validate(user)
        self.repository.save(user)
        self.email_service.send_welcome_email(user)
```

Здесь `RegistrationService` отвечает за **сценарий регистрации**, а конкретные операции делегированы специализированным компонентам.

---

# ⚠️ Не надо дробить классы бесконечно

SRP не означает:

```text
1 класс
 ↓
разделить на 10 классов
 ↓
разделить каждый ещё на 10
```

Можно получить обратную проблему:

```text
слишком много классов
       ↓
слишком много абстракций
       ↓
код сложнее понимать
```

Поэтому ответственность должна быть **логически цельной**, а не искусственно минимальной.

---

# 🎯 Как ответить, если попросят пример

> **Например, если класс одновременно отвечает за бизнес-логику пользователя, сохранение в БД и отправку email, то он нарушает SRP. Я бы разделил эти обязанности на `UserService`, `UserRepository` и `EmailService`. Тогда изменение БД не потребует изменения логики пользователя или email-сервиса.**

---

# 🧠 Главное

```text
SRP
 │
 ├── одна ответственность
 │
 └── одна причина для изменения
```

Не:

```text
❌ один класс = один метод
```

А:

```text
✅ один класс = одна логически связанная ответственность
```

Главный диагностический вопрос:

> **«Сколько независимых причин для изменения у этого класса?»**

Если несколько независимых причин — возможно, класс нарушает **Single Responsibility Principle**.

---

# 🧠 Формула для запоминания

**SRP = одна ответственность → одна причина для изменения.**

```text
UserRepository → БД
EmailService   → Email
UserService    → бизнес-логика
```

**Не количество методов определяет SRP, а количество независимых ответственностей класса.**

# Связанные темы:

[[SOLID|SOLID]]
