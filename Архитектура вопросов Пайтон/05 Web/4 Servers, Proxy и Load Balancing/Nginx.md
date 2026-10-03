# 🌐 Nginx

## 🎯 Ответ на собеседовании

**Nginx** — высокопроизводительный веб-сервер и **reverse proxy**, который часто используется перед Python-приложением.

В backend-проекте Nginx может:

* принимать HTTP/HTTPS-запросы;
* работать как reverse proxy;
* терминировать TLS;
* раздавать статические файлы;
* балансировать нагрузку;
* проксировать запросы к нескольким backend-серверам.

Типичная схема:

```text
Клиент
   ↓
Nginx
   ↓
Uvicorn / Gunicorn
   ↓
FastAPI / Django
   ↓
Database
```

Главная идея:

> **Nginx принимает внешние запросы и выступает посредником между клиентом и backend-приложением.**

---

## 🎤 Суперкоротко

```text
Nginx = Web Server + Reverse Proxy
```

Типичный Python backend:

```text
Client
  ↓
Nginx
  ↓
Uvicorn → FastAPI
```

или:

```text
Client
  ↓
Nginx
  ↓
Gunicorn → Django
```

---

# 🔄 Reverse Proxy

**Reverse proxy** — сервер, который принимает запросы от клиента и передаёт их внутренним серверам.

Клиент не обращается напрямую к приложению:

```text
Client
   ↓
Nginx
   ↓
Backend
```

Например, FastAPI работает на:

```text
127.0.0.1:8000
```

А пользователь обращается:

```text
https://example.com
```

Nginx принимает внешний запрос и проксирует его на:

```text
http://127.0.0.1:8000
```

---

# 🆚 Forward Proxy и Reverse Proxy

### Forward Proxy

Проксирует запросы **от клиентов наружу**:

```text
Client
   ↓
Proxy
   ↓
Internet
```

Proxy действует от имени клиента.

### Reverse Proxy

Проксирует запросы **к серверам**:

```text
Client
   ↓
Reverse Proxy
   ↓
Backend
```

Nginx часто используется именно как **reverse proxy**.

---

# ⚙️ Пример конфигурации

Простейший reverse proxy:

```python
server {
    listen 80;

    server_name example.com;

    location / {
        proxy_pass http://127.0.0.1:8000;
    }
}
```

Теперь:

```text
GET /users
```

попадает в Nginx:

```text
Client
   ↓
Nginx :80
   ↓
127.0.0.1:8000
   ↓
FastAPI
```

---

# 🚀 Nginx + FastAPI

Типичная схема:

```text
                    Internet
                       ↓
                  ┌─────────┐
                  │  Nginx  │
                  └────┬────┘
                       ↓
                  ┌─────────┐
                  │ Uvicorn │
                  └────┬────┘
                       ↓
                  ┌─────────┐
                  │ FastAPI │
                  └────┬────┘
                       ↓
                  PostgreSQL
```

Nginx принимает внешний трафик, Uvicorn запускает ASGI-приложение, FastAPI обрабатывает запрос.

---

# 🐍 Nginx + Django

Для Django часто используется:

```text
Client
   ↓
Nginx
   ↓
Gunicorn
   ↓
Django
```

Gunicorn работает как WSGI-сервер.

---

# 🔐 TLS / HTTPS

Nginx часто используется для завершения TLS-соединения.

Например:

```text
Client
   ↓
HTTPS
   ↓
Nginx
   ↓
HTTP
   ↓
FastAPI
```

То есть Nginx принимает зашифрованное HTTPS-соединение, расшифровывает его и передаёт запрос backend-приложению.

Это называется **TLS termination**.

Преимущество — не нужно отдельно настраивать TLS в каждом backend-сервисе.

---

# 📁 Раздача статических файлов

Nginx отлично подходит для раздачи:

* CSS;
* JavaScript;
* изображений;
* шрифтов;
* других статических файлов.

Например:

```text
/static/
```

можно отдавать непосредственно через Nginx:

```text
Client
   ↓
Nginx
   ↓
static files
```

При этом запросы к API идут дальше:

```text
/api/
   ↓
Nginx
   ↓
FastAPI
```

Это разгружает Python-приложение.

---

# ⚖️ Load Balancing

Nginx может распределять запросы между несколькими backend-серверами.

Например:

```text
                 Nginx
                   ↓
        ┌──────────┼──────────┐
        ↓          ↓          ↓
    FastAPI 1  FastAPI 2  FastAPI 3
```

Пример:

```python
upstream backend {
    server 127.0.0.1:8001;
    server 127.0.0.1:8002;
    server 127.0.0.1:8003;
}


server {
    listen 80;

    location / {
        proxy_pass http://backend;
    }
}
```

