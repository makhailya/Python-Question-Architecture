# 🌐 HTTP — HyperText Transfer Protocol

## 🎤 Суперкоротко

**HTTP** — протокол прикладного уровня для обмена данными между клиентом и сервером.

Основная модель:

```text
Клиент ── HTTP Request ──> Сервер
Клиент <─ HTTP Response ── Сервер
```

HTTP-запрос содержит:

* метод;
* URL;
* заголовки;
* тело — опционально.

HTTP-ответ содержит:

* статус-код;
* заголовки;
* тело — опционально.

**HTTP сам по себе не шифрует данные. HTTPS = HTTP + TLS.**

---

## 🎯 Ответ на собеседовании

**HTTP (HyperText Transfer Protocol)** — протокол прикладного уровня, предназначенный для обмена данными между клиентом и сервером.

Работа HTTP основана на модели **request-response**: клиент отправляет запрос, сервер его обрабатывает и возвращает ответ.

HTTP определяет формат запросов и ответов, методы, статус-коды, заголовки и правила взаимодействия клиента с сервером.

---

# 🧩 Где находится HTTP

HTTP относится к **прикладному уровню** сетевого стека.

Упрощённо:

```text
HTTP
 ↓
TCP
 ↓
IP
 ↓
Канальный уровень
```

Для HTTP/1.1 и HTTP/2 обычно используется TCP:

```text
HTTP/1.1 → TCP
HTTP/2   → TCP
```

HTTP/3 работает поверх QUIC:

```text
HTTP/3
 ↓
QUIC
 ↓
UDP
 ↓
IP
```

---

# 🔄 Модель Request → Response

HTTP работает по модели:

```text
        Request
Клиент ───────────> Сервер
       <───────────
        Response
```

Например:

```python
import requests

response = requests.get("https://example.com/users")

print(response.status_code)
print(response.text)
```

Логика:

```text
Python-приложение
       ↓
HTTP GET
       ↓
Сервер
       ↓
HTTP Response
       ↓
Python-приложение
```

---

# 📤 HTTP Request

HTTP-запрос состоит из нескольких основных частей:

```text
Request Line
Headers
Blank Line
Body
```

Например:

```text
GET /users HTTP/1.1
Host: example.com
Accept: application/json
Authorization: Bearer TOKEN

```

Разберём.

---

# 1️⃣ Request Line

Первая строка:

```text
GET /users HTTP/1.1
```

Содержит:

```text
METHOD + PATH + HTTP VERSION
```

В нашем примере:

```text
GET
 ↓
метод

/users
 ↓
ресурс

HTTP/1.1
 ↓
версия протокола
```

---

# 2️⃣ Headers

Заголовки передают дополнительную информацию.

Например:

```text
Host: example.com
Accept: application/json
Authorization: Bearer TOKEN
Content-Type: application/json
User-Agent: Mozilla/5.0
```

Заголовки могут описывать:

* тип данных;
* авторизацию;
* cookies;
* информацию о клиенте;
* кеширование;
* допустимые форматы ответа;
* длину тела запроса.

---

# 3️⃣ Body

Тело запроса используется для передачи данных.

Например, при `POST`:

```python
data = {
    "name": "Ilya",
    "age": 31
}
```

HTTP-запрос концептуально:

```text
POST /users HTTP/1.1
Content-Type: application/json

{"name": "Ilya", "age": 31}
```

Тело запроса **не является обязательным** для каждого HTTP-запроса.

Например, GET обычно отправляется без тела.

---

# 📥 HTTP Response

Ответ сервера также состоит из нескольких частей:

```text
Status Line
Headers
Blank Line
Body
```

Например:

```text
HTTP/1.1 200 OK
Content-Type: application/json

{"id": 1, "name": "Ilya"}
```

---

# 1️⃣ Status Code

Статус-код показывает результат обработки запроса.

Основные группы:

```text
1xx → информационные
2xx → успех
3xx → перенаправление
4xx → ошибка клиента
5xx → ошибка сервера
```

Примеры:

| Код | Значение              |
| --: | --------------------- |
| 200 | OK                    |
| 201 | Created               |
| 204 | No Content            |
| 301 | Moved Permanently     |
| 302 | Found                 |
| 400 | Bad Request           |
| 401 | Unauthorized          |
| 403 | Forbidden             |
| 404 | Not Found             |
| 409 | Conflict              |
| 422 | Unprocessable Content |
| 500 | Internal Server Error |
| 502 | Bad Gateway           |
| 503 | Service Unavailable   |

---

# 2️⃣ Response Headers

Сервер также отправляет заголовки.

Например:

```text
Content-Type: application/json
Content-Length: 42
Cache-Control: max-age=3600
Set-Cookie: session_id=abc123
```

---

# 3️⃣ Response Body

В теле ответа находятся данные.

Например:

```python
{
    "id": 1,
    "name": "Ilya"
}
```

Формат тела может быть разным:

* JSON;
* HTML;
* XML;
* текст;
* изображение;
* файл;
* бинарные данные.

---

# 🔧 HTTP Methods

Метод определяет, что клиент хочет сделать с ресурсом.

Основные:

```text
GET
POST
PUT
PATCH
DELETE
```

Например:

```text
GET    /users
POST   /users
GET    /users/10
PUT    /users/10
PATCH  /users/10
DELETE /users/10
```

Связь с CRUD:

```text
CREATE → POST
READ   → GET
UPDATE → PUT / PATCH
DELETE → DELETE
```

