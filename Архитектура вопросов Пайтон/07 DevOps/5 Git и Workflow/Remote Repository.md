# 🌐 Remote Repository в Git

## 🎤 Короткий ответ

**Remote Repository** — это удалённый Git-репозиторий, который хранится на другом сервере, например на [[GitHub]], [[GitLab]] или Bitbucket.

Он нужен для **обмена кодом между разработчиками, резервного хранения и совместной работы**.

Локальный репозиторий находится на моём компьютере, а remote repository — на удалённом сервере.

Основные команды для работы:

```bash
git remote -v
git fetch
git pull
git push
```

Например, `origin` — стандартное имя удалённого репозитория, которое Git обычно создаёт при `git clone`.

---

## 🗣️ Ответ на собеседовании

**Remote Repository** — это удалённый репозиторий Git, расположенный на сервере. Например, репозиторий проекта на GitHub.

Git позволяет работать с ним через команды `push`, `fetch` и `pull`.

При `git push` я отправляю свои локальные коммиты в удалённый репозиторий.

При `git fetch` получаю информацию о новых коммитах с сервера, но мои локальные ветки при этом автоматически не изменяются.

`git pull` обычно выполняет `fetch`, а затем интегрирует полученные изменения в текущую ветку через merge или rebase — в зависимости от настроек.

Удалённый репозиторий может иметь несколько имён. По умолчанию после `git clone` основной remote обычно называется `origin`.

---

## 🧭 Где я нахожусь

```text
07 DevOps
└── 05 Git и Workflow
    └── Git
        └── Репозиторий Git
            ├── Локальный репозиторий
            └── Remote Repository ← Я здесь
                ├── remote
                ├── origin
                ├── fetch
                ├── pull
                └── push
```

---

# 📚 Разбор поглубже

## 1. Что такое Remote Repository

Remote Repository — это **репозиторий Git, доступный через сеть**.

Например:

```text
GitHub
└── my-project.git
```

На компьютере разработчика:

```text
Мой компьютер
└── my-project/
    └── .git/
```

Получается:

```text
Локальный репозиторий  ←→  Remote Repository
       Git                  GitHub / GitLab
```

Важно:

> **Git не требует GitHub.**

Git — распределённая система контроля версий. GitHub — сервис, который предоставляет удалённое хранение Git-репозиториев и дополнительные возможности.

---

## 2. Зачем нужен Remote Repository

Основные задачи:

### Совместная работа

Несколько разработчиков могут работать с одним проектом:

```text
Developer A ──┐
              ├──→ Remote Repository
Developer B ──┤
              │
Developer C ──┘
```

### Резервное хранение

Если локальный компьютер сломался, история проекта может остаться на сервере.

### Обмен изменениями

Например:

```text
Developer A
    │
    │ git push
    ▼
Remote Repository
    │
    │ git fetch / pull
    ▼
Developer B
```

### Code Review

На GitHub/GitLab удалённый репозиторий используется как основа для Pull Request / Merge Request.

---

# 3. Remote в Git

Git хранит информацию о подключённых удалённых репозиториях.

Посмотреть их:

```bash
git remote
```

Например:

```text
origin
```

Получить URL:

```bash
git remote -v
```

Например:

```text
origin  git@github.com:user/project.git (fetch)
origin  git@github.com:user/project.git (push)
```

Здесь:

* `origin` — имя remote;
* `fetch` — URL для получения данных;
* `push` — URL для отправки данных.

---

# 4. Что такое `origin`

`origin` — **не специальное ключевое слово Git**.

Это просто стандартное имя remote, которое обычно получает репозиторий после:

```bash
git clone <url>
```

Например:

```bash
git clone git@github.com:user/project.git
```

Git автоматически создаст:

```text
origin → git@github.com:user/project.git
```

Можно назвать remote иначе:

```bash
git remote add github git@github.com:user/project.git
```

Тогда:

```text
github → git@github.com:user/project.git
```

То есть:

