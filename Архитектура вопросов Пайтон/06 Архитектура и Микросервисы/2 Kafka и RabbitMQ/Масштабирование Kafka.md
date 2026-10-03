# Масштабирование Kafka 🚀

## 🎯 Ответ на собеседовании

**Kafka масштабируется горизонтально:** данные распределяются по **партициям**, партиции — по **broker'ам**, а обработка масштабируется через **consumer groups**.

Основные уровни масштабирования:

* **Broker'ы** — увеличиваем количество серверов Kafka.
* **Partitions** — увеличиваем параллелизм внутри topic.
* **Consumers** — увеличиваем количество обработчиков.
* **Replication Factor** — повышаем отказоустойчивость, но увеличиваем нагрузку на сеть и диск.

Ключевая зависимость:

```text
Количество partitions
        ↓
максимальный параллелизм
        ↓
количество одновременно работающих consumers
```

В одной **consumer group** один partition в конкретный момент времени обрабатывается только одним consumer.

---

## 🎤 Суперкоротко

> Kafka масштабируется горизонтально за счёт broker'ов и partitions.
> Partitions позволяют распараллеливать запись и чтение.
> В consumer group каждый partition назначается одному consumer, поэтому количество активных consumers не может эффективно превышать количество partitions.
> Для отказоустойчивости используется replication factor.

---

# 1. Что вообще значит масштабирование Kafka?

Представим:

```text
Producer
   ↓
Kafka
   ↓
Consumer
```

При небольшом количестве сообщений этого достаточно.

Но нагрузка выросла:

```text
10 000 сообщений/сек
        ↓
     Kafka
        ↓
   Consumer
```

Один consumer может уже не справляться.

Тогда Kafka позволяет распределить нагрузку:

```text
                Kafka
                  │
        ┌─────────┼─────────┐
        ↓         ↓         ↓
   Partition  Partition  Partition
       0          1          2
        ↓         ↓         ↓
       C1        C2         C3
```

Теперь три consumer'а работают параллельно.

---

# 2. Горизонтальное масштабирование

Kafka в первую очередь масштабируется **горизонтально**.

То есть вместо того, чтобы бесконечно увеличивать мощность одного сервера, добавляем новые broker'ы.

### Было

```text
Broker 1
 ├── topic-A-0
 ├── topic-A-1
 └── topic-A-2
```

### Стало

```text
Broker 1          Broker 2          Broker 3
   │                 │                 │
   ├─ partition 0    ├─ partition 1   ├─ partition 2
   └─ partition 3    └─ partition 4   └─ partition 5
```

Нагрузка распределяется между серверами.

---

# 3. Что такое Broker в масштабировании?

**Broker** — сервер Kafka, который хранит и обслуживает partitions.

Kafka-кластер:

```text
Kafka Cluster

┌──────────┐
│ Broker 1 │
└──────────┘

┌──────────┐
│ Broker 2 │
└──────────┘

┌──────────┐
│ Broker 3 │
└──────────┘
```

Добавление broker'ов позволяет:

* распределять partitions;
* распределять нагрузку на CPU;
* распределять нагрузку на RAM;
* распределять дисковую нагрузку;
* увеличивать суммарную пропускную способность;
* повышать отказоустойчивость.

Но просто добавить broker недостаточно.

Kafka должна **распределить partitions** между ними.

---

# 4. Масштабирование через Partitions

Это один из главных механизмов Kafka.

Например:

```text
Topic orders

Partition 0
Partition 1
Partition 2
```

Три partition позволяют обрабатывать данные параллельно.

```text
Producer
   │
   ├────→ P0
   ├────→ P1
   └────→ P2
```

При увеличении partitions:

```text
3 partitions
      ↓
6 partitions
      ↓
больше потенциального параллелизма
```

---

# 5. Почему partitions дают масштабирование?

Потому что каждый partition — независимый последовательный log.

Например:

```text
P0: A B C D E
P1: F G H I J
P2: K L M N O
```

