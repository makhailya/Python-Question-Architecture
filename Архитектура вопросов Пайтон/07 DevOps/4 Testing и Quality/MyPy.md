# 🐍 MyPy

## 🎤 Короткий ответ

**MyPy** — статический анализатор типов для Python. Он проверяет аннотации типов **до запуска программы** и помогает находить ошибки типов на этапе разработки.

Например:

```python
def add(a: int, b: int) -> int:
    return a + b


result = add(10, "20")
```

MyPy обнаружит проблему:

```python
Argument 2 to "add" has incompatible type "str"; expected "int"
```

---

## 🎯 Формула для собеседования

> **MyPy = type hints + статический анализ → проверка типов без запуска программы.**

Главное:

```python
Код
 ↓
Type Hints
 ↓
MyPy
 ↓
Ошибки типов обнаружены ДО запуска
```

---

# 🔎 Что такое статическая типизация

Python является **динамически типизированным** языком.

Тип переменной определяется во время выполнения:

```python
x = 10
x = "hello"
```

Python это позволяет.

MyPy добавляет поверх Python **статическую проверку типов**:

```python
x: int = 10
x = "hello"
```

MyPy сообщит об ошибке.

При этом сам Python не становится статически типизированным — MyPy просто анализирует код.

---

# 🏷️ Type Hints

MyPy использует стандартные аннотации типов Python:

```python
name: str = "Ilya"
age: int = 31
is_active: bool = True
```

Для функций:

```python
def calculate(a: int, b: int) -> int:
    return a + b
```

MyPy проверяет:

* аргументы;
* возвращаемое значение;
* присваивания;
* типы переменных;
* совместимость типов;
* некоторые операции с объектами.

---

# ❌ Пример ошибки

```python
def greet(name: str) -> str:
    return "Hello " + name


greet(123)
```

MyPy:

```python
Argument 1 to "greet" has incompatible type "int"; expected "str"
```

Проблема обнаруживается **до запуска программы**.

---

# 📦 Коллекции

Можно указывать тип элементов:

```python
numbers: list[int] = [1, 2, 3]
names: list[str] = ["Anna", "Bob"]
```

Например:

```python
numbers: list[int] = [1, 2, "3"]
```

MyPy найдёт несовместимый `str`.

---

# 🗂️ Dict

```python
users: dict[int, str] = {
    1: "Anna",
    2: "Bob",
}
```

Здесь:

```text
key   → int
value → str
```

---

# 🧩 Optional

Если значение может быть `None`:

```python
def get_user_name(user_id: int) -> str | None:
    ...
```

Это означает:

```text
str
или
None
```

В современных версиях Python используется:

```python
str | None
```

В старом синтаксисе:

```python
Optional[str]
```

---

# 🆚 Any

`Any` фактически отключает строгую проверку типа для конкретного значения.

```python
from typing import Any

data: Any = get_data()
```

Можно сделать:

```python
data.foo()
data["name"]
data + 10
```

MyPy не будет нормально ограничивать операции с `Any`.

Поэтому чрезмерное использование `Any` снижает пользу статической типизации.

---

# 🧬 Union

`Union` означает, что значение может иметь несколько типов.

Современный синтаксис:

```python
def parse(value: str | int) -> str:
    ...
```

Например:

```python
value: str | int = 10
```

---

# 🔀 Type Narrowing

MyPy умеет сужать тип после проверки.

```python
def process(value: str | int) -> None:
    if isinstance(value, str):
        print(value.upper())
    else:
        print(value + 10)
```

До проверки:

```text
str | int
```

После:

```text
if isinstance(value, str):
    str
```

В `else`:

```text
int
```

Это называется **type narrowing**.

---

# 🏗️ Классы

MyPy проверяет совместимость типов объектов:

```python
class User:
    def __init__(self, name: str):
        self.name = name


user = User("Ilya")
```

Если:

```python
user = User(123)
```

MyPy обнаружит ошибку.

---

# 🧬 Наследование

MyPy учитывает наследование.

```python
class Animal:
    def speak(self) -> str:
        ...


class Dog(Animal):
    def speak(self) -> str:
        return "Woof"
```

Можно использовать:

```python
def make_sound(animal: Animal) -> str:
    return animal.speak()
```

`Dog` подходит туда, где ожидается `Animal`.

---

# 🔌 Protocol

`Protocol` позволяет описывать интерфейс объекта через его структуру.

```python
from typing import Protocol


class Logger(Protocol):
    def log(self, message: str) -> None:
        ...
```

Любой объект, у которого есть подходящий:

```python
log(message: str) -> None
```

может соответствовать этому Protocol.

Это связано с **structural typing** и хорошо сочетается с duck typing Python.

---

# ⚙️ Как установить MyPy

```python
pip install mypy
```

Или через Poetry:

```python
poetry add --group dev mypy
```

Проверка:

```python
mypy .
```

Конкретный файл:

```python
mypy app/main.py
```

---

# ⚙️ Конфигурация

Настройки можно хранить, например, в `pyproject.toml`.

```python
[tool.mypy]
python_version = "3.13"
strict = true
```

После этого:

```python
mypy .
```

будет использовать эти настройки.

---

# 🔒 Strict Mode

У MyPy есть более строгий режим:

```python
strict = true
```

Он включает набор дополнительных проверок.

Например, MyPy будет строже относиться к:

* отсутствующим аннотациям;
* `Any`;
* функциям;
* возвращаемым значениям;
* типам аргументов.

