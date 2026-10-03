# 📥 Staging Area в Git

## 🎤 Короткий ответ

**Staging Area** — это промежуточная область Git, куда я помещаю изменения, которые хочу включить в следующий commit.

После изменения файла он находится в **Working Directory**. Команда:

```bash
git add app.py
```

перемещает текущее состояние изменений файла в **Staging Area**.

Затем:

```bash
git commit -m "Add authentication"
```

создаёт commit на основе того, что находится в staging.

Главная идея:

```text
Working Directory
       ↓ git add
Staging Area
       ↓ git commit
Repository
```

---

## 🗣️ Ответ на собеседовании

> Staging Area — это промежуточная область между рабочей директорией и Git-репозиторием. Она позволяет заранее выбрать, какие именно изменения должны попасть в следующий commit.
>
> Например, я изменил три файла, но хочу закоммитить только два. Я могу выполнить `git add` только для этих двух файлов, а затем `git commit`. Третий файл останется изменённым, но в текущий commit не попадёт.
>
> Поэтому staging нужен не просто как технический промежуточный этап, а как механизм формирования содержимого следующего commit.
>
> Основная последовательность выглядит так: **изменил файл → `git add` → staging → `git commit` → repository**.

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
    ├── GitHub
    ├── Конфликты
    ├── Stash
    ├── Staging Area ← Я здесь
    │   ├── Working Directory
    │   ├── git add
    │   ├── git restore --staged
    │   └── git commit
    └── Pull Request / Code Review
```

Основной workflow Git:

```text
Working Directory
        │
        │ git add
        ▼
  Staging Area
        │
        │ git commit
        ▼
   Repository
        │
        │ git push
        ▼
     GitHub
```

---

# 📚 Разбор поглубже

## 1. Что такое Staging Area

Staging Area также называют:

* **index**;
* **staged area**;
* **область подготовки изменений**.

Это место, где Git хранит информацию о том, **какое состояние файлов должно попасть в следующий commit**.

Упрощённо:

```text
Working Directory
    ↓
  изменения
    ↓
Staging Area
    ↓
  commit
    ↓
Repository
```

---

# 2. Зачем вообще нужен Staging

Главная причина — возможность **сформировать commit выборочно**.

Допустим, ты изменил:

```text
app.py
models.py
tests.py
README.md
```

Но логически изменения относятся к двум разным задачам:

```text
Задача 1:
app.py
models.py

Задача 2:
tests.py
README.md
```

Можно сделать первый commit только с:

```bash
git add app.py models.py
git commit -m "Update models"
```

А остальные изменения оставить:

```text
tests.py
README.md
```

И позже сделать второй commit.

---

# 3. Working Directory → Staging Area

Предположим, есть:

```python
# app.py
timeout = 30
```

Ты изменил файл:

```python
# app.py
timeout = 60
```

Git видит:

```text
Working Directory
└── app.py → modified
```

Проверить:

```bash
git status
```

Можно получить:

```text
Changes not staged for commit:
  modified: app.py
```

Теперь:

```bash
git add app.py
```

Изменение попадает в staging.

```text
Changes to be committed:
  modified: app.py
```

Теперь Git сообщает:

> Это изменение будет включено в следующий commit.

---

# 4. `git add` не делает commit

Очень частая ошибка новичков:

```text
git add = сохранить commit
```

❌ Нет.

`git add` только добавляет текущее состояние изменений в staging.

```text
git add
   ↓
Staging Area
```

А commit создаётся отдельной командой:

```bash
git commit -m "Update timeout"
```

То есть:

```text
git add
   ↓
подготовить изменения

git commit
   ↓
зафиксировать подготовленные изменения
```

---

# 5. Staging позволяет выбрать часть файлов

Например:

```text
app.py       modified
models.py    modified
tests.py     modified
```

Выполняем:

```bash
git add app.py
git add models.py
```

Теперь:

```text
Staged:
    app.py
    models.py

Not staged:
    tests.py
```

После:

```bash
git commit -m "Update application models"
```

commit содержит только:

```text
app.py
models.py
```

`tests.py` остаётся изменённым в Working Directory.

---

# 6. Можно добавить всё сразу

```bash
git add .
```

Это добавляет изменения из текущего рабочего дерева в staging.

Также часто используют:

```bash
git add -A
```

`-A` означает добавление всех изменений, включая изменения, удаления и новые файлы в области действия команды.

Для собеседования важно понимать принцип:

> `git add` подготавливает изменения, а не создаёт commit.

---

# 7. `git status`

Одна из самых важных команд:

```bash
git status
```

Она показывает состояние между:

```text
Working Directory
        ↕