> `origin` — соглашение об именовании, а не обязательное имя.

---

# 5. Добавление Remote

Если локальный репозиторий уже существует:

```bash
git remote add origin git@github.com:user/project.git
```

Проверить:

```bash
git remote -v
```

Получим:

```text
origin  git@github.com:user/project.git (fetch)
origin  git@github.com:user/project.git (push)
```

---

# 6. `git push`

`push` отправляет локальные коммиты в remote repository.

Например:

```bash
git push origin main
```

Здесь:

```text
origin
  │
  └── remote repository

main
  │
  └── локальная ветка
```

Упрощённо:

```text
Local Repository
      │
      │ git push
      ▼
Remote Repository
```

Важно:

> `git push` отправляет **коммиты**, а не просто файлы.

Например, если у меня есть:

```text
A → B → C
```

и remote содержит:

```text
A → B
```

после:

```bash
git push
```

remote получит коммит `C`.

---

# 7. `git fetch`

`fetch` получает изменения из remote repository, но **не изменяет текущую рабочую ветку**.

```bash
git fetch origin
```

Например:

До:

```text
Local:

A → B → C
        ↑
       main
```

На сервере:

```text
A → B → C → D
            ↑
      origin/main
```

После:

```bash
git fetch
```

локальный Git узнает о `D`:

```text
A → B → C → D
        ↑    ↑
       main  origin/main
```

Но файлы рабочей директории при этом не переключаются на `D`.

---

# 8. `git pull`

`git pull` обычно состоит из двух этапов:

```text
git fetch
+
git merge
```

То есть:

```bash
git pull
```

обычно означает:

```bash
git fetch
git merge
```

Но поведение может быть настроено иначе, например через rebase:

```bash
git pull --rebase
```

Поэтому важно различать:

```text
fetch → получить изменения

pull → получить + интегрировать изменения
```

---

# 9. Remote-tracking branch

После работы с remote Git хранит ссылки на состояние удалённых веток.

Например:

```text
origin/main
```

Это **remote-tracking branch**.

Важно не путать:

```text
main
```

и

```text
origin/main
```

### `main`

Моя локальная ветка.

### `origin/main`

Моя локальная ссылка на состояние ветки `main` в remote `origin`.

Например:

```text
Local:
A → B → C
        ↑
       main

Remote:
A → B → C → D
            ↑
       origin/main
```

После:

```bash
git fetch
```

локальный Git узнаёт о `D`.

---

# 10. Upstream Branch

Локальная ветка может быть связана с удалённой веткой.

Например:

```text
main
  ↓
origin/main
```

Такая связь называется **upstream**.

Посмотреть:

```bash
git branch -vv
```

Например:

```text
* main  abc123 [origin/main] Add authentication
```

Теперь можно просто написать:

```bash
git pull
```

или:

```bash
git push
```

без явного указания:

```bash
origin main
```

---

# 11. Первый `push`

Если локальная ветка ещё не связана с remote:

```bash
git push -u origin main
```

Флаг:

```text
-u
```

устанавливает upstream для текущей ветки.

После этого обычно достаточно:

```bash
git push
```

и:

```bash
git pull
```

---

# 12. Несколько Remote

У репозитория может быть несколько remote.

Например:

```text
origin  → GitHub
upstream → основной проект
```

Проверить:

```bash
git remote -v
```

Например:

```text
origin    git@github.com:ilya/my-project.git
upstream  git@github.com:company/my-project.git
```

Это часто используется при работе с **fork**.

```text
Основной проект
      ↑
  upstream
      │
      │
    Fork
      │
    origin
      ↑
Локальный репозиторий
```

---

# 13. `origin` vs `upstream`

Типичный workflow при работе с fork:

```text
upstream
   │
   │ fetch
   ▼
origin
   │
   │ push
   ▼
мой GitHub
```

Например:

```bash
git fetch upstream
```

получает изменения из оригинального проекта.

А:

```bash
git push origin feature/auth
```

отправляет мою ветку в мой fork.

