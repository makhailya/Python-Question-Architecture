# Consumer Lag в Kafka 📊

## 🎯 Ответ на собеседовании

**Consumer Lag** — это отставание consumer'а от последних доступных сообщений в Kafka.

Упрощённо:

```text id="9n8x2p"
Consumer Lag =
последний доступный offset
−
последний обработанный / committed offset
```

Например:

```text id="q7j4mx"
Kafka partition:

0  1  2  3  4  5  6  7  8  9
                  ↑              ↑
              Consumer        Latest
              offset 5        offset 9

Lag = 9 - 5 = 4
```

**Lag показывает, насколько consumer не успевает обрабатывать поток сообщений.**

---

## 🎤 Суперкоротко

> **Consumer Lag — количество сообщений, на которое consumer отстаёт от текущего конца partition.**

```text id="6bq5vk"
Producer
   ↓
Kafka
   ↓
████████████████████████  ← новые сообщения
          ↑
       Consumer
```

Если Producer пишет быстрее, чем Consumer читает:

```text id="a5q6wd"
Producer rate > Consumer rate
            ↓
       Consumer Lag ↑
```

---

# 1. Offset и Lag

Чтобы понять Lag, нужно понимать **offset**.

Kafka хранит сообщения в partition с последовательными offset:

```text id="q1j0vd"
Partition 0

offset
  0 → event A
  1 → event B
  2 → event C
  3 → event D
  4 → event E
  5 → event F
```

Consumer читает сообщения и хранит свой прогресс через **consumer offset**.

Например:

```text id="5q4b8z"
Latest offset = 5
Consumer offset = 2
```

Упрощённо:

```text id="y4g8kc"
Lag = 5 - 2 = 3
```

> На практике при мониторинге важно учитывать семантику `log end offset`, `committed offset` и то, как конкретный инструмент считает границы offset'ов. Поэтому формулу лучше воспринимать как концептуальную.

---

# 2. Почему возникает Consumer Lag?

Основная причина:

> **Consumer обрабатывает сообщения медленнее, чем они появляются.**

Например:

```text id="v6n8hj"
Producer:
1000 msg/sec

Consumer:
700 msg/sec
```

Разница:

```text id="l3q7kf"
1000 - 700 = 300 msg/sec
```

Lag будет расти примерно на:

```text id="1o5j0b"
300 сообщений/сек
```

---

# 3. Причины большого Lag ⚠️

### 1. Consumer слишком медленный

Например:

```text id="g7k2d9"
Kafka
 ↓
Consumer
 ↓
сложная обработка
 ↓
PostgreSQL
 ↓
внешний API
```

Если обработка одного сообщения занимает много времени, consumer начинает отставать.

---

### 2. Недостаточно consumers

Например:

```text id="6g3p5d"
4 partitions

Consumer 1
Consumer 2
```

Два consumer'а не смогут одновременно обрабатывать все partitions максимально эффективно.

Можно увеличить количество consumers:

```text id="k2m8hz"
4 partitions

Consumer 1 → P0
Consumer 2 → P1
Consumer 3 → P2
Consumer 4 → P3
```

---

### 3. Мало partitions

Количество consumer'ов внутри одной Consumer Group не может дать больше параллелизма, чем количество partitions.

Например:

```text id="f8v3r1"
2 partitions
10 consumers
```

Максимально одновременно активно работают:

```text id="c0k6vp"
2 consumers
```

Остальные ждут.

Поэтому:

```text id="v4y1pm"
Parallelism ≈ количество partitions
```

---

### 4. Consumer упал

```text id="h3s7cz"
Consumer
   ↓
CRASH ❌

Kafka
████████████████████
        ↑
      messages
```

Producer продолжает писать сообщения.

Consumer не обрабатывает их → Lag растёт.

После восстановления consumer продолжит чтение с сохранённого offset.

---

### 5. База данных или внешний сервис тормозит

Например:

```text id="m8n1qs"
Kafka
 ↓
Consumer
 ↓
PostgreSQL 🐌
```

Consumer ждёт PostgreSQL.

В результате Kafka продолжает получать сообщения, а consumer обрабатывает их медленно.

---

### 6. Retry

Если обработка сообщения периодически завершается ошибкой:

```text id="e9v4ks"
Message
 ↓
Consumer
 ↓
ERROR
 ↓
Retry
 ↓
Retry
 ↓
Retry
```

Consumer тратит время на повторную обработку.

Lag может увеличиваться.

---

# 4. Lag бывает по каждой partition

Важно понимать:

> **Lag считается не только для Consumer Group, но и для каждой partition.**

Например:

```text id="x5c8qp"
Partition 0 → Lag 10
Partition 1 → Lag 15
Partition 2 → Lag 5000
Partition 3 → Lag 12
```

Это очень полезный сигнал.

Видим:

```text id="0q5m1d"
P2 → 5000
```

Значит проблема может быть именно с consumer'ом, который обрабатывает `Partition 2`.

---

# 5. Consumer Group Lag

Допустим:

```text id="7y9x4k"
Partition 0 → Lag 100
Partition 1 → Lag 50
Partition 2 → Lag 200
```

Суммарный lag группы:

```text id="d1j8mf"
100 + 50 + 200 = 350
```

Такой показатель часто используется в мониторинге.

Но важно смотреть не только сумму.

Например:

```text id="3k5p8a"
P0 → 10
P1 → 10
P2 → 10
P3 → 10000
```

Среднее значение может выглядеть терпимо, хотя одна partition серьёзно отстаёт.

---

# 6. Lag и Consumer Group 👥

Каждая Consumer Group имеет собственные offsets.

```text id="k8x3qn"
Kafka Topic
      ↓
 ┌────┴────┐
 ↓         ↓
Group A   Group B
```

Например:

```text id="9z4w7p"
Group A → Analytics
Group B → Notifications
```

У них будет независимый прогресс.

Поэтому одна группа может иметь:

```text id="4d9c7x"
Analytics:
Lag = 100

Notifications:
Lag = 5000
```

Это нормально с точки зрения Kafka: каждая группа читает поток независимо.

---

# 7. Что делать при большом Consumer Lag? 🛠️

Нужно сначала найти причину.

### Шаг 1 — проверить состояние consumers

```text id="q7j3m8"
Consumer работает?
        ↓
      Да / Нет
```

Если consumer упал — восстановить его.

---

### Шаг 2 — проверить нагрузку

Посмотреть:

* CPU;
* RAM;
* network;
* latency;
* время обработки сообщений.

---

### Шаг 3 — проверить внешние зависимости

Например:

```text id="n8v4k2"
Kafka
 ↓
Consumer
 ↓
PostgreSQL
 ↓
External API
```

Если тормозит PostgreSQL или API, масштабирование Kafka consumer'ов само по себе может не решить проблему.

---

### Шаг 4 — увеличить количество consumers

Если есть свободные partitions:

```text id="p4c7v9"
2 consumers
   ↓
4 consumers
```

Можно увеличить параллелизм.

---

### Шаг 5 — увеличить количество partitions

Если текущего количества partitions недостаточно:

```text id="w6m2xa"
2 partitions
      ↓
4 partitions
```

Но изменение количества partitions требует аккуратного проектирования, особенно если важен порядок сообщений или используется partition key.

---

### Шаг 6 — оптимизировать обработку

Например:

```text id="z2r5nq"
было:

Kafka
 ↓
Consumer
 ↓
INSERT по одному
 ↓
DB

стало:

Kafka
 ↓
Consumer
 ↓
batch
 ↓
DB
```

Batch processing может значительно увеличить throughput.

---

# 8. Lag ≠ ошибка ❗

Очень важный момент.

**Наличие Lag само по себе не означает проблему.**

Например:

```text id="d7h2p9"
Lag = 50
```

может быть совершенно нормально.

Если consumer стабильно обрабатывает сообщения:

```text id="q8k4vz"
Lag:
100
90
80
70
60
50
...
0
```

он догоняет поток.

Проблема:

```text id="r5n9mc"
Lag:
100
200
500
1000
2000
5000
```

Lag постоянно растёт.

Это означает, что consumer не успевает за producer'ом.

---

# 9. Lag и throughput

Ключевое соотношение:

```text id="f2p7qa"
Producer throughput
        vs
Consumer throughput
```

Если:

```text id="j5x8vk"
Producer = 1000 msg/s
Consumer = 1200 msg/s
```

consumer способен догонять поток.

Если:

```text id="w9c3lm"
Producer = 1000 msg/s
Consumer = 700 msg/s
```

Lag будет расти.

Упрощённо:

```text id="n7b4cx"
Producer rate > Consumer rate
          ↓
      Lag растёт

Producer rate < Consumer rate
          ↓
      Lag уменьшается
```

---

# 10. Lag и scaling 📈

Допустим:

```text id="p6d8zr"
Topic:

P0
P1
P2
P3
```

Один consumer:

```text id="v4k1mn"
Consumer 1
 ├── P0
 ├── P1
 ├── P2
 └── P3
```

Если он не справляется:

```text id="g2q7bx"
Consumer 1 → P0, P1
Consumer 2 → P2, P3
```

Производительность может вырасти.

Но:

```text id="m3w8kd"
4 partitions
10 consumers
```

не означает 10-кратное ускорение.

Максимум одновременно активно работает примерно 4 consumers в этой группе.

---

# 11. Lag и rebalance

Если consumer добавляется или удаляется из Consumer Group, Kafka выполняет **rebalance**.

Например:

```text id="s4p6my"
Consumer 1
Consumer 2
Consumer 3
```

Добавили:

```text id="h7q2vc"
Consumer 4
```

Kafka перераспределяет partitions.

```text id="j8n3rx"
P0 → C1
P1 → C2
P2 → C3
P3 → C4
```

Во время rebalance обработка может временно замедлиться, что способно увеличить Lag.

---

# 12. Lag Monitoring 📊

Consumer Lag — один из ключевых **Kafka monitoring metrics**.

Обычно мониторят:

```text id="b5x9qn"
Consumer Lag
Consumer Throughput
Processing Latency
Consumer Errors
Consumer Group State
Partition Distribution
```

Можно настроить alert:

```text id="k1z7wd"
Lag > 10 000
      ↓
   Alert 🚨
```

Но лучше использовать не только абсолютный Lag, а также:

```text id="r3m8kp"
Lag growth rate
```

То есть насколько быстро Lag увеличивается.

---

# 13. Пример из реального Backend

Допустим, интернет-магазин отправляет события:

```text id="t7x4qn"
OrderCreated
OrderPaid
OrderShipped
```

Kafka:

```text id="c6m2vz"
orders topic
     ↓
Consumer Group
"analytics"
```

Analytics consumer пишет данные в ClickHouse.

Если ClickHouse начал тормозить:

```text id="p8k5ws"
Kafka
████████████████████████
             ↑
          Consumer
             ↓
        ClickHouse 🐌
```

Lag начинает расти.

Мониторинг показывает:

```text id="a4v9xm"
Lag:
100
500
1000
5000
10000
```

Это сигнал:

> Consumer не успевает обрабатывать поток.

---

# ⚠️ Частые ошибки на собеседовании

### ❌ Неправильно

> Lag — это количество сообщений, которые ещё не прочитал consumer.

Точнее:

> **Lag — разница между текущей позицией конца partition и offset'ом consumer group.**

---

### ❌ Неправильно

> Чем больше consumers, тем быстрее Kafka.

Не всегда.

Нужно учитывать количество:

```text id="e3q7mk"
Partitions
```

и реальную производительность обработки.

---

### ❌ Неправильно

> Lag всегда означает проблему.

Нет.

Небольшой или временный Lag может быть нормальным.

Важнее смотреть:

```text id="c8p2zr"
Lag
+
скорость его изменения
+
throughput
+
latency
```

---

# 🧠 Формула

```text id="h5w9qb"
Consumer Lag

= насколько consumer отстаёт
  от конца partition
```

```text id="u7c3mx"
Producer
   ↓
Kafka Partition
   ↓
Latest Offset
   │
   │ ←──── Lag ────→
   │
Consumer Offset
```

### Для собеседования одной фразой:

> **Consumer Lag — метрика отставания consumer group от конца Kafka partition. Если producer публикует сообщения быстрее, чем consumer их обрабатывает, lag растёт. Для устранения проблемы анализируют throughput, latency, состояние consumers и partitions, а затем оптимизируют обработку или масштабируют consumer group.**
