# Ruff 🦀

## 🎤 Короткий ответ

**Ruff** — быстрый инструмент для Python, который выполняет **статический анализ (linting)** и умеет **форматировать код**.

Он может заменить сразу несколько инструментов, например [[Flake8]], [[Isort]] и в части проектов [[Black]].

> **Ruff = Linter + Formatter для Python.**

---

## 🎯 Формула для собеседования

**Python-код → Ruff → linting + formatting → CI**

```text id="r001"
Python Code
     ↓
   Ruff
  ↙    ↘
Lint   Format
  ↓      ↓
Ошибки  Единый стиль
```

---

# 1. Что такое Ruff

**Ruff** — инструмент статического анализа Python-кода, написанный на **Rust**.

Основная идея — выполнять проверки значительно быстрее многих традиционных Python-инструментов.

Ruff используется для:

* поиска проблем в коде;
* проверки стиля;
* поиска неиспользуемых импортов;
* проверки различных анти-паттернов;
* сортировки импортов;
* автоматического исправления части проблем;
* форматирования кода.

---

# 2. Ruff — это не только линтер

Изначально Ruff прежде всего воспринимался как **linter**.

Сейчас он включает несколько возможностей:

```text id="r002"
Ruff
 ├── Linting
 ├── Auto-fix
 ├── Import sorting
 └── Formatting
```

Поэтому можно построить Python-проект практически вокруг одного инструмента.

---

# 3. Установка

Через `pip`:

```python id="r003"
pip install ruff
```

Через Poetry:

```python id="r004"
poetry add --group dev ruff
```

Проверить установку:

```python id="r005"
ruff --version
```

---

# 4. Основная команда — `ruff check`

Запустить linting проекта:

```python id="r006"
ruff check .
```

Ruff анализирует Python-файлы и сообщает о найденных проблемах.

Например:

```text id="r007"
F401 [*] `os` imported but unused
```

---

# 5. Автоматическое исправление

Ruff умеет автоматически исправлять часть найденных проблем.

```python id="r008"
ruff check . --fix
```

Например:

```text id="r009"
unused import
       ↓
ruff --fix
       ↓
import удалён
```

Важно:

> **Не все проблемы можно безопасно исправить автоматически.**

Поэтому результат `--fix` всё равно стоит проверить.

---

# 6. Коды ошибок Ruff

Ruff использует коды диагностик.

Например:

```text id="r010"
F401
F841
E501
E711
```

Они позволяют понять, какое правило нарушено.

Например:

```text id="r011"
F401 → unused import
```

---

# 7. Источники правил

Ruff реализует множество правил и совместим с большим количеством популярных наборов проверок.

Например:

```text id="r012"
E / W → pycodestyle
F     → Pyflakes
I     → isort
B     → flake8-bugbear
UP    → pyupgrade
```

Поэтому Ruff может заменить сразу несколько отдельных инструментов.

---

# 8. Конфигурация Ruff

Конфигурацию удобно хранить в:

```text id="r013"
pyproject.toml
```

Например:

```python id="r014"
[tool.ruff]
line-length = 88

[tool.ruff.lint]
select = ["E", "F", "I"]
```

Здесь включаются группы правил:

* `E` — pycodestyle;
* `F` — Pyflakes;
* `I` — сортировка импортов.

Конкретный набор правил выбирается проектом.

---

# 9. Formatter Ruff

Ruff также умеет форматировать Python-код.

```python id="r015"
ruff format .
```

Проверить форматирование:

```python id="r016"
ruff format --check .
```

То есть:

```text id="r017"
ruff check .
        ↓
      linting

ruff format .
        ↓
    formatting
```

---

# 10. Ruff vs Black

Оба могут форматировать Python.

### Black

```text id="r018"
Black
 ↓
Formatting
```

### Ruff

```text id="r019"
Ruff
 ├── Linting
 └── Formatting
```

Поэтому возможны два подхода:

```text id="r020"
Вариант 1:
Black + Ruff

Вариант 2:
Ruff
```

Если Ruff используется для formatting, отдельный Black может быть не нужен.

---

# 11. Ruff vs Flake8

Исторически популярная схема:

```text id="r021"
Flake8
 ↓
Linting

isort
 ↓
Imports

Black
 ↓
Formatting
```

Современный вариант:

```text id="r022"
Ruff
 ├── Linting
 ├── Import sorting
 └── Formatting
```

Одно из преимуществ Ruff — скорость и консолидация нескольких инструментов.

---

# 12. Ruff vs isort

`isort` предназначен прежде всего для **сортировки импортов**.

Например:

```python id="r023"
from app.users import User
import os
import requests
```

После сортировки:

```python id="r024"
import os

import requests

from app.users import User
```

Ruff умеет выполнять проверки и исправления, связанные с сортировкой импортов:

```python id="r025"
ruff check . --select I --fix
```

---

# 13. Ruff vs MyPy

Это разные задачи.

### Ruff

```text id="r026"
linting
style
imports
static rules
```

### MyPy

```text id="r027"
type checking
```

Например:

```python id="r028"
def get_user(user_id: int) -> str:
    return user_id
```

Проблему типов должен обнаруживать type checker вроде MyPy, а не formatter.

---

# 14. Ruff vs pytest

### Ruff

Анализирует исходный код:

```text id="r029"
«Есть ли потенциальные проблемы
с точки зрения правил?»
```

### pytest

Запускает программу/тесты:

```text id="r030"
«Работает ли код так,
как ожидается?»
```

Поэтому они дополняют друг друга.

---

# 15. Ruff и PEP 8