Staging Area
        ↕
Repository
```

Например:

```text
Changes to be committed:
    modified: app.py

Changes not staged for commit:
    modified: models.py

Untracked files:
    tests/test_auth.py
```

Это означает:

```text
Staging:
    app.py

Working Directory:
    models.py

Untracked:
    tests/test_auth.py
```

---

# 8. Staged и Unstaged

Эти термины важно различать.

### Staged

Изменение уже добавлено в staging:

```text
git add
    ↓
staged
```

Оно потенциально попадёт в следующий commit.

### Unstaged

Файл изменён, но эти изменения ещё не добавлены в staging:

```text
изменение
    ↓
unstaged
```

Например:

```text
app.py       staged
models.py    unstaged
```

---

# 9. Один файл может быть одновременно staged и unstaged

Это важный нюанс.

Допустим, было:

```python
timeout = 30
```

Ты изменил:

```python
timeout = 60
```

и сделал:

```bash
git add app.py
```

Теперь изменение `60` находится в staging.

После этого ты снова изменил файл:

```python
timeout = 120
```

Получается:

```text
HEAD:
timeout = 30

Staging:
timeout = 60

Working Directory:
timeout = 120
```

То есть один и тот же файл имеет **три состояния**.

Это очень важная концепция Git.

---

# 10. Три состояния одного файла

```text
             app.py
               │
       ┌───────┼────────┐
       ▼       ▼        ▼
      HEAD   Staging   Working
       │       │        │
       30      60       120
```

Если выполнить:

```bash
git commit
```

в commit попадёт:

```python
timeout = 60
```

А изменение:

```python
timeout = 120
```

останется в Working Directory.

---

# 11. Как убрать файл из Staging

Допустим:

```bash
git add app.py
```

Но потом понял:

> Я пока не хочу включать этот файл в commit.

Можно выполнить:

```bash
git restore --staged app.py
```

Файл выйдет из staging.

При этом **само изменение не удаляется**.

Было:

```text
Working Directory
    app.py modified
```

После `git add`:

```text
Staging
    app.py modified
```

После:

```bash
git restore --staged app.py
```

получаем:

```text
Working Directory
    app.py modified
```

Изменения остаются.

---

# 12. `restore --staged` не удаляет изменения

Это важное различие.

```bash
git restore --staged app.py
```

означает:

> Убрать изменения из staging.

А не:

> Удалить мои изменения.

После команды файл всё ещё изменён.

---

# 13. `git reset HEAD <file>`

Старый способ убрать файл из staging:

```bash
git reset HEAD app.py
```

Сегодня для этого чаще используют:

```bash
git restore --staged app.py
```

Обе команды могут использоваться для unstaging, но `restore --staged` лучше непосредственно выражает современную модель операции.

---

# 14. Как посмотреть staged changes

Чтобы увидеть изменения, которые **уже попадут в commit**:

```bash
git diff --staged
```

Также работает:

```bash
git diff --cached
```

Они показывают:

```text
HEAD
 ↓
Staging
```

То есть разницу между последним commit и тем, что подготовлено к следующему commit.

---

# 15. `git diff` и `git diff --staged`

Это часто спрашивают на собеседовании.

### `git diff`

Показывает изменения:

```text
Working Directory
        ↓
    Staging Area
```

То есть **unstaged changes**.

```bash
git diff
```

### `git diff --staged`

Показывает:

```text
HEAD
 ↓
Staging Area
```

То есть **staged changes**.

```bash
git diff --staged
```

Схема:

```text
HEAD
 │
 │ git diff --staged
 ▼
Staging
 │
 │ git diff
 ▼
Working Directory
```

---

# 16. Добавление части изменений

Иногда файл содержит несколько изменений, но в commit нужно добавить только часть.

Можно использовать:

```bash
git add -p
```

Git будет показывать отдельные блоки изменений — **hunks** — и спрашивать, добавлять ли их в staging.

Например:

```text
Stage this hunk [y,n,q,a,d,s,e,?]?
```

Это позволяет сформировать аккуратный commit даже тогда, когда несколько изменений находятся в одном файле.

---

# 17. Staging и качество commit

Staging помогает делать commits логически целостными.

Плохой commit:

```text
Fix login + refactor database + update README + formatting
```

Хорошее разделение:

```text
Add login validation
Fix database query
Update README
```

Для этого staging используется как фильтр:

```text
Все изменения
     │
     ▼
Staging
     │
     ├── изменение A
     ├── изменение B
     └── изменение C
            │
            ▼
         Commit
