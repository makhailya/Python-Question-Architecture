# ⚙️ CI/CD

## 🎯 Ответ на собеседовании

**CI/CD** — это подход к автоматизации разработки, тестирования и доставки программного обеспечения.

**CI (Continuous Integration)** — непрерывная интеграция: разработчики регулярно отправляют изменения в репозиторий, после чего автоматически запускаются проверки — тесты, линтеры, сборка и другие проверки.

**CD (Continuous Delivery / Continuous Deployment)** — автоматизация доставки изменений. После успешного CI приложение может автоматически попасть на staging или production в зависимости от настроек pipeline.

Типичный процесс:

```text
Разработчик
    ↓
git push
    ↓
GitHub
    ↓
CI
 ├── Tests
 ├── Linter
 ├── Build
 └── Checks
    ↓
CD
    ↓
Deploy
    ↓
Production
```

---

## 🎤 Суперкоротко

**CI — автоматически проверяем изменения.**

**CD — автоматически доставляем и разворачиваем проверенные изменения.**

---

## 🔄 CI — Continuous Integration

**Continuous Integration** означает, что изменения разработчиков регулярно интегрируются в общий репозиторий и автоматически проверяются.

Например:

```text
git push
   ↓
CI
   ↓
pytest
   ↓
flake8
   ↓
build
   ↓
✅
```

Если тесты не прошли:

```text
git push
   ↓
pytest
   ↓
❌
   ↓
Pipeline failed
```

Изменение не должно попадать дальше по pipeline.

---

## 🧪 Что обычно делает CI

CI pipeline может выполнять:

* установку зависимостей;
* запуск тестов;
* проверку линтером;
* проверку форматирования;
* статический анализ;
* сборку Docker-образа;
* проверку миграций;
* security checks.

Для Python-проекта:

```text
Checkout code
      ↓
Install dependencies
      ↓
pytest
      ↓
flake8
      ↓
black --check
      ↓
Docker build
```

---

## 🚀 CD — Continuous Delivery

**Continuous Delivery** — автоматизация подготовки приложения к выпуску.

После успешного CI:

```text
CI
 ↓
Tests ✅
 ↓
Build ✅
 ↓
Artifact
 ↓
Staging
```

При Continuous Delivery production deployment может требовать ручного подтверждения.

```text
Staging
   ↓
Manual approval
   ↓
Production
```

---

## 🤖 Continuous Deployment

**Continuous Deployment** — следующий уровень автоматизации.

После успешного pipeline изменения автоматически разворачиваются в production.

```text
git push
   ↓
CI
   ↓
Tests ✅
   ↓
Build ✅
   ↓
Deploy
   ↓
Production
```

То есть:

**Continuous Delivery → production готов к deployment.**

**Continuous Deployment → deployment в production происходит автоматически.**

---

## 🆚 CI vs CD

| CI                  | CD                   |
| ------------------- | -------------------- |
| Проверяет изменения | Доставляет изменения |
| Тесты               | Deployment           |
| Линтеры             | Staging / Production |
| Build               | Release              |
| Контроль качества   | Доставка приложения  |

---

## 🧱 Pipeline

**Pipeline** — последовательность автоматизированных этапов обработки изменений.

Например:

```text
Push
 ↓
Lint
 ↓
Tests
 ↓
Build
 ↓
Docker image
 ↓
Deploy
```

Если один из обязательных этапов завершился ошибкой:

```text
Tests
  ↓
❌
  ↓
Pipeline остановлен
```

---

## 🐳 CI/CD и Docker

Docker часто используется для создания одинакового окружения.

```text
Code
 ↓
Docker build
 ↓
Docker Image
 ↓
Registry
 ↓
Server
 ↓
Container
```

Например:

```text
GitHub Actions
      ↓
docker build
      ↓
Docker Image
      ↓
Docker Hub / Registry
      ↓
Production
```

---

## 🐙 GitHub Actions

Для проектов на GitHub часто используется **GitHub Actions**.

Pipeline описывается YAML-файлом:

```text
.github/
└── workflows/
    └── ci.yml
```

Упрощённый пример:

```python
name: CI

on:
  push:
  pull_request:

jobs:
  tests:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Install Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.13"

      - name: Install dependencies
        run: pip install -r requirements.txt

      - name: Run tests
        run: pytest

      - name: Run linter
        run: flake8 .
```

После `git push` GitHub автоматически запускает workflow.

---

## 🔐 Secrets в CI/CD

CI/CD часто работает с секретами:

```text
DATABASE_URL
API_KEY
SECRET_KEY
DOCKER_PASSWORD
```

Их нельзя хранить непосредственно в коде или публичном репозитории.

Например:

```text
GitHub Secrets
      ↓
GitHub Actions
      ↓
Environment variables
```

---

## 📦 Artifact

**Artifact** — результат сборки, который можно передать на следующий этап pipeline.

Например:

```text
Source Code
    ↓
Build
    ↓
Docker Image
    ↓
Artifact
```

Artifact может быть:

* Docker image;
* wheel-пакет Python;
* архив приложения;
* собранный frontend;
* бинарный файл.

---

## 🌍 Типичный pipeline Backend

Для FastAPI-приложения:

```text
Developer
    ↓
git push
    ↓
GitHub
    ↓
GitHub Actions
    ↓
┌─────────────────┐
│ Install deps    │
│ Tests           │
│ Linter          │
│ Type checking   │
└────────┬────────┘
         ↓
   Docker build
         ↓
   Docker Registry
         ↓
      Deploy
         ↓
      Server
         ↓
     FastAPI
```

---

## 🔄 CI/CD и Git

Типичный workflow:

```text
feature branch
      ↓
Pull Request
      ↓
CI
      ↓
Tests ✅
      ↓
Code Review
      ↓
merge
      ↓
CD
      ↓
Deploy
```

Это позволяет не выкатывать непроверенный код.

---

## 🧪 Почему CI/CD нужен

Основные преимущества:

* автоматизация;
* раннее обнаружение ошибок;
* быстрый feedback;
* единый процесс сборки;
* уменьшение количества ручных операций;
* более предсказуемый deployment;
* возможность часто выпускать небольшие изменения.

---

## ⚠️ Что происходит при ошибке

Например:

```text
git push
   ↓
CI
   ↓
pytest
   ↓
❌ test failed
```

Pipeline становится **failed**.

Deployment не должен выполняться, если он зависит от успешного прохождения тестов.

После исправления:

```text
git push
   ↓
CI
   ↓
pytest ✅
   ↓
build ✅
   ↓
deploy
```

---

## 🧠 CI/CD ≠ только deployment

Это частая ошибка.

CI/CD — это не просто:

```text
git push → deploy
```

Pipeline может включать:

```text
Code
 ↓
Tests
 ↓
Lint
 ↓
Security
 ↓
Build
 ↓
Artifact
 ↓
Deploy
 ↓
Monitoring
```

---

## 🎯 Главное

```text
CI
 ↓
Continuous Integration
 ↓
автоматическая проверка изменений
 ↓
Tests + Lint + Build

CD
 ↓
Continuous Delivery / Deployment
 ↓
доставка приложения
 ↓
Staging / Production
```

### Формула для собеседования

**CI = проверить изменения автоматически.**

**Continuous Delivery = автоматически подготовить изменения к выпуску.**

**Continuous Deployment = автоматически выпустить изменения в production.**

### Типичный Python Backend pipeline

**git push → tests → lint → build → Docker image → registry → deploy**
