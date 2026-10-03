# 🐙 GitHub

## 🎤 Короткий ответ

**GitHub** — это платформа для хранения Git-репозиториев и совместной разработки.

Git — это система контроля версий, а GitHub — сервис, который хранит Git-репозитории удалённо и предоставляет инструменты для командной работы: **Pull Request, Code Review, Issues, Actions, Releases, Projects** и другие.

GitHub не заменяет Git. Обычно Git используется локально, а GitHub — как удалённый сервер и платформа для совместной разработки.

---

## 🗣️ Ответ на собеседовании

> GitHub — это платформа для размещения Git-репозиториев и организации совместной разработки.
>
> Git работает локально: у разработчика есть рабочая директория, staging area и локальный репозиторий. GitHub позволяет хранить копию репозитория удалённо и синхронизировать её через `push`, `fetch` и `pull`.
>
> Кроме хранения кода, GitHub предоставляет Pull Request для обсуждения и проверки изменений, Code Review, Issues для задач и багов, GitHub Actions для CI/CD и Releases для публикации версий.
>
> При этом Git и GitHub — разные вещи: Git является системой контроля версий, а GitHub — платформой, построенной вокруг Git.

---

## 🧭 Где я нахожусь

```text
07 DevOps
└── 05 Git и Workflow
    ├── Git
    ├── Репозиторий Git
    ├── Ветки Git
    ├── Merge
    ├── Rebase
    ├── GitHub ← Я здесь
    │   ├── Remote Repository
    │   ├── Push / Fetch / Pull
    │   ├── Pull Request
    │   ├── Code Review
    │   ├── Issues
    │   ├── GitHub Actions
    │   └── Releases
    └── Git Workflow
```

---

# 📚 Разбор поглубже

## 1. Git и GitHub — это не одно и то же

Главное различие:

```text
Git
│
├── система контроля версий
├── работает локально
├── commit
├── branch
├── merge
├── rebase
└── история изменений


GitHub
│
├── платформа вокруг Git
├── удалённые репозитории
├── Pull Request
├── Code Review
├── Issues
├── Actions
└── совместная работа
```

Можно использовать Git **без GitHub**.

Например:

```bash
git init
git add .
git commit -m "Initial commit"
```

Репозиторий уже существует локально.

GitHub становится нужен, когда мы хотим:

* хранить код удалённо;
* работать с другими разработчиками;
* создавать Pull Request;
* проводить Code Review;
* запускать CI/CD;
* публиковать версии проекта.

---

# 2. Repository на GitHub

На GitHub можно создать удалённый репозиторий:

```text
GitHub
└── my-project
    ├── README.md
    ├── pyproject.toml
    ├── src/
    ├── tests/
    └── .gitignore
```

Локальный Git-репозиторий можно связать с ним:

```bash
git remote add origin https://github.com/user/my-project.git
```

Здесь:

```text
origin
```

— имя удалённого репозитория.

`origin` — стандартное, но не обязательное название.

---

# 3. Local Repository и Remote Repository

После подключения получается:

```text
┌──────────────────────┐
│    Local Repository  │
│       Git            │
└──────────┬───────────┘
           │
           │ push / fetch / pull
           │
           ▼
┌──────────────────────┐
│   Remote Repository  │
│      GitHub          │
└──────────────────────┘
```

Локальный репозиторий находится на компьютере.

Удалённый репозиторий находится на GitHub.

---

# 4. Push

`push` отправляет локальные коммиты в удалённый репозиторий.

Например:

```bash
git push origin main
```

Смысл:

```text
local main
    │
    │ push
    ▼
GitHub main
```

Типичный workflow:

```bash
git add .
git commit -m "Add authentication"
git push origin main
```

---

# 5. Fetch

`fetch` получает информацию и новые коммиты с удалённого репозитория, **не изменяя текущую рабочую ветку напрямую**.

```bash
git fetch origin
```

Например:

```text
GitHub
  │
  │ fetch
  ▼
origin/main
```

После `fetch` Git обновляет remote-tracking references, например:

```text
origin/main
origin/feature/login
```

Но твоя локальная `main` автоматически не перемещается.

---

# 6. Pull

`git pull` — это высокоуровневая операция, которая обычно состоит из:

```text
fetch
  +
merge
```

Упрощённо:

```bash
git pull
```

≈

```bash
git fetch
git merge
```

Однако Git может быть настроен на использование rebase:

```bash
git pull --rebase
```

Тогда логика будет ближе к:

```text
fetch
  +
rebase
```

Поэтому важно не считать `pull` буквально всегда равным только `fetch + merge` — поведение может зависеть от конфигурации.

