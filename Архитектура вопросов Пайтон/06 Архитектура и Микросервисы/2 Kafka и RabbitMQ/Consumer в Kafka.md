# 📥 Consumer в Kafka

## 🎯 Ответ на собеседовании

**Consumer** — это приложение или компонент, который **читает и обрабатывает сообщения из Kafka**.

Consumer подключается к Topic, получает сообщения из Partition и после обработки фиксирует свою позицию — **Offset**.

```text
Kafka
  ↓
Topic
  ↓
Partition
  ↓
Consumer
  ↓
Обработка
  ↓
Offset
```

---

## 🎤 Суперкоротко

**Consumer — это получатель сообщений в Kafka. Он читает сообщения из Partition, обрабатывает их и отслеживает свою позицию с помощью Offset.**

---

## 🔄 Как работает Consumer

Допустим, в Kafka есть Topic `orders`:

```text
orders

Partition 0:
[0] [1] [2] [3] [4]
```

Consumer читает сообщения последовательно:

```text
0 → 1 → 2 → 3 → 4
```

После обработки Consumer фиксирует Offset.

```text
Message
   ↓
Consumer
   ↓
обработка
   ↓
commit offset
```

---

## 🔢 Offset

**Offset** — это позиция сообщения внутри конкретной Partition.

```text
Partition 0

Offset 0 → Message A
Offset 1 → Message B
Offset 2 → Message C
Offset 3 → Message D
```

Если Consumer обработал сообщения до Offset 2, он знает, откуда продолжить чтение.

Важно:

**Offset принадлежит Partition и отслеживается для Consumer Group.**

---

## 👥 Consumer Group

**Consumer Group** — группа Consumer'ов, которые совместно читают один Topic.

Например:

```text
Topic: orders

Partition 0 → Consumer 1
Partition 1 → Consumer 2
Partition 2 → Consumer 3
```

Каждая Partition внутри одной Consumer Group назначается одному Consumer.

Это позволяет распределять нагрузку.

---

## ⚖️ Масштабирование

Допустим:

```text
3 Partition
3 Consumer
```

Можно распределить:

```text
P0 → C1
P1 → C2
P2 → C3
```

Если Consumer'ов больше:

```text
3 Partition
5 Consumer
```

то часть Consumer'ов не получит Partition:

```text
P0 → C1
P1 → C2
P2 → C3

C4 → без Partition
C5 → без Partition
```

Поэтому максимальный параллелизм обработки внутри одной Consumer Group ограничен количеством Partition.

---

## 🔄 Несколько Consumer Group

Один Topic могут независимо читать несколько Consumer Group.

```text
                    orders
                       │
              ┌────────┴────────┐
              ↓                 ↓
        Group A             Group B
              ↓                 ↓
        Order Service     Analytics Service
```

Например, событие:

```text
order.created
```

может обработать одновременно:

```text
Order Service
Analytics Service
Notification Service
```

если они находятся в разных Consumer Group.

---

## 📦 Kafka не удаляет сообщение после чтения

Это важное отличие от классической очереди.

```text
Topic:

M1 → M2 → M3 → M4 → M5
```

Consumer Group A прочитала:

```text
M1 → M2 → M3
```

Сообщения всё ещё могут находиться в Kafka согласно политике хранения.

Другая Consumer Group может прочитать их независимо.

---

## 🔁 Повторное чтение

Consumer может снова прочитать старые сообщения, если изменить или сбросить Offset.

```text
M1 → M2 → M3 → M4 → M5
          ↑
       Consumer
```

Можно вернуть позицию назад:

```text
M1 → M2 → M3 → M4 → M5
     ↑
  Consumer
```

Это одна из сильных сторон Kafka как системы хранения и обработки событий.

---

## ⚠️ Дубликаты

Consumer может получить одно сообщение повторно.

Например:

```text
Consumer
   ↓
получил M1
   ↓
обработал M1
   ↓
❌ упал до commit Offset
   ↓
перезапуск
   ↓
снова получает M1
```

