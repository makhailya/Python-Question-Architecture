# isort 📦

## 🎤 Короткий ответ

**isort** — инструмент для Python, который **автоматически сортирует и форматирует импорты** в коде по заданным правилам.

Он группирует импорты, сортирует их по алфавиту и отделяет стандартную библиотеку от сторонних и локальных импортов.

> **isort = автоматический порядок импортов Python.**

---

## 🎯 Формула для собеседования

**Python-код → isort → сортировка импортов → единый стиль**

```text id="i001"
Python Code
     ↓
   isort
     ↓
┌──────────────────────┐
│ stdlib               │
│ third-party          │
│ local imports        │
└──────────────────────┘
```

---

# 1. Зачем нужен isort

Без автоматической сортировки каждый разработчик может располагать импорты по-разному:

```python id="i002"
from app.users import User
import os
from fastapi import FastAPI
import sys
from app.db import database
```

isort приведёт их к более структурированному виду:

```python id="i003"
import os
import sys

from fastapi import FastAPI

from app.db import database
from app.users import User
```

Таким образом:

* код становится единообразным;
* проще читать imports;
* проще Code Review;
* меньше конфликтов;
* не нужно вручную сортировать импорты.

---

# 2. Как isort группирует импорты

Обычно импорты разделяются на группы.

### 1. Standard Library

Стандартная библиотека Python:

```python id="i004"
import os
import sys
from pathlib import Path
```

### 2. Third-party

Внешние зависимости:

```python id="i005"
from fastapi import FastAPI
from pydantic import BaseModel
import requests
```

### 3. First-party / Local

Импорты собственного проекта:

```python id="i006"
from app.models import User
from app.services import UserService
```

В результате:

```text id="i007"
Standard Library
        ↓

Third-party
        ↓

First-party
```

---

# 3. Установка

Через `pip`:

```python id="i008"
pip install isort
```

Через Poetry:

```python id="i009"
poetry add --group dev isort
```

Проверить установку:

```python id="i010"
isort --version
```

---

# 4. Основная команда

Отсортировать импорты проекта:

```python id="i011"
isort .
```

Конкретный файл:

```python id="i012"
isort app/main.py
```

Директорию:

```python id="i013"
isort app/
```

isort изменяет файлы автоматически.

---

# 5. Проверка без изменения файлов

Для CI удобно использовать:

```python id="i014"
isort --check-only .
```

То есть:

> Проверить порядок импортов, но не изменять файлы.

Можно также посмотреть предполагаемые изменения:

```python id="i015"
isort --diff .
```

---

# 6. Пример

Было:

```python id="i016"
from app.models import User
import os
from fastapi import FastAPI
import sys
from app.database import db
```

После:

```python id="i017"
import os
import sys

from fastapi import FastAPI

from app.database import db
from app.models import User
```

---

# 7. isort не является линтером

Это важное различие.

**isort**:

```text id="i018"
сортирует импорты
```

**Flake8 / Ruff**:

```text id="i019"
анализируют код
```

**Black**:

```text id="i020"
форматирует код
```

Например:

```text id="i021"
isort
  ↓
порядок импортов

Black
  ↓
общее форматирование

Ruff
  ↓
linting
```

---

# 8. isort и Black

isort и Black часто использовались вместе:

```text id="i022"
isort
  ↓
imports

Black
  ↓
formatting
```

Но между ними исторически возникала проблема:

> Некоторые правила форматирования импортов и форматирования Black могли пересекаться.

Поэтому для совместной работы используются настройки, совместимые с Black.

Например:

```python id="i023"
[tool.isort]
profile = "black"
```

Это говорит isort использовать настройки, совместимые с Black.

---

# 9. isort и Ruff

Ruff также умеет проверять и автоматически исправлять порядок импортов.

Например:

```python id="i024"
ruff check . --select I --fix
```

Здесь `I` соответствует правилам сортировки импортов, совместимым с isort.

Поэтому современный проект может использовать:

```text id="i025"
Ruff
 ├── linting
 ├── import sorting
 └── formatting
```

и отдельный `isort` уже не понадобится.

---

# 10. isort и Flake8

Flake8 сам по себе не является полноценной заменой isort.

Исторически можно было встретить:

```text id="i026"
Flake8
 +
isort
 +
Black
```

Где:

```text id="i027"
Flake8 → linting
isort  → imports
Black  → formatting
```

---

# 11. isort и PEP 8

PEP 8 содержит рекомендации по оформлению импортов.

isort автоматизирует сортировку импортов в соответствии со своими правилами.

Важно:

```text id="i028"
PEP 8 ≠ isort
```

PEP 8 — рекомендации.

isort — инструмент автоматической сортировки.

---

