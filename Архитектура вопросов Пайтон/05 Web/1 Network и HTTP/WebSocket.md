# 🔌 WebSocket

## 🎯 Ответ на собеседовании

**WebSocket** — это протокол для организации **постоянного двустороннего соединения** между клиентом и сервером.

В отличие от обычного HTTP, где клиент отправляет запрос и получает ответ, WebSocket позволяет **обеим сторонам отправлять сообщения в любой момент времени** после установления соединения.

Упрощённо:

```text
HTTP:

Client → Request → Server
Client ← Response ← Server


WebSocket:

Client ←──────────────→ Server
        постоянное
        соединение
```

WebSocket особенно полезен для:

* чатов;
* онлайн-уведомлений;
* realtime-данных;
* онлайн-игр;
* биржевых котировок;
* совместного редактирования;
* отслеживания статуса в реальном времени.

---

## 🎤 Суперкоротко

```text
HTTP:
Request → Response

WebSocket:
постоянное соединение
Client ←→ Server
```

Главное:

> **WebSocket позволяет клиенту и серверу обмениваться данными в обе стороны без постоянного создания новых HTTP-запросов.**

---

# 🌐 Чем WebSocket отличается от HTTP

### HTTP

Клиент инициирует взаимодействие:

```text
Client → GET /messages
Server → Response

Client → GET /messages
Server → Response

Client → GET /messages
Server → Response
```

Если серверу нужно сообщить клиенту что-то новое, обычный HTTP сам по себе этого не позволяет — клиент должен снова обратиться к серверу.

---

### WebSocket

После установки соединения:

```text
Client ←→ Server
```

Сервер может самостоятельно отправить сообщение клиенту:

```text
Server → New message
```

И клиент может отправить сообщение серверу:

```text
Client → Hello
```

Соединение при этом остаётся открытым.

---

# 🤝 WebSocket Handshake

WebSocket-соединение начинается с HTTP-запроса.

Клиент отправляет специальный HTTP-запрос на установление WebSocket-соединения:

```text
GET /chat HTTP/1.1
Host: example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: ...
Sec-WebSocket-Version: 13
```

Сервер отвечает:

```text
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
```

После этого соединение переключается с HTTP на WebSocket.

```text
HTTP
 ↓
Handshake
 ↓
101 Switching Protocols
 ↓
WebSocket connection
```

---

# 🔄 Как работает WebSocket

Полный жизненный цикл:

```text
1. Client
      ↓
2. HTTP Handshake
      ↓
3. Server подтверждает upgrade
      ↓
4. WebSocket connection
      ↓
5. Обмен сообщениями
      ↕
   Client ↔ Server
      ↓
6. Закрытие соединения
```

---

# 📡 Двусторонняя связь

WebSocket предоставляет **full-duplex communication**.

Это означает, что обе стороны могут одновременно отправлять данные.

```text
Client                    Server

   ─────── message ───────→

   ←────── message ────────

   ─────── message ───────→

   ←────── message ────────
```

При этом серверу не нужно ждать нового HTTP-запроса от клиента.

---

# 💬 Пример: чат

Без WebSocket:

```text
Client → "Есть новые сообщения?"
Server → "Нет"

Client → "Есть новые сообщения?"
Server → "Да, вот сообщение"

Client → "Есть новые сообщения?"
Server → "Нет"
```

Это может быть реализовано через polling.

С WebSocket:

```text
Client ←──────────────→ Server
          connection

Server → "Новое сообщение"
```

Сервер сразу отправляет событие клиенту.

---

# 🆚 WebSocket и HTTP Polling

**Polling**:

```text
Client → Request
Server → Response

Client → Request
Server → Response

Client → Request
Server → Response
```

Проблема — множество лишних запросов.

**WebSocket**:

```text
Client ←────────→ Server
       connection

       message
       ←──────
```

Для realtime-сценариев WebSocket обычно эффективнее.

---

# ⚡ WebSocket и SSE

Ещё одна технология realtime — **Server-Sent Events (SSE)**.