Поэтому Consumer должен по возможности быть **идемпотентным**.

То есть повторная обработка одного события не должна приводить к неправильному результату.

---

## 📊 Consumer Lag

**Consumer Lag** — отставание Consumer от последних сообщений в Kafka.

Например:

```text
Последний Offset в Kafka: 1000

Consumer обработал: 900

Lag = 100
```

Если Lag постоянно увеличивается, Consumer не успевает обрабатывать поток сообщений.

Причины могут быть:

* недостаточно Consumer'ов;
* медленная обработка;
* недостаточно Partition;
* проблемы с БД;
* внешние API работают медленно;
* высокая нагрузка.

---

## 🐍 Consumer в Python

Пример:

```python
from kafka import KafkaConsumer

consumer = KafkaConsumer(
    "orders",
    bootstrap_servers="localhost:9092",
    group_id="order-service",
)

for message in consumer:
    print(message.value)
```

Здесь:

```text
orders
   ↓
Consumer
   ↓
group_id = order-service
```

Consumer читает сообщения из Topic `orders`.

---

## 🧩 Commit Offset

После обработки сообщения Consumer фиксирует Offset.

Упрощённо:

```text
Kafka
  ↓
Message
  ↓
Consumer
  ↓
обработка
  ↓
Commit Offset
```

При следующем запуске Consumer может продолжить с сохранённой позиции.

Важно понимать разницу:

**получить сообщение ≠ успешно обработать сообщение.**

Поэтому момент фиксации Offset имеет значение.

---

## 🚨 Что будет при падении Consumer

Допустим:

```text
P0:

M1 → M2 → M3 → M4
          ↑
      обработано
```

Consumer упал.

Kafka может передать Partition другому Consumer той же группы.

```text
Consumer 1 ❌

        ↓

Consumer 2
   ↓
продолжает обработку
```

Это называется **rebalance** — перераспределение Partition между Consumer'ами группы.

---

## 🔄 Rebalance

**Rebalance** — перераспределение Partition между Consumer'ами Consumer Group.

Например, было:

```text
P0 → C1
P1 → C2
P2 → C3
```

C2 отключился:

```text
C2 ❌
```

Kafka перераспределяет Partition:

```text
P0 → C1
P1 → C3
P2 → C1
```

Конкретное распределение зависит от используемого механизма назначения Partition.

---

## 🧠 Consumer в микросервисах

Например:

```text
Order Service
      │
      │ Producer
      ↓
    Kafka
      │
      ↓
orders
      │
 ┌────┼─────────────┐
 ↓    ↓             ↓
Billing  Notification  Analytics
Consumer Consumer      Consumer
```

Каждый сервис может самостоятельно обрабатывать событие `order.created`.

---

## 🆚 Consumer vs Producer

| Producer               | Consumer                 |
| ---------------------- | ------------------------ |
| Отправляет сообщения   | Читает сообщения         |
| Публикует события      | Обрабатывает события     |
| Записывает в Topic     | Читает из Partition      |
| Может использовать key | Работает с Offset        |
| Создаёт поток событий  | Потребляет поток событий |

---

## 🎯 Главное

```text
Consumer
   ↓
Consumer Group
   ↓
Topic
   ↓
Partition
   ↓
Message
   ↓
Processing
   ↓
Commit Offset
```

Ключевые понятия:

**Consumer** — читает сообщения.

**Consumer Group** — объединяет Consumer'ов для совместной обработки.

**Partition** — единица параллелизма.

**Offset** — позиция Consumer в Partition.

**Consumer Lag** — отставание Consumer от последних сообщений.

**Rebalance** — перераспределение Partition между Consumer'ами.

### Формула для собеседования

**Consumer → читает Partition → обрабатывает сообщение → фиксирует Offset.**

А если нужен параллелизм:

**Больше Partition → больше Consumer'ов в группе могут обрабатывать сообщения параллельно.**
