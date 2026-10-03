# 🔹 gRPC

## 🎯 Ответ на собеседовании

**gRPC** — это высокопроизводительный RPC-фреймворк для взаимодействия между сервисами. Он использует **HTTP/2** в качестве транспорта и обычно **Protocol Buffers (Protobuf)** для описания API и сериализации данных.

В микросервисной архитектуре gRPC часто используется для быстрого взаимодействия между внутренними сервисами.

Например:

```text
Order Service
      │
      │ gRPC
      ↓
Payment Service
```

API заранее описывается в `.proto`-файле, после чего генерируется клиентский и серверный код.

---

## 🎤 Суперкоротко

**gRPC — это RPC-фреймворк для быстрого взаимодействия между сервисами, использующий HTTP/2 и обычно Protobuf для сериализации и описания API.**

---

## 🧩 Что означает RPC

**RPC — Remote Procedure Call**, удалённый вызов процедуры.

Идея:

```python
payment_service.pay(order_id)
```

Выглядит как обычный вызов функции, но фактически запрос отправляется по сети в другой сервис.

```text
Client
  │
  │ pay(order_id)
  ↓
Network
  ↓
Payment Service
  │
  ↓
pay(order_id)
```

---

## 📦 Protocol Buffers

**Protocol Buffers (Protobuf)** — бинарный формат сериализации данных и язык описания структуры сообщений/API.

Пример `.proto`:

```python
syntax = "proto3";

service PaymentService {
    rpc CreatePayment (PaymentRequest) returns (PaymentResponse);
}

message PaymentRequest {
    int64 order_id = 1;
    double amount = 2;
}

message PaymentResponse {
    int64 payment_id = 1;
    string status = 2;
}
```

Из `.proto` можно сгенерировать код для клиента и сервера.

---

## ⚙️ Как работает gRPC

Упрощённо:

```text
.proto
  ↓
Protobuf compiler
  ↓
Generated client/server code
  ↓
Client
  ↓
HTTP/2
  ↓
Server
```

Клиент вызывает удалённый метод:

```python
response = client.CreatePayment(
    PaymentRequest(
        order_id=123,
        amount=1000
    )
)
```

gRPC сериализует запрос в Protobuf и отправляет его по HTTP/2.

Сервер десериализует запрос, выполняет метод и отправляет ответ обратно.

---

## 🚀 Почему gRPC быстрый

Основные причины:

### 1. HTTP/2

gRPC использует возможности HTTP/2:

* multiplexing;
* binary framing;
* header compression;
* persistent connections;
* streaming.

### 2. Protobuf

Protobuf использует компактное бинарное представление данных.

По сравнению с JSON:

```text
JSON
{
    "order_id": 123,
    "amount": 1000
}
```

Protobuf передаёт данные в бинарном формате.

Это обычно означает:

* меньший размер сообщений;
* более быструю сериализацию/десериализацию;
* меньший сетевой overhead.

---

## 🔄 Виды gRPC-вызовов

gRPC поддерживает **4 типа RPC**.

### 1. Unary RPC

Обычный запрос → обычный ответ.

```text
Client ───── Request ─────→ Server
Client ←──── Response ───── Server
```

Пример:

```python
rpc GetUser (GetUserRequest) returns (User);
```

Аналог обычного HTTP request/response.

---

### 2. Server Streaming

Клиент отправляет один запрос, сервер отправляет поток ответов.

```text
Client ───── Request ─────→ Server

Client ←──── Response ──── Server
Client ←──── Response ──── Server
Client ←──── Response ──── Server
```

Пример:

```python
rpc ListUsers (ListUsersRequest) returns (stream User);
```

---

### 3. Client Streaming

Клиент отправляет поток сообщений, сервер возвращает один ответ.

```text
Client ───── Request ─────→ Server
Client ───── Request ─────→ Server
Client ───── Request ─────→ Server
                           ↓
Client ←──── Response ───── Server
```

Пример:

```python
rpc UploadData (stream Data) returns (UploadResult);
```

---

### 4. Bidirectional Streaming

Обе стороны могут отправлять поток сообщений независимо друг от друга.

```text
Client ───── Request ─────→ Server
Client ←──── Response ───── Server
Client ───── Request ─────→ Server
Client ←──── Response ───── Server
```

Пример:

```python
rpc Chat (stream Message) returns (stream Message);
```

---

## 🌐 gRPC и HTTP

Важно не путать:

