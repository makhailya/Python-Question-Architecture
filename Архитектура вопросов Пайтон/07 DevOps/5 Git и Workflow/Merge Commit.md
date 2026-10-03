# 🔀 Merge Commit в Git

## 🎤 Короткий ответ

**Merge Commit** — это специальный commit, который Git создаёт при объединении двух веток, если их история разошлась и fast-forward невозможен.

Например:

```text
A---B---C-------M  main
     \         /
      D---E---    feature
```

`M` — merge commit.

У него обычно **два родительских коммита**:

```text
      M
     / \
    C   E
```

Один родитель — последний commit текущей ветки, второй — последний commit вливаемой ветки.

Merge commit сохраняет в истории информацию о том, что две независимые линии разработки были объединены.

---

## 🗣️ Ответ на собеседовании

> Merge commit — это commit, который появляется при объединении двух веток, когда их история после общего предка развивалась независимо и fast-forward невозможен.
>
> Например, `main` получила свои коммиты, а `feature` — свои. При `git merge feature` Git создаёт новый merge commit, который имеет два родителя: последний commit `main` и последний commit `feature`.
>
> Благодаря этому в истории сохраняется информация о параллельном развитии веток и моменте их объединения.
>
> Важно, что merge commit создаётся не при каждом merge. Если возможен fast-forward, Git просто перемещает указатель ветки и новый commit не создаёт.

---

## 🧭 Где я нахожусь

```text
07 DevOps
└── 05 Git и Workflow
    ├── Git
    ├── Репозиторий Git
    ├── Ветки Git
    ├── Merge
    │   ├── Fast-forward
    │   └── Merge Commit ← Я здесь
    ├── Rebase
    ├── GitHub
    ├── Конфликты
    ├── Stash
    └── Staging Area
```

Внутри концепции:

```text
Merge
├── Fast-forward
│   └── Новый commit не создаётся
│
└── Three-way merge
    └── Merge Commit
        ├── Parent 1
        └── Parent 2
```

---

# 📚 Разбор поглубже

## 1. Что такое Merge Commit

Обычный commit имеет одного родителя:

```text
A---B---C
        ↑
        C имеет parent B
```

Merge commit отличается тем, что обычно имеет **двух родителей**:

```text
A---B---C-------M
     \         /
      D---E---
```

У `M`:

```text
Parent 1 → C
Parent 2 → E
```

Поэтому merge commit можно представить как точку соединения двух историй.

---

# 2. Как появляется Merge Commit

Предположим, есть:

```text
A---B---C  main
     \
      D---E  feature
```

После создания `feature` обе ветки развивались независимо.

В `main` появился:

```text
C
```

В `feature` появились:

```text
D
E
```

Теперь выполняем:

```bash
git switch main
git merge feature
```

Git создаёт:

```text
A---B---C-------M  main
     \         /
      D---E---    feature
```

`M` — merge commit.

---

# 3. Почему Git создаёт отдельный commit

До merge существуют две разные линии:

```text
main:
A---B---C

feature:
A---B---D---E
```

Git должен сохранить информацию:

> Эти две истории были объединены.

Поэтому появляется новая точка:

```text
        M
       / \
      C   E
```

`M` связывает две истории в одном графе.

---

# 4. Два родителя

Это ключевая особенность merge commit.

Обычный commit:

```text
A---B---C

C
│
└── parent = B
```

Merge commit:

```text
A---B---C-------M
     \         /
      D---E---
```

У `M`:

```text
M
├── parent 1 = C
└── parent 2 = E
```

Именно наличие нескольких родителей позволяет Git представить объединение двух историй.

---

# 5. Merge Commit не равен обычному Commit

Оба являются объектами Git типа **commit**, но merge commit имеет несколько родителей.

Обычный:

```text
Commit C
    │
    └── parent B
```

Merge:

```text
Merge Commit M
    ├── parent C
    └── parent E
```

Поэтому merge commit можно считать обычным Git commit с важной особенностью:

> **У него два или более родителей.**

В обычном типичном merge — два.

---

# 6. Как посмотреть Merge Commit

Историю удобно смотреть:

```bash
git log --graph --oneline --all
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

Здесь:

```text
91abc12
```

— merge commit.

Граф показывает, что у него две линии родителей.

---

# 7. Как Git определяет необходимость Merge Commit

Главный вопрос:

> Может ли Git выполнить fast-forward?

### Сценарий 1 — Fast-forward возможен

```text
A---B  main
     \
      C---D  feature
```

`main` является предком `feature`.

Можно просто передвинуть `main`:

```text
A---B---C---D
            ↑
           main
```

Merge commit не нужен.

---

### Сценарий 2 — Fast-forward невозможен

```text
A---B---C  main
     \
      D---E  feature
```

Ветки имеют собственные commits после общего предка.

Git создаёт:

```text
A---B---C-------M
     \         /
      D---E---
```

`M` — merge commit.

---

# 8. `git merge --no-ff`

Git можно заставить создать merge commit даже тогда, когда возможен fast-forward:

```bash
git merge --no-ff feature
```

Например, было:

```text
A---B---C  main
         \
          D---E  feature
```

Обычный merge:

```text
A---B---C---D---E
```

С `--no-ff`:

```text
A---B---C-------M
         \     /
          D---E
```

Таким образом, в истории явно остаётся факт:

> Коммиты `D` и `E` были частью отдельной feature-ветки.

---

# 9. Зачем использовать `--no-ff`

Одна из причин — сохранить структуру feature-веток в истории.

Например:

```text
A---B-------M1-------M2
     \     /  \     /
      C---D    E---F