---

# 14. SSH и HTTPS

Remote URL может использовать разные протоколы.

### HTTPS

```text
https://github.com/user/project.git
```

### SSH

```text
git@github.com:user/project.git
```

SSH часто удобен для постоянной работы с GitHub/GitLab, потому что после настройки SSH-ключа не требуется постоянно вводить учётные данные.

---

# 15. Remote Repository ≠ GitHub

Это важное различие на собеседовании.

```text
Git
│
├── Local Repository
│
└── Remote Repository
       │
       ├── GitHub
       ├── GitLab
       ├── Bitbucket
       └── собственный Git-сервер
```

Git — система контроля версий.

GitHub — сервис вокруг Git.

Remote repository может находиться не только на GitHub.

---

# 16. Типичный workflow

Обычная работа разработчика:

```text
git clone
    ↓
git switch -c feature/auth
    ↓
изменение файлов
    ↓
git add
    ↓
git commit
    ↓
git push
    ↓
Pull Request
    ↓
Code Review
    ↓
Merge
```

После того как коллеги внесли изменения:

```bash
git fetch
```

или:

```bash
git pull
```

---

# 17. Частая ошибка: путать remote и remote-tracking branch

❌ Неправильно:

> `origin/main` — это ветка, которая непосредственно находится на GitHub.

Точнее:

> `origin/main` — это локальная remote-tracking reference, которая показывает состояние ветки `main` в remote на момент последнего получения информации через Git.

То есть:

```text
GitHub
  │
  │ fetch
  ▼
origin/main
```

`origin/main` обновляется локальным Git после получения данных из remote.

---

# 18. Основные команды

| Команда             | Назначение                          |
| ------------------- | ----------------------------------- |
| `git remote`        | Показать remote                     |
| `git remote -v`     | Показать URL remote                 |
| `git remote add`    | Добавить remote                     |
| `git remote remove` | Удалить remote                      |
| `git remote rename` | Переименовать remote                |
| `git fetch`         | Получить изменения без интеграции   |
| `git pull`          | Получить и интегрировать изменения  |
| `git push`          | Отправить локальные коммиты         |
| `git branch -r`     | Показать remote-tracking branches   |
| `git branch -vv`    | Показать upstream и состояние веток |

---

# 19. Главное различие `push / fetch / pull`

```text
                 Remote Repository
                  /            \
                 /              \
            fetch              push
               ↑                 ↓
               │                 │
        Local Repository ←───────┘
               │
              pull
               ↓
      Local Repository
      + интеграция изменений
```

Упрощённо:

```text
push  → отправить

fetch → получить информацию

pull  → получить + интегрировать
```

---

## 🎤 Вопросы на собеседовании

### Что такое Remote Repository?

Удалённый Git-репозиторий, доступный через сеть. Используется для хранения и обмена коммитами между разработчиками.

### Что такое `origin`?

Стандартное имя первого remote, обычно создаваемого при `git clone`. Это просто имя, его можно изменить.

### Чем `git fetch` отличается от `git pull`?

`fetch` получает изменения и обновляет remote-tracking references, но не интегрирует их в текущую ветку.

`pull` обычно выполняет `fetch` и затем `merge` либо `rebase`.

### Чем `git push` отличается от `git fetch`?

`push` отправляет локальные коммиты в remote.

`fetch` получает информацию о новых коммитах из remote.

### Что такое `origin/main`?

Remote-tracking reference, которая показывает состояние ветки `main` в remote `origin`, известное локальному Git после последнего `fetch`.

### Может ли быть несколько remote?

Да:

```text
origin
upstream
```

и другие.

### Remote Repository — это GitHub?

Нет.

GitHub — один из сервисов, предоставляющих удалённые Git-репозитории. Remote может находиться на GitHub, GitLab, Bitbucket или собственном Git-сервере.

### Что делает `git push -u origin main`?

Отправляет локальную ветку `main` в `origin` и устанавливает для неё upstream-связь с `origin/main`.
