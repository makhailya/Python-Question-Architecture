# Hotfix 🚑

## 🎤 Короткий ответ

**Hotfix** — это срочное исправление критической ошибки в уже работающей версии приложения, обычно в **production**.

Обычно создают отдельную ветку от текущей production-версии, быстро исправляют проблему, тестируют и вливают исправление обратно в `main` и/или ветку релиза.

> **Hotfix = срочное исправление production → отдельная ветка → fix → test → merge → deploy.**

---

## 🎯 Формула для собеседования

```text
Production
    ↓
Критический баг
    ↓
hotfix/*
    ↓
Исправление
    ↓
Тесты / CI
    ↓
Merge
    ↓
Deploy
```

---

# 1. Зачем нужен Hotfix

Представим:

```text
main → v1.5.0 → Production
                  ↓
             критический баг
```

Например:

* API перестал отвечать;
* авторизация сломалась;
* платежи не проходят;
* приложение падает;
* критическая ошибка безопасности.

Обычная feature-разработка может ждать следующего релиза.

**Hotfix нужен для срочного исправления проблемы.**

---

# 2. Как выглядит Hotfix-ветка

Например, production сейчас находится на:

```text
main
  ↓
A → B → C
         ↑
       v1.5.0
```

Создаём:

```text
A → B → C
         \
          H
          ↑
      hotfix/payment
```

После исправления:

```text
A → B → C ──────── D
         \        /
          H ──────
```

В зависимости от используемого workflow hotfix может быть влит обратно в `main`, release-ветку и/или другие актуальные ветки.

---

# 3. Типичный workflow

Например, обнаружили ошибку оплаты.

Создаём ветку:

```python id="g1t101"
git switch main
git pull
git switch -c hotfix/payment-error
```

Исправляем код.

Проверяем:

```python id="g1t102"
pytest
```

Смотрим изменения:

```python id="g1t103"
git diff
git status
```

Commit:

```python id="g1t104"
git add .
git commit -m "Fix payment processing error"
```

Push:

```python id="g1t105"
git push -u origin hotfix/payment-error
```

После этого создаётся **Merge Request / Pull Request** и выполняются CI-проверки.

---

# 4. Hotfix vs Feature

|              | Feature                | Hotfix                   |
| ------------ | ---------------------- | ------------------------ |
| Цель         | новая функциональность | срочное исправление      |
| Срочность    | обычная                | высокая                  |
| Причина      | новая задача           | критический баг          |
| Ветка        | `feature/*`            | `hotfix/*`               |
| Production   | обычно не горит        | проблема уже существует  |
| Тестирование | обычный процесс        | быстрое, но обязательное |
| Deploy       | плановый               | срочный                  |

Например:

```text
feature/login
```

— добавляем новую авторизацию.

```text
hotfix/login-crash
```

— исправляем падение уже работающей авторизации.

---

# 5. Hotfix vs Bugfix

Термины могут использоваться немного по-разному в разных командах.

### Bugfix

Обычное исправление ошибки:

```text
bug → исправление → следующий релиз
```

### Hotfix

Срочное исправление критической проблемы:

```text
critical production bug
        ↓
      hotfix
        ↓
   срочный deploy
```

То есть **hotfix — это прежде всего про срочность и production-контекст**, а не отдельный тип программного кода.

---

# 6. Hotfix и CI/CD

Даже срочное исправление желательно пропускать через автоматические проверки:

```text
Hotfix branch
      ↓
   Git push
      ↓
      CI
      ├── pytest
      ├── lint
      ├── type checking
      └── build
          ↓
        Merge
          ↓
       Deploy
```

Срочность не означает:

> «можно вообще не тестировать».

Наоборот, ошибка в hotfix может привести к ещё одному production-инциденту.

---

# 7. Hotfix и Rollback

**Hotfix** и **rollback** — разные вещи.

### Hotfix

Исправляем проблему новым изменением:

```text
v1.5.0
  ↓
bug
  ↓
hotfix
  ↓
v1.5.1
```

### Rollback

Возвращаем приложение к предыдущей рабочей версии:

```text
v1.5.0 → v1.5.1
              ↓
           проблема
              ↓
          rollback
              ↓
           v1.5.0
```

Иногда сначала делают rollback, чтобы быстро восстановить сервис, а затем отдельно готовят hotfix.

---

# 8. Hotfix и Release

После hotfix часто выпускается новая patch-версия.

Например:

```text
v2.3.0
   ↓
обнаружен production bug
   ↓
hotfix
   ↓
v2.3.1
```

Если используется **Semantic Versioning**:

```text
MAJOR.MINOR.PATCH
```

то исправление ошибки без изменения публичного API обычно может получить увеличение `PATCH`.

---

# 9. Hotfix в Git Flow

В классическом Git Flow существуют ветки:

```text
main
develop
feature/*
release/*
hotfix/*
```

Hotfix обычно создаётся от `main`, потому что `main` представляет production-ready состояние.

Пример:

```text
             feature
                ↓
develop ──────────────
                       \
main ──────────────────●
                       ↑
                    production
                       \
                    hotfix/*
```

После исправления hotfix обычно вливается обратно в `main` и синхронизируется с `develop`, чтобы исправление не потерялось в следующей разработке.

---

# 10. Важно: Hotfix — не обязательно Git Flow

Название:

```text
hotfix/*
```

— это **соглашение команды**, а не специальный объект Git.

Git не имеет отдельного типа «hotfix branch».

Для Git это обычная ветка:

```python id="g1t106"
git switch -c hotfix/payment-error
```

Особенность hotfix определяется **процессом разработки**.

---

# 11. Пример для Python Backend

Допустим, FastAPI-приложение в production начало возвращать:

```text
HTTP 500
```

после запроса:

```text
POST /payments
```

Workflow:

```text
Production
    ↓
500 на /payments
    ↓
создаём hotfix/payment-error
    ↓
исправляем Python-код
    ↓
добавляем regression test
    ↓
pytest
    ↓
CI
    ↓
Merge Request
    ↓
merge
    ↓
Docker Image
    ↓
Deploy
    ↓
проверка метрик/логов
```

Особенно полезно добавить **regression test**, который воспроизводит найденный баг и гарантирует, что он не вернётся.

---

# 12. Hotfix и Regression Test

Хорошая практика:

```text
Bug
 ↓
Написать тест, воспроизводящий bug
 ↓
Исправить код
 ↓
Тест проходит
 ↓
Deploy
```

Например:

```python id="g1t107"
def test_payment_with_zero_amount():
    response = client.post(
        "/payments",
        json={"amount": 0},
    )

    assert response.status_code == 400
```

Такой тест фиксирует ожидаемое поведение после исправления.

---

# 🎤 Как ответить на собеседовании

> **Hotfix — это срочное исправление критической ошибки в production. Обычно создают отдельную ветку от актуальной production-версии, исправляют проблему, добавляют или обновляют тесты, проходят CI, после чего изменения вливаются и выполняется срочный deploy. Hotfix отличается от обычного bugfix прежде всего срочностью и необходимостью быстро восстановить корректную работу production.**

---

## 🧠 Главное

```text
Feature
→ новая функциональность

Bugfix
→ обычное исправление ошибки

Hotfix
→ срочное исправление критической production-проблемы

Rollback
→ возврат к предыдущей версии
```

### Ключевая формула

> **Production bug → `hotfix/*` → fix + tests → CI → merge → deploy.**
