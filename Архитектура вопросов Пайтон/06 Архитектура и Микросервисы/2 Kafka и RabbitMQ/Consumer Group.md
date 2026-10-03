# Consumer Group в Kafka 👥

## 🎯 Ответ на собеседовании

**Consumer Group** — это группа Kafka-консьюмеров, которые совместно потребляют сообщения из топика.

Главное правило:

> **Внутри одной Consumer Group один partition в конкретный момент времени назначается только одному consumer.**

Это позволяет распределять обработку сообщений между несколькими экземплярами приложения.

Например:

```text id="q7m2xa"
Topic: orders

Partition 0 ──→ Consumer 1
Partition 1 ──→ Consumer 2
Partition 2 ──→ Consumer 3
Partition 3 ──→ Consumer 4
```

Consumer Group также хранит **offsets**, поэтому Kafka знает, с какого места продолжить чтение после перезапуска consumer'а.

---

## 🎤 Суперкоротко

```text id="c4k8zp"
Topic
 │
 ├── Partition 0 → Consumer 1
 ├── Partition 1 → Consumer 2
 ├── Partition 2 → Consumer 3
 └── Partition 3 → Consumer 4
              │
              ↓
        Consumer Group
```

**Consumer Group = несколько consumers, совместно читающих partitions одного topic.**

---

# 1. Зачем нужна Consumer Group?

Основная задача — **горизонтально масштабировать обработку сообщений**.

Допустим, есть один consumer:

```text id="k3n7vy"
Topic
├── P0
├── P1
├── P2
└── P3
     ↓
 Consumer
```

Один процесс должен обрабатывать все partitions.

Если сообщений становится много, можно запустить несколько consumers:

```text id="m9x2qa"
Topic
├── P0 ──→ Consumer 1
├── P1 ──→ Consumer 2
├── P2 ──→ Consumer 3
└── P3 ──→ Consumer 4
```

Теперь работа распределяется между четырьмя экземплярами приложения.

---

# 2. Связь Consumer Group и Partition

Количество partitions определяет максимальный параллелизм **в рамках одной Consumer Group**.

Например:

```text id="p4y8km"
4 partitions
4 consumers
```

Можно получить:

```text id="v2n6rx"
P0 → C1
P1 → C2
P2 → C3
P3 → C4
```

Все consumers заняты.

---

## Если consumers больше partitions

Например:

```text id="x8q3mz"
2 partitions
4 consumers
```

Получится примерно:

```text id="h5w9kc"
P0 → C1
P1 → C2

C3 → idle
C4 → idle
```

Дополнительные consumers не получают отдельные partitions.

Поэтому:

> **Для полного параллелизма внутри группы число активных consumers не должно превышать число partitions.**

---

# 3. Если consumers меньше partitions

Например:

```text id="n7c4bx"
4 partitions
2 consumers
```

Kafka распределит несколько partitions между consumers:

```text id="z6p2qa"
Consumer 1 → P0 + P1
Consumer 2 → P2 + P3
```

Каждый consumer может обрабатывать несколько partitions.

---

# 4. Несколько Consumer Groups

Один Kafka topic может одновременно читать несколько независимых Consumer Groups.

Например:

```text id="w3k8nv"
                 orders
                   │
          ┌────────┼────────┐
          ↓        ↓        ↓
       Group A  Group B  Group C
       Billing  Analytics Notifications
```

Каждая группа имеет собственные offsets.

Например:

```text id="q9m4xt"
Billing:
offset = 100

Analytics:
offset = 80

Notifications:
offset = 95
```

Они могут находиться на разных позициях одного и того же topic.

---

# 5. Почему сообщения получают несколько групп?

Это одно из ключевых свойств Kafka.

Допустим, есть событие:

```text id="j6r2wp"
OrderCreated
```

Его могут независимо обработать:

```text id="y8k5mc"
Billing Group
      ↓
создать платёж

Analytics Group
      ↓
записать статистику

Notification Group
      ↓
отправить уведомление
```

То есть сообщение не обязательно «доставляется одному consumer'у вообще».

