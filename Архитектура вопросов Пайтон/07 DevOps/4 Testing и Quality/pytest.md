# pytest 🧪

## 🎤 Короткий ответ

**pytest** — популярный фреймворк для написания и запуска тестов в Python. Он позволяет проверять, что отдельные функции, модули и компоненты приложения работают ожидаемым образом.

> **pytest = тесты → запуск кода → проверка результата → PASS / FAIL.**

---

## 🎯 Формула для собеседования

**Код → Test → pytest → выполнение → Assertion → PASS / FAIL**

```text
Python-код
    ↓
Тест
    ↓
pytest
    ↓
Запуск
    ↓
assert
    ↓
┌─────────────┐
│ PASS        │
│ или         │
│ FAIL        │
└─────────────┘
```

---

# 1. Что такое тестирование

**Тестирование** — проверка того, что программа работает согласно ожидаемому поведению.

Например, есть функция:

```python
def add(a, b):
    return a + b
```

Мы ожидаем:

```python
add(2, 3) == 5
```

Тест:

```python
def test_add():
    assert add(2, 3) == 5
```

pytest запускает этот тест и проверяет `assert`.

---

# 2. Что такое pytest

**pytest** — framework для тестирования Python-приложений.

Он позволяет:

* писать тесты;
* запускать тесты;
* делать assertions;
* использовать fixtures;
* параметризовать тесты;
* группировать тесты;
* работать с исключениями;
* запускать отдельные тесты;
* формировать отчёты;
* интегрироваться с CI/CD.

---

# 3. Установка

Через `pip`:

```python
pip install pytest
```

Через Poetry:

```python
poetry add --group dev pytest
```

Проверить:

```python
pytest --version
```

---

# 4. Первый тест

Допустим:

```python
def add(a, b):
    return a + b
```

Создаём:

```text
tests/
└── test_math.py
```

Внутри:

```python
from app.math import add


def test_add():
    assert add(2, 3) == 5
```

Запускаем:

```python
pytest
```

Результат:

```text
1 passed
```

---

# 5. Как pytest находит тесты

По умолчанию pytest ищет:

```text
test_*.py
*_test.py
```

А функции тестов обычно начинаются с:

```text
test_
```

Например:

```python
def test_create_user():
    ...
```

Класс тестов обычно называется:

```python
class TestUser:
    ...
```

---

# 6. `assert`

Главный механизм проверки результата:

```python
assert actual == expected
```

Например:

```python
def test_add():
    result = add(2, 3)

    assert result == 5
```

Если условие истинно:

```text
PASS
```

Если ложно:

```text
FAIL
```

---

# 7. Что происходит при FAIL

Допустим:

```python
def test_add():
    assert add(2, 3) == 10
```

pytest покажет, что ожидалось одно значение, а фактически получилось другое.

```text
Expected: 10
Actual:    5
```

Это помогает быстро найти причину ошибки.

---

# 8. Запуск pytest

Запустить все тесты:

```python
pytest
```

Подробный вывод:

```python
pytest -v
```

Запустить конкретный файл:

```python
pytest tests/test_math.py
```

Конкретный тест:

```python
pytest tests/test_math.py::test_add
```

Остановиться после первой ошибки:

```python
pytest -x
```

---

# 9. Fixtures

**Fixture** — подготовленная зависимость или данные, которые используются тестами.

Например:

```python
import pytest


@pytest.fixture
def user():
    return {
        "name": "Ilya",
        "age": 31,
    }


def test_user(user):
    assert user["name"] == "Ilya"
```

pytest автоматически передаст fixture в тест.

```text
fixture
   ↓
подготовка данных
   ↓
test
```

---

# 10. Зачем нужны fixtures

Они позволяют вынести общую подготовку тестов.

Например, несколько тестов используют одного пользователя:

```python
def test_user_name(user):
    assert user["name"] == "Ilya"


def test_user_age(user):
    assert user["age"] == 31
```

Вместо повторения:

```python
user = {
    "name": "Ilya",
    "age": 31,
}
```

используется одна fixture.

---

# 11. Scope fixture

Fixture может иметь разный жизненный цикл.

Основные:

```text
function
class
module
package
session
```

Например:

```python
@pytest.fixture(scope="session")
def database():
    ...
```

Такая fixture создаётся на уровень тестовой сессии.

### Важно

Чем шире scope, тем дольше живёт fixture.

---

# 12. `yield` в fixture

`yield` удобно использовать для setup/teardown.

```python
@pytest.fixture
def database():
    db = create_database()

    yield db

    db.close()
```

Получается:

