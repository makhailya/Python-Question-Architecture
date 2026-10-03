# Non-data дескрипторы

## 🎯 Ответ на собеседовании

**Non-data дескриптор** — это дескриптор, который реализует только метод `__get__`, но не реализует `__set__` или `__delete__`.

Главная особенность:

> **Non-data дескриптор имеет меньший приоритет, чем `instance.__dict__`.**

Поэтому атрибут экземпляра может **перекрыть** Non-data дескриптор.

---

## 📌 Пример

```python id="x8k4mz"
class Descriptor:
    def __get__(self, instance, owner):
        return "descriptor"


class User:
    name = Descriptor()
```

Создаём объект:

```python id="q3v7np"
user = User()

print(user.name)
```

Результат:

```text id="f5m2qa"
descriptor
```

Потому что `name` — Non-data дескриптор.

---

## 🔥 Атрибут экземпляра может его перекрыть

Добавим `name` в `__dict__`:

```python id="n7c1vx"
user.__dict__["name"] = "Ilya"
```

Теперь:

```python id="r4m8kd"
print(user.name)
```

получим:

```text id="p2z6wc"
Ilya
```

Почему?

Потому что порядок поиска примерно такой:

```text id="j8q3mv"
1. Data Descriptor
        ↓
2. instance.__dict__  ← здесь нашли name
        ↓
3. Non-data Descriptor
```

До дескриптора Python уже не доходит.

---

## 🆚 Data vs Non-data

|                                          | Data       | Non-data |
| ---------------------------------------- | ---------- | -------- |
| `__get__`                                | ✅          | ✅        |
| `__set__`                                | ✅          | ❌        |
| `__delete__`                             | Может быть | ❌        |
| Приоритет над `instance.__dict__`        | ✅          | ❌        |
| Может быть перекрыт атрибутом экземпляра | ❌          | ✅        |

Запоминалка:

```text id="k1w9sq"
__set__ / __delete__
        ↓
Data Descriptor
        ↓
имеет высокий приоритет

только __get__
        ↓
Non-data Descriptor
        ↓
может быть перекрыт __dict__
```

---

# 🧩 Обычные методы — Non-data дескрипторы

Это особенно важно для понимания Python.

Когда мы пишем:

```python id="v5n2qa"
class User:
    def hello(self):
        print("Hello")
```

`hello` — обычный объект-функция, находящийся в классе.

Функции реализуют `__get__`, поэтому они являются **Non-data дескрипторами**.

Когда пишем:

```python id="b7k4xp"
user.hello()
```

Python получает функцию через её `__get__()` и создаёт **bound method**, привязанный к `user`.

Упрощённо:

```text id="m8q3cz"
User.hello
    ↓
function.__get__()
    ↓
bound method
    ↓
user.hello()
```

---

## ⚠️ Можно перекрыть метод

Поскольку функция — Non-data дескриптор, атрибут экземпляра может её перекрыть:

```python id="s6v9ka"
class User:
    def hello(self):
        print("Hello")


user = User()

user.hello = lambda: print("Hi")

user.hello()
```

Результат:

```text id="w2p7md"
Hi
```

В `user.__dict__` теперь есть:

```python id="d9k4xc"
{
    "hello": <lambda>
}
```

Атрибут экземпляра имеет приоритет над Non-data дескриптором-функцией.

---

# 🏗️ `staticmethod` и `classmethod`

Они также используют **дескрипторный механизм**.

Например:

```python id="q5m8vz"
class User:
    @staticmethod
    def validate():
        ...
```

`staticmethod` реализует дескрипторный протокол и относится к Non-data дескрипторам.

`classmethod` также использует дескрипторный механизм.

---

## 🧠 Главное

* Non-data дескриптор реализует **только `__get__`**.
* Не имеет `__set__` и `__delete__`.
* Имеет меньший приоритет, чем `instance.__dict__`.
* Поэтому атрибут экземпляра может его перекрыть.
* Обычные функции класса — Non-data дескрипторы.
* Через `function.__get__()` формируются bound methods.
* `staticmethod` и `classmethod` также основаны на дескрипторном механизме.

## 🎤 Суперкоротко

> **Non-data дескриптор** — это дескриптор, который реализует `__get__`, но не `__set__` и `__delete__`. Он имеет меньший приоритет, чем `instance.__dict__`, поэтому атрибут экземпляра может его перекрыть. Обычные методы класса являются Non-data дескрипторами.