Методы подробно разбираются отдельной темой.

---

# 📍 URL

URL определяет, куда отправляется запрос.

Например:

```text
https://example.com:443/users/10?active=true
```

Можно разделить:

```text
https
  ↓
scheme

example.com
  ↓
host

443
  ↓
port

/users/10
  ↓
path

active=true
  ↓
query parameters
```

---

# ❓ Query Parameters

Query-параметры находятся после `?`.

Например:

```text
/users?limit=10&offset=20
```

Здесь:

```text
limit = 10
offset = 20
```

Они часто используются для:

* фильтрации;
* сортировки;
* пагинации;
* поиска.

Например:

```text
/products?category=laptop&sort=price
```

---

# 🧭 Path Parameters

Path-параметр является частью пути.

```text
/users/123
```

Здесь:

```text
123
```

может быть идентификатором пользователя.

Сравнение:

```text
/users/123
       ↑
       path parameter
```

```text
/users?id=123
       ↑
       query parameter
```

---

# 🏷️ Content-Type

`Content-Type` сообщает, какой формат данных находится в теле.

Например:

```text
Content-Type: application/json
```

означает JSON.

Другие варианты:

```text
text/html
text/plain
application/xml
multipart/form-data
application/x-www-form-urlencoded
```

---

# 📋 Accept

`Accept` сообщает серверу, какой формат ответа предпочитает клиент.

Например:

```text
Accept: application/json
```

То есть:

> «Я хочу получить JSON».

Важно:

```text
Content-Type
```

описывает **текущие данные**.

```text
Accept
```

описывает **желаемый формат ответа**.

---

# 🍪 Cookies

HTTP является **stateless** протоколом.

Это означает, что каждый запрос сам по себе не обязан содержать состояние предыдущих запросов.

Для хранения состояния могут использоваться cookies.

Сервер отправляет:

```text
Set-Cookie: session_id=abc123
```

Клиент затем отправляет:

```text
Cookie: session_id=abc123
```

Таким образом сервер может связать несколько запросов с одной сессией.

---

# 🔐 HTTP и HTTPS

HTTP:

```text
HTTP
 ↓
TCP
```

HTTPS:

```text
HTTP
 ↓
TLS
 ↓
TCP
```

То есть:

**HTTPS — это HTTP, защищённый TLS.**

HTTP сам по себе не предоставляет шифрование.

---

# 🧠 Stateless

HTTP называют **stateless**.

Это означает, что протокол не требует от сервера помнить состояние предыдущего запроса для обработки следующего.

Например:

```text
Request 1 → /users
Request 2 → /products
Request 3 → /orders
```

Каждый запрос содержит информацию, необходимую для его обработки, либо использует дополнительные механизмы состояния — например cookies, sessions или токены.

Важно:

**stateless ≠ сервер вообще не хранит состояние.**

Это означает, что само HTTP не требует встроенного состояния между запросами.

---

# 🔁 Persistent Connection

В HTTP/1.0 соединение исторически часто создавалось отдельно для каждого запроса.

HTTP/1.1 по умолчанию поддерживает **persistent connections**.

То есть одно TCP-соединение можно использовать для нескольких HTTP-запросов.

```text
TCP connection
 │
 ├── HTTP Request
 ├── HTTP Response
 ├── HTTP Request
 ├── HTTP Response
 └── ...
```

Это уменьшает накладные расходы на создание новых TCP-соединений.

---

# 🚦 HTTP/1.1, HTTP/2 и HTTP/3

### HTTP/1.1

```text
HTTP/1.1
   ↓
TCP
```

Текстовый протокол, persistent connections, chunked transfer и другие механизмы.

### HTTP/2

```text
HTTP/2
   ↓
TCP
```

Основные идеи:

* бинарные кадры;
* multiplexing;
* stream;
* header compression;
* несколько запросов через одно TCP-соединение.

### HTTP/3

```text
HTTP/3
   ↓
QUIC
   ↓
UDP
```

Основные идеи:

* QUIC;
* multiplexing без TCP head-of-line blocking между потоками;
* встроенная криптографическая защита через TLS 1.3.

---

# 🧩 HTTP в REST API

REST API обычно использует HTTP как транспортный протокол.

Например:

```text
GET    /users
POST   /users
GET    /users/10
PATCH  /users/10
DELETE /users/10
```

HTTP предоставляет:

```text
Методы
Статус-коды
Headers
Request
Response
```

А REST определяет архитектурные принципы взаимодействия с ресурсами.

Поэтому:

```text
HTTP ≠ REST
```

REST API может использовать HTTP, но HTTP сам по себе не является REST.

---

# 🎯 Главное для собеседования

Нужно уверенно знать структуру:

```text
HTTP Request
 ├── Method
 ├── URL
 ├── Headers
 └── Body

HTTP Response
 ├── Status Code
 ├── Headers
 └── Body
```

И понимать модель:

```text
Клиент
   │
   │ HTTP Request
   ↓
Сервер
   │
   │ HTTP Response
   ↓
Клиент
```

---

# 🧠 Формула для запоминания

**HTTP — протокол прикладного уровня для обмена данными между клиентом и сервером по модели request-response.**

**Request = method + URL + headers + body.**

**Response = status code + headers + body.**

**HTTP stateless по своей природе.**

**HTTPS = HTTP + TLS.**

**HTTP/1.1 и HTTP/2 обычно работают поверх TCP, HTTP/3 — поверх QUIC/UDP.**