```

---

# 18. Staging и Commit

Важно понимать их различие.

### Staging

Отвечает на вопрос:

> **Что я хочу включить в следующий commit?**

### Commit

Отвечает на вопрос:

> **Какую версию проекта я зафиксировал в истории?**

Поэтому:

```text
git add
```

— формирует содержимое следующего commit.

```text
git commit
```

— создаёт объект commit в истории Git.

---

# 19. Staging и GitHub

GitHub здесь появляется только после commit.

Типичный путь:

```text
Working Directory
       │
       │ git add
       ▼
Staging Area
       │
       │ git commit
       ▼
Local Repository
       │
       │ git push
       ▼
GitHub
```

То есть нельзя сделать:

```text
Working Directory
       ↓
GitHub
```

через обычный `git push`.

`push` отправляет **commit'ы**, а не просто содержимое staging.

---

# 20. Staging и Stash

Эти механизмы тоже важно не путать.

### Staging

Подготовка изменений к commit:

```text
Working Directory
       ↓
   git add
       ↓
Staging
       ↓
   git commit
```

### Stash

Временное сохранение незакоммиченных изменений:

```text
Working Directory
       ↓
   git stash
       ↓
Stash
```

То есть:

```text
Staging → я готовлю изменения к commit

Stash   → я временно убираю незавершённые изменения
```

---

# 21. Что происходит при `git commit`

Допустим:

```text
HEAD:
A

Working Directory:
B

Staging:
B
```

После:

```bash
git commit -m "Update code"
```

получаем:

```text
A---B
    ↑
   HEAD
```

Staging очищается относительно нового commit.

Упрощённо:

```text
Working Directory
       │
       ▼
Staging
       │
       ▼
Commit
       │
       ▼
HEAD
```

---

# 22. Важная модель Git

Для собеседования полезно держать в голове четыре состояния:

```text
┌───────────────────┐
│ Working Directory │
└─────────┬─────────┘
          │ git add
          ▼
┌───────────────────┐
│   Staging Area     │
└─────────┬─────────┘
          │ git commit
          ▼
┌───────────────────┐
│ Local Repository   │
└─────────┬─────────┘
          │ git push
          ▼
┌───────────────────┐
│ Remote Repository  │
│      GitHub        │
└───────────────────┘
```

Именно здесь находится роль Staging Area:

> **Staging Area — это граница между изменениями, которые просто существуют локально, и изменениями, которые разработчик решил включить в следующий commit.**

---

# 23. Основные команды

| Команда                        | Что делает                                  |
| ------------------------------ | ------------------------------------------- |
| `git add file.py`              | Добавляет файл в staging                    |
| `git add .`                    | Добавляет изменения в staging               |
| `git add -A`                   | Добавляет все изменения                     |
| `git add -p`                   | Выборочно добавляет hunks                   |
| `git status`                   | Показывает состояние working tree и staging |
| `git diff`                     | Показывает unstaged changes                 |
| `git diff --staged`            | Показывает staged changes                   |
| `git restore --staged file.py` | Убирает файл из staging                     |
| `git commit`                   | Создаёт commit из staged changes            |

---

## 🎤 Вопросы на собеседовании

### Что такое Staging Area?

> Промежуточная область Git, в которой формируется содержимое следующего commit.

### Зачем нужен staging?

> Чтобы выборочно определить, какие изменения попадут в следующий commit.

### Что делает `git add`?

> Добавляет текущее состояние изменений файла или части изменений в Staging Area.

### `git add` создаёт commit?

> Нет. `git add` только подготавливает изменения. Commit создаётся командой `git commit`.

### Как убрать файл из staging?

```bash
git restore --staged file.py
```

### Удалятся ли изменения после `git restore --staged`?

> Нет. Они останутся в Working Directory, просто перестанут быть staged.

### Чем отличаются `git diff` и `git diff --staged`?

> `git diff` показывает изменения между Working Directory и Staging Area, а `git diff --staged` — изменения между последним commit и Staging Area.

### Может ли один файл одновременно иметь staged и unstaged изменения?

> Да. Например, сначала сделать `git add`, а затем снова изменить тот же файл. В staging останется первая версия изменений, а новые изменения будут только в Working Directory.

### Чем Staging отличается от Stash?

> Staging используется для подготовки изменений к commit, а Stash — для временного сохранения незакоммиченной работы.

### Попадает ли Staging напрямую на GitHub?

> Нет. Сначала staged changes становятся commit через `git commit`, и уже commit'ы можно отправить на GitHub через `git push`.