```

По истории видно:

```text
M1 → feature A была объединена
M2 → feature B была объединена
```

Без merge commit история могла бы выглядеть просто линейно:

```text
A---B---C---D---E---F
```

И факт существования отдельных feature-веток в графе был бы менее очевиден.

---

# 10. Merge Commit и Pull Request

На GitHub merge Pull Request может приводить к созданию merge commit — если выбран соответствующий способ merge.

Упрощённо:

```text
feature
   │
   ▼
Pull Request
   │
   ▼
Code Review
   │
   ▼
Merge
   │
   ▼
main
```

При создании merge commit:

```text
A---B---C-------M  main
     \         /
      D---E---    feature
```

Но GitHub также поддерживает другие стратегии объединения, например squash и rebase merge.

Поэтому:

> **Pull Request не означает автоматически, что будет создан merge commit.**

---

# 11. Merge Commit vs Squash

Допустим, feature содержит:

```text
D---E---F
```

### Merge

Сохраняет все commits и добавляет merge commit:

```text
A---B---C-------M
     \         /
      D---E---F
```

### Squash

Объединяет изменения feature в один commit:

```text
A---B---C---S
```

где `S` содержит итоговые изменения feature.

Получается:

```text
Merge:
несколько commits + merge commit

Squash:
один итоговый commit
```

---

# 12. Merge Commit vs Rebase

Это фундаментальное сравнение.

### Merge

```text
A---B---C-------M
     \         /
      D---E---
```

Сохраняет историю ветвления.

### Rebase

```text
A---B---C---D'---E'
```

Переносит commits и создаёт новые версии.

Поэтому:

```text
Merge Commit
    ↓
история сохраняется

Rebase
    ↓
история переписывается
```

---

# 13. Merge Commit и конфликт

Merge commit может появиться после разрешения merge conflict.

Например:

```bash
git merge feature
```

Возник конфликт:

```text
CONFLICT
```

Исправляем:

```bash
git add .
```

Затем:

```bash
git commit
```

Git создаёт merge commit:

```text
A---B---C-------M
     \         /
      D---E---
```

То есть конфликт не отменяет возможность создания merge commit.

---

# 14. Merge Commit и `git log`

Можно посмотреть родителей конкретного commit:

```bash
git show --no-patch --pretty=raw <commit>
```

В merge commit можно увидеть несколько строк:

```text
parent <sha1>
parent <sha2>
```

У обычного commit будет один:

```text
parent <sha1>
```

У merge commit — два или больше.

---

# 15. Почему Merge Commit полезен

Merge commit может сохранять контекст истории:

```text
feature/login
      │
      ├── Add login
      ├── Add JWT
      └── Add tests
             │
             ▼
          Merge
             │
             ▼
            main
```

В истории можно увидеть:

* где была создана ветка;
* какие commits относились к ней;
* когда ветка была объединена;
* какие две линии истории были соединены.

Это может быть полезно при анализе истории проекта.

---

# 16. Недостаток большого количества Merge Commit

Если команда постоянно выполняет merge, история может стать сложной:

```text
A---B-------M1-------M2------M3
     \     /  \     /  \    /
      C---D    E---F    G--H
```

При большом количестве параллельных веток граф становится менее линейным.

Именно поэтому некоторые команды предпочитают:

* rebase;
* squash merge;
* комбинацию этих подходов.

Конкретный workflow зависит от команды и проекта.

---

# 17. Как отличить Merge Commit

Можно посмотреть:

```bash
git log --oneline --graph --all
```

или:

```bash
git show <commit>
```

Если commit имеет несколько родителей, это merge commit.

Также можно использовать:

```bash
git rev-list --parents -n 1 <commit>
```

Для обычного commit:

```text
commit_sha parent_sha
```

Для merge commit:

```text
commit_sha parent1_sha parent2_sha
```

---

# 18. Основная схема

```text
                 main
                  │
                  C
                 / \
                /   \
               /     \
              D       \
              │        \
              E         \
               \        /
                \      /
                  M
                  ↑
            merge commit
```

Упрощённо:

```text
M
├── parent 1 → C
└── parent 2 → E
```

---

# 19. Важные команды

| Команда                                | Назначение                          |
| -------------------------------------- | ----------------------------------- |
| `git merge feature`                    | Объединить feature с текущей веткой |
| `git merge --no-ff feature`            | Принудительно создать merge commit  |
| `git log --graph --oneline --all`      | Посмотреть граф истории             |
| `git show <commit>`                    | Посмотреть commit                   |
| `git rev-list --parents -n 1 <commit>` | Посмотреть родителей commit         |
| `git merge --abort`                    | Отменить незавершённый merge        |

---

## 🎤 Вопросы на собеседовании

### Что такое Merge Commit?

> Это commit, созданный при объединении двух разошедшихся историй. В обычном случае он имеет двух родителей.

### Сколько родителей у merge commit?

> Обычно два: последний commit текущей ветки и последний commit вливаемой ветки.

### Всегда ли `merge` создаёт merge commit?

> Нет. Если возможен fast-forward, Git просто перемещает указатель ветки.

### Как принудительно создать merge commit?

```bash
git merge --no-ff feature
```

### Чем merge commit отличается от обычного commit?

> Обычный commit обычно имеет одного родителя, а merge commit — двух или более.

### Зачем нужен merge commit?

> Он сохраняет в истории факт объединения двух независимых линий разработки и позволяет восстановить структуру ветвления.

### Чем Merge Commit отличается от Rebase?

> Merge commit объединяет существующие истории, сохраняя их структуру. Rebase переносит commits на новую базу и создаёт их новые версии.

### Что произойдёт при merge conflict?

> Git остановит merge. После разрешения конфликтов, `git add` и `git commit` завершат merge и создадут merge commit.

### Главная формула

> **Merge Commit = точка объединения двух историй + два родителя + сохранение факта ветвления.**