Оно может быть прочитано **один раз каждой Consumer Group**.

---

# 6. Consumer Group и Offset

Каждая Consumer Group хранит своё состояние чтения.

Упрощённо:

```text id="m5q8vz"
Topic
│
├── P0
│   ├── 0
│   ├── 1
│   ├── 2
│   └── 3 ← Group A
│
└── P1
    ├── 0
    ├── 1
    ├── 2 ← Group A
    └── 3
```

Group знает:

```text id="a3x7kp"
P0 → offset 3
P1 → offset 2
```

После перезапуска consumer'а Kafka может продолжить чтение с сохранённой позиции.

---

# 7. Consumer Group и Consumer Lag 📊

Consumer Group напрямую связана с **Consumer Lag**.

Например:

```text id="u8n4qm"
Partition 0

Latest offset    = 1000
Committed offset = 900

Lag = 100
```

Если в группе несколько partitions:

```text id="k6p2rx"
P0 → Lag 100
P1 → Lag 50
P2 → Lag 200
```

Общий lag группы концептуально:

```text id="z5c9vn"
100 + 50 + 200 = 350
```

Но на практике важно смотреть Lag **по каждой partition**, а не только суммарное значение.

---

# 8. Что происходит при падении Consumer? 💥

Допустим:

```text id="r4m8sy"
P0 → C1
P1 → C2
P2 → C3
```

Consumer 2 упал:

```text id="v7q3nx"
P0 → C1
P1 → ❌
P2 → C3
```

Kafka обнаруживает изменение состава группы и выполняет **rebalance**.

После перераспределения:

```text id="p8k2wd"
P0 → C1
P1 → C3
P2 → C3
```

Consumer 3 временно получает дополнительную partition.

---

# 9. Rebalance 🔄

**Rebalance** — перераспределение partitions между consumers внутри Consumer Group.

Он может произойти, когда:

* consumer подключился;
* consumer отключился;
* consumer упал;
* изменилось количество consumers;
* изменились некоторые параметры группы.

Например:

До:

```text id="j4x9mc"
P0 → C1
P1 → C2
P2 → C3
P3 → C4
```

Добавили C5:

```text id="s7q2vp"
P0 → C1
P1 → C2
P2 → C3
P3 → C4
```

Kafka перераспределяет partitions согласно выбранному механизму назначения.

Возможный результат:

```text id="h6m3zx"
P0 → C1
P1 → C2
P2 → C3
P3 → C4

C5 → пока не получил partition
```

Если partitions меньше consumers, часть consumers будет простаивать.

---

# 10. Consumer Group ≠ один Consumer

Это важное различие.

```text id="w5q8zn"
Consumer Group
      │
 ┌────┼────┐
 ↓    ↓    ↓
 C1   C2   C3
```

**Consumer** — конкретный экземпляр приложения.

**Consumer Group** — логическая группа этих экземпляров.

Например:

```text id="g2k7xp"
docker-compose:

consumer-1
consumer-2
consumer-3
```

Все могут иметь:

```text id="n8v4qm"
group_id = "orders-service"
```

Тогда они работают как одна Consumer Group.

---

# 11. Один Consumer — несколько Groups

Один и тот же consumer-процесс концептуально может читать разные topics/groups в зависимости от конфигурации клиентов, но стандартный и наиболее понятный сценарий:

```text id="f3m7kc"
Consumer instance
       ↓
Consumer Group
       ↓
Topic
```

В backend обычно каждый логический сервис имеет свою Consumer Group.

Например:

```text id="q5x9vp"
Billing Service
→ group_id = "billing"

Analytics Service
→ group_id = "analytics"

Notification Service
→ group_id = "notifications"
```

---

# 12. Consumer Group и микросервисы 🏗️

Очень типичный сценарий:

```text id="r8m3zw"
                  Kafka
                    │
             orders topic
                    │
        ┌───────────┼───────────┐
        ↓           ↓           ↓
     Billing     Analytics   Notification
      Group         Group       Group
```

Каждый сервис независимо читает события.

Например:

