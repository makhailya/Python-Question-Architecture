# `classmethod`

## 🎯 Ответ на собеседовании

**`classmethod`** — это метод класса, который при вызове автоматически получает **класс (`cls`)** в качестве первого аргумента вместо экземпляра (`self`).

Создаётся с помощью декоратора:

```python id="n3q7ka"
@classmethod
```

Пример:

```python id="v8m2xp"
class User:
    count = 0

    @classmethod
    def get_count(cls):
        return cls.count
```

Вызов:

```python id="k4w9zc"
User.get_count()
```

---

## 📌 `self` vs `cls`

### Обычный метод

```python id="q7m3vd"
class User:
    def hello(self):
        print(self)
```

Получает **экземпляр**:

```text id="x2n8ma"
user.hello()
     ↓
self = user
```

### `classmethod`

```python id="p5k1rz"
class User:
    @classmethod
    def create(cls):
        return cls()
```

Получает **класс**:

```text id="c9v4qb"
User.create()
     ↓
cls = User
```

---

# 🏗️ Для чего используется?

## 1. Альтернативные конструкторы

Это одно из самых частых применений.

```python id="h6q2mx"
class User:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    @classmethod
    def from_string(cls, data):
        name, age = data.split(",")
        return cls(name, int(age))
```

Теперь можно создать объект двумя способами:

```python id="z8m3vp"
user = User("Ilya", 31)

user2 = User.from_string("Ilya,31")
```

`from_string()` выступает как **альтернативный конструктор**.

---

## 2. Работа с атрибутами класса

```python id="r5k7xa"
class User:
    count = 0

    def __init__(self):
        type(self).count += 1

    @classmethod
    def get_count(cls):
        return cls.count
```

Можно вызвать:

```python id="u3q9nd"
User.get_count()
```

Метод работает с состоянием **класса**, а не конкретного объекта.

---

# 🧬 Почему лучше использовать `cls`, а не имя класса?

Потому что `cls` учитывает **наследование**.

```python id="m7v2kc"
class User:
    @classmethod
    def create(cls):
        return cls()
```

Есть наследник:

```python id="a4x8pz"
class Admin(User):
    pass
```

Вызов:

```python id="j6q3wb"
Admin.create()
```

передаст:

```text id="e8n5mv"
cls = Admin
```

Поэтому:

```python id="s2k7qa"
return cls()
```

создаст `Admin`, а не всегда `User`.

---

# 🔧 `classmethod` и дескрипторы

`classmethod` реализован через **дескрипторный механизм Python**.

Когда пишем:

```python id="c5v9nx"
class User:
    @classmethod
    def create(cls):
        ...
```

объект `classmethod` находится в словаре класса.

При обращении:

```python id="w8m3kp"
User.create()
```

дескриптор связывает функцию с классом и передаёт его как `cls`.

Упрощённо:

```text id="p4q7za"
User.create
     ↓
classmethod.__get__()
     ↓
create(User)
```

---

# 🆚 `staticmethod` vs `classmethod` vs обычный метод

|                           | Обычный метод                         | `classmethod`               | `staticmethod`                  |
| ------------------------- | ------------------------------------- | --------------------------- | ------------------------------- |
| Первый аргумент           | `self`                                | `cls`                       | Нет                             |
| Работает с экземпляром    | ✅                                     | Не обязательно              | ❌                               |
| Работает с классом        | Через `type(self)` / `self.__class__` | ✅                           | Только если передать класс явно |
| Можно вызвать через класс | Да, с явным экземпляром               | Да                          | Да                              |
| Частое применение         | Работа с объектом                     | Альтернативные конструкторы | Утилитарные функции             |

Пример:

```python id="n2v8cx"
class User:

    def hello(self):
        print("instance")

    @classmethod
    def create(cls):
        return cls()

    @staticmethod
    def validate_name(name):
        return bool(name)
```

---

# ⚠️ Важный нюанс

`classmethod` не означает:

> «метод, который можно вызвать только через класс».

Его можно вызвать и через экземпляр:

```python id="q6m4vz"
user = User()

user.create()
```

Но `cls` всё равно будет ссылаться на класс объекта.

---

# 🧠 Главное

* `@classmethod` превращает функцию в **метод класса**.
* Первый аргумент автоматически получает **`cls`**.
* `cls` — это класс, через который выполняется вызов.
* Частое применение — **альтернативные конструкторы**.
* Хорошо работает с наследованием благодаря `cls()`.
* Реализован через **дескрипторный механизм**.
* Можно вызывать как через класс, так и через экземпляр.

## 🎤 Суперкоротко

> `classmethod` — это метод, который вместо экземпляра получает класс в первом аргументе `cls`. Он часто используется для альтернативных конструкторов и работы с состоянием класса. Реализован через дескрипторный механизм Python.
