# Retention в Kafka 📦

## 🎯 Ответ на собеседовании

**Retention** — политика хранения сообщений в Kafka.

В отличие от классической очереди, Kafka **не удаляет сообщение только потому, что consumer его прочитал или сделал commit offset**.

Сообщения хранятся в partition в соответствии с настройками retention, а затем Kafka удаляет устаревшие данные.

Например:

```text id="q8m4xp"
Producer
   ↓
Kafka Topic
   ↓
Partition
   ↓
Consumer
   ↓
Commit Offset

        ↓

Сообщение всё ещё находится в Kafka
        ↓
Удаляется только после выполнения
retention policy
```

> **Чтение и commit offset не означают удаление сообщения из Kafka.**

---

## 🎤 Суперкоротко

```text id="v5k9mz"
Kafka

Message
   ↓
Partition
   ↓
Consumer читает
   ↓
Commit offset
   ↓
Message остаётся
   ↓
Retention
   ↓
Удаление
```

**Retention определяет, сколько Kafka хранит данные.**

---

# 1. Зачем Kafka хранит прочитанные сообщения?

Главное преимущество — возможность **повторного чтения событий**.

Например:

```text id="n7c3qx"
10:00 → OrderCreated
10:01 → PaymentCreated
10:02 → OrderShipped
```

Consumer обработал события.

Но сообщения остаются:

```text id="r4m8vk"
Kafka
├── OrderCreated
├── PaymentCreated
└── OrderShipped
```

Другой consumer может прочитать их позже.

Например:

```text id="x8q2mp"
orders topic
       │
       ├──→ Billing Group
       │
       ├──→ Analytics Group
       │
       └──→ New Service
```

Новый consumer может начать чтение доступной истории.

---

# 2. Retention по времени ⏱️

Kafka может хранить сообщения определённое количество времени.

Например:

```text id="m5v9cz"
retention = 7 дней
```

Сообщение:

```text id="k3x7qp"
1 сентября
```

может храниться примерно до:

```text id="p8n4mw"
8 сентября
```

После чего Kafka сможет удалить его согласно политике хранения.

Важно:

> **Retention — это не точный таймер удаления каждого сообщения.** Kafka работает с сегментами логов и периодически удаляет устаревшие сегменты.

---

# 3. Retention по размеру 💾

Можно ограничивать не только время хранения, но и размер данных.

Например:

```text id="z6q2vn"
Topic
↓
максимальный размер = 100 GB
```

Когда объём превышает заданные ограничения, старые данные могут удаляться согласно политике retention.

Таким образом Kafka может ограничиваться:

```text id="w4m8xp"
временем
+
размером
```

---

# 4. Retention не зависит от Consumer

Это принципиально.

Допустим:

```text id="j7q3kc"
Consumer A
прочитал сообщение

Consumer B
ещё не прочитал

Consumer C
вообще не существует
```

Kafka не удаляет сообщение просто потому, что Consumer A его прочитал.

```text id="u5x9mq"
Message
  │
  ├── Consumer A ✓
  ├── Consumer B ?
  └── Consumer C ?
```

Сообщение продолжает существовать, пока оно доступно по retention policy.

---

# 5. Retention и Consumer Group

Представим:

```text id="c8m4zr"
Kafka
  ↓
orders
  ↓
Group A
```

Group A прочитала:

```text id="n2q7vp"
offset 100
```

Но Kafka продолжает хранить сообщения:

```text id="s5k9xm"
offset 0
offset 1
offset 2
...
offset 100
...
```

до момента, когда они попадут под retention.

Поэтому Consumer Group может **отставать от Producer**, но пока нужные данные ещё существуют, consumer может догнать поток.

---

# 6. Что если Consumer долго не работал? 💤

Допустим:

```text id="x3v7qn"
Retention = 7 дней
```

Consumer остановился на:

```text id="m8k2zp"
offset 100
```

За это время Kafka продолжила получать данные.

Через 10 дней старые сообщения могли быть удалены.

