# 🦊 GitLab

## 🎤 Короткий ответ

**GitLab** — это платформа для разработки программного обеспечения, которая использует Git и предоставляет инструменты для хранения репозиториев, совместной разработки, Code Review и автоматизации CI/CD.

GitLab позволяет хранить **Remote Repository**, работать с ветками и Merge Request, а также запускать автоматические проверки и деплой через **GitLab CI/CD**.

Важно различать:

> **Git — система контроля версий, GitLab — платформа вокруг Git.**

---

## 🗣️ Ответ на собеседовании

GitLab — это платформа для работы с Git-репозиториями и организации процесса разработки.

В GitLab можно хранить удалённые репозитории, создавать ветки, делать Merge Request, проводить Code Review, управлять Issues и запускать CI/CD pipelines.

Сам Git при этом остаётся отдельной системой контроля версий. Например, команды `git commit`, `git branch`, `git merge`, `git push` и `git pull` относятся к Git, а Merge Request, GitLab CI/CD, Issues и интерфейс управления репозиториями — это функциональность GitLab.

GitLab может использоваться как облачный сервис или быть установлен на собственных серверах компании.

---

## 🧭 Где я нахожусь

```text
07 DevOps
└── 05 Git и Workflow
    └── Git
        ├── Репозиторий Git
        ├── Remote Repository
        │   ├── GitHub
        │   └── GitLab ← Я здесь
        ├── Ветки
        ├── Merge
        ├── Rebase
        └── Git Workflow
```

---

# 📚 Разбор поглубже

## 1. Что такое GitLab

GitLab объединяет несколько частей процесса разработки:

```text
GitLab
├── Git Repository
├── Branches
├── Merge Requests
├── Code Review
├── Issues
├── CI/CD
├── Package Registry
├── Container Registry
└── Releases
```

Поэтому GitLab — это не просто место, куда разработчик загружает код.

Он может использоваться как единая платформа для:

```text
Код
 ↓
Code Review
 ↓
Tests
 ↓
Build
 ↓
Deploy
```

---

# 2. Git и GitLab — разные вещи

Это один из важных вопросов на собеседовании.

### Git

Git отвечает за контроль версий:

```text
git init
git add
git commit
git branch
git merge
git rebase
git fetch
git pull
git push
```

### GitLab

GitLab предоставляет инфраструктуру и инструменты вокруг Git:

```text
Remote Repository
Merge Request
Code Review
CI/CD
Issues
Registry
Permissions
Web UI
```

Можно представить так:

```text
Git
│
└── система контроля версий

GitLab
│
├── Git
├── Remote Repository
├── Merge Request
├── Code Review
├── CI/CD
└── другие инструменты
```

---

# 3. GitLab как Remote Repository

GitLab может выступать сервером для удалённого Git-репозитория.

Например:

```text
Локальный компьютер
       │
       │ git push
       ▼
GitLab
       │
       │ git fetch / pull
       ▼
Другой разработчик
```

Можно посмотреть remote:

```bash
git remote -v
```

Например:

```text
origin  git@gitlab.com:user/project.git (fetch)
origin  git@gitlab.com:user/project.git (push)
```

После этого обычные Git-команды работают так же:

```bash
git push origin main
git fetch origin
git pull origin main
```

---

# 4. Merge Request

**Merge Request (MR)** — механизм GitLab для предложения объединить изменения одной ветки с другой.

Например:

```text
main
  ↑
  │
Merge Request
  ↑
feature/auth
```

Разработчик:

```bash
git switch -c feature/auth
```

делает изменения:

```bash
git add .
git commit -m "Add authentication"
git push -u origin feature/auth
```

После этого в GitLab создаётся:

```text
feature/auth → main
```

Merge Request позволяет:

* посмотреть изменения;
* провести Code Review;
* оставить комментарии;
* запустить CI/CD;
* проверить тесты;
* после одобрения выполнить merge.

---

# 5. Merge Request ≠ Git Merge

Это важное различие.

### `git merge`

Команда Git:

```bash
git merge feature/auth
```

Она непосредственно объединяет истории Git.

### Merge Request

Функциональность GitLab:

```text
Developer
    ↓
Push branch
    ↓
GitLab
    ↓
Merge Request
    ↓
Code Review
    ↓
CI/CD
    ↓
Merge
```

То есть:

> **Merge — операция Git. Merge Request — процесс и интерфейс GitLab для организации этой операции.**

---

# 6. Code Review

В Merge Request разработчики могут посмотреть diff:

```text
- старый код
+ новый код
```

Коллеги могут:

* оставить комментарии;
* предложить изменения;
* запросить исправления;
* подтвердить изменения.

Например:

```text
feature/payment
       │
       ▼
Merge Request
       │
       ├── Code Review
       ├── Tests
       ├── Linter
       └── Security checks
       │
       ▼
     main
```

---

# 7. GitLab CI/CD

Одна из ключевых возможностей GitLab — **CI/CD**.

CI/CD позволяет автоматически выполнять действия после изменения кода.

Например:

```text
git push
   ↓
GitLab
   ↓
Pipeline
   ├── lint
   ├── tests
   ├── build
   └── deploy
```

Для настройки обычно используется файл:

```text
.gitlab-ci.yml
```

Например, упрощённо:

```yaml
stages:
  - test

tests:
  stage: test
  script:
    - pip install -r requirements.txt
    - pytest
```

