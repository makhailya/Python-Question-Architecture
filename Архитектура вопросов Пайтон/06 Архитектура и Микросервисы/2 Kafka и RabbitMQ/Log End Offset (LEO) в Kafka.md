# Log End Offset (LEO) в Kafka 📍

## 🎯 Ответ на собеседовании

**Log End Offset (LEO) — это offset следующей записи, которая будет добавлена в конец Partition.**

Иными словами, LEO показывает, **где сейчас находится конец лога Partition**.

Например:

```text
Partition 0:

offset 0
offset 1
offset 2
offset 3
offset 4
        ↑
       LEO = 5
```

Последняя существующая запись имеет:

```text
offset = 4
```

а **LEO равен 5**, потому что `5` — это следующий Offset, который будет назначен новой записи.

---

## 🎤 Суперкоротко

```text
LEO = следующий Offset после последней записи
```

Если:

```text
Последняя запись = offset 99
```

то:

```text
LEO = 100
```

---

# Почему LEO называется End Offset

Kafka хранит Partition как последовательный log:

```text
Partition 0

0 → 1 → 2 → 3 → 4 → 5 → ...
```

LEO указывает на **конец этого лога**.

Например:

```text
0  1  2  3  4
         ↑
      последняя запись

LEO = 5
```

---

# LEO — это не Offset последнего сообщения

Это одна из самых частых ошибок.

Если в Partition есть:

```text
offset 0
offset 1
offset 2
offset 3
```

то:

```text
последний offset = 3
LEO = 4
```

То есть:

```text
LEO = last_offset + 1
```

---

# Пример

Представим Partition:

```text
P0:

offset 0 → event A
offset 1 → event B
offset 2 → event C
offset 3 → event D
offset 4 → event E
```

Последняя запись:

```text
offset = 4
```

Следующий свободный Offset:

```text
5
```

Поэтому:

```text
LEO = 5
```

---

# LEO и Consumer

Допустим:

```text
LEO = 1000
```

Consumer Group имеет позицию:

```text
offset = 800
```

Упрощённо:

```text
1000 - 800 = 200
```

Получаем:

```text
Consumer Lag ≈ 200
```

То есть Consumer отстаёт от конца Partition.

---

# Важный нюанс: Current Position и Committed Offset

При расчёте lag нужно понимать, какую позицию мы сравниваем с LEO.

Есть:

```text
LEO
```

и позиция Consumer Group:

```text
committed offset
```

Например:

```text
LEO = 1000
Committed Offset = 800
```

Концептуально:

```text
Lag = 1000 - 800 = 200
```

Но в реальных инструментах мониторинга конкретное определение текущего/коммитнутого offset может отличаться, поэтому важно смотреть документацию конкретного инструмента.

---

# Визуально

```text
Partition 0

0  1  2  3  4  5  6  7  8  9
                  ↑           ↑
             Consumer       LEO
              offset=6      =10
```

Получаем:

```text
LEO = 10
Consumer offset = 6

Lag = 10 - 6 = 4
```

---

# Что происходит при записи нового сообщения

Было:

```text
id="5j9k8x"
0  1  2  3  4
            ↑
          LEO=5
```

Producer добавляет сообщение.

Новая запись получает:

```text
offset = 5
```

Теперь:

```text
0  1  2  3  4  5
               ↑
             LEO=6
```

То есть после записи:

```text
LEO увеличился с 5 до 6
```

---

# LEO и Producer

Producer добавляет записи в конец Partition.

Например:

```text
Producer
   ↓
Partition
   ↓
LEO
```

Если Producer продолжает писать:

```text
LEO:
100
101
102
103
...
```

конец лога постоянно перемещается вправо.

---

# LEO и Consumer Lag

Представим Producer пишет быстрее Consumer:

```text
Producer
   ↓
LEO растёт быстро

Consumer
   ↓
offset растёт медленно
```

Получаем:

```text
LEO       = 10000
Consumer  = 7000

Lag       = 3000
```

Если ситуация сохраняется:

```text
Lag:
100
500
1000
2000
3000
...
```

это сигнал, что Consumer Group не успевает за поступлением данных.

---

# Когда LEO может расти, а Consumer Lag не растёт

Если Consumer успевает за Producer:

```text
LEO       = 1000
Consumer  = 995
```

через некоторое время:

```text
LEO       = 1100
Consumer  = 1095
```

Lag остаётся примерно:

```text
5
```

То есть сам по себе высокий LEO ничего не говорит о проблеме.

Важно смотреть **разницу** между концом лога и позицией Consumer.

---

# LEO и Retention

Kafka удаляет старые данные согласно политике Retention.

Например:

```text
Partition

100 101 102 103 104 105 106
 ↑                       ↑
старые                   LEO
```

После удаления старых сегментов начало доступных данных может сдвинуться.

Но:

```text
LEO
```

отражает конец текущего log.

Поэтому:

```text
Retention
→ влияет на доступность старых данных

LEO
→ показывает конец log
```

---

# LEO и Offset

