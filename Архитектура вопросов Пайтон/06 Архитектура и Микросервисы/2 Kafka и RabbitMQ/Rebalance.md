# Rebalance в Kafka 🔄

## 🎯 Ответ на собеседовании

**Rebalance** — это процесс перераспределения partitions между consumers внутри одной **Consumer Group**.

Он происходит, когда состав группы или состояние consumers меняется.

Например, было:

```text
Topic
├── P0 → Consumer 1
├── P1 → Consumer 2
├── P2 → Consumer 3
└── P3 → Consumer 4
```

`Consumer 2` упал:

```text
Consumer 2 ❌
```

Kafka запускает rebalance и перераспределяет его partitions:

```text
P0 → Consumer 1
P1 → Consumer 3
P2 → Consumer 3
P3 → Consumer 4
```

> **Rebalance нужен, чтобы partitions были распределены между актуальными участниками Consumer Group.**

---

## 🎤 Суперкоротко

```text
Consumer Group
      ↓
изменился состав
      ↓
Rebalance
      ↓
Kafka перераспределяет partitions
```

Типичные причины:

```text
Consumer добавился
Consumer отключился
Consumer упал
Consumer перестал отправлять heartbeat
```

---

# 1. Зачем нужен Rebalance?

Главная задача — сохранить **распределение нагрузки** между consumers.

Допустим:

```text
4 partitions
4 consumers
```

```text
P0 → C1
P1 → C2
P2 → C3
P3 → C4
```

Если `C2` упал:

```text
P0 → C1
P1 → ?
P2 → C3
P3 → C4
```

Partition `P1` нельзя оставить без consumer'а.

Kafka запускает rebalance:

```text
P0 → C1
P1 → C3
P2 → C4
P3 → C1
```

Конкретное распределение зависит от выбранного assignor.

---

# 2. Когда происходит Rebalance?

## Consumer добавился

Было:

```text
P0 → C1
P1 → C2
P2 → C1
P3 → C2
```

Добавился:

```text
C3
```

Kafka может перераспределить:

```text
P0 → C1
P1 → C2
P2 → C3
P3 → C1
```

Теперь нагрузка распределена между тремя consumers.

---

## Consumer удалился

Было:

```text
P0 → C1
P1 → C2
P2 → C3
```

`C2` остановился:

```text
P0 → C1
P1 → ?
P2 → C3
```

После rebalance:

```text
P0 → C1
P1 → C3
P2 → C1
```

---

# 3. Consumer Crash 💥

Один из типичных сценариев:

```text
Consumer
   ↓
CRASH
```

Kafka не узнаёт об этом мгновенно.

Она определяет состояние consumer'а через механизм **heartbeats**.

```text
Consumer
   │
   ├── heartbeat
   ├── heartbeat
   ├── heartbeat
   │
   X
```

Если consumer перестаёт отвечать в течение допустимого времени:

```text
Consumer считается вышедшим
        ↓
Rebalance
```

---

# 4. Heartbeat ❤️

Consumer периодически отправляет heartbeat **Group Coordinator**.

Упрощённо:

```text
Consumer
    │
    │ heartbeat
    ↓
Group Coordinator
```

Heartbeat сообщает:

> «Я жив и всё ещё являюсь участником этой Consumer Group».

Если heartbeat перестали приходить, Kafka может исключить consumer из группы.

---

# 5. Group Coordinator

**Group Coordinator** — broker, который отвечает за управление конкретной Consumer Group.

Он участвует в:

* отслеживании membership группы;
* heartbeats;
* обнаружении выхода consumers;
* управлении group state;
* координации rebalance.

Упрощённо:

```text
             Kafka Cluster
                   │
          ┌────────┴────────┐
          ↓                 ↓
       Broker 1          Broker 2
          │
          ↓
 Group Coordinator
          │
     Consumer Group
      /     |     \
     ↓      ↓      ↓
    C1     C2     C3
```

---

# 6. Что происходит во время Rebalance?

Упрощённо процесс выглядит так:

```text
1. Kafka обнаруживает изменение группы
            ↓
2. Начинается rebalance
            ↓
3. Определяется список consumers
            ↓
4. Определяется список partitions
            ↓
5. Выбирается распределение
            ↓
6. Partitions назначаются consumers
            ↓
7. Consumers продолжают обработку
```

Схема:

```text
Old Assignment
      ↓
Rebalance
      ↓
New Assignment
```

---

# 7. Partition Assignment

Kafka должна решить:

> **Какие partitions назначить каким consumers?**

