# 🔄 Refresh Token

## 🎯 Ответ на собеседовании

**Refresh Token** — это долгоживущий токен, который используется для **получения нового Access Token**, когда старый истёк или скоро истечёт.

Он нужен, чтобы пользователю **не приходилось повторно вводить логин и пароль** после истечения короткоживущего Access Token.

Типичная схема:

```text id="m8x4qz"
Login + Password
      ↓
   Auth Server
      ↓
┌───────────────┐
│ Access Token  │ → короткоживущий
│ Refresh Token │ → долгоживущий
└───────────────┘
```

---

## 🎤 Суперкоротко

**Refresh Token = токен для получения нового Access Token.**

```text id="a4k7ps"
Access Token истёк
        ↓
Refresh Token
        ↓
Новый Access Token
```

---

# 🔄 Как работает

После успешной аутентификации сервер выдаёт оба токена:

```text id="q7f3md"
Access Token  → 15 минут
Refresh Token → 30 дней
```

Клиент использует Access Token для API:

```python id="p9w2kc"
GET /api/profile
Authorization: Bearer <access_token>
```

Через некоторое время Access Token истекает:

```text id="z3n6rv"
Access Token ❌ expired
```

Клиент отправляет Refresh Token на endpoint обновления:

```python id="v5m8qa"
POST /api/token/refresh
```

```python id="c2r7hx"
{
    "refresh": "<refresh_token>"
}
```

Сервер проверяет Refresh Token и выдаёт новый Access Token:

```text id="k8q4pd"
Refresh Token
      ↓
проверка
      ↓
New Access Token
```

---

# ⏱️ Почему Access Token короткоживущий

Access Token используется часто и отправляется с запросами к API:

```text id="n6p3vk"
Client → Access Token → API
```

Если он будет украден, злоумышленник сможет использовать его до истечения срока действия.

Поэтому Access Token обычно делают короткоживущим.

```text id="r4m7xs"
Access Token
     ↓
короткий TTL
     ↓
меньше окно для злоупотребления
```

---

# 🕐 Почему Refresh Token долгоживущий

Refresh Token используется значительно реже — только для получения новых Access Token.

Например:

```text id="e8c2mq"
Access Token  → 15 минут
Refresh Token → 30 дней
```

Благодаря этому пользователь может оставаться авторизованным длительное время, не вводя пароль каждые 15 минут.

---

# 🔐 Refresh Token нельзя использовать для обычного API

Важно различать назначение токенов:

```text id="v6j2pw"
Access Token
    ↓
доступ к API

Refresh Token
    ↓
получение нового Access Token
```

Обычно Refresh Token **не отправляется на обычные endpoints API**.

---

# 🛡️ Безопасность

Refresh Token является чувствительным credential, поэтому его необходимо защищать.

При его компрометации злоумышленник потенциально сможет получать новые Access Token в течение срока действия Refresh Token.

Поэтому применяют:

* HTTPS;
* безопасное хранение на клиенте;
* ограниченный срок жизни;
* отзыв токенов;
* rotation Refresh Token;
* защиту от повторного использования.

Для браузерных приложений часто используют защищённые cookie с подходящими флагами, например `HttpOnly` и `Secure`.

---

# 🔄 Refresh Token Rotation

При **Refresh Token Rotation** после использования старого Refresh Token сервер выдаёт новый:

```text id="k5p8ds"
Refresh Token A
      ↓
   refresh
      ↓
Access Token B
Refresh Token C
```

Старый Refresh Token `A` становится недействительным.

Это позволяет обнаруживать и ограничивать повторное использование украденного refresh-токена.

---

# 🆚 Access Token vs Refresh Token

|                    | Access Token               | Refresh Token                                  |
| ------------------ | -------------------------- | ---------------------------------------------- |
| Назначение         | Доступ к API               | Получение нового Access Token                  |
| Срок жизни         | Обычно короткий            | Обычно длиннее                                 |
| Отправляется       | С API-запросами            | На endpoint обновления                         |
| Используется часто | Да                         | Нет                                            |
| При утечке         | Ограниченное время доступа | Потенциально можно получать новые Access Token |

---

# 🧩 Полный цикл

```text id="d7m3qa"
          Login + Password
                 ↓
            Auth Server
                 ↓
       ┌─────────┴─────────┐
       ↓                   ↓
 Access Token        Refresh Token
       ↓                   ↓
    API-запросы          хранится
       ↓                   │
 Access Token             │
   expired                │
       ↓                   │
       └─────────→ Refresh Endpoint
                         ↓
                  Новый Access Token
```

---

# ⚠️ Частая ошибка

Неправильно:

> Refresh Token используется для доступа к API.

Правильно:

> **Refresh Token используется для получения нового Access Token после его истечения.**

И ещё одна важная мысль:

**Refresh Token не заменяет Access Token.**

У них разные задачи.

---

# 🧠 Главное

```text id="x9f4kc"
Access Token
→ доступ к API
→ короткий срок жизни

Refresh Token
→ получение нового Access Token
→ долгий срок жизни
```

**Refresh Token — это долгоживущий credential, который позволяет получить новый Access Token без повторной аутентификации пользователя.**

Главная формулировка для собеседования:

> **Refresh Token используется для обновления короткоживущего Access Token. Когда Access Token истекает, клиент отправляет Refresh Token на специальный endpoint, сервер проверяет его и выдаёт новый Access Token.**