Очень важно различать:

### Offset записи

```text
offset = 42
```

Позиция конкретной записи.

### LEO

```text
LEO = 43
```

Конец Partition после этой записи.

Получается:

```text
offset 42 → последняя существующая запись
LEO 43    → следующая позиция
```

---

# LEO и High Watermark

В Kafka существует ещё одно важное понятие — **High Watermark (HW)**.

Не стоит смешивать:

```text
LEO
HW
Consumer Offset
```

Упрощённо:

```text
LEO
↓
конец локального log

High Watermark
↓
граница записей, доступных Consumer'ам
```

LEO и High Watermark могут отличаться, например во время репликации.

---

# LEO vs High Watermark

Упрощённая схема:

```text
Partition:

0  1  2  3  4  5  6  7
            ↑        ↑
            HW       LEO
```

Например:

```text
HW  = 5
LEO = 8
```

Это означает, что локальный log может содержать записи дальше High Watermark, но Consumer не обязательно сможет читать их как доступные записи.

**High Watermark связан с репликацией и безопасной видимостью записей, а LEO — с концом локального log.**

---

# Почему LEO важен

LEO используется как ориентир для понимания:

* насколько далеко продвинулся log;
* сколько данных находится после позиции Consumer;
* Consumer Lag;
* состояния Partition;
* работы репликации.

Особенно важно:

```text
LEO
 ↓
Consumer Lag
 ↓
Monitoring
```

---

# Пример из production

Допустим:

```text
Topic: orders
Partition: 3
```

Метрики:

```text
LEO = 1 500 000
Consumer Offset = 1 490 000
```

Получаем:

```text
Lag = 10 000
```

Если lag продолжает расти:

```text
10k
20k
50k
100k
```

значит Consumer Group всё сильнее отстаёт.

Тогда ищем причину:

```text
Consumer
   ↓
processing
   ↓
DB / API / CPU / network
```

---

# LEO не означает количество сообщений

Например, если Partition начинается с Offset:

```text
1000
```

и LEO:

```text
1500
```

это не значит, что в Partition обязательно ровно 1500 доступных сообщений.

Доступный диапазон может быть:

```text
1000 ... 1499
```

то есть около:

```text
500 записей
```

Потому что старые Offset могли быть удалены Retention.

---

# Важный момент после Retention

Допустим:

```text
LEO = 1000
```

а старые записи:

```text
0 ... 499
```

уже удалены.

Тогда доступные записи начинаются примерно с:

```text
500
```

и заканчиваются перед:

```text
1000
```

Получается:

```text
Log Start Offset ≈ 500
LEO = 1000
```

Это показывает ещё одну важную границу Partition.

---

# LEO и Log Start Offset

Можно представить Partition так:

```text
Log Start                          LEO
   ↓                                ↓
   500 501 502 ... 997 998 999
   │                                │
   └──────── доступный log ─────────┘
```

### Log Start Offset

Начало доступного log.

### LEO

Конец log.

Таким образом:

```text
Log Start Offset
       ↓
   available log
       ↓
      LEO
```

---

# 🎯 Частые вопросы на собеседовании

### Что такое LEO?

> Log End Offset — Offset следующей записи, которая будет добавлена в конец Partition.

### Если последняя запись имеет Offset 99, чему равен LEO?

```text
100
```

### LEO — это Offset последнего сообщения?

Нет.

```text
last offset = LEO - 1
```

если речь идёт о непрерывной последовательности.

### Как LEO связан с Lag?

Упрощённо:

```text
Lag = LEO - Consumer Offset
```

### Может ли LEO расти?

Да. Каждая новая запись в Partition увеличивает LEO.

### Удаляет ли LEO сообщения?

Нет. LEO — это метрика/позиция конца log, а не механизм удаления.

### Чем LEO отличается от High Watermark?

> LEO показывает конец локального log, а High Watermark связан с границей записей, доступных для чтения с учётом состояния репликации.

### Чем LEO отличается от Consumer Offset?

> LEO показывает конец Partition, а Consumer Offset показывает позицию Consumer.

---

# 🧠 Сводная схема

```text
                    Kafka Partition

Log Start                                LEO
    ↓                                     ↓
    500  501  502  503  ...  998  999
     │                              │
     │                         последняя запись
     │                              │
     └──────── available log ───────┘
                                     
                                      1000 ← LEO
```

Consumer:

```text
Consumer Offset = 900
LEO             = 1000

Lag ≈ 100
```

### Связь основных понятий

```text
Producer
   ↓
добавляет записи
   ↓
LEO растёт
   ↓
Consumer читает
   ↓
Consumer Offset растёт
   ↓
LEO - Consumer Offset
   ↓
Consumer Lag
```

### Формула для собеседования

> **Log End Offset — это Offset следующей записи в конце Partition. Если последняя запись имеет Offset 99, LEO равен 100. LEO используется как ориентир конца лога и участвует в расчёте Consumer Lag: концептуально `Lag = LEO - Consumer Offset`.**