---

# 7. Remote

Посмотреть подключённые remote:

```bash
git remote -v
```

Например:

```text
origin  https://github.com/user/project.git (fetch)
origin  https://github.com/user/project.git (push)
```

Здесь:

```text
origin
```

— имя remote.

```text
https://github.com/user/project.git
```

— URL репозитория.

---

# 8. Clone

Если проект уже находится на GitHub, его можно получить:

```bash
git clone https://github.com/user/project.git
```

Git создаст:

```text
project/
├── .git/
├── README.md
├── src/
└── tests/
```

При `clone` создаётся локальная копия Git-репозитория и обычно автоматически добавляется remote:

```text
origin
```

---

# 9. Типичный workflow разработчика

Представим, что проект находится на GitHub.

```text
GitHub
   │
   │ clone
   ▼
Local repository
   │
   ├── create branch
   │
   ├── write code
   │
   ├── git add
   │
   ├── git commit
   │
   └── git push
           │
           ▼
        GitHub
           │
           ▼
     Pull Request
           │
           ▼
      Code Review
           │
           ▼
         Merge
```

Команды:

```bash
git clone <repository-url>

cd project

git switch -c feature/login

# пишем код

git add .
git commit -m "Add login"

git push -u origin feature/login
```

После этого на GitHub можно создать Pull Request.

---

# 10. Pull Request

**Pull Request (PR)** — механизм GitHub для предложения изменений из одной ветки в другую.

Например:

```text
feature/login
      │
      │ Pull Request
      ▼
    main
```

PR позволяет:

* показать изменения;
* обсудить код;
* провести Code Review;
* запустить автоматические проверки;
* внести дополнительные коммиты;
* после одобрения объединить изменения.

Важно:

> **Pull Request — это не команда Git.**

Это функция GitHub и других платформ, например GitLab использует термин **Merge Request**.

---

# 11. Code Review

Code Review позволяет другим разработчикам проверить изменения до их попадания в целевую ветку.

Например:

```text
feature/login
      │
      ▼
Pull Request
      │
      ├── CI
      ├── Code Review
      ├── комментарии
      └── исправления
              │
              ▼
            Merge
```

Разработчик может получить комментарий:

```text
Please add a test for invalid credentials.
```

Он исправляет код:

```bash
git add .
git commit -m "Add invalid credentials test"
git push
```

Новый коммит автоматически появляется в том же Pull Request.

---

# 12. GitHub Actions

**GitHub Actions** — система автоматизации на GitHub.

Она используется для:

* CI/CD;
* запуска тестов;
* линтинга;
* сборки проекта;
* публикации Docker-образов;
* деплоя.

Например, после:

```bash
git push
```

GitHub Actions может автоматически выполнить:

```text
push
 │
 ▼
GitHub Actions
 │
 ├── install dependencies
 ├── run flake8
 ├── run pytest
 ├── build Docker image
 └── deploy
```

Конфигурация обычно находится в:

```text
.github/workflows/
```

Например:

```text
.github/
└── workflows/
    └── tests.yml
```

---

# 13. Issues

**GitHub Issues** используются для управления задачами, багами и предложениями.

Например:

```text
Issue #42
Title: Add password reset

Tasks:
- create endpoint
- send email
- add tests
```

Issue можно связать с Pull Request.

Получается цепочка:

```text
Issue
  ↓
Feature branch
  ↓
Pull Request
  ↓
Code Review
  ↓
Merge
```

---

# 14. Releases

**GitHub Releases** позволяют публиковать версии проекта.

Например:

```text
v1.0.0
v1.1.0
v1.2.0
```

Release обычно связывается с определённым Git tag:

```text
commit
  ↓
tag v1.2.0
  ↓
GitHub Release
```

Например:

```bash
git tag v1.0.0
git push origin v1.0.0
```

---

# 15. GitHub и GitHub Repository

Важно различать:

```text
GitHub
└── платформа

GitHub Repository
└── конкретный удалённый Git-репозиторий
```

Например:

```text
GitHub
├── repository A
├── repository B
└── repository C
```

---

# 16. GitHub и GitLab

На собеседовании иногда спрашивают различия.

Обе платформы предоставляют:

* Git-репозитории;
* удалённую работу с кодом;
* Pull/Merge Requests;
* Code Review;
* CI/CD;
* Issues;
* управление проектами.

Терминология различается:

| GitHub            | GitLab            |
| ----------------- | ----------------- |
| Pull Request      | Merge Request     |
| GitHub Actions    | GitLab CI/CD      |
| GitHub Issues     | Issues            |
| GitHub Repository | GitLab Repository |