Получается:

```text id="q6n4yc"
Consumer хочет читать
с offset 100

Но:

offset 100 уже удалён
```

Consumer больше не может прочитать эту часть истории.

Тогда важным становится:

```text id="a7p3mk"
auto.offset.reset
```

Например:

```text id="v9x5qz"
earliest
latest
```

Это напрямую связывает **Retention и Offset**.

---

# 7. Retention + Offset + Replay 🔄

Эти три понятия лучше понимать вместе:

```text id="k4m8vp"
Retention
    ↓
сколько хранится история
    ↓
Offset
    ↓
где находится Consumer
    ↓
Replay
    ↓
можно ли перечитать историю
```

Например:

```text id="z5q7mx"
Kafka хранит 30 дней

Consumer:
offset = 1000

Можно вернуть consumer
на более ранний доступный offset
и перечитать события.
```

Но если событие уже удалено retention policy:

```text id="n8c3vy"
Replay ❌
```

---

# 8. Retention и RabbitMQ 🐇

Это одно из самых важных сравнений.

### RabbitMQ

Упрощённо:

```text id="p7m2xk"
Producer
   ↓
Queue
   ↓
Consumer
   ↓
ACK
   ↓
Message удаляется
```

### Kafka

```text id="r4v8qn"
Producer
   ↓
Topic
   ↓
Consumer
   ↓
Commit Offset
   ↓
Message остаётся
   ↓
Retention
   ↓
Message удаляется
```

Главная разница:

> **RabbitMQ ориентирован на доставку сообщения, Kafka — на хранение и чтение потока событий.**

---

# 9. Retention ≠ ACK

Очень важно не путать.

В RabbitMQ:

```text id="y3k8mc"
ACK
↓
сообщение подтверждено
↓
может быть удалено
```

В Kafka:

```text id="q6p4vz"
Commit Offset
↓
зафиксирован прогресс consumer group
↓
сообщение НЕ удаляется
```

То есть:

```text id="x8m2kp"
ACK
→ подтверждение обработки

Commit Offset
→ сохранение позиции чтения
```

Это совершенно разные механизмы.

---

# 10. Retention и Consumer Lag 📊

Представим:

```text id="m5q9vx"
Producer
  ↓
Kafka
  ↓
Consumer
```

Consumer медленный:

```text id="j7c3mk"
Lag ↑
```

Пока сообщения не вышли за retention:

```text id="r8n4qp"
Lag ↑
Consumer догоняет
       ↓
обрабатывает старые сообщения
```

Но если consumer слишком сильно отстал:

```text id="z2x7vn"
Lag ↑↑↑
       ↓
старые сообщения удаляются
       ↓
Consumer не может прочитать
часть истории
```

Поэтому при проектировании системы важно учитывать:

```text id="p4m8yc"
Consumer throughput
+
Maximum expected downtime
+
Retention period
```

---

# 11. Retention и большие данные 💾

Kafka может хранить значительные объёмы событий.

Например:

```text id="k8q3mz"
10 MB/s
×
86 400 секунд
≈
864 GB/day
```

Если хранить неделю:

```text id="v5n7xp"
≈ 6 TB
```

Поэтому retention напрямую связан с:

* дисковым пространством;
* количеством partitions;
* replication factor;
* стоимостью инфраструктуры.

---

# 12. Retention и Replication

Kafka обычно хранит несколько копий partition.

Например:

```text id="q3m8vz"
Replication Factor = 3

Partition 0
├── Broker 1
├── Broker 2
└── Broker 3
```

Retention удаляет старые данные из логов, но при этом Kafka должна поддерживать согласованное состояние реплик.

Поэтому увеличение retention влияет не только на объём данных, но и на общий объём дискового пространства с учётом репликации.

Упрощённо:

```text id="x7k4pn"
Data
 ×
Replication Factor
 =
необходимое хранилище
```

---

# 13. Log Compaction 🧹

Отдельная важная концепция Kafka — **Log Compaction**.

