# `__dict__`

## 🎯 Ответ на собеседовании

**`__dict__`** — это словарь, в котором Python хранит атрибуты объекта или класса, если они представлены через обычное словарное пространство атрибутов.

Для экземпляра:

```python id="k4m8qx"
class User:
    pass


user = User()

user.name = "Ilya"
user.age = 31

print(user.__dict__)
```

Получим:

```python id="r7v2nc"
{
    "name": "Ilya",
    "age": 31
}
```

То есть `__dict__` содержит **атрибуты конкретного экземпляра**.

---

## 📌 Атрибуты экземпляра

```python id="m8q3vp"
class User:
    def __init__(self, name):
        self.name = name


user = User("Ilya")
```

Внутри:

```python id="x5k9za"
user.__dict__
```

будет:

```python id="n2v7mc"
{
    "name": "Ilya"
}
```

Можно изменить его напрямую:

```python id="q6m4px"
user.__dict__["age"] = 31

print(user.age)
# 31
```

---

# 🏛️ `__dict__` класса

У самого класса тоже может быть `__dict__`:

```python id="v3k8qa"
class User:
    name = "Unknown"

    def hello(self):
        print("Hello")
```

Можно посмотреть:

```python id="c7m2xn"
print(User.__dict__)
```

Там будут находиться, среди прочего:

```text id="j5q9vc"
name
hello
__module__
__doc__
...
```

Важно: `User.__dict__` — это не обычный изменяемый `dict`, а специальный **mappingproxy**.

---

# 🔥 Связь с дескрипторами

`__dict__` особенно важен при понимании **порядка поиска атрибутов**.

Когда выполняется:

```python id="w4n8kp"
user.name
```

Python учитывает атрибуты класса, дескрипторы и `user.__dict__`.

Упрощённо:

```text id="q8m3va"
Data Descriptor
       ↓
instance.__dict__
       ↓
Non-data Descriptor
       ↓
обычный атрибут класса
```

Именно поэтому **Data-дескриптор имеет приоритет над `instance.__dict__`**.

---

# 🧩 Пример с Non-data descriptor

```python id="s6k2mx"
class Descriptor:
    def __get__(self, instance, owner):
        return "descriptor"


class User:
    name = Descriptor()
```

Создадим объект:

```python id="p9v4qc"
user = User()

print(user.name)
# descriptor
```

Теперь положим значение непосредственно в `__dict__`:

```python id="a3m7xz"
user.__dict__["name"] = "Ilya"
```

Теперь:

```python id="h5q8vn"
print(user.name)
```

получим:

```text id="e2k6mc"
Ilya
```

Потому что `Descriptor` — **Non-data descriptor**, а `instance.__dict__` имеет перед ним приоритет.

---

# 🆚 С Data-дескриптором

```python id="r8m3qa"
class Descriptor:
    def __get__(self, instance, owner):
        return "descriptor"

    def __set__(self, instance, value):
        pass
```

Теперь это Data-дескриптор.

Даже если:

```python id="k4v7xp"
user.__dict__["name"] = "Ilya"
```

при:

```python id="z6m2nc"
user.name
```

будет вызван:

```python id="w9q3va"
Descriptor.__get__()
```

потому что Data-дескриптор имеет более высокий приоритет.

---

# ⚠️ Не каждый объект имеет `__dict__`

Например, класс со `__slots__` может не иметь словаря экземпляра:

```python id="n7k4mx"
class User:
    __slots__ = ("name",)
```

Тогда:

```python id="q2v8pc"
user = User()

user.name = "Ilya"

print(user.__dict__)
```

может привести к:

```text id="f5m9qa"
AttributeError
```

Потому что `__slots__` позволяет хранить атрибуты без обычного `__dict__` экземпляра.

---

# 🧠 Главное

* `instance.__dict__` хранит обычные атрибуты экземпляра.
* `Class.__dict__` содержит атрибуты и методы класса.
* `Class.__dict__` — `mappingproxy`, а не обычный изменяемый словарь.
* `instance.__dict__` играет важную роль в поиске атрибутов.
* **Data-дескриптор имеет приоритет над `instance.__dict__`.**
* **Non-data дескриптор уступает `instance.__dict__`.**
* `__slots__` может убрать `__dict__` у экземпляров.

## 🎤 Суперкоротко

> `__dict__` — это пространство атрибутов объекта или класса в виде словаря. Для экземпляра в нём обычно хранятся его обычные атрибуты. При поиске атрибута `instance.__dict__` имеет меньший приоритет, чем Data-дескриптор, но больший, чем Non-data дескриптор.
