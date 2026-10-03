# Data-дескрипторы

## 🎯 Ответ на собеседовании

**Data-дескриптор** — это дескриптор, который реализует метод `__set__()` и/или `__delete__()`.

Главная особенность:

> **Data-дескриптор имеет приоритет над `instance.__dict__` при доступе к атрибуту.**

Например:

```python id="k4n8sp"
class Descriptor:
    def __get__(self, instance, owner):
        return "descriptor"

    def __set__(self, instance, value):
        pass
```

Наличие `__set__` делает `Descriptor` **Data-дескриптором**.

---

## 📌 Пример

```python id="x7q2mv"
class Name:
    def __get__(self, instance, owner):
        return instance._name

    def __set__(self, instance, value):
        instance._name = value


class User:
    name = Name()
```

Используем:

```python id="j5v8cq"
user = User()

user.name = "Ilya"
print(user.name)
```

Происходит:

```text id="e8r1nz"
user.name = "Ilya"
       ↓
Name.__set__()
       ↓
user._name = "Ilya"

user.name
       ↓
Name.__get__()
       ↓
"Ilya"
```

---

## 🔥 Почему Data-дескриптор имеет приоритет?

Рассмотрим:

```python id="p2c6vk"
user.__dict__["name"] = "instance"
```

При этом в классе существует Data-дескриптор:

```python id="q8m4xa"
class User:
    name = Name()
```

Тогда:

```python id="v9k3jd"
print(user.name)
```

Python сначала увидит **Data-дескриптор** и вызовет его `__get__()`.

Значение из:

```python id="s6w2qp"
user.__dict__["name"]
```

не будет использовано как обычный атрибут.

---

# 🆚 Data vs Non-Data Descriptor

|                                   | Data-дескриптор | Non-Data дескриптор    |
| --------------------------------- | --------------- | ---------------------- |
| `__get__`                         | ✅               | ✅                      |
| `__set__`                         | ✅               | ❌                      |
| `__delete__`                      | Может быть      | ❌                      |
| Приоритет над `instance.__dict__` | ✅               | ❌                      |
| Пример                            | `property`      | обычная функция класса |

Главное различие:

```text id="m7x1qd"
__set__ / __delete__
        ↓
Data Descriptor
```

Только `__get__`:

```text id="c9v4ks"
__get__
  ↓
Non-Data Descriptor
```

---

## 🔍 Порядок поиска атрибута

Когда выполняется:

```python id="j4m8qa"
user.name
```

упрощённо Python ищет так:

```text id="r2f7nw"
1. Data Descriptor в классе
        ↓
2. instance.__dict__
        ↓
3. Non-Data Descriptor / атрибут класса
```

Поэтому:

```text id="w8k3pb"
Data Descriptor
      ↑
   высокий
   приоритет

instance.__dict__

Non-Data Descriptor
      ↑
   низкий
   приоритет
```

---

## 🏗️ `property` — Data-дескриптор

Например:

```python id="n6v2xc"
class User:
    def __init__(self, age):
        self._age = age

    @property
    def age(self):
        return self._age

    @age.setter
    def age(self, value):
        self._age = value
```

`property` реализует дескрипторный протокол.

Когда есть setter:

```python id="b8q5mz"
@age.setter
```

`property` является **Data-дескриптором**.

---

## ⚠️ Важный нюанс

Для классификации достаточно запомнить:

```text id="q1m7cx"
__set__ или __delete__
        ↓
Data Descriptor
```

Например, дескриптор без `__set__`:

```python id="d5v9ka"
class Descriptor:
    def __get__(self, instance, owner):
        return "value"
```

является **Non-Data Descriptor**.

---

## 🧠 Главное

* Data-дескриптор реализует `__set__` и/или `__delete__`.
* Он имеет **приоритет над `instance.__dict__`**.
* При чтении `obj.attr` Python сначала проверяет Data-дескриптор в классе.
* `property` — распространённый пример Data-дескриптора.
* Обычная функция класса — Non-Data Descriptor.

## 🎤 Суперкоротко

> **Data-дескриптор** — дескриптор с `__set__` или `__delete__`. Его главное свойство — приоритет над `instance.__dict__` при доступе к атрибуту. Типичный пример — `property`.
