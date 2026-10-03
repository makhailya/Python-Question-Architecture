# 🔀 Merge в Git

## 🎤 Короткий ответ

**Merge** — это операция Git, которая объединяет изменения из одной ветки в другую.

Например, если я нахожусь в `main` и выполняю:

```bash
git merge feature/login
```

Git пытается добавить изменения из `feature/login` в текущую ветку `main`.

В зависимости от истории Git может сделать **fast-forward merge** или создать отдельный **merge commit**. Если изменения конфликтуют, возникает **merge conflict**, который нужно разрешить вручную.

---

## 🗣️ Ответ на собеседовании

> Merge в Git используется для объединения истории двух веток. Обычно я сначала переключаюсь на ветку, в которую хочу внести изменения, например `main`, и выполняю `git merge feature/login`.
>
> Если `main` не содержит новых коммитов после создания `feature/login`, Git может выполнить fast-forward и просто переместить указатель `main` вперёд. Если обе ветки развивались независимо, Git обычно создаёт отдельный merge commit с двумя родителями.
>
> Если Git не может автоматически объединить изменения, возникает конфликт. Тогда я исправляю конфликтующие участки файлов, добавляю исправленные файлы через `git add` и завершаю merge коммитом.
>
> В отличие от rebase, merge сохраняет факт существования двух параллельных линий разработки и не переписывает существующую историю коммитов.

---

## 🧭 Где я нахожусь

```text
07 DevOps
└── 05 Git и Workflow
    ├── Git
    ├── Репозиторий Git
    ├── Ветки Git
    ├── Merge ← Я здесь
    ├── Rebase
    ├── Remote
    └── Pull Request / Merge Request
```

Внутри самого Git:

```text
Git
└── Branch
    └── Несколько веток
        └── Merge
            ├── Fast-forward
            ├── Merge commit
            └── Merge conflict
```

---

## 📚 Разбор поглубже

### 1. Что такое Merge

`merge` объединяет историю текущей ветки с указанной веткой.

Общий синтаксис:

```bash
git merge <branch>
```

Например:

```bash
git switch main
git merge feature/login
```

Здесь важно понимать направление:

```text
feature/login ──►
                \
main ────────────► merge
```

Мы **находимся в `main`** и вливаем в неё изменения из `feature/login`.

То есть:

```bash
git merge feature/login
```

не означает «переключиться на feature/login».

Он означает:

> «Возьми изменения из `feature/login` и объедини их с текущей веткой».

---

# 2. Простой пример

Изначально:

```text
A---B---C  main
     \
      D---E  feature
```

Переключаемся на `main`:

```bash
git switch main
```

Затем:

```bash
git merge feature
```

Если обе ветки имеют независимые изменения, Git создаст новый коммит:

```text
A---B---C-------M  main
     \         /
      D---E---    feature
```

`M` — **merge commit**.

У него два родителя:

```text
        M
       / \
      C   E
```

То есть merge commit объединяет две линии истории.

---

# 3. Fast-forward merge

Иногда отдельный merge commit не нужен.

Например:

```text
A---B  main
     \
      C---D  feature
```

`main` не двигалась после создания `feature`.

Если выполнить:

```bash
git switch main
git merge feature
```

Git может просто переместить указатель `main`:

```text
A---B---C---D  main, feature
```

Это называется **fast-forward**.

Git буквально говорит:

> «История `main` уже является предком `feature`, поэтому достаточно передвинуть указатель».

При этом нового коммита не создаётся.

---

# 4. Merge commit

Теперь другой сценарий:

```text
A---B---C  main
     \
      D---E  feature
```

Обе ветки после разделения получили новые коммиты.

Выполняем:

```bash
git switch main
git merge feature
```

Получаем:

```text
A---B---C-------M  main
     \         /
      D---E---    feature
```

`M` — merge commit.

Проверить историю удобно так:

```bash
git log --oneline --graph --all
```

Например:

```text
*   91abc12 Merge branch 'feature/login'
|\
| * 73def45 Add login endpoint
| * 62abc11 Add authentication
* | 45abc99 Update API
|/
* 1234567 Initial commit
```

---

# 5. Что происходит при Merge

Упрощённо Git сравнивает три состояния:

```text
             merge base
                │
          ┌─────┴─────┐
          ▼           ▼
       current      other
```

Например:

```text
A---B---C  main
     \
      D---E  feature
```

Общая база:

```text
B
```

Git смотрит:

* что изменилось в `main` относительно `B`;
* что изменилось в `feature` относительно `B`;
* можно ли объединить эти изменения автоматически.

Если да — merge выполняется.

Если нет — возникает конфликт.

---

# 6. Merge Conflict

Конфликт возникает, когда Git не может однозначно объединить изменения.

Например, в `main`:

```python
timeout = 10
```

А в `feature`:

```python
timeout = 30
```

Git не может самостоятельно определить, какое значение должно остаться.

После:

```bash
git merge feature
```

может появиться:

```text
<<<<<<< HEAD
timeout = 10
=======
timeout = 30
>>>>>>> feature
```