Consumer'ы могут читать их одновременно:

```text
C1 → P0
C2 → P1
C3 → P2
```

Таким образом:

```text
              Kafka
                │
       ┌────────┼────────┐
       ↓        ↓        ↓
      P0       P1       P2
       ↓        ↓        ↓
      C1       C2       C3
```

---

# 6. Partitions ↔ Consumers

Очень важная тема для собеседования.

Внутри одной consumer group:

```text
Partitions = 3
Consumers  = 3
```

Получаем:

```text
P0 → C1
P1 → C2
P2 → C3
```

Максимальный параллелизм используется.

---

## Consumers меньше partitions

```text
Partitions = 6
Consumers  = 2
```

Например:

```text
C1 → P0 P1 P2
C2 → P3 P4 P5
```

Каждый consumer обрабатывает несколько partitions.

---

## Consumers равно partitions

```text
Partitions = 3
Consumers  = 3
```

```text
C1 → P0
C2 → P1
C3 → P2
```

Оптимальный вариант для такого количества partitions.

---

## Consumers больше partitions

```text
Partitions = 3
Consumers  = 5
```

Получится примерно:

```text
C1 → P0
C2 → P1
C3 → P2

C4 → idle
C5 → idle
```

Часть consumers не получает partitions.

### Главное

> В одной consumer group количество одновременно работающих consumers эффективно ограничено количеством partitions.

---

# 7. Почему нельзя просто добавлять consumers?

Допустим:

```text
Topic
10 partitions

Consumer Group
10 consumers
```

Все consumers работают.

Добавляем ещё:

```text
10 partitions
20 consumers
```

Результат:

```text
10 consumers → работают
10 consumers → idle
```

Поэтому:

```text
Consumers > Partitions
```

не увеличивает параллелизм обработки внутри этой группы.

---

# 8. Пример масштабирования Consumer Group

Допустим, приложение обрабатывает события заказов.

Было:

```text
orders
├── P0
└── P1

Consumers:
C1
C2
```

```text
P0 → C1
P1 → C2
```

Нагрузка выросла.

Увеличиваем partitions:

```text
orders
├── P0
├── P1
├── P2
├── P3
├── P4
└── P5
```

И consumers:

```text
C1 → P0
C2 → P1
C3 → P2
C4 → P3
C5 → P4
C6 → P5
```

Теперь обработка может идти параллельно в 6 потоков на уровне consumers.

---

# 9. Масштабирование Producer

Producer тоже можно масштабировать.

Например:

```text
Producer 1 ─┐
Producer 2 ─┼──→ Kafka
Producer 3 ─┘
```

Kafka принимает записи параллельно.

Producer определяет, в какой partition попадёт сообщение.

Обычно используется:

* partition key;
* partitioner;
* явное указание partition.

Например:

```python
producer.send(
    "orders",
    key="user-123",
    value="order-created",
)
```

Если используется ключ, связанные сообщения могут попадать в одну partition.

Это важно для сохранения порядка сообщений по конкретному ключу.

---

# 10. Масштабирование Broker'ов

Представим:

```text
Broker 1
 ├── P0
 ├── P1
 ├── P2
 └── P3
```

Он перегружен.

Добавляем:

```text
Broker 1       Broker 2       Broker 3
   │               │               │
  P0              P1              P2
  P3              P4              P5
```

Но существующие partitions автоматически не превращаются в идеально сбалансированную систему только от факта добавления broker'а.

Нужно учитывать **распределение partitions и реплик**.

---

# 11. Replication Factor

Kafka поддерживает репликацию partitions.

Например:

```text
Replication Factor = 3
```

Partition имеет:

```text
P0
├── Replica → Broker 1
├── Replica → Broker 2
└── Replica → Broker 3
```

Если один broker упадёт:

```text
Broker 1 ❌

Broker 2 → replica
Broker 3 → replica
```

Данные остаются доступными при корректной настройке кластера.

---