Для production-проектов это может значительно повысить качество типизации, но внедрять strict mode в старый проект иногда приходится постепенно.

---

# 🧪 MyPy в CI/CD

MyPy удобно запускать в CI.

Например:

```python
pytest
mypy .
flake8 .
black --check .
```

Pipeline:

```python
Git Push
   ↓
CI
   ├── pytest
   ├── mypy
   ├── flake8
   └── black --check
   ↓
Build
```

Если MyPy обнаружил ошибку:

```text
CI ❌
```

и pipeline может остановить дальнейшую сборку.

---

# 🆚 MyPy vs pytest

| MyPy                       | pytest                    |
| -------------------------- | ------------------------- |
| Проверяет типы             | Проверяет поведение       |
| Статический анализ         | Выполнение тестов         |
| Не запускает бизнес-логику | Запускает код             |
| Находит ошибки типов       | Находит логические ошибки |
| Использует Type Hints      | Использует тесты          |

Они дополняют друг друга.

Например:

```python
def divide(a: int, b: int) -> float:
    return a / b
```

MyPy может проверить типы.

А pytest может проверить:

```python
assert divide(10, 2) == 5
```

---

# 🆚 MyPy vs линтер

**MyPy**:

```text
Типы
```

**Flake8/Ruff**:

```text
Стиль + потенциальные проблемы кода
```

**Black**:

```text
Форматирование
```

**pytest**:

```text
Поведение приложения
```

Вместе:

```python
MyPy
  +
Ruff / Flake8
  +
Black
  +
pytest
```

дают более полную автоматическую проверку проекта.

---

# 🐍 Пример Python Backend

Допустим, есть FastAPI:

```python
from fastapi import FastAPI

app = FastAPI()


def get_user(user_id: int) -> str:
    return f"User {user_id}"


@app.get("/users/{user_id}")
def user(user_id: int) -> str:
    return get_user(user_id)
```

MyPy позволяет контролировать типы между слоями приложения:

```text
API
 ↓
Service
 ↓
Repository
 ↓
Database
```

Например:

```python
def get_user(user_id: int) -> User:
    ...
```

и:

```python
def get_user(user_id: str) -> User:
    ...
```

Несоответствие типов может быть найдено ещё до запуска приложения.

---

# ⚠️ Что MyPy НЕ делает

MyPy не гарантирует отсутствие всех ошибок.

Он не проверяет автоматически:

* бизнес-логику;
* корректность SQL;
* доступность PostgreSQL;
* HTTP-ошибки;
* производительность;
* runtime-данные;
* правильность алгоритма.

Например:

```python
def calculate_price(price: float) -> float:
    return price * 100
```

С точки зрения типов всё может быть корректно, хотя бизнес-логика может быть неправильной.

---

# 🧠 Статическая проверка vs Runtime

Важно понимать разницу:

```python
MyPy
 ↓
проверка до запуска
```

и:

```python
Python
 ↓
выполнение программы
 ↓
runtime errors
```

Например, данные из API могут иметь неожиданный формат:

```python
data = external_api()
```

Даже если переменная типизирована:

```python
data: User
```

реальные данные сами по себе не становятся `User`.

Для runtime-валидации используются инструменты вроде:

```text
Pydantic
```

---

# 🆚 MyPy и Pydantic

Это особенно важно для Backend.

### MyPy

Проверяет типы **статически**:

```python
user_id: int
```

### Pydantic

Проверяет/валидирует **реальные данные во время выполнения**:

```python
class User(BaseModel):
    id: int
    name: str
```

Например, FastAPI активно использует Pydantic для runtime-валидации входных данных.

Упрощённо:

```text
MyPy
  ↓
"Код типизирован правильно?"

Pydantic
  ↓
"Реальные данные соответствуют схеме?"
```

---

# 🎤 Как рассказать на собеседовании

> **MyPy — статический анализатор типов для Python. Он использует type hints и проверяет совместимость типов без запуска программы. Например, может обнаружить передачу `str` в функцию, которая ожидает `int`. MyPy часто запускают вместе с pytest, Ruff или Flake8 и Black в CI/CD pipeline. При этом MyPy не заменяет runtime-валидацию — для реальных входных данных, например в FastAPI, используются Pydantic и другие механизмы валидации.**

---

# ❓ Частые вопросы

### MyPy делает Python статически типизированным?

**Нет.** Python остаётся динамически типизированным. MyPy выполняет дополнительный статический анализ.

### Нужны ли Type Hints для MyPy?

Да, Type Hints являются основным источником информации о типах.

### Запускает ли MyPy программу?

**Нет.** Он анализирует исходный код статически.

### Может ли MyPy найти все ошибки?

Нет. Он специализируется прежде всего на типах и связанных с ними проблемах.

### Чем MyPy отличается от pytest?

MyPy проверяет типы, pytest проверяет поведение программы через выполнение тестов.

### Чем MyPy отличается от Pydantic?

MyPy — статическая проверка типов, Pydantic — runtime-валидация данных.

### Что такое `Any`?

`Any` означает, что значение может считаться совместимым практически с любым типом, поэтому чрезмерное использование `Any` ослабляет статическую проверку.

---

## 🔑 Главное

```python
Type Hints
    ↓
   MyPy
    ↓
Static Type Checking
    ↓
Ошибки типов найдены до запуска
```

**MyPy — это инструмент статической проверки типов Python-кода.**

```python
MyPy   → типы
pytest → поведение
Ruff   → код/стиль
Black  → форматирование
Pydantic → runtime-валидация
```
