# Protocol

## 🎯 Ответ на собеседовании

**`Protocol`** — это механизм из [[typing]], который позволяет описывать **интерфейс объекта** без необходимости явно наследоваться от этого интерфейса.

Это пример **структурной типизации**:

> Если объект имеет нужные методы и атрибуты — он соответствует `Protocol`.

В отличие от `ABC`, классу не обязательно явно наследоваться от `Protocol`.

---

## 📌 Простой пример

```python id="m5p2x8"
from typing import Protocol


class Speaker(Protocol):
    def speak(self) -> None:
        ...
```

Есть два независимых класса:

```python id="q8v4ka"
class Dog:
    def speak(self) -> None:
        print("Гав")


class Robot:
    def speak(self) -> None:
        print("Beep")
```

Они **не наследуются** от `Speaker`.

Но оба соответствуют этому Protocol:

```python id="h3k7wd"
def make_sound(speaker: Speaker) -> None:
    speaker.speak()
```

```python id="j2r6fz"
make_sound(Dog())
make_sound(Robot())
```

Статический анализатор понимает, что оба объекта подходят, потому что у них есть `speak()`.

---

## 🧩 Структурная типизация

`Protocol` основан на принципе:

> **Важно не то, от какого класса объект наследуется, а то, какие методы и атрибуты он предоставляет.**

Например:

```python id="q4m8zs"
class Dog:
    def speak(self):
        ...


class Cat:
    def speak(self):
        ...
```

Если `Protocol` требует:

```python id="x6t2pa"
class Speaker(Protocol):
    def speak(self):
        ...
```

то и `Dog`, и `Cat` подходят.

Схематично:

```text id="n5f9qw"
        Speaker
       /       \
   Dog          Cat
```

Но наследования здесь нет.

---

## 🆚 Protocol vs ABC

Это один из самых популярных вопросов на собеседовании.

|                     | ABC                    | Protocol            |
| ------------------- | ---------------------- | ------------------- |
| Типизация           | Номинальная            | Структурная         |
| Нужно наследоваться | Обычно да              | Нет                 |
| Контракт            | Явный                  | По форме объекта    |
| Duck typing         | Ограниченно            | Да                  |
| `@abstractmethod`   | Используется           | Не нужен            |
| Основная роль       | Общая иерархия классов | Описание интерфейса |

### ABC

```python id="w7k2nb"
class Repository(ABC):
    @abstractmethod
    def save(self, obj):
        ...
```

Класс должен явно наследоваться:

```python id="r9m3cx"
class UserRepository(Repository):
    ...
```

### Protocol

```python id="a4q8vk"
class Repository(Protocol):
    def save(self, obj):
        ...
```

Класс может просто иметь нужный метод:

```python id="p6s1zd"
class UserRepository:
    def save(self, obj):
        ...
```

Явное наследование не требуется.

---

## 🐍 Protocol + Duck Typing

`Protocol` можно рассматривать как способ **формализовать duck typing для статической типизации**.

Без Protocol:

```python id="k3n8pq"
def save(repository):
    repository.save()
```

Python просто попробует вызвать `save()` во время выполнения.

С Protocol:

```python id="c5v2mx"
class Repository(Protocol):
    def save(self) -> None:
        ...


def process(repository: Repository):
    repository.save()
```

Теперь IDE и статический анализатор могут заранее проверить, соответствует ли объект нужному интерфейсу.

---

## 💼 Backend-пример

Представим сервис, которому нужно хранилище:

```python id="u8k4pd"
from typing import Protocol


class UserRepository(Protocol):
    def get_by_id(self, user_id: int) -> User:
        ...
```

Есть PostgreSQL-реализация:

```python id="s2m7qa"
class PostgresUserRepository:
    def get_by_id(self, user_id: int) -> User:
        ...
```

И тестовая in-memory реализация:

```python id="d6w9rx"
class InMemoryUserRepository:
    def get_by_id(self, user_id: int) -> User:
        ...
```

Сервису не важно, какой именно класс используется:

```python id="h1q5vc"
class UserService:
    def __init__(self, repository: UserRepository):
        self.repository = repository
```

Оба репозитория подходят, если реализуют нужный интерфейс.

Это особенно удобно для:

* Dependency Injection;
* тестирования;
* слабой связанности;
* реализации Dependency Inversion Principle.

---

## ⚠️ Важный нюанс

`Protocol` в первую очередь предназначен для **статической типизации**.

Сам Python во время выполнения не будет автоматически запрещать:

```python id="z4x8mc"
class Wrong:
    pass
```

передать `Wrong` туда, где ожидается `Protocol`.

Ошибка обнаруживается статическим анализатором или возникнет при попытке вызвать отсутствующий метод.

---

## 🔍 `@runtime_checkable`

Protocol можно сделать проверяемым во время выполнения:

```python id="v7m2qa"
from typing import Protocol, runtime_checkable


@runtime_checkable
class Speaker(Protocol):
    def speak(self) -> None:
        ...
```

Теперь:

```python id="n3k8wp"
isinstance(Dog(), Speaker)
```

может вернуть:

```text id="x4q1mz"
True
```

Но runtime-проверка ограничена проверкой наличия соответствующих атрибутов/методов и **не заменяет полноценную проверку типов**.

---

## 🧠 Главное

* `Protocol` описывает **интерфейс объекта**.
* Использует **структурную типизацию**.
* Явное наследование от `Protocol` не требуется.
* Объект подходит, если соответствует требуемой структуре.
* Это формализация **duck typing** для статической типизации.
* Часто используется с Dependency Injection и для слабой связанности.
* `ABC` → «ты должен явно быть наследником».
* `Protocol` → «мне всё равно, от кого ты наследуешься, главное — чтобы у тебя был нужный интерфейс».

## 🎤 Суперкоротко

> **`Protocol`** позволяет описать интерфейс через структурную типизацию. Класс не обязан наследоваться от `Protocol` — достаточно реализовать требуемые методы и атрибуты. Это способ формализовать duck typing для статических анализаторов типов.