# 12. Масштабирование и Replication Factor

Replication Factor повышает:

* отказоустойчивость;
* доступность данных;
* устойчивость к отказу broker'ов.

Но увеличивает:

* использование диска;
* сетевой трафик;
* нагрузку на broker'ы;
* стоимость инфраструктуры.

Например:

```text
100 GB данных

RF = 1
≈ 100 GB

RF = 3
≈ 300 GB
```

Упрощённо, без учёта дополнительных накладных расходов.

---

# 13. Throughput

**Throughput** — сколько данных система способна обработать за единицу времени.

Например:

```text
Kafka:
100 000 messages/sec
```

Масштабирование partitions и broker'ов может увеличить throughput.

Упрощённо:

```text
1 partition
    ↓
ограниченный throughput

10 partitions
    ↓
больше параллелизма

10 partitions + несколько brokers
    ↓
ещё больше потенциального throughput
```

Но это не означает:

```text
10 partitions = ровно 10× throughput
```

Реальная производительность зависит от:

* размера сообщений;
* дисков;
* сети;
* CPU;
* compression;
* replication;
* producer;
* consumer;
* настроек batching;
* скорости обработки downstream-систем.

---

# 14. Kafka масштабирует не только чтение

Важно разделять:

```text
Producer scaling
        ↓
запись в partitions

Broker scaling
        ↓
хранение и обслуживание данных

Partition scaling
        ↓
параллелизм

Consumer scaling
        ↓
обработка данных
```

Полная система должна масштабироваться по всей цепочке.

---

# 15. Что происходит при росте нагрузки?

Допустим:

```text
Producer
1000 msg/s
```

Consumer успевает:

```text
1000 msg/s
```

Lag:

```text
≈ 0
```

Нагрузка выросла:

```text
Producer
10 000 msg/s
```

А consumer всё ещё:

```text
1000 msg/s
```

Тогда:

```text
Producer
10 000 msg/s
       ↓
     Kafka
       ↓
Consumer
1 000 msg/s

       ↓
   Consumer Lag ↑
```

Что можно сделать:

```text
1. увеличить consumers
2. если partitions недостаточно → увеличить partitions
3. оптимизировать обработку
4. масштабировать downstream
5. увеличить Kafka cluster при необходимости
```

---

# 16. Важная зависимость

Главная схема:

```text
Partitions
    ↓
определяют максимальный параллелизм
    ↓
Consumers в одной группе
    ↓
обрабатывают partitions параллельно
```

Например:

```text
2 partitions
10 consumers
```

Не получится эффективно обработать 10 partitions параллельно, потому что их всего 2.

А:

```text
10 partitions
2 consumers
```

означает, что каждый consumer может получить несколько partitions.

---

# 17. Можно ли увеличить количество partitions?

Да.

Например:

```text
orders:
3 partitions
```

можно расширить:

```text
orders:
6 partitions
```

Это позволяет увеличить потенциальный параллелизм.

Но есть важный нюанс:

> Увеличение количества partitions может повлиять на распределение сообщений по ключам и порядок обработки.

Если приложение рассчитывает на определённое распределение ключей, изменение числа partitions нужно делать осознанно.

---

# 18. Почему порядок важен?

Kafka гарантирует порядок сообщений **внутри одной partition**.

```text
P0:

1 → 2 → 3 → 4
```

Порядок сохраняется.

Но между partitions:

```text
P0: 1 → 3 → 5
P1: 2 → 4 → 6
```

Kafka не гарантирует глобальный порядок:

```text
1, 2, 3, 4, 5, 6
```

Поэтому если важен порядок событий конкретного объекта, часто используют key:

```text
user_id
order_id
account_id
```

Например:

```text
order_id = 123

event 1 → P2
event 2 → P2
event 3 → P2
```

Тогда события этого заказа находятся в одной partition и сохраняют порядок внутри неё.

---

# 19. Масштабирование Kafka: общая схема

