# 🔀 Merge Request (MR)

## 🎤 Короткий ответ

**Merge Request (MR)** — это запрос на **слияние изменений из одной Git-ветки в другую** с предварительным обсуждением и проверкой кода.

Обычно разработчик создаёт отдельную ветку, делает изменения, отправляет её на Git-сервер и открывает MR:

```text
feature → Merge Request → code review → CI/CD → merge → main
```

В **GitHub** аналогичный механизм называется **Pull Request (PR)**.

## 🎯 Формула для собеседования

> **MR = предложение объединить изменения одной ветки с другой + Code Review + автоматические проверки.**

---

# 🔹 Зачем нужен Merge Request

MR позволяет не отправлять изменения непосредственно в `main`.

Вместо:

```text
Developer
    ↓
main
```

используется:

```text
Developer
    ↓
feature branch
    ↓
Merge Request
    ↓
Code Review
    ↓
CI
    ↓
Merge
    ↓
main
```

Это позволяет проверить изменения **до попадания в основную ветку**.

---

# 🔹 Типичный workflow

Допустим, нужно добавить авторизацию.

Создаём ветку:

```python id="g4m8qx"
git checkout -b feature/jwt-auth
```

Разрабатываем:

```text
feature/jwt-auth
        ↓
     commits
        ↓
     push
        ↓
Merge Request
```

После этого команда проверяет изменения.

Если всё нормально:

```text
MR
 ↓
Approve
 ↓
Merge
 ↓
main
```

---

# 🔹 Основные этапы

```text
1. Создать branch
       ↓
2. Написать код
       ↓
3. Сделать commits
       ↓
4. Push branch
       ↓
5. Создать MR
       ↓
6. Code Review
       ↓
7. CI checks
       ↓
8. Исправить замечания
       ↓
9. Approve
       ↓
10. Merge
```

---

# 🔹 Code Review

**Code Review** — проверка изменений другими разработчиками.

Reviewer может проверить:

* корректность кода;
* архитектуру;
* читаемость;
* тесты;
* обработку ошибок;
* безопасность;
* производительность;
* соответствие coding standards.

Например:

```text
Reviewer:
"Здесь возможен N+1 query.
Используй select_related()."
```

Разработчик исправляет:

```text
Push new commit
       ↓
CI запускается снова
       ↓
Reviewer проверяет изменения
```

---

# 🔹 CI в Merge Request

MR часто запускает автоматические проверки.

Например:

```text
Merge Request
      ↓
     CI
      │
      ├── tests
      ├── lint
      ├── formatting
      ├── type checking
      └── build Docker image
```

Например:

```text
pytest       ✅
flake8       ✅
black        ✅
mypy         ✅
Docker build ✅
```

Если тесты падают:

```text
CI ❌
```

Merge может быть запрещён правилами репозитория.

---

# 🔹 Что такое Approval

**Approval** — подтверждение reviewer, что изменения прошли code review.

Например:

```text
Developer → MR
              ↓
           Reviewer
              ↓
           Approved
              ↓
             Merge
```

В команде могут требовать:

```text
1 approval
2 approvals
approval from CODEOWNERS
```

Конкретные правила зависят от проекта.

---

# 🔹 Merge

**Merge** — объединение изменений двух веток.

Например:

```text
main
  │
  ├───────────────┐
  │               │
  │          feature/auth
  │               │
  │           commits
  │               │
  └─────── Merge ─┘
          ↓
         main
```

После merge изменения из feature-ветки становятся частью целевой ветки.

---

# 🔹 Source и Target Branch

У MR есть две основные ветки.

### Source branch

Откуда приходят изменения:

```text
feature/auth
```

### Target branch

Куда они должны попасть:

```text
main
```

Получается:

```text
feature/auth
      ↓
    MR
      ↓
    main
```

---

# 🔹 Merge Request ≠ Merge

Это важно.

**Merge Request**:

> предложение выполнить объединение.

**Merge**:

> само объединение веток.

То есть:

```text
MR → review → approval → merge
```

---

# 🔹 MR ≠ Commit

**Commit** — отдельная фиксация изменений в Git.

```text
commit 1
commit 2
commit 3
```

MR объединяет целый набор изменений:

```text
feature branch
 ├── commit 1
 ├── commit 2
 └── commit 3
        ↓
       MR
        ↓
      main
```

---

# 🔹 MR ≠ Branch

**Branch** — указатель на линию разработки.

**Merge Request** — механизм предложения изменений из одной ветки в другую.

```text
Branch:
feature/payment

MR:
feature/payment → main
```

---

# 🔹 Merge Strategies

Существуют разные способы объединения истории Git.

### Merge Commit

Создаётся отдельный merge commit:

```text
A ── B ─────── M
      \       /
       C ────
```

### Squash Merge

Несколько commits объединяются в один:

```text
commit A
commit B
commit C
    ↓
commit S
```

История `main` становится более компактной.

### Rebase

Commits переносятся поверх актуальной версии целевой ветки:

```text
main:    A ── B
               \
feature:         C ── D
```

после rebase:

```text
main:    A ── B
               \
                C' ── D'
```

Rebase переписывает commit history, поэтому с общими ветками его используют осторожно.

---

# 🔹 Что происходит при конфликте

Допустим:

```text
main
 ↓
file.py
```

и одновременно:

```text
feature
 ↓
изменение той же строки
```

Git не может автоматически решить, какую версию оставить.

Возникает:

```text
Merge Conflict
```

Например:

```python id="v8m3qx"
<<<<<<< HEAD
timeout = 10
=======
timeout = 30
>>>>>>> feature
```

Разработчик должен вручную выбрать правильный вариант:

```python id="q4m7vx"
timeout = 30
```

После этого:

```text
git add
git commit
git push
```

и MR снова проходит проверки.

---

# 🔹 Draft Merge Request

**Draft MR** — незавершённый MR.

Используется, когда разработчик хочет получить ранний feedback, но код ещё не готов к merge.

```text
Draft MR
   ↓
Early Code Review
   ↓
Development
   ↓
Ready for review
   ↓
Approve
   ↓
Merge
```

---

# 🔹 Protected Branch

Основную ветку часто защищают.

Например:

```text
main
```

нельзя изменить напрямую.

Вместо:

```text
git push origin main
```

требуется:

```text
feature
   ↓
MR
   ↓
CI
   ↓
Review
   ↓
Merge
```

Это снижает вероятность попадания непроверенного кода в production branch.

---

# 🔹 Хороший Merge Request

Хороший MR обычно:

* делает одну логически связанную задачу;
* имеет понятное название;
* содержит описание изменений;
* содержит тесты;
* проходит CI;
* не содержит лишних изменений;
* имеет небольшой и обозримый diff.

Например:

```text
feat: add JWT authentication
```

Описание:

```text
Что сделано:
- добавлена JWT-аутентификация
- добавлены login/refresh endpoints
- добавлены tests

Как проверить:
- pytest
```

---

# 🔹 Маленькие MR

Большой MR:

```text
5000 lines changed
```

сложно качественно проверить.

Лучше разделять:

```text
MR #1 → database models
MR #2 → authentication
MR #3 → API endpoints
MR #4 → tests
```

Но дробить нужно по **логическим изменениям**, а не искусственно.

---

# 🔹 MR и CI/CD

MR обычно связан с CI:

```text
Developer
   ↓
Push
   ↓
Merge Request
   ↓
CI
 ┌─┴─────────────┐
 ↓               ↓
Tests           Lint
 ↓               ↓
 └──────┬────────┘
        ↓
     Approved
        ↓
      Merge
        ↓
    Deployment
```

При этом **CI и CD — разные вещи**:

* CI — автоматическая проверка изменений;
* CD — доставка/развёртывание изменений.

---

# 🔹 GitLab и GitHub

Терминология отличается:

| GitLab        | GitHub                |
| ------------- | --------------------- |
| Merge Request | Pull Request          |
| MR            | PR                    |
| Merge         | Merge                 |
| Pipeline      | Actions / CI workflow |

По смыслу MR и PR выполняют похожую задачу.

---

# 🔹 Пример для Python Backend

Допустим, есть FastAPI-проект:

```text
main
```

Создаём:

```python id="w5m8qx"
git checkout -b feature/orders
```

Пишем endpoint:

```python id="j7q3mc"
@app.post("/orders")
async def create_order():
    ...
```

Добавляем тесты:

```python id="x4m8vx"
def test_create_order():
    ...
```

Commit:

```python id="n6q2vk"
git add .
git commit -m "feat: add order creation"
```

Push:

```python id="p8m4qx"
git push origin feature/orders
```

Создаём:

```text
feature/orders → main
```

После этого:

```text
CI
 ↓
pytest
 ↓
flake8
 ↓
black
 ↓
Code Review
 ↓
Approval
 ↓
Merge
```

---

# 🔹 Как выглядит жизненный цикл MR

```text
                 ┌──────────────┐
                 │ Feature      │
                 │ Branch       │
                 └──────┬───────┘
                        ↓
                     Push
                        ↓
                ┌───────────────┐
                │ Merge Request │
                └───────┬───────┘
                        ↓
              ┌─────────┴─────────┐
              ↓                   ↓
           Code Review            CI
              ↓                   ↓
           Changes?             Tests
              │                   │
              └─────────┬─────────┘
                        ↓
                     Approve
                        ↓
                      Merge
                        ↓
                       main
```

---

# 🔹 Частые вопросы на собеседовании

### Что такое Merge Request?

> Запрос на слияние изменений из одной Git-ветки в другую с возможностью code review и автоматических проверок.

### Чем MR отличается от commit?

> Commit фиксирует конкретное изменение в Git, а MR объединяет набор изменений и служит процессом их проверки и интеграции.

### Чем MR отличается от branch?

> Branch — линия разработки, MR — запрос на перенос изменений из одной ветки в другую.

### Что происходит после создания MR?

> Обычно запускается CI, выполняется code review, разработчик исправляет замечания, после approval изменения merge'ятся в целевую ветку.

### Что такое Code Review?

> Проверка изменений другим разработчиком до их интеграции в основную ветку.

### Что делать, если CI упал?

> Посмотреть причину ошибки, исправить код, выполнить необходимые проверки и отправить новый commit в ту же ветку MR.

### Что делать при Merge Conflict?

> Обновить локальную ветку относительно target branch, разрешить конфликт вручную, проверить тесты и отправить исправление в MR.

### Что такое Draft MR?

> Незавершённый MR, который ещё не готов к финальному merge, но может использоваться для раннего review.

### Что такое Protected Branch?

> Ветка с ограничениями на прямое изменение, например обязательным review и успешным CI перед merge.

### GitLab MR и GitHub PR — это одно и то же?

> По назначению — практически аналогичные механизмы: предложение изменений из одной ветки в другую с review и проверками.

---

# 🔑 Главное

```text
Branch
  ↓
Commit
  ↓
Push
  ↓
Merge Request
  ↓
Code Review + CI
  ↓
Approve
  ↓
Merge
  ↓
main
```

> **Branch** → где разрабатываем
> **Commit** → фиксируем изменения
> **Push** → отправляем изменения на сервер
> **MR/PR** → предлагаем интегрировать изменения
> **Code Review** → проверяем код
> **CI** → автоматически проверяем проект
> **Merge** → объединяем ветки
