# 🌿 Ветки Git

## 🎤 Короткий ответ

**Ветка (branch)** в Git — это именованная ссылка на коммит, которая позволяет вести отдельную линию разработки.

Ветка не является копией всего проекта. Упрощённо это указатель на конкретный commit, который перемещается вперёд при создании новых коммитов.

```text
A ── B ── C   main
      \
       D ── E feature
```

Основные команды:

```bash
git branch
git switch main
git switch -c feature
git merge feature
git branch -d feature
```

---

## 🗣️ Ответ на собеседовании

Ветка в Git — это именованная ссылка на коммит. Она позволяет вести отдельную линию разработки независимо от другой ветки.

Например, от `main` можно создать `feature`, сделать в ней несколько коммитов, а затем объединить изменения обратно через `merge` или другим способом, например через `rebase`.

Важный момент: branch — это не копия каталога проекта. Это лёгкая ссылка на commit. Когда создаётся новый commit в текущей ветке, указатель ветки перемещается на этот новый commit.

Обычно `main` используется как основная ветка, а для разработки отдельных задач создаются feature-ветки.

---

## 🧭 Где я нахожусь

```text
07 DevOps
└── 05 Git и Workflow
    ├── Git
    ├── Репозиторий Git
    ├── Ветки ← Я здесь
    │   ├── Создание и переключение
    │   ├── Локальные ветки
    │   ├── Remote branches
    │   ├── Merge
    │   └── Rebase
    ├── Remote Repository
    └── Git Workflow
```

---

## 📚 Разбор поглубже

### 1. Что такое branch

Ветка — это **именованный указатель на commit**.

Например:

```text
A ── B ── C
          ↑
         main
```

Здесь `main` указывает на `C`.

Создаём новую ветку:

```bash
git branch feature
```

Получаем:

```text
A ── B ── C
          ↑
       main
       feature
```

Обе ветки пока указывают на один и тот же commit.

---

### 2. Ветка не копирует проект

Это важная особенность Git.

Когда выполняем:

```bash
git branch feature
```

Git не делает:

```text
копия проекта №1
копия проекта №2
```

Ветка — это небольшая ссылка на commit.

Поэтому создание ветки — дешёвая операция.

---

### 3. Создание ветки

Создать ветку:

```bash
git branch feature
```

Посмотреть ветки:

```bash
git branch
```

Например:

```text
* main
  feature
```

`*` показывает текущую ветку.

---

### 4. Создать и сразу переключиться

Современный вариант:

```bash
git switch -c feature
```

Он объединяет две операции:

```bash
git branch feature
git switch feature
```

Старый распространённый вариант:

```bash
git checkout -b feature
```

Обе формы используются, но `git switch` лучше выражает именно операцию работы с ветками.

---

### 5. Переключение веток

```bash
git switch feature
```

После этого:

```text
HEAD
 ↓
feature
 ↓
C
```

Проверить текущую ветку:

```bash
git branch
```

или:

```bash
git status
```

---

### 6. Как ветка перемещается

Допустим:

```text
A ── B ── C
          ↑
         main
```

Создали:

```bash
git switch -c feature
```

Теперь:

```text
A ── B ── C
          ↑
      main, feature
```

Делаем commit:

```bash
git commit -m "Add authorization"
```

Получаем:

```text
A ── B ── C ── D
          ↑     ↑
         main  feature
```

`feature` переместилась с `C` на `D`.

---

### 7. `HEAD` и ветка

Обычно структура выглядит так:

```text
HEAD
 ↓
feature
 ↓
D
```

То есть `HEAD` указывает на текущую ветку, а ветка — на текущий commit.

Когда выполняем:

```bash
git switch main
```

получаем:

```text
HEAD
 ↓
main
 ↓
C
```

---

### 8. Типичный сценарий разработки

Есть основная ветка:

```text
main
```

Нужно реализовать новую функцию.

Создаём:

```bash
git switch main
git switch -c feature/auth
```

Работаем:

```text
feature/auth
```

Создаём коммиты:

```bash
git add .
git commit -m "Add authentication"
```

В результате:

```text
A ── B ── C ── D
          ↑     ↑
         main  feature/auth
```

После завершения работы изменения интегрируются в `main`.

---

## 9. Merge веток

