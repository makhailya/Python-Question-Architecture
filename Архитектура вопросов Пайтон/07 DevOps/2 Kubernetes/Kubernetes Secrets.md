# 🔐 Kubernetes Secrets

## 🎯 Ответ на собеседовании

**Kubernetes Secret** — это объект Kubernetes для хранения **чувствительных данных**, например паролей, токенов, API-ключей и TLS-сертификатов.

Secret позволяет не хранить такие данные непосредственно в `Deployment`, `Pod` или исходном коде приложения.

Например:

```text id="q7x3na"
Secret
 ├── DB_USER
 ├── DB_PASSWORD
 └── API_KEY
       ↓
     Pod
       ↓
   Application
```

---

## 🎤 Суперкоротко

**Secret = объект Kubernetes для хранения и передачи чувствительных данных в Pod.**

Например:

```text id="h8p2km"
DB_PASSWORD
API_KEY
JWT_SECRET
```

---

# 📦 Создание Secret

Secret можно создать через `kubectl`:

```python id="m2v6rx"
kubectl create secret generic app-secret \
  --from-literal=DB_PASSWORD=secret123
```

После этого Kubernetes создаст объект:

```text id="w4c9pj"
Secret
└── app-secret
    └── DB_PASSWORD
```

---

# 📄 Secret в YAML

Secret можно описать декларативно:

```python id="v9q1ks"
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
data:
  DB_PASSWORD: c2VjcmV0MTIz
```

Здесь значение в `data` должно быть представлено в **Base64**.

Например:

```python id="a7f4nd"
echo -n "secret123" | base64
```

Результат:

```python id="j2k8qx"
c2VjcmV0MTIz
```

---

# ⚠️ Base64 ≠ шифрование

Это очень частый вопрос на собеседовании.

**Base64 не является шифрованием.**

Это всего лишь кодирование.

```text id="d6m3pz"
"secret123"
      ↓
   Base64
      ↓
"c2VjcmV0MTIz"
```

Любой человек может декодировать значение обратно.

Поэтому нельзя считать:

> "Secret безопасен, потому что данные закодированы в Base64."

Это неправильно.

---

# 🔑 StringData

Вместо ручного Base64 можно использовать `stringData`:

```python id="r3n8wv"
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
stringData:
  DB_PASSWORD: secret123
```

Kubernetes сам преобразует значение в нужный формат.

Для пользователя это удобнее, но сам Secret всё равно требует соответствующих мер защиты.

---

# 💉 Передача Secret в Pod

Есть два основных способа.

## 1. Environment Variables

Secret можно передать в контейнер как переменную окружения:

```python id="k6w1za"
env:
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: app-secret
        key: DB_PASSWORD
```

В приложении:

```python id="n4q7cs"
import os

password = os.getenv("DB_PASSWORD")
```

---

## 2. Volume

Secret можно смонтировать в контейнер как файлы:

```python id="s8p3mv"
volumes:
  - name: secret-volume
    secret:
      secretName: app-secret
```

После монтирования приложение может получить секрет из файла.

```text id="z5q9fk"
Pod
 │
 └── /etc/secrets/
       └── DB_PASSWORD
```

---

# 🏷️ Типы Secret

Наиболее распространённый:

```python id="c4m7xb"
type: Opaque
```

Используется для произвольных секретов.

Также существуют специальные типы, например:

```python id="j8s2qd"
kubernetes.io/tls
```

для TLS-сертификатов и ключей.

---

# 🛡️ Безопасность

Важно понимать:

**Kubernetes Secret сам по себе не делает секрет безопасным автоматически.**

Нужно контролировать:

* RBAC — кто может читать Secret;
* доступ к Kubernetes API;
* доступ к etcd;
* шифрование Secret в etcd;
* права пользователей и ServiceAccount.

В production часто используют внешние системы управления секретами, например:

```text id="b6m4qw"
HashiCorp Vault
AWS Secrets Manager
Azure Key Vault
Google Secret Manager
```

---

# 🆚 ConfigMap vs Secret

|                       | ConfigMap | Secret |
| --------------------- | --------- | ------ |
| Обычная конфигурация  | ✅         | ❌      |
| Пароли                | ❌         | ✅      |
| API-ключи             | ❌         | ✅      |
| Токены                | ❌         | ✅      |
| TLS-ключи             | ❌         | ✅      |
| Чувствительные данные | ❌         | ✅      |

Например:

```text id="q3h7np"
ConfigMap
→ APP_PORT
→ LOG_LEVEL
→ DEBUG

Secret
→ DB_PASSWORD
→ API_KEY
→ JWT_SECRET
```

---

# 🏗️ Пример для FastAPI

Допустим, FastAPI подключается к PostgreSQL.

Secret:

```python id="w9x5ka"
apiVersion: v1
kind: Secret
metadata:
  name: backend-secret
type: Opaque
stringData:
  DB_USER: postgres
  DB_PASSWORD: mypassword
```

Deployment получает их:

```python id="u6c2fz"
env:
  - name: DB_USER
    valueFrom:
      secretKeyRef:
        name: backend-secret
        key: DB_USER

  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: backend-secret
        key: DB_PASSWORD
```

FastAPI получает значения через environment:

```python id="p4k8yc"
import os

DB_USER = os.getenv("DB_USER")
DB_PASSWORD = os.getenv("DB_PASSWORD")
```

Пароль при этом не находится в исходном коде приложения.

---

# ⚠️ Частая ошибка

Нельзя считать такой YAML безопасным только потому, что используется `Secret`:

```python id="t2m7vx"
data:
  PASSWORD: cGFzc3dvcmQ=
```

Base64 легко декодируется.

Правильное понимание:

```text id="e5k9rb"
Secret
   ↓
управление чувствительными данными
   ↓
Base64 ≠ encryption
```

Безопасность зависит также от настроек Kubernetes, RBAC и защиты хранилища.

---

# 🧠 Главное

```text id="x7p3md"
Kubernetes Secret
       ↓
чувствительные данные
       ↓
┌─────────────────┐
│ password        │
│ API key         │
│ token           │
│ TLS certificate │
└─────────────────┘
       ↓
     Pod
```

Основные способы использования:

```text id="f8q2vk"
Secret
  ├── Environment Variable
  └── Volume
```

Главное, что нужно сказать на собеседовании:

> **Kubernetes Secret — это объект Kubernetes для хранения и передачи чувствительных данных, таких как пароли, токены и API-ключи. Secret можно передавать в Pod через environment variables или volumes. Значения в `data` представлены в Base64, но Base64 не является шифрованием.**
