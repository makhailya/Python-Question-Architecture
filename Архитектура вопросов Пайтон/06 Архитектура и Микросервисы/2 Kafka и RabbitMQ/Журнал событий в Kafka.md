# Журнал событий в Kafka 📝

## 🎯 Ответ на собеседовании

**Kafka хранит сообщения в виде последовательного append-only журнала событий (log).**

Каждый **partition** представляет собой отдельный упорядоченный log:

```text id="j8q2m4"
Partition 0

offset:
  0 → Event A
  1 → Event B
  2 → Event C
  3 → Event D
  4 → Event E
```

Новые события **добавляются в конец** журнала, а уже записанные сообщения обычно не изменяются.

Kafka не удаляет сообщение сразу после того, как consumer его прочитал. Сообщения хранятся согласно политике **retention**.

---

## 🎤 Суперкоротко

> Kafka использует append-only log. Каждый partition — независимый упорядоченный журнал событий, где каждому сообщению соответствует offset. Consumer читает журнал по offset, а сообщения удаляются не после чтения, а согласно retention policy.

---

# 1. Что такое Log в Kafka?

В Kafka **log** — это последовательность записей, которая хранится внутри partition.

Например:

```text id="f4n7k2"
Topic: orders

Partition 0:

0  OrderCreated
1  PaymentStarted
2  PaymentCompleted
3  OrderConfirmed
4  OrderDelivered
```

Это и есть журнал событий.

Новые записи добавляются:

```text id="w2m8q5"
0
1
2
3
4
↓
5 ← новое событие
```

---

# 2. Append-only

Kafka использует модель **append-only log**.

Это означает:

> Новые записи добавляются в конец журнала.

Например:

```text id="k6r1p9"
До:

0 → A
1 → B
2 → C
```

Добавляем событие:

```text id="v3n8q2"
3 → D
```

Получаем:

```text id="s5m2x7"
0 → A
1 → B
2 → C
3 → D
```

Kafka не требует изменения существующих записей для добавления нового события.

---

# 3. Partition — это отдельный Log

Это ключевой момент.

Topic сам по себе не является одним физическим журналом.

Например:

```text id="q7p3m8"
Topic orders

Partition 0:
0 → A
1 → B
2 → C

Partition 1:
0 → D
1 → E
2 → F

Partition 2:
0 → G
1 → H
2 → I
```

У каждого partition:

* свой log;
* свои offsets;
* свой порядок сообщений.

---

# 4. Offset

Каждая запись внутри partition имеет **offset**.

```text id="r4k9x2"
Partition 0

Offset    Event
  0       OrderCreated
  1       PaymentStarted
  2       PaymentCompleted
  3       OrderConfirmed
```

Offset определяет положение записи в конкретном partition.

Важно:

```text id="u8m2q5"
(topic, partition, offset)
```

можно рассматривать как уникальную позицию записи в Kafka.

---

# 5. Offset не является глобальным ID

Например:

```text id="c3n7v1"
Partition 0:
0 → A
1 → B
2 → C

Partition 1:
0 → D
1 → E
2 → F
```

Offset `0` существует в обоих partitions.

Поэтому:

```text id="a9p4k6"
P0 + offset 0
```

и:

```text id="b2x8m3"
P1 + offset 0
```

— разные записи.

---

# 6. Порядок событий

Kafka гарантирует порядок **внутри partition**.

Например:

```text id="g6q1r8"
Partition 0:

0 → A
1 → B
2 → C
3 → D
```

Consumer прочитает их в этом порядке:

```text id="h5m9v2"
A → B → C → D
```

Но между разными partitions глобального порядка нет:

```text id="p7k3x1"
P0: A → C → E

P1: B → D → F
```

Kafka не гарантирует:

```text id="z4n8q6"
A → B → C → D → E → F
```

---

# 7. Как сообщения попадают в partition?

Producer отправляет сообщение в topic.

Kafka выбирает partition.

Упрощённо:

```text id="m2r7k4"
Producer
   ↓
Topic
   ↓
Partition
```

Если указан key:

```python id="v8q3n5"
producer.send(
    "orders",
    key="order-123",
    value="OrderCreated",
)
```

partitioner использует key для выбора partition.

Это позволяет связанные события направлять в один partition.

---

# 8. Почему это важно для порядка?

Допустим:

```text id="n6p2x8"
order_id = 123
```

Есть события:

```text id="y4m7q1"
OrderCreated
PaymentStarted
PaymentCompleted
OrderConfirmed
```

Если они попали в один partition:

```text id="c9r3k5"
P2:

0 → OrderCreated
1 → PaymentStarted
2 → PaymentCompleted
3 → OrderConfirmed
```

Порядок сохраняется.

---

# 9. Consumer читает Log

Consumer не «забирает» сообщение из Kafka в классическом смысле очереди.

Он читает записи по их offset.

```text id="e8q2m6"
Kafka:

0 → A
1 → B
2 → C
3 → D
4 → E

Consumer
   ↓
читает 0
читает 1
читает 2
```

После обработки consumer может сохранить свой offset.

```text id="k5r9p3"
Consumer Group offset = 3
```

---

# 10. Чтение не удаляет сообщение

Это одно из главных отличий Kafka от классической очереди.

```text id="q3m8x1"
Kafka Log:

0 → A
1 → B
2 → C
3 → D
```

Consumer прочитал:

```text id="u7n4p6"
A
B
C
```

Но Kafka не удаляет их просто потому, что они были прочитаны.

Они продолжают находиться в log до удаления согласно retention.

---

# 11. Replay

Поскольку сообщения остаются в log, consumer может **прочитать их повторно**.

Это называется **replay**.

Например:

```text id="a6q2m9"
Kafka Log

0 → A
1 → B
2 → C
3 → D
```

Consumer сначала обработал:

```text id="x8r4k1"
A → B → C
```

Позже можно изменить offset и снова прочитать:

```text id="p5m7v3"
A → B → C → D
```

Это очень важное преимущество Kafka.

---

# 12. Зачем нужен Replay?

Например, появился новый сервис аналитики.

До этого события уже были:

```text id="j3n9q5"
OrderCreated
PaymentCompleted
OrderDelivered
```

Новый consumer может прочитать исторические события и построить свою базу аналитики.

```text id="w6k2m8"
Kafka Log
   ↓
historical events
   ↓
Analytics Service
```

Не обязательно генерировать события заново.

---

# 13. Retention

Kafka хранит события согласно **retention policy**.

Например:

```text id="r8p3v6"
retention = 7 days
```

Упрощённо:

```text id="m4q9x2"
День 1 → Event A
День 2 → Event B
...
День 7 → Event G
День 8 → старые данные начинают удаляться
```

Важно:

> Retention не зависит напрямую от того, прочитал consumer сообщение или нет.

---

# 14. Consumer может отстать

Например:

```text id="k1v7n4"
Kafka:
1 000 000 events

Consumer:
обработал 800 000
```

Consumer отстаёт.

Но Kafka продолжает хранить события, пока они не будут удалены retention policy.

```text id="f9m3q8"
Producer
   ↓
Kafka Log
   ↓
Consumer
   ↑
  lag
```

Если consumer слишком долго отстаёт и retention уже удалил старые записи, он может потерять возможность прочитать их из Kafka.

---

# 15. Log Segment

Kafka не хранит весь partition как один огромный файл.

Log разделяется на **segments**.

Упрощённо:

```text id="p6r2m9"
Partition 0

Segment 0
├── events
├── events
└── events

Segment 1
├── events
├── events
└── events

Segment 2
├── events
├── events
└── events
```

Это облегчает:

* хранение;
* поиск;
* удаление старых данных;
* управление большими объёмами log.

Retention обычно удаляет **старые segments**, а не отдельные сообщения произвольным образом.

---

# 16. Log End Offset

У partition есть конец журнала.

Например:

```text id="q4n8x2"
Partition 0

0 → A
1 → B
2 → C
3 → D
```

Следующая позиция:

```text id="j7m3p5"
4
```

Концептуально это связано с **Log End Offset (LEO)**.

Если consumer находится на позиции:

```text id="r2k9v6"
offset = 2
```

а конец log:

```text id="s5q1m8"
LEO = 4
```

то между consumer и концом есть непрочитанные записи.

---

# 17. Consumer Lag и Log

Теперь связываем несколько тем:

```text id="n8x4p2"
Kafka Log
0  1  2  3  4  5  6
            ↑        ↑
         Consumer    Log End
```

Consumer отстаёт:

```text id="m6r9q3"
Lag ≈ Log End Offset - Consumer Group Position
```

Например:

```text id="k2v7p5"
Log End = 1000
Consumer position = 700

Lag ≈ 300
```

---

# 18. Несколько Consumer Groups

Один Kafka log могут читать разные группы независимо.

```text id="w3n8q1"
                 Kafka Log
                     │
          ┌──────────┼──────────┐
          ↓          ↓          ↓
       Group A    Group B    Group C
          │          │          │
       Service A Service B Service C
```

