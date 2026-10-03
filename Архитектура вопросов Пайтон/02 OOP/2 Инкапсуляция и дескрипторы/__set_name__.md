# `__set_name__`

## 🎯 Ответ на собеседовании

`__set_name__` — специальный метод, который Python вызывает **при создании класса**, если объект класса реализует этот метод.

Он передаёт дескриптору:

* класс-владелец;
* имя атрибута, под которым дескриптор был объявлен.

Сигнатура:

```python id="8f4m2q"
def __set_name__(self, owner, name):
    ...
```

---

## 📌 Пример

```python id="k7v3pa"
class Descriptor:
    def __set_name__(self, owner, name):
        print(owner)
        print(name)


class User:
    name = Descriptor()
```

При создании `User` Python автоматически вызовет:

```python id="m2q8vx"
Descriptor.__set_name__(User, "name")
```

То есть дескриптор узнает:

```text id="c5n9kd"
owner = User
name  = "name"
```

---

## 🔥 Зачем это нужно?

Главное применение — **автоматически узнать имя атрибута**, под которым дескриптор находится в классе.

Например:

```python id="p8w4mz"
class Field:
    def __set_name__(self, owner, name):
        self.name = name
```

Теперь:

```python id="x3q7ka"
class User:
    name = Field()
    age = Field()
```

Python вызовет:

```text id="n6v2xc"
Field.__set_name__(User, "name")
Field.__set_name__(User, "age")
```

И каждый дескриптор будет знать своё имя.

---

# 🧩 Практический пример

Можно автоматически хранить значение во внутреннем атрибуте:

```python id="r5m8qw"
class Field:
    def __set_name__(self, owner, name):
        self.name = name
        self.storage_name = f"_{name}"

    def __get__(self, instance, owner):
        if instance is None:
            return self

        return getattr(instance, self.storage_name)

    def __set__(self, instance, value):
        setattr(instance, self.storage_name, value)
```

Используем:

```python id="j9k3vp"
class User:
    name = Field()
    age = Field()
```

Теперь:

```python id="a6q2mx"
user = User()

user.name = "Ilya"
user.age = 31
```

Внутри получится примерно:

```python id="v4n7cz"
user._name
user._age
```

При этом нам не пришлось вручную писать:

```python id="s8m2qa"
self.storage_name = "_name"
self.storage_name = "_age"
```

`__set_name__` сделал это автоматически.

---

## 🔄 Когда вызывается?

Важно:

```python id="w3q8nc"
class User:
    name = Field()
```

При создании класса:

```text id="k5m1vx"
Создание User
      ↓
Python видит Field()
      ↓
вызывает __set_name__(User, "name")
      ↓
класс создан
```

Это **не происходит при каждом**:

```python id="r7c2mz"
user.name
```

и не происходит при:

```python id="e4n9qa"
user.name = "Ilya"
```

Для них используются соответственно:

```text id="v8k3px"
user.name
    ↓
__get__()

user.name = ...
    ↓
__set__()
```

---

# 🆚 `__set_name__` vs `__set__`

Очень легко перепутать.

| Метод          | Когда вызывается       | Что делает                                  |
| -------------- | ---------------------- | ------------------------------------------- |
| `__set_name__` | При создании класса    | Передаёт дескриптору `owner` и имя атрибута |
| `__set__`      | При `obj.attr = value` | Обрабатывает присваивание                   |

Запоминалка:

```text id="p2w7kc"
__set_name__
     ↓
«Как меня зовут?»

__set__
     ↓
«Мне устанавливают значение»
```

---

## ⚠️ Важный нюанс

`__set_name__` не обязан использоваться только дескрипторами.

Это специальный механизм для объектов, находящихся в теле класса, которым нужно узнать имя, под которым они были объявлены.

Но чаще всего его рассматривают именно **в контексте дескрипторов**.

---

## 🧠 Главное

* `__set_name__` вызывается **при создании класса**.
* Получает:

```python id="x4m8qa"
__set_name__(self, owner, name)
```

* `owner` → класс, в котором находится объект.
* `name` → имя атрибута.
* Позволяет дескриптору узнать, **под каким именем он объявлен**.
* Часто используется для автоматического создания внутреннего имени хранения.
* Не путать с `__set__`.

## 🎤 Суперкоротко

> `__set_name__` вызывается при создании класса и передаёт дескриптору класс-владелец и имя атрибута. Он позволяет дескриптору узнать, под каким именем он был объявлен. Например, `name = Field()` получает `name` через `__set_name__`.