```text
create_database()
       ↓
     yield
       ↓
      test
       ↓
   db.close()
```

---

# 13. Параметризация

**Parametrize** позволяет запускать один тест с разными входными данными.

```python
import pytest


@pytest.mark.parametrize(
    "a,b,result",
    [
        (2, 3, 5),
        (10, 5, 15),
        (1, 1, 2),
    ],
)
def test_add(a, b, result):
    assert add(a, b) == result
```

pytest запустит тест несколько раз:

```text
test_add[2-3-5] → PASS
test_add[10-5-15] → PASS
test_add[1-1-2] → PASS
```

---

# 14. Тестирование исключений

pytest позволяет проверять, что код выбрасывает ожидаемое исключение.

```python
import pytest


def divide(a, b):
    return a / b


def test_divide_by_zero():
    with pytest.raises(ZeroDivisionError):
        divide(10, 0)
```

Если исключение действительно возникло:

```text
PASS
```

Если нет:

```text
FAIL
```

---

# 15. Mock

**Mock** — подмена реального объекта или зависимости тестовым объектом.

Например, приложение обращается к внешнему API:

```text
FastAPI
   ↓
Payment API
```

В тесте необязательно делать реальный HTTP-запрос.

Можно заменить API mock-объектом:

```text
FastAPI
   ↓
Mock
```

Это позволяет:

* ускорить тесты;
* избежать зависимости от внешнего сервиса;
* контролировать ответ;
* воспроизводить ошибки.

В Python часто используется `unittest.mock`.

---

# 16. Unit Test

**Unit test** проверяет небольшую изолированную часть программы.

Например:

```python
def add(a, b):
    return a + b
```

Тест:

```python
def test_add():
    assert add(2, 3) == 5
```

```text
Function
   ↓
Unit Test
```

Обычно unit-тесты:

* быстрые;
* изолированные;
* не требуют PostgreSQL;
* не требуют реального API.

---

# 17. Integration Test

**Integration test** проверяет взаимодействие нескольких компонентов.

Например:

```text
FastAPI
   ↓
PostgreSQL
```

Тест может проверить:

```text
POST /users
    ↓
FastAPI
    ↓
ORM
    ↓
PostgreSQL
```

Это уже не просто проверка одной функции.

---

# 18. E2E Test

**End-to-End test** проверяет сценарий целиком.

Например:

```text
Client
  ↓
API
  ↓
Auth
  ↓
PostgreSQL
  ↓
Response
```

Пример сценария:

```text
Регистрация
    ↓
Авторизация
    ↓
Получение JWT
    ↓
Запрос защищённого endpoint
    ↓
Проверка результата
```

---

# 19. Unit vs Integration vs E2E

| Тип         | Что проверяет                    |
| ----------- | -------------------------------- |
| Unit        | отдельную функцию/компонент      |
| Integration | взаимодействие компонентов       |
| E2E         | полный пользовательский сценарий |

Условно:

```text
Unit
 ↓
маленький и быстрый

Integration
 ↓
несколько компонентов

E2E
 ↓
вся цепочка
```

---

# 20. pytest и FastAPI

Для backend-разработчика pytest особенно важен.

Например:

```python
from fastapi.testclient import TestClient

from app.main import app


client = TestClient(app)


def test_get_users():
    response = client.get("/users")

    assert response.status_code == 200
```

Тест проверяет HTTP endpoint.

---

# 21. pytest и PostgreSQL

Integration-тест может работать с тестовой базой:

```text
pytest
   ↓
FastAPI
   ↓
SQLAlchemy
   ↓
PostgreSQL test DB
```

Важно не использовать production database для тестов.

Для тестовой среды обычно применяют:

* отдельную БД;
* Docker Compose;
* временные БД;
* fixtures;
* транзакции/rollback;
* подготовленные test data.

---

# 22. pytest и CI/CD

Типичный pipeline:

```text
Git Push
    ↓
CI
    ↓
Ruff
    ↓
MyPy
    ↓
pytest
    ↓
Docker Build
    ↓
Deploy
```

Если тест падает:

```text
pytest
   ↓
FAILED
   ↓
CI FAILED
   ↓
Merge blocked
```

Это один из основных способов автоматической проверки качества проекта.

---

# 23. Coverage

**Test Coverage** показывает, какая часть кода была выполнена во время тестов.

Например:

```text
Coverage: 82%
```

Популярный инструмент:

```text
pytest-cov
```

Установка:

```python
pip install pytest-cov
```

Запуск:

```python
pytest --cov=app
```

Можно получить отчёт:

```text
TOTAL    82%
```

### Важно