```text id="x4q7vn"
OrderCreated
      │
      ├──→ Billing
      ├──→ Analytics
      └──→ Notification
```

Это хорошо подходит для **event-driven architecture**.

---

# 13. Consumer Group и порядок сообщений

Kafka гарантирует порядок сообщений **внутри partition**.

Например:

```text id="b6m2xy"
Partition 0:

100 → OrderCreated
101 → PaymentCreated
102 → OrderShipped
```

Consumer читает их последовательно.

Но если события находятся в разных partitions:

```text id="n7q3kc"
P0:
100 → A
101 → B

P1:
100 → C
101 → D
```

Kafka не предоставляет глобального порядка между P0 и P1.

Поэтому если порядок событий для одной сущности важен, часто используют **одинаковый partition key**.

Например:

```text id="v9x4mp"
order_id = 123
      ↓
одинаковый partition
      ↓
OrderCreated
PaymentCreated
OrderShipped
```

---

# 14. Consumer Group и масштабирование

Главная схема:

```text id="c5k8rz"
             Topic
               │
       ┌───────┼───────┐
       ↓       ↓       ↓
      P0      P1      P2
       │       │       │
       ↓       ↓       ↓
      C1      C2      C3
               │
        Consumer Group
```

Чтобы увеличить throughput:

```text id="m2v7qx"
1 consumer
     ↓
2 consumers
     ↓
3 consumers
```

Но только пока есть свободные partitions.

```text id="a8q4zn"
3 partitions
3 consumers
→ полный параллелизм

3 partitions
5 consumers
→ 2 consumers простаивают
```

---

# 15. Consumer Group vs Queue в RabbitMQ

Это важное сравнение после темы **Kafka vs RabbitMQ**.

### RabbitMQ

Несколько consumers могут читать одну очередь:

```text id="r6m3wp"
Queue
 ├── Consumer 1
 ├── Consumer 2
 └── Consumer 3
```

Сообщение обрабатывается одним consumer'ом.

### Kafka

Несколько consumers внутри одной группы:

```text id="t8x2vk"
Topic
 ├── P0 → Consumer 1
 ├── P1 → Consumer 2
 └── P2 → Consumer 3
```

А другая группа может прочитать **те же события независимо**:

```text id="y4n7mc"
             Topic
            /     \
           ↓       ↓
       Group A   Group B
```

---

# 16. Типичная ошибка ❌

Неправильно:

> «Каждое сообщение Kafka получает один consumer».

Правильно:

> **В рамках одной Consumer Group конкретное сообщение из partition обрабатывается одним consumer'ом группы. Но другие Consumer Groups могут независимо прочитать это же сообщение.**

---

# 17. Связь всех понятий

```text id="q8m5zx"
Producer
   ↓
 Topic
   ↓
Partition
   ↓
Consumer Group
   ↓
Consumer
   ↓
Offset
   ↓
Consumer Lag
```

Если consumer не успевает:

```text id="w3k7pn"
Producer
   ↓
Kafka
   ↓
Consumer
   ↓
обработка медленная
   ↓
Offset растёт медленно
   ↓
Lag ↑
```

Если добавить consumers:

```text id="n5x9qm"
Partition 0 → Consumer 1
Partition 1 → Consumer 2
Partition 2 → Consumer 3
```

можно увеличить параллелизм — **если количество partitions позволяет**.

---

# 🧠 Шпаргалка

```text id="z4q8vk"
Consumer
→ экземпляр приложения

Consumer Group
→ группа consumers

Partition
→ единица параллелизма

Offset
→ позиция чтения группы

Rebalance
→ перераспределение partitions

Consumer Lag
→ отставание группы от конца partition
```

### Главная формула

```text id="c7m2xp"
1 Consumer Group
        ↓
N Consumers
        ↓
M Partitions

Максимальный параллелизм
≈ min(N, M)
```

> **Consumer Group позволяет горизонтально масштабировать Kafka-consumers: partitions распределяются между экземплярами группы, а offsets позволяют каждой группе независимо отслеживать прогресс чтения.**