Для этого используются **partition assignors**.

Основные варианты:

```text
RangeAssignor
RoundRobinAssignor
StickyAssignor
CooperativeStickyAssignor
```

Их задача — определить распределение partitions между consumers.

---

# 8. Range Assignor

Partitions распределяются диапазонами.

Например:

```text
6 partitions
3 consumers
```

Условно:

```text
C1 → P0, P1
C2 → P2, P3
C3 → P4, P5
```

Идея:

```text
[ P0 P1 ] → C1
[ P2 P3 ] → C2
[ P4 P5 ] → C3
```

---

# 9. Round Robin Assignor

Partitions распределяются циклически.

Например:

```text
P0 → C1
P1 → C2
P2 → C3
P3 → C1
P4 → C2
P5 → C3
```

Получается:

```text
C1 → P0, P3
C2 → P1, P4
C3 → P2, P5
```

Это может давать более равномерное распределение.

---

# 10. Sticky Assignor

**Sticky** старается:

1. распределить partitions равномерно;
2. сохранить существующее распределение при rebalance, насколько возможно.

Это важно, потому что изменение assignment может приводить к лишним перемещениям partitions.

Например:

```text
До:

C1 → P0, P1
C2 → P2, P3
```

Добавили C3.

Sticky assignor старается не менять всё распределение с нуля:

```text
C1 → P0
C2 → P2
C3 → P1, P3
```

Конкретный результат зависит от assignment и конфигурации.

---

# 11. Cooperative Rebalance

Современный подход — **cooperative rebalancing**.

Идея:

> Не обязательно полностью отзывать все partitions у consumers перед новым распределением.

Вместо этого Kafka старается менять только необходимую часть assignment.

Упрощённо:

```text
Eager:

Все partitions
      ↓
отозвать
      ↓
назначить заново
```

Cooperative:

```text
Изменить только необходимую часть
        ↓
минимум перемещений
```

Это помогает уменьшить влияние rebalance на работающую систему.

---

# 12. Eager vs Cooperative

|            | Eager             | Cooperative                |
| ---------- | ----------------- | -------------------------- |
| Подход     | Полный rebalance  | Постепенный                |
| Partitions | Широко отзываются | Меняется необходимая часть |
| Влияние    | Может быть больше | Обычно меньше              |
| Цель       | Простота          | Минимизация disruption     |

Для собеседования достаточно:

> **Eager rebalance может временно отзывать partitions у всех участников, а cooperative старается перераспределять только необходимую часть partitions.**

---

# 13. Rebalance и Consumer Lag 📊

Во время rebalance обработка может временно замедлиться.

Например:

```text
Lag
 ↓
100
150
200
250
300
```

Причина:

```text
Rebalance
   ↓
часть времени не обрабатываем сообщения
   ↓
Lag ↑
```

После завершения:

```text
Consumer processing
       ↓
Lag ↓
```

Поэтому частые rebalance могут негативно влиять на throughput.

---

# 14. Почему частые Rebalance — плохо? ⚠️

Представим:

```text
Consumer Group

C1
C2
C3
```

Каждые несколько секунд:

```text
Join
 ↓
Rebalance
 ↓
Leave
 ↓
Rebalance
 ↓
Join
 ↓
Rebalance
```

Получаем:

```text
стабильной обработки мало
       ↓
rebalance много
       ↓
throughput падает
       ↓
lag растёт
```

Поэтому в production важно избегать **rebalance storm**.

---

# 15. Причины частых Rebalance

Например:

### Consumer постоянно падает

```text
C1
 ↓
CRASH
 ↓
Rebalance
 ↓
C1 restart
 ↓
Rebalance
```

---

### Consumer слишком долго обрабатывает poll

Если consumer долго не вызывает `poll()`, Kafka может решить, что consumer перестал нормально участвовать в группе.

Поэтому важно понимать параметр:

```text
max.poll.interval.ms
```

Он ограничивает максимальный интервал между вызовами `poll()`.

Если обработка сообщения занимает слишком много времени, consumer может быть исключён из группы.

---

# 16. Session Timeout

Другой важный параметр:

```text
session.timeout.ms
```

Он связан с heartbeat.

Упрощённо:

```text
Heartbeat
   ↓
Heartbeat
   ↓
нет heartbeat
   ↓
session timeout
   ↓
Consumer считается недоступным
   ↓
Rebalance
```

---

# 17. max.poll.interval.ms vs session.timeout.ms

Их часто путают.

### `session.timeout.ms`

Связан с:

```text
heartbeats
```

и определением, жив ли consumer с точки зрения group membership.

### `max.poll.interval.ms`

Связан с:

```text
poll()
```

и максимальным временем между вызовами `poll()`.

Упрощённо:

```text
session.timeout
→ "Consumer вообще жив?"

max.poll.interval
→ "Consumer продолжает нормально забирать сообщения?"
```

---

# 18. Rebalance и масштабирование

Rebalance — нормальная часть горизонтального масштабирования.

Например:

```text
Было:

P0 → C1
P1 → C2
P2 → C1
P3 → C2
```

Добавляем:

```text
C3
```

Kafka:

```text
Rebalance
   ↓
P0 → C1
P1 → C2
P2 → C3
P3 → C1
```

Теперь:

```text
3 consumers
4 partitions
```

и работа распределена между ними.

---

# 19. Rebalance и количество partitions

Допустим:

```text
2 partitions
5 consumers
```

После rebalance:

```text
P0 → C1
P1 → C2

C3 → idle
C4 → idle
C5 → idle
```

Добавление consumers выше количества partitions не даст дополнительного параллелизма.

```text
Max useful consumers ≈ partitions
```

---

# 20. Практический пример

Есть сервис обработки заказов:

```text
orders topic
├── P0
├── P1
├── P2
└── P3
```

Запущено:

```text
orders-consumer × 2
```

Получаем:

```text
C1 → P0, P1
C2 → P2, P3
```

Нагрузка выросла.

Добавляем третий экземпляр:

```text
orders-consumer × 3
```

Происходит:

```text
Rebalance
```

После него возможна схема:

```text
C1 → P0
C2 → P1, P3
C3 → P2
```

Теперь три экземпляра обрабатывают четыре partitions.

---

# 21. Связь Rebalance с Offset

Очень важная цепочка:

```text
Consumer
   ↓
обрабатывает Partition
   ↓
Commit Offset
   ↓
Consumer падает
   ↓
Rebalance
   ↓
Partition получает другой Consumer
   ↓
новый Consumer читает с committed offset
```

Например:

```text
P0

offset 100 → обработан
offset 100 → committed

Consumer 1 ❌

Rebalance

P0 → Consumer 2

Consumer 2
   ↓
продолжает с сохранённого offset
```

Если сообщение было обработано, но offset не успел закоммититься:

```text
Process ✓
Commit  ✗
Crash   💥
```

новый consumer может обработать сообщение повторно.

Отсюда:

```text
Rebalance
    ↓
Offset
    ↓
At-least-once
    ↓
Idempotency
```

---

# ⚠️ Частые ошибки на собеседовании

### ❌ «Rebalance происходит при каждом новом сообщении»

Нет.

Rebalance связан с **изменением состава Consumer Group или её состояния**, а не с обычным чтением сообщений.

---

### ❌ «Rebalance распределяет сообщения между consumers»

Точнее:

> **Rebalance распределяет partitions между consumers.**

А сообщения уже находятся внутри этих partitions.

---

### ❌ «Consumer Group может иметь только столько consumers, сколько partitions»

Нет.

Consumers может быть больше.

Просто часть будет простаивать:

```text
4 partitions
6 consumers

4 active
2 idle
```

---

### ❌ «Rebalance всегда означает ошибку»

Нет.

Rebalance — штатный механизм Kafka.

Проблемой являются **частые или длительные rebalance**, которые мешают стабильной обработке.

---

# 🧠 Шпаргалка

```text
Rebalance
→ перераспределение partitions

Причины:
→ consumer добавился
→ consumer ушёл
→ consumer упал
→ heartbeat/session timeout
→ изменения membership

Group Coordinator
→ управляет состоянием Consumer Group

Assignment
→ кому какие partitions

Heartbeat
→ consumer сообщает, что жив

session.timeout.ms
→ контроль доступности через heartbeat

max.poll.interval.ms
→ максимум между poll()

Rebalance
→ может временно увеличить Lag
```

## Главная схема

```text
Consumer Group
      │
      ├── Consumer 1
      ├── Consumer 2
      └── Consumer 3
              │
              ↓
       Group Coordinator
              │
              ↓
          Rebalance
              │
              ↓
     Partition Assignment
              │
              ↓
       Consumers получают
          partitions
```

> **Rebalance — это механизм Kafka для поддержания корректного распределения partitions между участниками Consumer Group. Он необходим при изменении состава группы, но слишком частые rebalance могут снижать throughput и увеличивать Consumer Lag.**
