# 🤝 Three-Way Handshake

## 🎯 Ответ на собеседовании

**Three-Way Handshake** — это процесс установления TCP-соединения между клиентом и сервером.

Он состоит из **трёх этапов**:

```text
Client                    Server
  │                         │
  │ ─────── SYN ──────────→ │
  │                         │
  │ ←──── SYN + ACK ─────── │
  │                         │
  │ ─────── ACK ──────────→ │
  │                         │
  │      Connection         │
  │       established       │
```

После успешного трёхстороннего рукопожатия TCP-соединение считается установленным и стороны могут передавать данные.

---

## 🎤 Суперкоротко

**Three-Way Handshake = SYN → SYN-ACK → ACK.**

```text
1. Client → SYN
2. Server → SYN + ACK
3. Client → ACK
```

После этого TCP-соединение установлено.

---

# 1. 📤 SYN

Клиент хочет установить TCP-соединение и отправляет серверу пакет с флагом **SYN**.

```text
Client → Server
        SYN
```

Одновременно клиент отправляет свой **Initial Sequence Number (ISN)** — начальный номер последовательности.

Условно:

```text
Client:
Seq = 1000
SYN = 1
```

Клиент переходит в состояние:

```text
SYN-SENT
```

---

# 2. 📥 SYN + ACK

Сервер получает SYN и отвечает пакетом с двумя флагами:

```text
Server → Client
       SYN + ACK
```

Сервер:

* подтверждает получение SYN клиента;
* отправляет свой собственный SYN;
* сообщает свой начальный Sequence Number.

Например:

```text
Server:
Seq = 5000
Ack = 1001
SYN = 1
ACK = 1
```

Почему `Ack = 1001`?

Потому что **SYN занимает один номер последовательности**:

```text
Client Seq = 1000
SYN → занимает 1 номер
ACK = 1001
```

Сервер переходит в состояние:

```text
SYN-RECEIVED
```

---

# 3. 📤 ACK

Клиент получает SYN + ACK от сервера и отправляет подтверждение:

```text
Client → Server
        ACK
```

Например:

```text
Client:
Seq = 1001
Ack = 5001
ACK = 1
```

Сервер получает ACK и TCP-соединение устанавливается.

```text
ESTABLISHED
```

---

# 🔢 Что происходит с Sequence Number

Допустим:

```text
Client ISN = 1000
Server ISN = 5000
```

Обмен:

```text
Client → Server
SYN
Seq = 1000

Server → Client
SYN + ACK
Seq = 5000
Ack = 1001

Client → Server
ACK
Seq = 1001
Ack = 5001
```

После этого обе стороны знают начальные номера последовательности друг друга.

---

# 🧩 Зачем нужны три шага

Главная задача — **синхронизировать состояние TCP между двумя сторонами**.

После handshake:

```text
Client                       Server
  │                            │
  │ знает Seq сервера           │
  │ знает Seq клиента           │
  │                            │
  └──── TCP connection ────────┘
```

Обе стороны подтверждают:

1. клиент доступен;
2. сервер доступен;
3. обе стороны согласовали начальные Sequence Numbers.

---

# 🌐 Пример с HTTP

Когда клиент обращается к HTTP/HTTPS-серверу через TCP, сначала устанавливается TCP-соединение:

```text
Client
  │
  │ SYN
  ↓
Server
  │
  │ SYN + ACK
  ↓
Client
  │
  │ ACK
  ↓
TCP connection established
  │
  │ HTTP request
  ↓
Server
```

То есть HTTP-запрос передаётся **после установления TCP-соединения**.

Для HTTPS дополнительно выполняется **TLS handshake**.

```text
TCP Three-Way Handshake
        ↓
TLS Handshake
        ↓
HTTPS data
```

---

# ⚠️ Частая ошибка

Не стоит говорить:

> Three-Way Handshake нужен для передачи данных.

Правильнее:

> **Three-Way Handshake используется для установления TCP-соединения и синхронизации Sequence Numbers между клиентом и сервером.**

---

# 🆚 TCP vs UDP

### TCP

Перед передачей данных устанавливает соединение:

```text
SYN
 ↓
SYN-ACK
 ↓
ACK
 ↓
Data
```

### UDP

Не устанавливает соединение таким способом:

```text
Data → Server
```

Поэтому UDP не использует Three-Way Handshake.

---

# 🧠 Главное

```text
Three-Way Handshake

1. SYN
   Client → Server

2. SYN + ACK
   Server → Client

3. ACK
   Client → Server

        ↓

TCP connection established
```

**SYN** — клиент хочет установить соединение.

**SYN + ACK** — сервер согласен и подтверждает SYN клиента.

**ACK** — клиент подтверждает SYN сервера.

### Формула

```text
SYN → SYN-ACK → ACK
```

**Three-Way Handshake = трёхэтапное установление TCP-соединения с синхронизацией начальных Sequence Numbers.**