Здесь:

```text
<<<<<<< HEAD
```

начало текущей версии.

```text
=======
```

разделитель.

```text
>>>>>>> feature
```

версия из вливаемой ветки.

Разработчик должен выбрать правильный вариант.

Например:

```python
timeout = 30
```

После исправления:

```bash
git add .
```

И затем:

```bash
git commit
```

Merge завершается.

---

# 7. Как выглядит процесс Merge

Типичный workflow:

```bash
git switch main
```

Обновляем локальную информацию:

```bash
git fetch origin
```

При необходимости обновляем `main`:

```bash
git pull
```

Вливаем feature:

```bash
git merge feature/login
```

Если конфликтов нет — готово.

Если есть конфликт:

```bash
# исправляем файлы

git add <исправленные-файлы>
git commit
```

---

# 8. Как отменить Merge

Если merge ещё находится в процессе и возникли конфликты:

```bash
git merge --abort
```

Git постарается вернуть рабочее состояние до начала merge.

Например:

```bash
git merge feature
# возникли конфликты

git merge --abort
```

После этого можно изменить подход и попробовать снова.

---

# 9. `git merge --no-ff`

Git можно попросить **всегда создавать merge commit**, даже если возможен fast-forward:

```bash
git merge --no-ff feature
```

Было:

```text
A---B  main
     \
      C---D  feature
```

Обычный merge:

```text
A---B---C---D  main
```

С `--no-ff`:

```text
A---B-------M  main
     \     /
      C---D
```

Это позволяет явно сохранить в истории факт объединения feature-ветки.

---

# 10. Merge vs Rebase

Это один из важных вопросов на собеседовании.

### Merge

```text
A---B---C-------M
     \         /
      D---E---
```

Merge:

* сохраняет историю ветвления;
* может создавать merge commit;
* не переписывает существующие коммиты;
* хорошо показывает факт объединения веток.

### Rebase

```text
A---B---C---D'---E'
```

Rebase:

* переносит коммиты на новую базу;
* создаёт новые версии коммитов;
* делает историю линейной;
* переписывает историю переносимых коммитов.

Ключевое различие:

> **Merge объединяет две линии истории, а rebase перестраивает историю одной линии относительно другой базы.**

---

# 11. Merge не удаляет исходную ветку

После:

```bash
git merge feature
```

ветка `feature` сама по себе не удаляется.

Например:

```text
A---B---C---M  main
     \       /
      D---E  feature
```

Обе ссылки продолжают существовать.

Если feature больше не нужна:

```bash
git branch -d feature
```

Удаляется именно **ссылка на ветку**, а не коммиты, которые уже доступны из `main`.

---

# 12. Merge и удалённый репозиторий

Важно разделять Git и GitHub/GitLab.

Merge выполняется самим Git:

```bash
git merge feature
```

А GitHub/GitLab предоставляют интерфейс **Pull Request / Merge Request**, через который merge часто выполняется командой проекта.

Типичный workflow:

```text
Локальная feature
       │
       ▼
     push
       │
       ▼
GitHub/GitLab
       │
       ▼
Pull Request / Merge Request
       │
       ▼
Review + CI
       │
       ▼
Merge
       │
       ▼
main
```

Сам **Merge — это операция Git**, а Pull Request / Merge Request — механизм платформы вокруг этой операции.

---

# 13. Важные команды

| Команда                           | Назначение                       |
| --------------------------------- | -------------------------------- |
| `git merge feature`               | Влить `feature` в текущую ветку  |
| `git merge --no-ff feature`       | Создать merge commit даже при FF |
| `git merge --abort`               | Отменить незавершённый merge     |
| `git status`                      | Посмотреть состояние merge       |
| `git log --graph --oneline --all` | Посмотреть граф истории          |
| `git branch -d feature`           | Удалить слитую ветку             |

---

# 14. Типичная ошибка

❌ Неправильно понимать:

```bash
git merge feature
```

как:

> «Я объединяю текущую ветку с feature, находясь где угодно».

На самом деле направление определяется **текущей веткой**.

Например:

```bash
git switch main
git merge feature
```

означает:

```text
feature
   │
   ▼
 main
```

А если сделать:

```bash
git switch feature
git merge main
```

то направление будет уже другим:

```text
main
  │
  ▼
feature
```

Поэтому перед merge полезно проверить:

```bash
git branch --show-current
```

---

# 15. Главное различие Merge и Fast-forward

Не каждый merge создаёт merge commit.

### Fast-forward

```text
A---B---C  main
         ↑
       feature
```

После:

```text
A---B---C  main, feature
```

Нового коммита нет.

### Настоящее трёхстороннее объединение

```text
A---B---C-------M  main
     \         /
      D---E---
```

Создаётся `M`.

Поэтому фраза:

> «Merge всегда создаёт новый коммит»

❌ неверна.

Точнее:

> **При merge Git либо может выполнить fast-forward без нового коммита, либо создать merge commit, если истории разошлись.**