| WebSocket                            | SSE                                   |
| ------------------------------------ | ------------------------------------- |
| Двусторонняя связь                   | Сервер → клиент                       |
| Client ↔ Server                      | Server → Client                       |
| Поддерживает сообщения в обе стороны | Клиент отправляет данные обычным HTTP |
| Чаты, игры, realtime-интерактив      | Уведомления, ленты событий            |
| WebSocket-протокол                   | HTTP                                  |

Если клиенту нужно только получать события от сервера, SSE может быть проще.

---

# 🐍 WebSocket в FastAPI

FastAPI поддерживает WebSocket.

Пример:

```python
from fastapi import FastAPI, WebSocket

app = FastAPI()


@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()

    while True:
        message = await websocket.receive_text()
        await websocket.send_text(f"Server: {message}")
```

Здесь:

```text
websocket.accept()
```

принимает соединение.

```text
websocket.receive_text()
```

получает сообщение от клиента.

```text
websocket.send_text()
```

отправляет сообщение клиенту.

---

# 🔌 Почему WebSocket связан с ASGI

WebSocket — один из важных сценариев для **ASGI**.

WSGI ориентирован прежде всего на классическую модель HTTP:

```text
Request → Response
```

ASGI поддерживает событийную модель:

```text
HTTP
WebSocket
другие типы событий
```

Например:

```text
Client
   ↓
Uvicorn
   ↓
ASGI
   ↓
FastAPI
   ↓
WebSocket
```

---

# 🧩 WebSocket Events

В ASGI взаимодействие происходит через события.

Упрощённо:

```text
Client
   ↓
websocket.connect
   ↓
Application

Client
   ↓
websocket.receive
   ↓
Application

Application
   ↓
websocket.send
   ↓
Client

Client
   ↓
websocket.disconnect
```

Поэтому WebSocket хорошо сочетается с асинхронной моделью ASGI.

---

# 🔐 WebSocket и безопасность

WebSocket может использовать защищённое соединение:

```text
ws://
```

или:

```text
wss://
```

`wss://` — WebSocket поверх TLS.

Аналогия:

```text
http://  → https://
ws://    → wss://
```

Для production обычно используется `wss://`.

---

# 📦 Формат данных

WebSocket может передавать:

* текст;
* бинарные данные.

Например:

```python
await websocket.send_text("Hello")
```

или:

```python
await websocket.send_bytes(data)
```

Часто поверх WebSocket передают JSON:

```python
await websocket.send_json(
    {
        "type": "message",
        "text": "Hello",
    }
)
```

Важно:

> **WebSocket — это транспорт, а JSON — формат данных.**

---

# ⚠️ WebSocket ≠ HTTP

Несмотря на то что WebSocket начинается с HTTP handshake, после переключения протокол становится другим.

```text
HTTP request
     ↓
Handshake
     ↓
101 Switching Protocols
     ↓
WebSocket
```

Поэтому нельзя говорить:

> «WebSocket — это просто постоянный HTTP-запрос».

Правильнее:

> **WebSocket использует HTTP для первоначального handshake, после которого устанавливается отдельное постоянное WebSocket-соединение.**

---

# 🆚 HTTP vs WebSocket

| HTTP                     | WebSocket                              |
| ------------------------ | -------------------------------------- |
| Request → Response       | Двусторонний обмен                     |
| Обычно короткие запросы  | Долгоживущее соединение                |
| Клиент инициирует запрос | Обе стороны могут отправлять сообщения |
| Подходит для REST API    | Подходит для realtime                  |
| HTTP/1.1, HTTP/2, HTTP/3 | Отдельный WebSocket-протокол           |
| Простая модель           | Событийная модель                      |

---

# 🎯 Главное

```text
WebSocket
↓
HTTP Handshake
↓
101 Switching Protocols
↓
постоянное соединение
↓
Client ←→ Server
```

Запомнить:

```text
HTTP
→ запросил
→ получил ответ
→ соединение может завершиться

WebSocket
→ установил соединение
→ соединение остаётся открытым
→ клиент и сервер могут отправлять сообщения
```

```text
ws://  → WebSocket
wss:// → WebSocket + TLS
```

### Формула для собеседования

> **WebSocket — протокол для постоянного двустороннего обмена данными между клиентом и сервером. Соединение устанавливается через HTTP handshake, после чего обе стороны могут отправлять сообщения независимо друг от друга.**