Nginx распределяет запросы между серверами из `upstream`.

---

# 🧠 Зачем нужен Load Balancing

Если один backend не справляется с нагрузкой:

```text
1000 requests
      ↓
   Backend
```

можно добавить несколько экземпляров:

```text
1000 requests
      ↓
    Nginx
      ↓
 ┌────┼────┐
 ↓    ↓    ↓
App1 App2 App3
```

Это позволяет масштабировать приложение горизонтально.

---

# 📦 Headers

При reverse proxy Nginx часто передаёт backend информацию о первоначальном запросе.

Например:

```python
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto $scheme;
```

Это важно, потому что backend должен понимать:

* какой был исходный host;
* IP клиента;
* какой протокол использовался;
* через какие proxy прошёл запрос.

---

# 🌐 Почему backend обычно не выставляют напрямую в Internet

Допустим, FastAPI слушает:

```text
0.0.0.0:8000
```

Можно отправлять запросы напрямую:

```text
Internet
   ↓
FastAPI
```

Но часто предпочтительнее:

```text
Internet
   ↓
Nginx
   ↓
FastAPI
```

Nginx может централизованно выполнять:

* TLS;
* proxy;
* ограничения;
* маршрутизацию;
* раздачу статики;
* балансировку.

---

# 🛡️ Nginx и безопасность

Nginx может участвовать в защите приложения:

```text
Client
   ↓
Nginx
   ↓
Backend
```

Например, на уровне Nginx можно настроить:

* ограничения размера запросов;
* rate limiting;
* TLS;
* определённые HTTP-заголовки;
* доступ к отдельным маршрутам;
* IP-based ограничения.

Но Nginx **не заменяет** полноценную authentication/authorization в приложении.

---

# 🔀 Nginx как маршрутизатор

Можно направлять разные URL в разные сервисы:

```text
/api/users/
      ↓
User Service

/api/orders/
      ↓
Order Service

/static/
      ↓
Nginx
```

Например:

```python
location /api/users/ {
    proxy_pass http://users_service;
}

location /api/orders/ {
    proxy_pass http://orders_service;
}

location /static/ {
    root /var/www;
}
```

Это особенно полезно в микросервисной архитектуре.

---

# 🆚 Nginx vs Uvicorn

Это разные уровни.

| Nginx                      | Uvicorn                   |
| -------------------------- | ------------------------- |
| Web server / reverse proxy | ASGI server               |
| Работает перед приложением | Запускает ASGI-приложение |
| TLS termination            | ASGI                      |
| Static files               | HTTP/WebSocket            |
| Load balancing             | Event loop                |
| Proxy                      | FastAPI/Starlette         |

Типичная схема:

```text
Nginx
  ↓
Uvicorn
  ↓
FastAPI
```

---

# 🆚 Nginx vs Gunicorn

| Nginx                         | Gunicorn                    |
| ----------------------------- | --------------------------- |
| Reverse proxy / web server    | Application server          |
| Может принимать внешний HTTPS | Запускает Python-приложение |
| Статика                       | WSGI                        |
| Load balancing                | Workers                     |
| TLS termination               | Django/Flask                |

Типичная схема:

```text
Nginx
  ↓
Gunicorn
  ↓
Django
```

---

# 🔗 Связка всего Backend

Для FastAPI:

```text
                     Client
                        ↓
                      HTTPS
                        ↓
                     Nginx
                        ↓
                    Uvicorn
                        ↓
                      ASGI
                        ↓
                    FastAPI
                        ↓
                     Service
                        ↓
                   Repository
                        ↓
                    PostgreSQL
```

Для Django:

```text
                     Client
                        ↓
                      HTTPS
                        ↓
                     Nginx
                        ↓
                    Gunicorn
                        ↓
                      WSGI
                        ↓
                     Django
                        ↓
                       ORM
                        ↓
                   PostgreSQL
```

---

# 🧠 Главное

Запомнить роли:

```text
Nginx
↓
принимает внешний трафик
↓
reverse proxy
↓
TLS termination
↓
static files
↓
load balancing
↓
backend
```

И всю цепочку:

```text
Nginx
  ↓
Uvicorn
  ↓
ASGI
  ↓
FastAPI
```

или:

```text
Nginx
  ↓
Gunicorn
  ↓
WSGI
  ↓
Django
```

### Формула для собеседования

> **Nginx — веб-сервер и reverse proxy, который часто используется перед Python-приложением. Он может принимать HTTPS, завершать TLS, проксировать запросы к Uvicorn или Gunicorn, раздавать статику и выполнять балансировку нагрузки.**

```text
Nginx = Web Server
      + Reverse Proxy
      + TLS Termination
      + Static Files
      + Load Balancing
```
