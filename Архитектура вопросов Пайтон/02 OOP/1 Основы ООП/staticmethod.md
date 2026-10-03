# `staticmethod`

## 🎯 Ответ на собеседовании

**`staticmethod`** — это метод класса, который не получает автоматически ни экземпляр (`self`), ни класс (`cls`).

Создаётся декоратором:

```python id="q4n8vm"
@staticmethod
```

По сути, это обычная функция, логически размещённая внутри класса.

---

## 📌 Пример

```python id="k7m2xp"
class User:
    @staticmethod
    def validate_age(age):
        return age >= 18
```

Вызов:

```python id="z3q8nc"
User.validate_age(20)
```

Результат:

```text id="v5m1qa"
True
```

Метод не получает автоматически:

```text id="h8k4rx"
self
cls
```

---

## 🔄 Обычный метод vs `staticmethod`

Обычный метод:

```python id="p6v2mc"
class User:
    def hello(self):
        print(self)
```

При:

```python id="a9k3wd"
user.hello()
```

Python автоматически передаёт:

```text id="s4x7pz"
self = user
```

`staticmethod`:

```python id="m2q8vn"
class User:
    @staticmethod
    def hello():
        print("Hello")
```

Никакой автоматической передачи аргумента нет.

```python id="j7c5ka"
User.hello()
```

---

# 📌 Зачем нужен `staticmethod`?

Обычно — для функции, которая **логически относится к классу**, но не нуждается ни в состоянии экземпляра, ни в состоянии класса.

Например:

```python id="f8m3qx"
class User:
    @staticmethod
    def is_valid_email(email):
        return "@" in email
```

Функция работает только с переданным аргументом.

Ей не нужны:

```text id="r2k6wb"
self
cls
```

Поэтому её можно логически объединить с `User`.

---

# 🧩 Можно ли просто сделать обычную функцию?

Да.

Например:

```python id="w4q9nc"
def is_valid_email(email):
    return "@" in email
```

и:

```python id="d7m2xp"
class User:
    ...
```

Это тоже нормально.

`staticmethod` имеет смысл, когда функция **концептуально относится к классу** и её удобно использовать как часть API класса:

```python id="n5k8va"
User.is_valid_email(...)
```

---

# 🔧 `staticmethod` и дескрипторы

`staticmethod` связан с **дескрипторным механизмом Python**.

```python id="q3v7mx"
class User:
    @staticmethod
    def validate():
        ...
```

`staticmethod` хранится в классе как специальный объект.

При обращении:

```python id="h8k2qa"
User.validate
```

дескриптор возвращает исходную функцию **без привязки к экземпляру или классу**.

Упрощённо:

```text id="m4p9zc"
User.validate
      ↓
staticmethod.__get__()
      ↓
обычная функция
```

Поэтому `self` и `cls` автоматически не передаются.

---

# 🆚 `staticmethod` vs `classmethod`

|                               | `staticmethod`      | `classmethod`              |
| ----------------------------- | ------------------- | -------------------------- |
| `self`                        | ❌                   | ❌                          |
| `cls`                         | ❌                   | ✅                          |
| Доступ к классу автоматически | ❌                   | ✅                          |
| Частое применение             | Утилитарная функция | Альтернативный конструктор |
| Дескриптор                    | ✅                   | ✅                          |

Пример:

```python id="c6x1vn"
class User:

    @staticmethod
    def validate_age(age):
        return age >= 18

    @classmethod
    def create(cls):
        return cls()
```

---

# ⚠️ Важный нюанс

`staticmethod` **не означает**, что метод нельзя вызвать через экземпляр.

Можно:

```python id="z9m4qa"
user = User()

user.validate_age(20)
```

Но `self` автоматически передан **не будет**.

То есть это:

```python id="s5k7xp"
user.validate_age(20)
```

по смыслу остаётся вызовом обычной функции с аргументом `20`.

---

# 💼 Backend-пример

Например, валидатор:

```python id="r8q3mw"
class UserService:

    @staticmethod
    def normalize_email(email: str) -> str:
        return email.strip().lower()
```

Использование:

```python id="u4n7kc"
email = UserService.normalize_email("  TEST@MAIL.COM  ")

print(email)
# test@mail.com
```

Методу не нужен ни объект `UserService`, ни сам класс.

---

## 🧠 Главное

* `@staticmethod` превращает метод в функцию без автоматической передачи `self` или `cls`.
* Не имеет доступа к экземпляру/классу через автоматически переданный аргумент.
* Используется для функций, которые логически относятся к классу, но не используют его состояние.
* Можно вызвать через класс или экземпляр.
* Реализован через дескрипторный механизм.
* Если функция вообще не имеет отношения к классу, часто лучше сделать обычную функцию на уровне модуля.

## 🎤 Суперкоротко

> `staticmethod` — это метод, который не получает автоматически ни `self`, ни `cls`. Это обычная функция, логически помещённая в класс. Используется, когда функция связана с классом по смыслу, но не зависит от его состояния.