Переключаемся на ветку, **в которую хотим добавить изменения**:

```bash
git switch main
```

Затем:

```bash
git merge feature/auth
```

Например:

```text
До:

A ── B ── C       main
      \
       D ── E     feature
```

После merge:

```text
A ── B ── C ── M   main
      \         /
       D ── E ────
```

`M` — merge commit.

---

### 10. Fast-forward merge

Если `main` не изменялся:

```text
A ── B ── C       main
      \
       D ── E     feature
```

Можно просто передвинуть `main`:

```text
A ── B ── C ── D ── E
                    ↑
              main, feature
```

Это **fast-forward**.

Отдельный merge commit не создаётся.

---

### 11. Удаление ветки

После объединения ветка часто больше не нужна:

```bash
git branch -d feature
```

`-d` — безопасное удаление, Git проверяет, что ветка уже объединена.

Принудительное удаление:

```bash
git branch -D feature
```

Использовать осторожно: можно удалить ветку с ещё не интегрированными коммитами.

---

## 12. Локальные и удалённые ветки

До сих пор мы говорили о локальных ветках:

```text
main
feature/auth
```

Но у Git есть и remote-tracking branches:

```text
origin/main
origin/feature/auth
```

Например:

```text
Local:
main

Remote-tracking:
origin/main
```

`origin/main` показывает состояние ветки `main`, которое Git последний раз получил с remote.

---

### 13. Remote branch

Отправить локальную ветку:

```bash
git push -u origin feature/auth
```

После этого на remote появляется ветка:

```text
origin/feature/auth
```

Опция:

```bash
-u
```

устанавливает upstream-связь.

После этого часто можно просто использовать:

```bash
git push
git pull
```

без явного указания remote и ветки.

---

### 14. Upstream branch

Допустим:

```text
local:
feature/auth

upstream:
origin/feature/auth
```

Git запоминает соответствие.

Можно посмотреть:

```bash
git branch -vv
```

Например:

```text
* feature/auth  abc123 [origin/feature/auth] Add auth
```

Это показывает, с какой удалённой веткой связана локальная.

---

### 15. `git fetch` и ветки

Предположим:

```text
Local:
main → A

Remote:
main → A → B
```

Выполняем:

```bash
git fetch origin
```

Теперь:

```text
main        → A
origin/main → B
```

Локальная `main` сама не переместилась.

Чтобы интегрировать изменения:

```bash
git merge origin/main
```

или:

```bash
git rebase origin/main
```

в зависимости от выбранного workflow.

---

### 16. Merge vs Rebase для веток

Есть:

```text
main:
A ── B ── C

feature:
      └── D ── E
```

### Merge

```text
A ── B ── C ── M
      \       /
       D ── E
```

История сохраняет факт объединения веток.

### Rebase

```text
A ── B ── C ── D' ── E'
```

Коммиты feature переносятся поверх новой базы.

Ключевой момент:

> `rebase` переписывает коммиты, поэтому его нужно осторожно использовать для уже опубликованных веток, которыми работают другие разработчики.

---

## 17. Конфликт при merge

Конфликт возникает, когда Git не может автоматически определить, как объединить изменения.

Например, две ветки изменили одну и ту же часть файла.

Git может оставить:

```text
<<<<<<< HEAD
print("Hello")
=======
print("Hi")
>>>>>>> feature
```

Разработчик должен выбрать правильный вариант:

```python
print("Hello")
```

После исправления:

```bash
git add app.py
git commit
```

Для merge-конфликта commit завершает merge.

---

## 18. `git merge --abort`

Если во время merge возник конфликт и нужно отказаться от операции:

```bash
git merge --abort
```

Git попытается вернуть состояние до начала merge.

Это полезно, если нужно сначала разобраться в изменениях или выбрать другой способ интеграции.

---

## 19. Типичная схема веток

В реальном проекте могут использоваться:

```text
main
 │
 ├── feature/auth
 ├── feature/payments
 ├── feature/orders
 └── bugfix/login
```

Например:

```text
main
 │
 ├── feature/auth
 │
 ├── feature/orders
 │
 └── bugfix/payment
```

Каждая задача развивается независимо.

После code review изменения интегрируются в основную ветку.

---

## 20. Ветки и Pull Request