```text
gRPC
  ↓
RPC framework
  ↓
использует HTTP/2
```

То есть:

**HTTP/2 — транспортный протокол, а gRPC — фреймворк удалённого взаимодействия.**

---

## 🆚 gRPC vs REST

| gRPC                         | REST                                     |
| ---------------------------- | ---------------------------------------- |
| RPC-подход                   | Ресурсный подход                         |
| Обычно Protobuf              | Обычно JSON                              |
| HTTP/2                       | Часто HTTP/1.1 или HTTP/2                |
| Бинарная сериализация        | Текстовый JSON                           |
| Высокая производительность   | Простота и широкая совместимость         |
| Хорош для service-to-service | Хорош для публичных API                  |
| Поддерживает streaming       | Streaming возможен, но подход отличается |
| Строгий контракт `.proto`    | Контракт обычно OpenAPI                  |

---

## 🔌 Где используют gRPC

Частый сценарий:

```text
                Internet
                   │
                   ↓
              API Gateway
                   │
             ┌─────┴─────┐
             ↓           ↓
        User Service   Order Service
                           │
                         gRPC
                           ↓
                     Payment Service
```

Например:

* frontend → REST/HTTP;
* API Gateway → REST;
* внутренние сервисы → gRPC.

Это распространённая комбинация.

---

## 🐍 gRPC и Python

Для Python существует библиотека `grpcio`.

Типичный процесс:

```text
payment.proto
      ↓
protoc
      ↓
payment_pb2.py
payment_pb2_grpc.py
      ↓
Python client/server
```

`payment_pb2.py` содержит сгенерированные Protobuf-сообщения.

`payment_pb2_grpc.py` содержит сгенерированный gRPC-код для клиента и сервера.

---

## 🔐 Безопасность

gRPC может работать поверх **TLS**.

```text
Client
   │
   │ gRPC + TLS
   ↓
Server
```

TLS обеспечивает:

* шифрование трафика;
* целостность;
* аутентификацию сервера.

Также gRPC поддерживает различные механизмы аутентификации через credentials/interceptors.

---

## ⚠️ Недостатки gRPC

### 1. Сложнее для браузера

Обычный gRPC не так удобен для непосредственного использования из браузера.

Для браузерных клиентов часто используется **gRPC-Web**.

### 2. Protobuf сложнее JSON

JSON легко посмотреть:

```python
{
    "id": 123,
    "name": "Ilya"
}
```

Бинарный Protobuf менее удобен для ручной отладки.

### 3. Требует генерации кода

Нужно работать с:

```text
.proto
   ↓
protoc
   ↓
generated code
```

### 4. Не всегда нужен

Для небольшого публичного API использование gRPC может добавить ненужную сложность.

---

## 🧠 gRPC в микросервисах

Один из типичных вариантов:

```text
             API Gateway
                  │
        ┌─────────┴─────────┐
        ↓                   ↓
   User Service        Order Service
                            │
                           gRPC
                            ↓
                     Payment Service
                            │
                           gRPC
                            ↓
                    Notification Service
```

Преимущества:

* строгий контракт;
* высокая производительность;
* компактные сообщения;
* streaming;
* хорошо подходит для service-to-service communication.

---

## 🎯 Что важно сказать на собеседовании

Если спрашивают **«Что такое gRPC?»**:

> **gRPC — это RPC-фреймворк для взаимодействия между сервисами. Он использует HTTP/2 и обычно Protocol Buffers для сериализации данных и описания API. gRPC поддерживает unary и streaming-вызовы и хорошо подходит для высокопроизводительного взаимодействия между микросервисами.**

Если спрашивают **«Почему gRPC может быть быстрее REST?»**:

> **За счёт HTTP/2 и бинарной сериализации Protobuf. HTTP/2 поддерживает multiplexing и streaming, а Protobuf обычно передаёт более компактные сообщения и быстрее сериализуется, чем текстовый JSON.**

---

## 💡 Главное

```text
gRPC
 │
 ├── RPC framework
 │
 ├── HTTP/2
 │
 ├── Protobuf
 │
 ├── строгий контракт .proto
 │
 ├── code generation
 │
 └── streaming
       ├── Unary
       ├── Server Streaming
       ├── Client Streaming
       └── Bidirectional Streaming
```

### Формула для собеседования

**gRPC = RPC + HTTP/2 + Protobuf + строгий контракт + streaming**

Главный сценарий:

**микросервис → gRPC → микросервис**