После push GitLab CI может автоматически запустить этот pipeline.

---

# 8. Pipeline

**Pipeline** — последовательность автоматических этапов обработки изменений.

Например:

```text
Pipeline
│
├── lint
│
├── tests
│
├── build
│
└── deploy
```

Каждый этап может содержать один или несколько **jobs**.

Например:

```text
Pipeline
│
├── test
│   ├── unit-tests
│   └── integration-tests
│
├── build
│   └── docker-build
│
└── deploy
    └── deploy-production
```

---

# 9. Runner

**GitLab Runner** — агент, который фактически выполняет CI/CD jobs.

Упрощённо:

```text
GitLab
   │
   │ назначает job
   ▼
GitLab Runner
   │
   ├── запускает команды
   ├── выполняет тесты
   ├── собирает проект
   └── выполняет deploy
```

Например:

```yaml
tests:
  script:
    - pytest
```

GitLab хранит конфигурацию pipeline, а Runner выполняет команду `pytest`.

---

# 10. GitLab CI/CD и Docker

Для Backend-разработчика часто встречается такой pipeline:

```text
Developer
    │
    │ git push
    ▼
GitLab
    │
    ▼
GitLab CI
    │
    ├── pytest
    ├── flake8
    ├── Docker build
    └── Docker push
             │
             ▼
       Container Registry
```

Например, после успешного тестирования можно автоматически собрать Docker image.

---

# 11. Container Registry

GitLab может предоставлять **Container Registry** для хранения Docker images.

Например:

```text
GitLab
└── Container Registry
    └── my-project
        ├── backend:latest
        ├── backend:1.0.0
        └── backend:1.1.0
```

CI/CD pipeline может:

```text
Dockerfile
    ↓
docker build
    ↓
Docker image
    ↓
docker push
    ↓
GitLab Container Registry
```

---

# 12. GitLab и GitHub

GitLab и GitHub решают много похожих задач:

| Возможность         | GitLab        | GitHub         |
| ------------------- | ------------- | -------------- |
| Git-репозитории     | ✅             | ✅              |
| Remote Repository   | ✅             | ✅              |
| Branches            | ✅             | ✅              |
| Code Review         | Merge Request | Pull Request   |
| Issues              | ✅             | ✅              |
| CI/CD               | GitLab CI/CD  | GitHub Actions |
| Container Registry  | ✅             | ✅              |
| Self-hosted вариант | ✅             | ✅              |

Названия отличаются:

```text
GitLab → Merge Request
GitHub → Pull Request
```

Но концепция похожая: предложить изменения для интеграции в целевую ветку и провести проверку.

---

# 13. GitLab Self-Managed

GitLab можно использовать не только как облачный сервис.

Компания может установить GitLab на собственной инфраструктуре.

Например:

```text
Компания
│
├── GitLab Server
├── GitLab Runner
├── PostgreSQL
├── Container Registry
└── Internal Network
```

Это используется, когда компании важно самостоятельно контролировать инфраструктуру и данные.

---

# 14. Типичный GitLab Workflow

Для Backend-разработчика типичный процесс может выглядеть так:

```text
main
 │
 └── feature/auth
        │
        ├── код
        ├── git add
        ├── git commit
        └── git push
                │
                ▼
          GitLab
                │
                ▼
        Merge Request
                │
        ┌───────┴────────┐
        ▼                ▼
   Code Review        CI/CD
        │                │
        └───────┬────────┘
                ▼
             Merge
                │
                ▼
              main
```

---

# 15. GitLab vs Remote Repository

Важно не смешивать понятия.

**Remote Repository** — это непосредственно удалённый Git-репозиторий.

**GitLab** — платформа, которая может предоставлять этот репозиторий и множество дополнительных возможностей.

То есть:

```text
GitLab
└── Project
    ├── Git Repository
    ├── Merge Requests
    ├── Issues
    ├── CI/CD
    └── Registry
```

---

# 16. Частые вопросы на собеседовании

### Что такое GitLab?

Платформа для разработки, предоставляющая Git-репозитории, Merge Requests, Code Review, CI/CD и другие инструменты.

### GitLab и Git — одно и то же?

Нет.

Git — распределённая система контроля версий.

GitLab — платформа, использующая Git и предоставляющая дополнительные инструменты вокруг него.

### Что такое Merge Request?

Механизм GitLab для предложения объединить изменения одной ветки с другой с возможностью Code Review и автоматических проверок.

### Чем Merge Request отличается от `git merge`?

`git merge` — команда Git для объединения историй.

Merge Request — механизм GitLab для организации процесса проверки и последующего merge.

### Что такое GitLab CI/CD?

Система автоматизации GitLab, позволяющая запускать pipeline для тестирования, сборки и доставки приложения.

### Что такое GitLab Runner?

Агент, который выполняет jobs GitLab CI/CD.

### Где хранится конфигурация GitLab CI/CD?

Обычно в файле:

```text
.gitlab-ci.yml
```

### Можно ли установить GitLab на свой сервер?

Да. Существует вариант GitLab Self-Managed.

### Что такое Pipeline?

Набор автоматизированных этапов и jobs, которые выполняются в рамках CI/CD.

### GitLab может быть Remote Repository?

Да. GitLab может хранить удалённый Git-репозиторий, к которому локальный Git обращается через `push`, `fetch` и `pull`.