> **Высокий coverage не гарантирует отсутствие ошибок.**

Можно выполнить много строк кода, но плохо проверить их результаты.

---

# 24. Regression Test

**Regression test** проверяет, что ранее исправленный баг не появился снова.

Например:

```text
Bug
 ↓
Fix
 ↓
Regression Test
 ↓
CI
```

Особенно полезно при hotfix.

---

# 25. Test Isolation

Хороший тест должен быть максимально независимым от других тестов.

Плохо:

```text
test_A создаёт пользователя
       ↓
test_B ожидает этого пользователя
```

Если `test_A` не запустился, `test_B` тоже ломается.

Лучше:

```text
test_A → собственные данные
test_B → собственные данные
test_C → собственные данные
```

---

# 26. Fixtures + Mock + Parametrize

Это три важных механизма pytest:

```text
Fixture
 ↓
подготовка данных

Mock
 ↓
подмена зависимости

Parametrize
 ↓
несколько наборов данных
```

Например:

```python
@pytest.mark.parametrize(
    "value,expected",
    [
        (1, 2),
        (2, 4),
        (3, 6),
    ],
)
def test_double(value, expected):
    assert value * 2 == expected
```

---

# 27. pytest vs unittest

В Python есть встроенный модуль:

```text
unittest
```

и сторонний framework:

```text
pytest
```

### unittest

* входит в стандартную библиотеку;
* использует классы и `TestCase`;
* имеет свой стиль assertions и fixtures.

### pytest

* простой синтаксис;
* обычные функции;
* мощная fixture-система;
* параметризация;
* большое количество plugins;
* удобный вывод ошибок.

Например, pytest:

```python
def test_add():
    assert add(2, 3) == 5
```

Это одна из причин его популярности в Python-проектах.

---

# 28. Что pytest не заменяет

pytest не заменяет:

* линтер;
* formatter;
* type checker;
* Code Review;
* observability;
* нагрузочное тестирование.

Типичный набор:

```text
Ruff
 ↓
код соответствует правилам

MyPy
 ↓
типы корректны

pytest
 ↓
поведение корректно

Code Review
 ↓
архитектура и качество решения
```

---

# 🎤 Вопросы на собеседовании

### Что такое pytest?

> pytest — популярный Python-фреймворк для написания и запуска автоматических тестов.

### Что такое `assert`?

> Проверка условия. Если условие истинно, тест проходит, если ложно — pytest сообщает об ошибке.

### Что такое fixture?

> Механизм pytest для подготовки общих данных или зависимостей для тестов и выполнения setup/teardown.

### Что такое parametrization?

> Возможность запускать один тест с несколькими наборами входных данных.

### Что такое mock?

> Подмена реального объекта или зависимости контролируемым тестовым объектом, чтобы изолировать тест и не обращаться к реальному внешнему ресурсу.

### Что такое Unit Test?

> Тест отдельной небольшой части программы, обычно в изоляции от внешних зависимостей.

### Чем Integration Test отличается от Unit Test?

> Integration Test проверяет взаимодействие нескольких компонентов, например приложения и PostgreSQL, а Unit Test обычно проверяет один компонент изолированно.

### Что такое E2E?

> End-to-End тест проверяет полный сценарий работы системы от начала до конца.

### Что такое Test Coverage?

> Метрика, показывающая, какая часть кода была выполнена во время тестов. Она не гарантирует качество самих тестов.

### Как проверить исключение?

```python
with pytest.raises(ValueError):
    some_function()
```

### Как запустить конкретный тест?

```python
pytest tests/test_users.py::test_create_user
```

### Зачем pytest в CI?

> Чтобы автоматически запускать тесты при изменениях и не допускать в основную ветку код, который нарушает существующее поведение.

---

## 🧠 Ключевая схема

```text
                     pytest 🧪
                         │
          ┌──────────────┼──────────────┐
          ↓              ↓              ↓
       Unit Tests   Integration       E2E
          │           Tests            Tests
          └──────────────┼──────────────┘
                         ↓
                    Assertions
                         ↓
                   PASS / FAIL
                         ↓
                        CI
```

### Самая важная формула

> **pytest = автоматический запуск тестов + проверка ожидаемого поведения программы.**

```text
Fixture      → подготовка данных/зависимостей
Parametrize  → разные входные данные
Mock         → подмена зависимости
assert       → проверка результата
pytest.raises → проверка исключения
Coverage     → измерение охвата кода
```

### Для Python Backend

```text
Ruff   → lint + format
MyPy   → type checking
pytest → tests
Black  → formatting
isort  → imports
```
