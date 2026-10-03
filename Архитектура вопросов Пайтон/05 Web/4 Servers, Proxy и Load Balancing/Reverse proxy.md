# 🔄 Reverse Proxy

## 🎯 Ответ на собеседовании

**Reverse Proxy** — это сервер-посредник, который принимает запросы от клиентов и **перенаправляет их на один или несколько внутренних серверов**.

Клиент взаимодействует с reverse proxy, а не напрямую с backend-сервером.

Например:

```text
Client
   ↓
Nginx (Reverse Proxy)
   ↓
FastAPI
```

Или несколько backend-серверов:

```text
                 ┌→ FastAPI 1
Client → Nginx ──┼→ FastAPI 2
                 └→ FastAPI 3
```

Для клиента всё выглядит как один сервер.

---

## 🎤 Суперкоротко

**Reverse Proxy = посредник между клиентом и backend-серверами.**

```text
Client → Reverse Proxy → Backend
```

Например, **Nginx** может выступать в роли reverse proxy.

---

# 🔄 Как работает

Клиент отправляет запрос:

```python
GET /api/users
```

Но запрос приходит не напрямую в FastAPI:

```text
Client
   ↓
Nginx
   ↓
FastAPI
```

Nginx принимает запрос, выбирает backend и передаёт ему запрос.

Ответ возвращается обратно:

```text
FastAPI
   ↓
Nginx
   ↓
Client
```

---

# 🏗️ Reverse Proxy с несколькими серверами

Reverse Proxy может выполнять **балансировку нагрузки**.

```text
                 ┌→ FastAPI 1
                 │
Client → Nginx ──┼→ FastAPI 2
                 │
                 └→ FastAPI 3
```

Nginx распределяет запросы между backend-серверами.

Например:

```text
Request 1 → FastAPI 1
Request 2 → FastAPI 2
Request 3 → FastAPI 3
```

---

# 🔐 TLS Termination

Reverse Proxy часто принимает HTTPS-соединение и выполняет TLS-шифрование/расшифрование.

```text
Client
   │
   │ HTTPS :443
   ↓
Nginx
   │
   │ HTTP
   ↓
FastAPI
```

Это называется **TLS Termination**.

Backend при этом может работать без непосредственного обслуживания HTTPS.

---

# 📁 Статические файлы

Nginx может самостоятельно отдавать статические файлы, не передавая запрос FastAPI.

```text
Client
   ↓
Nginx
   ├── /static → отдаёт сам
   │
   └── /api → FastAPI
```

Это снижает нагрузку на backend.

---

# ⚖️ Балансировка нагрузки

Reverse Proxy может распределять запросы между несколькими серверами:

```text
                 ┌→ Server 1
                 │
Client → Nginx ──┼→ Server 2
                 │
                 └→ Server 3
```

Можно использовать разные алгоритмы:

```text
Round Robin
Weighted Round Robin
Least Connections
IP Hash
Random
```

---

# 🛡️ Зачем нужен Reverse Proxy

Reverse Proxy позволяет:

* скрыть внутренние backend-серверы;
* централизовать HTTPS;
* выполнять балансировку нагрузки;
* отдавать статические файлы;
* выполнять кэширование;
* применять ограничения запросов;
* централизованно обрабатывать некоторые HTTP-заголовки;
* упростить масштабирование backend.

---

# 🆚 Forward Proxy vs Reverse Proxy

Это частый вопрос на собеседовании.

### Forward Proxy

Прокси работает **от имени клиента**.

```text
Client
   ↓
Forward Proxy
   ↓
Internet
```

Сервер назначения может не видеть реальный IP клиента.

Пример — корпоративный proxy-сервер.

### Reverse Proxy

Прокси работает **перед серверами**.

```text
Client
   ↓
Reverse Proxy
   ↓
Backend
```

Клиент не взаимодействует напрямую с backend.

---

# 🧩 Reverse Proxy vs Load Balancer

Они не одно и то же.

**Reverse Proxy** — более широкое понятие:

```text
Client
   ↓
Reverse Proxy
   ↓
Backend
```

**Load Balancer** — компонент или функция, задача которой — распределять нагрузку между несколькими серверами:

```text
             ┌→ Server 1
Client → LB ─┼→ Server 2
             └→ Server 3
```

Reverse Proxy **может выполнять функции Load Balancer**.

Например, Nginx может одновременно быть:

```text
Nginx
├── Reverse Proxy
├── Load Balancer
├── TLS termination
└── Static file server
```

---

# 🐍 Пример для FastAPI

Типичная production-схема:

```text
Internet
   ↓
HTTPS :443
   ↓
Nginx
   ↓
Gunicorn / Uvicorn
   ↓
FastAPI
   ↓
PostgreSQL
```

Nginx принимает внешний HTTP/HTTPS-трафик и проксирует запросы приложению.

Например:

```python
location /api/ {
    proxy_pass http://backend:8000;
}
```

---

# ⚠️ Важный нюанс

Не нужно говорить:

> Reverse Proxy — это сервер, который просто перенаправляет запросы.

Точнее:

> **Reverse Proxy — это промежуточный сервер, который принимает запросы клиентов и от имени клиента взаимодействует с внутренними серверами.**

Он может не только проксировать запросы, но и выполнять:

```text
TLS termination
Load balancing
Caching
Rate limiting
Static files
Access control
```

---

# 🧠 Главное

```text
                 ┌→ Backend 1
                 │
Client → Nginx ──┼→ Backend 2
                 │
                 └→ Backend 3
```

**Reverse Proxy скрывает внутреннюю инфраструктуру от клиента и выступает единой точкой входа в backend.**

Главная формулировка для собеседования:

> **Reverse Proxy — это сервер-посредник, который принимает запросы клиентов и передаёт их внутренним backend-серверам. Например, Nginx может выступать reverse proxy, выполнять TLS termination, балансировку нагрузки, кэширование и отдачу статических файлов.**
