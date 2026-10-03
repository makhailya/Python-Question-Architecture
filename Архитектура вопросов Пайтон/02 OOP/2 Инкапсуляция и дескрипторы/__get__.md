# `__get__`

## 🎯 Ответ на собеседовании

`__get__` — специальный метод дескриптора, который вызывается при **получении атрибута**.

То есть при:

```python id="x8m2qa"
obj.attribute
```

Python может вызвать:

```python id="v5k7nc"
descriptor.__get__(obj, type(obj))
```

---

## 📌 Пример

```python id="m3q8vx"
class Descriptor:
    def __get__(self, instance, owner):
        return "Hello"


class User:
    name = Descriptor()
```

Теперь:

```python id="q7w2ka"
user = User()

print(user.name)
```

Результат:

```text id="f4n8mz"
Hello
```

Потому что:

```text id="h6p3vc"
user.name
   ↓
Descriptor.__get__()
   ↓
"Hello"
```

---

## 🔹 Параметры `__get__`

```python id="n9k4qa"
def __get__(self, instance, owner):
    ...
```

### `self`

Сам объект-дескриптор:

```text id="p2m7xv"
User.name
    ↓
Descriptor instance
```

### `instance`

Экземпляр класса, через который обращаются к атрибуту:

```python id="r5k8mz"
user.name
```

Тогда:

```text id="c3q7nw"
instance = user
```

### `owner`

Класс, которому принадлежит атрибут:

```text id="v8m2qa"
owner = User
```

---

## ⚠️ Обращение через класс

Если написать:

```python id="k6p3xz"
User.name
```

то `instance` будет:

```python id="s4q9mv"
None
```

а:

```python id="j8n2kc"
owner = User
```

Поэтому часто пишут:

```python id="q5v7mx"
class Descriptor:
    def __get__(self, instance, owner):
        if instance is None:
            return self

        return instance._name
```

Это позволяет корректно работать и с:

```python id="z3m8qa"
User.name
```

и:

```python id="h7q2vc"
user.name
```

---

## 🧩 Практический пример

```python id="m8k4xz"
class Field:
    def __get__(self, instance, owner):
        if instance is None:
            return self

        return instance.__dict__.get("value")


class User:
    value = Field()
```

Использование:

```python id="r2n7qc"
user = User()

user.__dict__["value"] = 42

print(user.value)
# 42
```

При `user.value` вызывается `Field.__get__()`.

---

## 🔗 Связь с обычными методами

Функции, объявленные внутри класса, тоже имеют `__get__`.

```python id="c9m5vx"
class User:
    def hello(self):
        print("Hello")
```

При:

```python id="w4q8nz"
user.hello
```

Python использует механизм `function.__get__()` и получает **bound method**, где `self` привязан к `user`.

Поэтому:

```python id="a7k2mc"
user.hello()
```

работает без явной передачи `self`.

---

## 🧠 Главное

* `__get__` вызывается при **чтении атрибута**.
* Сигнатура:

```python id="q8v3mx"
__get__(self, instance, owner)
```

* `self` → объект-дескриптор.
* `instance` → экземпляр объекта.
* `owner` → класс.
* При обращении через класс `instance is None`.
* Используется `property`, функциями классов и другими дескрипторами.

## 🎤 Суперкоротко

> `__get__` — метод дескриптора, который управляет чтением атрибута. Он получает сам дескриптор, экземпляр объекта и класс-владелец: `__get__(self, instance, owner)`.