Обычный retention отвечает на вопрос:

> **Когда удалить старые сообщения?**

Compaction отвечает на другой вопрос:

> **Какие записи с одинаковым key можно сохранить как актуальное состояние?**

Например:

```text id="m6q2vz"
key = user:123

v1 → name = Ilya
v2 → name = Alex
v3 → name = Ivan
```

После compaction Kafka может сохранить последнюю актуальную запись:

```text id="k8x4qp"
user:123 → name = Ivan
```

Это позволяет использовать Kafka не только как поток событий, но и как журнал текущего состояния.

---

# 14. Retention vs Log Compaction

|               | Retention             | Log Compaction                     |
| ------------- | --------------------- | ---------------------------------- |
| Основная идея | Удалять старые данные | Сохранять последнее значение key   |
| Критерий      | Время / размер        | Key                                |
| История       | Постепенно удаляется  | Может сохраняться последняя версия |
| Использование | Event streams         | State / snapshots / changelog      |

Например:

```text id="z4n7mc"
Retention:

A → B → C → D → старое удаляется


Compaction:

user:1 → A
user:1 → B
user:1 → C

↓

user:1 → C
```

---

# 15. Tombstone 🪦

В compacted topic специальное значение может использоваться для удаления key.

Например:

```text id="r5m8xq"
user:123 → null
```

Такая запись называется **tombstone**.

Она сообщает:

> Состояние с этим key нужно удалить.

После соответствующей compaction Kafka сможет удалить старую запись для этого key.

---

# 16. Практический Backend-пример

Представим сервис заказов:

```text id="v7q3mk"
Order Service
     ↓
Kafka
     ↓
orders topic
```

Настройка:

```text id="j4x8pn"
retention = 7 days
```

События:

```text id="m9c2vz"
OrderCreated
OrderPaid
OrderShipped
```

Analytics Service может обработать их сегодня.

Через три дня можно запустить новую Consumer Group:

```text id="x6k4qp"
analytics-v2
```

и прочитать доступную историю.

Это возможно именно потому, что:

```text id="z8m3vn"
Kafka
не удаляет событие
после чтения.
```

---

# ⚠️ Частые ошибки на собеседовании

### ❌ «Kafka хранит сообщение, пока его не прочитает consumer»

Нет.

Сообщение хранится согласно **retention policy**, независимо от того, прочитал его consumer или нет.

---

### ❌ «Commit offset удаляет сообщение»

Нет.

```text id="c5q9mx"
Commit
→ сохраняет прогресс

Retention
→ определяет срок хранения
```

---

### ❌ «Retention — это TTL каждого сообщения»

Упрощённо можно так объяснять, но технически Kafka работает с **log segments**, поэтому удаление происходит сегментами и не обязательно ровно в момент истечения возраста каждого отдельного сообщения.

---

### ❌ «Retention и Compaction — одно и то же»

Нет.

```text id="n7x4kp"
Retention
→ время / размер

Compaction
→ key / последнее значение
```

---

# 🧠 Шпаргалка

```text id="w8m3qz"
Retention
→ политика хранения данных

Time Retention
→ хранить N времени

Size Retention
→ хранить до определённого объёма

Commit Offset
→ прогресс Consumer Group

Retention
→ срок жизни данных

Replay
→ повторное чтение доступной истории

Log Compaction
→ сохранять последнее значение по key

Tombstone
→ удалить состояние key в compacted topic
```

### Главная цепочка

```text id="q4k8vn"
Producer
   ↓
Kafka Topic
   ↓
Partition
   ↓
Message + Offset
   ↓
Consumer
   ↓
Commit Offset
   ↓
Message остаётся
   ↓
Retention / Compaction
   ↓
Удаление или уплотнение
```

> **Kafka отделяет обработку сообщения от его хранения: consumer commit'ит offset, а Kafka продолжает хранить данные согласно retention policy. Именно это позволяет делать replay и независимо подключать новые Consumer Groups.**
