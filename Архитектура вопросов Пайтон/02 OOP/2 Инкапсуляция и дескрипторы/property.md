# `property`

## 🎯 Ответ на собеседовании

**`property`** — это механизм Python, который позволяет обращаться к методам как к обычным атрибутам.

Он используется для **контролируемого доступа к атрибутам**: можно выполнять логику при чтении, записи или удалении значения.

`property` реализован через **дескрипторный протокол**.

---

## 📌 Простой пример

Без `property`:

```python id="m7k2px"
class User:
    def __init__(self, age):
        self._age = age

    def get_age(self):
        return self._age
```

Использование:

```python id="q4v8nc"
user.get_age()
```

С `property`:

```python id="z3m6ka"
class User:
    def __init__(self, age):
        self._age = age

    @property
    def age(self):
        return self._age
```

Теперь:

```python id="n8x2qp"
user.age
```

Хотя `age` фактически связан с методом:

```python id="s5c9vd"
def age(self):
    ...
```

---

# 🔧 Getter

Декоратор:

```python id="r7m3xa"
@property
```

создаёт getter.

```python id="v2q8nk"
class User:
    @property
    def age(self):
        return self._age
```

При:

```python id="j6k4pw"
user.age
```

вызывается getter.

Упрощённо:

```text id="f9v3mq"
user.age
   ↓
property.__get__()
   ↓
getter
```

---

# ✏️ Setter

Если нужно контролировать присваивание:

```python id="c8m2qa"
class User:
    def __init__(self, age):
        self._age = age

    @property
    def age(self):
        return self._age

    @age.setter
    def age(self, value):
        if value < 0:
            raise ValueError("Возраст не может быть отрицательным")

        self._age = value
```

Теперь:

```python id="w5q7kc"
user.age = 31
```

вызовет setter.

```text id="a3n8vz"
user.age = 31
       ↓
property.__set__()
       ↓
age.setter
```

---

# 🗑️ Deleter

Можно контролировать удаление:

```python id="p4x9md"
class User:
    def __init__(self, age):
        self._age = age

    @property
    def age(self):
        return self._age

    @age.deleter
    def age(self):
        del self._age
```

Теперь:

```python id="q8m2va"
del user.age
```

вызовет deleter.

---

# 🧩 Полный пример

```python id="k6v3px"
class User:
    def __init__(self, age):
        self._age = age

    @property
    def age(self):
        return self._age

    @age.setter
    def age(self, value):
        if not 0 <= value <= 150:
            raise ValueError("Некорректный возраст")

        self._age = value

    @age.deleter
    def age(self):
        del self._age
```

Использование выглядит как работа с обычным атрибутом:

```python id="m9q2vc"
user = User(31)

print(user.age)

user.age = 32

del user.age
```

Но за каждой операцией стоит логика getter/setter/deleter.

---

# 🔥 `property` — дескриптор

Это ключевой момент.

`property` — встроенный **Data-дескриптор**.

Он реализует механизм:

```text id="w3k7na"
__get__()
__set__()
__delete__()
```

Поэтому:

```python id="b8m4qx"
@property
```

— это не просто удобный синтаксис для getter.

Это использование встроенного дескриптора `property`.

---

# ⚠️ Почему `_age`, а не `age`?

Внутри обычно используется другое имя:

```python id="n5x8kc"
self._age
```

а наружу предоставляется:

```python id="r2m6va"
user.age
```

Это позволяет избежать рекурсии.

❌ Неправильно:

```python id="c7q3wp"
@property
def age(self):
    return self.age
```

Получится бесконечный вызов `property`.

Правильно:

```python id="j4n8xz"
@property
def age(self):
    return self._age
```

---

# 🆚 Обычный атрибут vs property

### Обычный атрибут

```python id="x6m2qa"
user.age = -100
```

Python просто запишет значение.

### `property`

```python id="p9k4vc"
user.age = -100
```

setter может проверить значение:

```python id="f3w7na"
if value < 0:
    raise ValueError
```

То есть `property` позволяет сохранить удобный интерфейс:

```python id="h8m5qx"
user.age
user.age = 31
```

но добавить контролируемое поведение.

---

# 💼 Где используется?

`property` полезен, когда нужно:

* валидировать значение;
* вычислять значение;
* преобразовывать значение;
* скрывать внутреннее представление;
* сделать атрибут только для чтения;
* сохранить API атрибута при изменении внутренней реализации.

Например, read-only property:

```python id="q2v7mc"
class User:
    @property
    def full_name(self):
        return f"{self.first_name} {self.last_name}"
```

Можно:

```python id="n4k8za"
print(user.full_name)
```

Но нельзя:

```python id="s6m3xp"
user.full_name = "..."
```

если setter не определён.

---

# 🧠 Главное

* `property` позволяет использовать метод как обычный атрибут.
* `@property` → getter.
* `@name.setter` → setter.
* `@name.deleter` → deleter.
* `property` реализован через **дескрипторы**.
* `property` — **Data-дескриптор**.
* Позволяет контролировать чтение, изменение и удаление атрибута.
* Часто используется для валидации и вычисляемых атрибутов.
* Внутри обычно хранится значение под другим именем: `_age`.

## 🎤 Суперкоротко

> `property` — это встроенный Data-дескриптор, который позволяет обращаться к методам как к атрибутам. Через `@property`, `@setter` и `@deleter` можно контролировать чтение, запись и удаление значения.