Ruff может проверять ряд правил, связанных со стилем Python.

Но:

```text id="r031"
PEP 8 ≠ Ruff
```

**PEP 8** — рекомендации по стилю.

**Ruff** — инструмент, который автоматически анализирует код по выбранным правилам.

---

# 16. Ruff в CI

Типичный pipeline:

```text id="r032"
Git Push
    ↓
CI
    ↓
ruff check .
    ↓
ruff format --check .
    ↓
mypy .
    ↓
pytest
    ↓
Docker build
```

Если Ruff обнаружит проблему:

```text id="r033"
Ruff → FAILED
      ↓
CI → FAILED
```

Merge может быть запрещён правилами репозитория.

---

# 17. Ruff + Pre-commit

Ruff удобно использовать через `pre-commit`.

Например:

```text id="r034"
git commit
    ↓
pre-commit
    ↓
Ruff
 ├── lint
 └── format
    ↓
commit
```

Это позволяет автоматически проверять код до попадания изменений в репозиторий.

---

# 18. Ruff в IDE

Ruff можно интегрировать в IDE.

Тогда проблемы отображаются непосредственно во время разработки:

```text id="r035"
import os
       ^^^
F401 unused import
```

Разработчик исправляет проблему до:

```text id="r036"
git commit
```

---

# 19. Ruff и Code Review

Без автоматических проверок reviewer может тратить время на:

* неиспользуемые импорты;
* стиль;
* сортировку импортов;
* простые статические ошибки.

Ruff переносит значительную часть таких проверок в автоматический процесс:

```text id="r037"
Developer
    ↓
Ruff
    ↓
CI
    ↓
Code Review
```

Reviewer больше сосредотачивается на:

* бизнес-логике;
* архитектуре;
* корректности решения;
* безопасности;
* производительности.

---

# 20. Пример Python Backend

Допустим, проект FastAPI.

В `pyproject.toml`:

```python id="r038"
[tool.ruff]
line-length = 88

[tool.ruff.lint]
select = ["E", "F", "I"]
```

Проверяем:

```python id="r039"
ruff check .
```

Форматируем:

```python id="r040"
ruff format .
```

Проверяем формат:

```python id="r041"
ruff format --check .
```

Автоматически исправляем lint-проблемы:

```python id="r042"
ruff check . --fix
```

---

# 21. Типичный набор инструментов Python Backend

Современный проект может выглядеть так:

```text id="r043"
Python Project
      │
      ├── Ruff
      │    ├── lint
      │    └── format
      │
      ├── MyPy
      │    └── type checking
      │
      └── Pytest
           └── tests
```

Pipeline:

```text id="r044"
Code
 ↓
Ruff
 ↓
MyPy
 ↓
Pytest
 ↓
Build
 ↓
Deploy
```

---

# 22. Почему Ruff быстрый

Ruff написан на **Rust**, а не на Python.

Это позволяет ему быстро обрабатывать большие проекты и запускать большое количество проверок.

Для разработчика это означает:

```text id="r045"
изменил код
    ↓
Ruff
    ↓
быстрая проверка
```

Поэтому его удобно запускать:

* локально;
* при сохранении;
* в pre-commit;
* в CI.

---

# 23. Что Ruff НЕ делает

Ruff не заменяет полностью:

### pytest

Не проверяет бизнес-логику приложения.

### MyPy

Не является полноценной заменой type checker для строгой проверки типов.

### Code Review

Не оценивает архитектуру так, как человек.

### Integration/E2E tests

Не проверяет взаимодействие всех компонентов приложения.

---

# 🎤 Вопросы на собеседовании

### Что такое Ruff?

> Ruff — быстрый инструмент статического анализа Python-кода, который выполняет linting и также умеет форматировать код.

### Почему Ruff используют вместо Flake8?

> Ruff выполняет множество lint-проверок значительно быстрее и может заменить несколько отдельных инструментов.

### Может ли Ruff форматировать код?

> Да. У Ruff есть собственный formatter: `ruff format`.

### Чем Ruff отличается от Black?

> Black специализируется на форматировании, а Ruff может выполнять и linting, и formatting.

### Чем Ruff отличается от MyPy?

> Ruff в основном выполняет linting и formatting, а MyPy специализируется на статической проверке типов.

### Что делает `ruff check`?

> Запускает lint-проверки Python-кода.

### Что делает `ruff check --fix`?

> Автоматически исправляет те найденные lint-проблемы, для которых Ruff поддерживает безопасное исправление.

### Что делает `ruff format`?

> Автоматически форматирует Python-код.

### Что делает `ruff format --check`?

> Проверяет форматирование без изменения файлов. Особенно удобно использовать в CI.

### Можно ли использовать Ruff в CI?

> Да. Обычно в CI запускают `ruff check .` и `ruff format --check .` вместе с MyPy и pytest.

---

## 🧠 Ruff в одном экране

```text
                       Ruff 🦀
                          │
             ┌────────────┴────────────┐
             ↓                         ↓
          Linting                  Formatting
             │                         │
        ruff check .             ruff format .
             │                         │
             ↓                         ↓
       Поиск проблем              Единый стиль
             │                         │
             └────────────┬────────────┘
                          ↓
                         CI
                          ↓
                     Code Review
```

### Самая важная формула

> **Ruff = быстрый Python linter + formatter.**

```text id="r046"
ruff check .          → linting
ruff check . --fix    → auto-fix
ruff format .         → formatting
ruff format --check . → проверка formatting
```

### Связка для собеседования

```text id="r047"
Ruff   → lint + format
MyPy   → type checking
Pytest → tests
Black  → formatting
```