# 12. isort не проверяет бизнес-логику

Например:

```python id="i029"
from payments import charge

charge(user, amount)
```

isort может проверить только порядок импорта.

Он не определяет:

* правильно ли рассчитывается `amount`;
* можно ли списывать деньги;
* существует ли пользователь;
* корректна ли бизнес-логика.

Для этого нужны тесты и другие инструменты.

---

# 13. Конфигурация

Настройки можно хранить в:

```text id="i030"
pyproject.toml
```

Например:

```python id="i031"
[tool.isort]
profile = "black"
line_length = 88
```

Можно также настроить собственные категории импортов.

Например, определить, какие пакеты считать first-party.

---

# 14. isort и first-party imports

Для большого Python-проекта важно правильно определить локальные модули.

Например:

```text id="i032"
project/
├── app/
│   ├── models/
│   ├── services/
│   └── api/
└── tests/
```

Импорты:

```python id="i033"
from app.models import User
from app.services import UserService
```

должны восприниматься как **first-party imports**, а не third-party.

---

# 15. isort в CI

Типичный старый pipeline:

```text id="i034"
Git Push
    ↓
CI
    ↓
isort --check-only .
    ↓
black --check .
    ↓
flake8 .
    ↓
mypy .
    ↓
pytest
```

Если импорты расположены неправильно:

```text id="i035"
isort → FAILED
```

Pipeline может завершиться ошибкой.

---

# 16. isort + pre-commit

Можно запускать isort перед commit:

```text id="i036"
git commit
    ↓
pre-commit
    ↓
isort
    ↓
Black
    ↓
Flake8 / Ruff
    ↓
commit
```

Это позволяет автоматически поддерживать порядок импортов.

---

# 17. Современный вариант с Ruff

В современном Python-проекте вместо:

```text id="i037"
isort
Black
Flake8
```

можно использовать:

```text id="i038"
Ruff
```

Например:

```python id="i039"
ruff check . --fix
ruff format .
```

Ruff способен:

* выполнять linting;
* сортировать импорты;
* автоматически исправлять часть проблем;
* форматировать код.

Но это не означает, что `isort`, `Black` и `Flake8` исчезли: они всё ещё встречаются в существующих проектах и поддерживаемых командами конфигурациях.

---

# 18. Типичный Python Backend

### Классическая схема

```text id="i040"
Python Backend
      │
      ├── isort  → imports
      ├── Black  → formatting
      ├── Flake8 → linting
      ├── MyPy   → types
      └── Pytest → tests
```

### Современная схема

```text id="i041"
Python Backend
      │
      ├── Ruff → lint + imports + format
      ├── MyPy → type checking
      └── Pytest → tests
```

---

# 19. Основные команды

| Команда                | Назначение                        |
| ---------------------- | --------------------------------- |
| `isort .`              | отсортировать импорты             |
| `isort file.py`        | отсортировать конкретный файл     |
| `isort --check-only .` | проверить без изменения           |
| `isort --diff .`       | показать предполагаемые изменения |
| `isort --version`      | показать версию                   |

---

# 🎤 Вопросы на собеседовании

### Что такое isort?

> isort — инструмент для автоматической сортировки и форматирования импортов Python-кода.

### Что именно сортирует isort?

> Он группирует и сортирует импорты, например отделяет стандартную библиотеку, сторонние зависимости и локальные импорты проекта.

### isort — это линтер?

> Нет. Его основная задача — порядок и форматирование импортов.

### isort — это formatter?

> В узком смысле он форматирует именно блоки импортов, но не является универсальным форматтером всего Python-кода, как Black или Ruff Formatter.

### Чем isort отличается от Black?

> isort занимается импортами, а Black форматирует весь Python-код.

### Чем isort отличается от Ruff?

> Ruff умеет выполнять linting, форматирование и сортировку импортов, поэтому во многих современных проектах может заменить отдельный isort.

### Как проверить импорты без изменения файлов?

```python id="i042"
isort --check-only .
```

### Зачем `profile = "black"`?

> Чтобы настроить isort на стиль, совместимый с Black и избежать конфликтов между инструментами форматирования.

---

## 🧠 Ключевая схема

```text id="i043"
                 Python Code
                      │
                      ▼
                    isort
                      │
                      ▼
               Import Sorting
                      │
          ┌───────────┼───────────┐
          ↓           ↓           ↓
       stdlib     third-party   local
          │           │           │
          └───────────┼───────────┘
                      ↓
               Единый порядок
```

### Самая важная формула

> **isort = автоматическая сортировка импортов Python.**

```text id="i044"
isort  → imports
Black  → formatting
Flake8 → linting
Ruff   → lint + imports + format
MyPy   → types
Pytest → tests
```