```text
                    Kafka Cluster
                         │
          ┌──────────────┼──────────────┐
          ↓              ↓              ↓
      Broker 1       Broker 2       Broker 3
          │              │              │
       P0 P3           P1 P4           P2 P5
          │              │              │
          └──────────────┼──────────────┘
                         ↓
                  Consumer Group
                         │
          ┌──────────────┼──────────────┐
          ↓              ↓              ↓
         C1             C2             C3
```

Здесь:

* **Broker** — физический узел Kafka.
* **Partition** — единица хранения и параллелизма.
* **Consumer** — обработчик.
* **Consumer Group** — логическая группа обработчиков.
* **Replication** — копии partitions для отказоустойчивости.

---

# 20. Что масштабировать при проблеме?

| Проблема                | Что смотреть                                        |
| ----------------------- | --------------------------------------------------- |
| Consumer Lag растёт     | Consumers / partitions / скорость обработки         |
| Broker перегружен       | Добавление broker'ов / перераспределение partitions |
| Не хватает параллелизма | Увеличение partitions                               |
| Consumers простаивают   | Возможно, partitions слишком мало                   |
| Много данных            | Retention / диски / broker'ы                        |
| Broker падает           | Replication Factor                                  |
| Producer не успевает    | Producer instances / batching / network             |
| Consumer не успевает    | Consumer instances / обработка / downstream         |

---

# 21. Важное ограничение

Kafka нельзя масштабировать бесконечно простым увеличением всего подряд.

Например:

```text
+ partitions
+ consumers
+ brokers
```

не гарантирует пропорционального роста производительности.

Появляются bottleneck'и:

```text
Producer
   ↓
Kafka
   ↓
Consumer
   ↓
Database
```

Если Kafka может обработать:

```text
100 000 msg/s
```

но PostgreSQL может принять:

```text
20 000 msg/s
```

то реальный bottleneck:

```text
PostgreSQL
```

И увеличение consumers только увеличит нагрузку на БД.

---

# 22. Kafka Scaling vs Consumer Scaling

Это часто путают.

### Масштабирование Kafka

Увеличиваем:

```text
Broker
Partition
Replication
```

Цель:

> увеличить ёмкость, throughput и отказоустойчивость Kafka.

### Масштабирование consumers

Увеличиваем:

```text
Consumer instances
```

Цель:

> увеличить скорость обработки сообщений.

Но consumers ограничены количеством partitions:

```text
10 partitions
↓
до 10 активных consumers в одной группе
```

---

# 23. Типичный сценарий на собеседовании

**Вопрос:**

> Consumer Lag постоянно растёт. Что будете делать?

Хороший ответ:

> Сначала проверю, действительно ли consumer не успевает обрабатывать сообщения, и посмотрю lag по partitions. Затем проверю нагрузку и состояние downstream-систем — например, БД или внешнего API. Если проблема именно в недостатке consumer throughput, увеличу количество consumers, но сначала проверю количество partitions. Если partitions недостаточно для нужного параллелизма, потребуется увеличить их количество. При этом нужно учитывать порядок сообщений, распределение по ключам и нагрузку на downstream.

---

# 24. Главное для собеседования

```text
Kafka масштабируется горизонтально.

Broker'ы
    ↓
масштабируют кластер

Partitions
    ↓
дают параллелизм

Consumers
    ↓
обрабатывают partitions

Consumer Group
    ↓
распределяет partitions между consumers

Replication Factor
    ↓
даёт отказоустойчивость
```

### Самая важная формула

```text
Максимальный параллелизм consumer group
≈ количество partitions
```

Поэтому:

```text
Partitions < Consumers
→ часть consumers простаивает

Partitions = Consumers
→ каждый consumer может получить partition

Partitions > Consumers
→ consumers обрабатывают несколько partitions
```

### И главное различие

```text
Partitions → параллелизм
Consumers  → обработка
Brokers    → инфраструктурная ёмкость
Replication → отказоустойчивость
```