В Git нет самого понятия Pull Request — это функция платформ вроде GitHub/GitLab.

Обычно процесс выглядит:

```text
main
  │
  └── feature/auth
          │
          ├── commit
          ├── commit
          └── commit
                │
                ▼
        Push to remote
                │
                ▼
        Pull / Merge Request
                │
                ▼
             Review
                │
                ▼
          Merge into main
```

То есть Git предоставляет механизм веток и истории, а GitHub/GitLab добавляют интерфейс для code review и Pull/Merge Requests.

---

## 21. Именование веток

Часто используют понятные имена:

```text
feature/auth
feature/user-profile
bugfix/login-error
hotfix/payment
refactor/user-service
```

Конкретный формат зависит от соглашений команды.

Хорошее имя должно показывать назначение ветки.

---

## 22. Не стоит создавать commit прямо в чужой ветке

В командной работе обычно не работают непосредственно в:

```text
main
```

для каждой задачи.

Чаще:

```text
main
  ↓
feature/auth
  ↓
работа
  ↓
review
  ↓
merge
```

Это позволяет изолировать изменения и провести code review до интеграции.

---

## 23. Ветка не является копией remote

Важно различать:

```text
main
origin/main
```

Это не одна и та же сущность.

```text
main
```

— локальная ветка.

```text
origin/main
```

— remote-tracking reference, показывающая известное локальному Git состояние удалённой ветки `main`.

После `git fetch` она может обновиться, а локальная `main` — нет.

---

## 24. Полезные команды

### Посмотреть ветки

```bash
git branch
```

### Посмотреть локальные и remote-tracking

```bash
git branch -a
```

### Создать ветку

```bash
git branch feature
```

### Создать и переключиться

```bash
git switch -c feature
```

### Переключиться

```bash
git switch feature
```

### Удалить локальную ветку

```bash
git branch -d feature
```

### Принудительно удалить

```bash
git branch -D feature
```

### Объединить ветку

```bash
git merge feature
```

### Посмотреть связь с remote

```bash
git branch -vv
```

### Отправить ветку

```bash
git push -u origin feature
```

---

## 25. Главная модель веток

Запомнить можно через такую схему:

```text
                    feature
                       ↓
A ── B ── C ───────── D ── E
      ↑                ↑
     main             HEAD
```

* `main` — ветка;
* `feature` — другая ветка;
* каждая ветка указывает на commit;
* `HEAD` показывает текущую позицию;
* новые commits перемещают текущую ветку вперёд.

---

## 🎤 Вопросы на собеседовании

**Что такое branch в Git?**

Именованная ссылка на commit, позволяющая вести отдельную линию разработки.

**Является ли branch копией проекта?**

Нет. Ветка — лёгкая ссылка на commit, а не полная копия рабочей директории.

**Как создать ветку?**

```bash
git branch feature
```

или сразу создать и переключиться:

```bash
git switch -c feature
```

**Как переключиться на другую ветку?**

```bash
git switch feature
```

**Что такое `HEAD`?**

Ссылка на текущую позицию Git, обычно текущую ветку и её commit.

**Что происходит с веткой после нового commit?**

Указатель ветки перемещается на новый commit.

**Что такое merge?**

Объединение истории одной ветки с другой.

**Что такое fast-forward?**

Случай, когда Git может просто передвинуть указатель ветки вперёд без создания merge commit.

**Чем merge отличается от rebase?**

`merge` объединяет истории, а `rebase` переносит коммиты на новую базу и переписывает их идентификаторы.

**Что такое `origin/main`?**

Remote-tracking reference, отражающая известное локальному Git состояние ветки `main` на remote `origin`.

**Что делает `git push -u origin feature`?**

Отправляет локальную ветку `feature` на remote `origin` и устанавливает upstream-связь с удалённой веткой.

**Что делать при конфликте merge?**

Исправить конфликтующие файлы, выполнить `git add` для разрешённых файлов и завершить merge commit.

**Формула для собеседования:**

> **Branch — это именованный указатель на commit, который позволяет вести отдельную линию разработки. Ветка не является копией проекта. При создании новых commit указатель текущей ветки перемещается вперёд. Ветки можно объединять через merge или перестраивать через rebase; локальные ветки можно синхронизировать с remote через push и fetch/pull.**
