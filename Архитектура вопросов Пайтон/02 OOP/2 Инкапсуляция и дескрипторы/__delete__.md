# `__delete__`

## 🎯 Ответ на собеседовании

`__delete__` — специальный метод дескриптора, который вызывается при удалении атрибута:

```python
del obj.attribute
```

Python передаёт дескриптору экземпляр объекта:

```python
descriptor.__delete__(obj)
```

---

## 📌 Простой пример

```python
class Descriptor:
    def __get__(self, instance, owner):
        return instance._value

    def __set__(self, instance, value):
        instance._value = value

    def __delete__(self, instance):
        del instance._value
```

Используем:

```python
class User:
    value = Descriptor()
```

```python
user = User()

user.value = 42

print(user.value)
# 42

del user.value

print(user.value)
# AttributeError
```

Путь выполнения:

```text
user.value = 42
        ↓
    __set__()

user.value
        ↓
    __get__()

del user.value
        ↓
    __delete__()
```

---

## 🔹 Параметр `instance`

Сигнатура:

```python
def __delete__(self, instance):
    ...
```

### `self`

Сам объект-дескриптор.

### `instance`

Экземпляр класса, у которого удаляют атрибут:

```python
del user.value
```

Здесь:

```text
instance = user
```

В отличие от `__get__`, здесь нет `owner`, потому что для удаления достаточно самого экземпляра.

---

## 💡 Зачем нужен `__delete__`?

Он позволяет выполнить дополнительную логику при удалении атрибута.

Например, удалить значение из хранилища:

```python
class Field:
    def __get__(self, instance, owner):
        return instance.__dict__.get("_value")

    def __set__(self, instance, value):
        instance.__dict__["_value"] = value

    def __delete__(self, instance):
        instance.__dict__.pop("_value", None)
```

Теперь:

```python
user.value = 100
del user.value
```

вызовет именно `__delete__()`.

---

## 🔥 Связь с Data-дескрипторами

Если дескриптор реализует:

```python
__set__()
```

или:

```python
__delete__()
```

он является **Data-дескриптором**.

```text
__set__ / __delete__
        ↓
Data Descriptor
```

Поэтому `__delete__` сам по себе уже достаточно, чтобы дескриптор считался Data-дескриптором.

---

## 🧩 `__delete__` и `property`

У `property` также существует механизм удаления через `deleter`:

```python
class User:
    def __init__(self, name):
        self._name = name

    @property
    def name(self):
        return self._name

    @name.deleter
    def name(self):
        del self._name
```

Теперь:

```python
user = User("Ilya")

del user.name
```

вызовет функцию, зарегистрированную как deleter.

---

## 🆚 Четыре метода дескриптора

| Метод          | Операция                                         |
| -------------- | ------------------------------------------------ |
| `__get__`      | `obj.attr`                                       |
| `__set__`      | `obj.attr = value`                               |
| `__delete__`   | `del obj.attr`                                   |
| `__set_name__` | сообщает дескриптору его имя при создании класса |

Последний работает **не при обращении к атрибуту**, а при создании класса.

---

## 🧠 Главное

* `__delete__` перехватывает `del obj.attr`.
* Сигнатура:

```python
__delete__(self, instance)
```

* `instance` — объект, у которого удаляется атрибут.
* `__delete__` вместе с `__set__` делает дескриптор Data-дескриптором.
* Часто используется вместе с `__get__` и `__set__`.

## 🎤 Суперкоротко

> `__delete__` — метод дескриптора, который вызывается при `del obj.attr`. Он получает экземпляр объекта и позволяет контролировать или изменить поведение удаления атрибута.