При этом **Git остаётся отдельной системой контроля версий**.

---

# 17. GitHub не является самим Git

Это один из самых важных моментов для собеседования.

❌ Неправильно:

> Git — это GitHub.

✅ Правильно:

> Git — распределённая система контроля версий, а GitHub — платформа для размещения Git-репозиториев и совместной разработки.

Можно использовать:

```text
Git
├── GitHub
├── GitLab
├── Bitbucket
└── собственный сервер
```

---

# 18. GitHub в реальном Backend-проекте

Например, Python-проект:

```text
my-api/
├── .github/
│   └── workflows/
│       └── tests.yml
├── app/
│   ├── api/
│   ├── models/
│   ├── services/
│   └── repositories/
├── tests/
├── Dockerfile
├── docker-compose.yml
├── pyproject.toml
├── README.md
└── .gitignore
```

Workflow:

```text
Developer
    │
    ▼
feature/payment
    │
    │ push
    ▼
GitHub
    │
    ▼
Pull Request
    │
    ├── GitHub Actions
    │      ├── pytest
    │      ├── flake8
    │      └── build
    │
    ├── Code Review
    │
    ▼
Merge → main
```

Это уже типичный элемент CI/CD workflow.

---

# 19. Основные команды при работе с GitHub

| Команда                      | Назначение                                     |
| ---------------------------- | ---------------------------------------------- |
| `git clone URL`              | Клонировать репозиторий                        |
| `git remote -v`              | Посмотреть remote                              |
| `git remote add origin URL`  | Добавить remote                                |
| `git fetch origin`           | Получить изменения/ссылки с remote             |
| `git pull`                   | Получить изменения и интегрировать их          |
| `git push`                   | Отправить локальные коммиты                    |
| `git push -u origin feature` | Опубликовать новую ветку и установить upstream |
| `git branch -r`              | Посмотреть remote-tracking ветки               |
| `git branch -a`              | Посмотреть локальные и remote-tracking ветки   |

---

# 20. `origin` и `upstream` — не одно и то же

Это важный нюанс.

`origin` — обычно remote основного репозитория, с которым работает локальный clone.

Например:

```text
origin
   ↓
https://github.com/user/project.git
```

`upstream` часто используют в workflow с **fork**:

```text
Официальный репозиторий
        ↑
     upstream
        │
        │
      fork
        │
      origin
        │
        ▼
   твой GitHub
```

Например:

```bash
git fetch upstream
```

получает изменения из оригинального проекта.

---

# 21. GitHub Fork

**Fork** — копия репозитория другого пользователя или организации в твоём аккаунте GitHub.

Например:

```text
Original repository
        │
        │ fork
        ▼
Your repository
```

Затем можно:

```text
original
    ↑
    │ Pull Request
    │
your fork
    │
    ▼
feature branch
```

Это особенно распространено при работе с open-source проектами.

---

# 🎤 Вопросы на собеседовании

### Что такое GitHub?

> Платформа для размещения Git-репозиториев и совместной разработки. Она предоставляет Pull Requests, Code Review, Issues, Actions и другие инструменты.

### Чем Git отличается от GitHub?

> Git — система контроля версий. GitHub — платформа, использующая Git для хранения репозиториев и организации совместной работы.

### Что такое `origin`?

> Обычно это имя удалённого репозитория, который был добавлен при clone или вручную.

### Чем `push` отличается от `fetch`?

> `push` отправляет локальные коммиты в remote, а `fetch` получает информацию и новые коммиты из remote, не интегрируя их непосредственно в текущую ветку.

### Чем `fetch` отличается от `pull`?

> `fetch` только получает изменения. `pull` обычно выполняет fetch и затем интеграцию изменений через merge или, в зависимости от настройки, rebase.

### Что такое Pull Request?

> Механизм GitHub для предложения изменений из одной ветки в другую с возможностью Code Review, CI-проверок и последующего merge.

### Что такое GitHub Actions?

> Система автоматизации GitHub, используемая для CI/CD, тестирования, линтинга, сборки и деплоя.

### Что такое Fork?

> Копия репозитория в другом аккаунте GitHub, которая позволяет независимо работать с проектом и при необходимости предложить изменения через Pull Request.

### Что такое `origin` и `upstream`?

> Это просто имена remote. `origin` обычно указывает на репозиторий, с которым работает разработчик, а `upstream` часто используется для оригинального репозитория при работе через fork.

### Можно ли использовать Git без GitHub?

> Да. Git полностью работает локально. GitHub — только одна из платформ, предоставляющих удалённое хранение и дополнительные инструменты вокруг Git.
