# `__set__`

## 🎯 Ответ на собеседовании

`__set__` — специальный метод дескриптора, который вызывается при **присваивании значения атрибуту**.

То есть при:

```python
obj.attribute = value
```

Python может вызвать:

```python
descriptor.__set__(obj, value)
```

---

## 📌 Пример

```python
class Descriptor:
    def __set__(self, instance, value):
        print(f"Установлено: {value}")


class User:
    name = Descriptor()
```

Теперь:

```python
user = User()

user.name = "Ilya"
```

Результат:

```text
Установлено: Ilya
```

Вместо обычной записи в `user.__dict__` Python вызвал:

```text
user.name = "Ilya"
       ↓
Descriptor.__set__(user, "Ilya")
```

---

## 🔹 Параметры

Сигнатура:

```python
def __set__(self, instance, value):
    ...
```

### `self`

Сам объект-дескриптор.

### `instance`

Экземпляр, которому устанавливается атрибут:

```python
user.name = "Ilya"
```

Здесь:

```text
instance = user
```

### `value`

Новое значение:

```text
value = "Ilya"
```

---

## 💡 Практический пример: валидация

Дескрипторы часто используют, чтобы контролировать допустимые значения.

```python
class PositiveNumber:
    def __set__(self, instance, value):
        if value < 0:
            raise ValueError("Число должно быть положительным")

        instance._value = value


class Product:
    price = PositiveNumber()
```

Теперь:

```python
product = Product()

product.price = 100   # OK
product.price = -10   # ValueError
```

`__set__` перехватил присваивание и проверил значение.

---

## 🔥 Почему это Data-дескриптор?

Если класс дескриптора реализует:

```python
__set__()
```

то он становится **Data-дескриптором**.

```text
__set__
   ↓
Data Descriptor
   ↓
имеет приоритет над instance.__dict__
```

Например:

```python
class Descriptor:
    def __get__(self, instance, owner):
        return "descriptor"

    def __set__(self, instance, value):
        print("set:", value)
```

Теперь:

```python
class User:
    name = Descriptor()
```

Даже если:

```python
user.__dict__["name"] = "Ilya"
```

при:

```python
user.name
```

Data-дескриптор имеет приоритет.

---

## 🧩 Обычно `__set__` работает вместе с `__get__`

Например:

```python
class PositiveNumber:
    def __get__(self, instance, owner):
        return instance._value

    def __set__(self, instance, value):
        if value < 0:
            raise ValueError("Must be positive")

        instance._value = value
```

Использование:

```python
class Product:
    price = PositiveNumber()
```

```python
product = Product()

product.price = 100
print(product.price)
```

Получаем:

```text
100
```

Путь выполнения:

```text
product.price = 100
        ↓
   __set__()

product.price
        ↓
   __get__()
```

---

## 🔗 Связь с `property`

Когда используется:

```python
class User:
    @property
    def age(self):
        return self._age

    @age.setter
    def age(self, value):
        self._age = value
```

setter `property` позволяет контролировать присваивание:

```python
user.age = 31
```

Это реализуется через дескрипторный механизм `property`.

---

## ⚠️ Важный нюанс

`__set__` не вызывается при обычном чтении:

```python
user.name
```

Для чтения используется:

```python
__get__()
```

А для:

```python
user.name = "Ilya"
```

используется:

```python
__set__()
```

---

## 🧠 Главное

* `__set__` перехватывает **присваивание атрибута**.
* Сигнатура:

```python
__set__(self, instance, value)
```

* `instance` → объект, которому устанавливают значение.
* `value` → новое значение.
* Наличие `__set__` делает дескриптор **Data-дескриптором**.
* Можно использовать для валидации, преобразования и хранения значений.

## 🎤 Суперкоротко

> `__set__` — метод дескриптора, который вызывается при присваивании `obj.attr = value`. Он получает экземпляр и новое значение. Наличие `__set__` делает дескриптор Data-дескриптором.