Каждая consumer group имеет собственные offsets.

Например:

```text id="f7m2x9"
Group A → offset 100
Group B → offset 500
Group C → offset 50
```

Все они могут читать один и тот же log независимо.

---

# 19. Kafka Log vs RabbitMQ Queue

Это очень полезное сравнение.

### RabbitMQ

Упрощённо:

```text id="q5r8m3"
Queue
 ↓
Consumer
 ↓
ACK
 ↓
сообщение может быть удалено
```

### Kafka

```text id="v2n6p9"
Log
 ↓
Consumer
 ↓
commit offset
 ↓
сообщение остаётся
 ↓
retention
 ↓
удаление
```

Поэтому:

```text id="x8k4m1"
RabbitMQ
→ доставка сообщения

Kafka
→ хранение и чтение event stream
```

Это упрощённое, но полезное для собеседования различие.

---

# 20. Log Compaction

Кроме обычного retention в Kafka существует **log compaction**.

При compaction Kafka может сохранять последнее значение для каждого key.

Например:

```text id="r3m7q2"
key=user-1 → name=Ilya
key=user-1 → name=Alex
key=user-1 → name=Bob
```

После compaction может остаться актуальное значение:

```text id="p9x4n6"
key=user-1 → name=Bob
```

Это отличается от обычного retention.

### Retention

Удаляет старые данные по времени/размеру.

### Compaction

Сохраняет актуальное состояние по key.

---

# 21. Tombstone

При log compaction можно использовать **tombstone** — запись с `null` value для удаления ключа из логически актуального состояния.

Например:

```text id="h2q8m5"
user-1 → Bob
user-1 → null
```

`null` может обозначать:

```text id="d7r3p9"
user-1 deleted
```

После соответствующей compaction-зачистки старые записи этого key могут быть удалены.

---

# 22. Почему Kafka называют Distributed Commit Log?

Kafka часто описывают как **distributed commit log**.

Идея:

```text id="c4n8x2"
последовательность событий
        ↓
append-only log
        ↓
распределён по broker'ам
        ↓
реплицируется
```

Producer добавляет записи.

Consumers читают записи по offset.

Это очень близко к модели журнала событий.

---

# 23. Полная схема

```text id="v7m2q9"
                    PRODUCER
                       │
                       ▼
                     TOPIC
                       │
          ┌────────────┼────────────┐
          ↓            ↓            ↓
     Partition 0   Partition 1   Partition 2
          │            │            │
          ↓            ↓            ↓
       ┌──────┐     ┌──────┐     ┌──────┐
       │ Log  │     │ Log  │     │ Log  │
       ├──────┤     ├──────┤     ├──────┤
       │ 0 A  │     │ 0 D  │     │ 0 G  │
       │ 1 B  │     │ 1 E  │     │ 1 H  │
       │ 2 C  │     │ 2 F  │     │ 2 I  │
       └──────┘     └──────┘     └──────┘
          │            │            │
          └────────────┼────────────┘
                       ↓
                 CONSUMER GROUP
                       ↓
                  read by offset
                       ↓
                 commit offset
                       ↓
                    retention
```

---

# 24. Связь всех понятий Kafka

Теперь можно собрать уже изученные темы:

```text id="s6p1m8"
                     Kafka
                       │
                       ↓
                     Topic
                       │
               ┌───────┴───────┐
               ↓               ↓
          Partition 0      Partition 1
               │               │
               ↓               ↓
             Log             Log
               │               │
          Offset 0...      Offset 0...
               │               │
               └───────┬───────┘
                       ↓
                Consumer Group
                       │
                       ↓
                    Consumer
                       │
                       ↓
                 Commit Offset
                       │
                       ↓
                  Consumer Lag
```

При этом:

```text id="x9m3q7"
Retention
    ↓
определяет хранение

Replication
    ↓
защищает данные

Partitions
    ↓
дают параллелизм

Offsets
    ↓
определяют позицию чтения

Consumer Groups
    ↓
распределяют обработку
```

---

# 25. Типичный вопрос на собеседовании

### ❓ Что такое Kafka Log?

> Kafka Log — это последовательность записей внутри partition, в которую новые сообщения добавляются в конец. Каждая запись имеет offset, а удаление происходит согласно политике хранения, а не после чтения consumer'ом.

### ❓ Kafka удаляет сообщение после чтения?

> Нет. Consumer коммитит offset, но сообщение остаётся в log до момента удаления по retention или в с
