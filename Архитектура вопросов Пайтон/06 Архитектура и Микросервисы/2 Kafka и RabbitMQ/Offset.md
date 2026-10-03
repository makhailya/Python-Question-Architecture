# Offset в Kafka 📍

## 🎯 Ответ на собеседовании

**Offset** — это уникальная последовательная позиция сообщения внутри конкретной Kafka partition.

Consumer использует offset, чтобы понимать, **на каком месте он находится при чтении partition**.

Например:

```text id="h7k3xp"
Partition 0

Offset 0 → Event A
Offset 1 → Event B
Offset 2 → Event C
Offset 3 → Event D
Offset 4 → Event E
```

Если consumer обработал сообщения до offset `2`, он знает, с какого места продолжать чтение.

> **Offset принадлежит partition, а прогресс consumer group хранится отдельно для каждой partition.**

---

## 🎤 Суперкоротко

```text id="q4m8vz"
Partition

0 → A
1 → B
2 → C
3 → D
4 → E
    ↑
  offset
```

**Offset = позиция сообщения внутри partition.**

---

# 1. Offset не является глобальным ID сообщения

Это важный момент.

Offset уникален **только внутри конкретной partition**.

Например:

```text id="x9c5mn"
Partition 0:

offset 0 → A
offset 1 → B
offset 2 → C
```

И:

```text id="r7k2qp"
Partition 1:

offset 0 → X
offset 1 → Y
offset 2 → Z
```

`offset = 2` существует в обеих partitions.

Поэтому сообщение идентифицируется комбинацией:

```text id="m5v8cx"
(topic, partition, offset)
```

Например:

```text id="d3q7kn"
orders, 2, 154
```

---

# 2. Как Consumer использует Offset

Допустим:

```text id="w6p3za"
Partition 0

0 → A
1 → B
2 → C
3 → D
4 → E
```

Consumer читает:

```text id="k8x2mv"
A → B → C
```

Его позиция находится около:

```text id="f4n9qp"
offset = 2
```

После следующего poll он может получить:

```text id="v7m3xc"
D → E
```

Таким образом consumer последовательно продвигается по partition.

---

# 3. Committed Offset 💾

Kafka должна знать, **до какого места consumer group зафиксировала свой прогресс**.

Для этого используется **committed offset**.

Например:

```text id="p5q8zr"
Partition 0:

0 → A
1 → B
2 → C
3 → D
4 → E

Committed offset → 3
```

Это означает, что группа сохранила прогресс чтения примерно до этой позиции.

При перезапуске consumer Kafka может продолжить работу с сохранённого offset.

---

# 4. Offset и Consumer Group

Offset связан не просто с consumer'ом, а с его **Consumer Group**.

Допустим:

```text id="c7m4xn"
Topic: orders

Partition 0
Partition 1
```

Есть группа:

```text id="z8q2vp"
Group: billing
```

Она может иметь:

```text id="n3w6ky"
P0 → offset 100
P1 → offset 250
```

Другая группа:

```text id="a5r9mx"
Group: analytics

P0 → offset 80
P1 → offset 200
```

Они читают один и тот же topic, но находятся на разных позициях.

---

# 5. Offset после перезапуска 🔄

Это одна из главных причин существования committed offsets.

Сценарий:

```text id="u4k8zp"
Consumer
   ↓
читает сообщения
   ↓
commit offset
   ↓
Consumer CRASH 💥
```

После запуска:

```text id="j6m2qx"
Consumer
   ↓
получает сохранённый offset
   ↓
продолжает чтение
```

То есть Kafka не обязана начинать чтение topic с самого начала.

---

# 6. Что такое Log End Offset?

**Log End Offset (LEO)** — позиция конца partition, то есть место, до которого Kafka уже записала сообщения.

Например:

```text id="q9v3mk"
Partition:

0 → A
1 → B
2 → C
3 → D
4 → E
```

Условно:

```text id="s7x5np"
Log End Offset → 5
```

Здесь важно не путать:

```text id="m8k2zr"
Log End Offset
→ конец доступного лога

Committed Offset
→ прогресс consumer group
```

---

# 7. Offset и Consumer Lag 📊

Теперь можно точно понять Lag.

Упрощённо:

```text id="y5q8cx"
Lag =
Log End Offset
−
Consumer Group Offset
```

Например:

```text id="g4m7vp"
Log End Offset      = 1000
Committed Offset    = 900

Lag = 100
```

Схематично:

```text id="x8n3mq"
0 ─────────────────────── 900 ───────────── 1000
                          ↑                  ↑
                       Consumer             LEO
                       
                       ←── Lag = 100 ──→
```

Поэтому:

> **Consumer Lag показывает, насколько consumer group отстаёт от конца partition.**

---

# 8. Current Position vs Committed Offset

Это важный нюанс.

У consumer есть **текущая позиция чтения**, а отдельно существует **committed offset**.

Например:

```text id="k3v7xp"
Consumer уже прочитал:
0 → 1 → 2 → 3 → 4

Current position = 5

Но commit ещё не сделал:

Committed offset = 3
```

Получается:

```text id="w6m2qz"
Current position
      ↓
      5

Committed offset
      ↓
      3
```

Если consumer сейчас упадёт, после восстановления он может продолжить с **committed offset**, а не с текущей позиции.

Отсюда потенциально появляются **повторные обработки**.

---

# 9. Почему появляются дубликаты?

Рассмотрим:

```text id="r8p4my"
1. Consumer получил сообщение
2. Обработал сообщение
3. База данных успешно обновилась
4. Consumer ещё НЕ успел commit offset
5. Consumer упал 💥
```

После восстановления:

```text id="q5n7vx"
Kafka считает:
"offset ещё не committed"

        ↓

Consumer получает сообщение снова
```

Получается:

```text id="m4z8kc"
Message
  ↓
обработка
  ↓
CRASH
  ↓
повторная обработка
```

Поэтому Kafka-приложения часто должны быть **идемпотентными**.

Это напрямую связано с темами:

```text id="x7p3mn"
Retry
Idempotency
Deduplication
Unique Constraint
```

---

# 10. Auto Commit

Kafka consumer может автоматически фиксировать offsets.

Например, концептуально:

```python id="k8r4vq"
consumer = KafkaConsumer(
    "orders",
    enable_auto_commit=True,
)
```

При `auto commit` consumer периодически коммитит offset автоматически.

Плюс:

* меньше ручного кода.

Минус:

* сложнее точно контролировать момент фиксации offset;
* возможны повторные или потенциально пропущенные обработки в зависимости от момента commit и архитектуры обработки.

Для критичных систем важно понимать, **когда именно offset считается подтверждённым относительно бизнес-операции**.

---

# 11. Manual Commit

Можно управлять commit самостоятельно.

Концептуально:

```python id="p7m2xz"
for message in messages:
    process(message)

    consumer.commit()
```

Идея:

```text id="v9q4kc"
Получили сообщение
      ↓
Обработали
      ↓
Успешно
      ↓
Commit offset
```

Это позволяет связать продвижение offset с успешной обработкой сообщения.

---

# 12. At-Least-Once и Offset

На практике Kafka часто используется с семантикой:

```text id="c5x8mn"
At-Least-Once
```

То есть сообщение может быть обработано повторно.

Сценарий:

```text id="z4q7vp"
Read
 ↓
Process
 ↓
Commit
```

Если `Process` завершился успешно, но `Commit` не произошёл:

```text id="h6m3xr"
Process ✓
Commit  ✗
```

После перезапуска:

```text id="w8k2mq"
Read again
   ↓
Process again
```

Поэтому consumer должен быть готов к повторной обработке.

---

# 13. auto.offset.reset

Этот параметр определяет, **откуда начинать чтение, если у Consumer Group нет подходящего committed offset**.

Основные варианты:

```text id="n7x4qp"
earliest
latest
```

### `earliest`

Начать с самого раннего доступного сообщения.

```text id="m5k8vz"
0 → 1 → 2 → 3 → 4
↑
start
```

Используется, например, когда нужно обработать доступную историю.

### `latest`

Начать с конца доступного лога.

```text id="q3r7xm"
0 → 1 → 2 → 3 → 4
                  ↑
                start
```

Consumer будет получать новые сообщения.

### Важно

`auto.offset.reset` **не означает «каждый раз начинать отсюда»**.

Он используется, когда для группы нет подходящего offset.

---

# 14. Offset можно сбросить

Иногда нужно заново обработать события.

Например:

```text id="y8m3kc"
Consumer Group
      ↓
reset offsets
      ↓
earliest
      ↓
перечитать историю
```

Это используется для:

* повторной обработки данных;
* восстановления после ошибок;
* пересчёта аналитики;
* тестирования.

Например:

```text id="p4v7xn"
orders topic
      ↓
analytics group
      ↓
offset → earliest
      ↓
replay событий
```

---

# 15. Offset и Retention

Offset существует независимо от того, сколько Kafka хранит сами сообщения.

Например:

```text id="s6q2mw"
Kafka retention = 7 дней
```

После удаления старых сообщений:

```text id="j8x4kp"
старые offsets
      ↓
могут больше не соответствовать
доступным данным
```

Если consumer слишком долго не читает partition и его offset указывает на уже удалённые данные, поведение зависит от настроек и доступных offset'ов; в частности, может сработать `auto.offset.reset`.

---

# 16. Offset и порядок сообщений

Offset позволяет определить порядок сообщений **внутри partition**:

```text id="u5n8qc"
0 → Event A
1 → Event B
2 → Event C
3 → Event D
```

Поэтому:

```text id="k7m3vp"
offset 1
```

идёт раньше:

```text id="f8x4zn"
offset 2
```

Но между partitions глобального порядка нет:

```text id="a6q9mc"
P0:
0 → A
1 → B

P1:
0 → X
1 → Y
```

Нельзя сказать, что:

```text id="d3w7kp"
A → B → X → Y
```

является глобальным порядком Kafka.

---

# 17. Главная схема Offset

```text id="v9m4xq"
                  Kafka Partition

0    1    2    3    4    5    6    7    8
│    │    │    │    │    │    │    │    │
A    B    C    D    E    F    G    H    I
                         ↑
                  Committed Offset
                         
                              ↑
                         Current Position

                                            ↑
                                      Log End Offset
```

Упрощённо:

```text id="p5k8mz"
Committed Offset
       ↓
       5
       │
       │ ←── Consumer Lag ──→
       │
       9
       ↑
Log End Offset
```

---

# 18. Связь с Consumer Group

```text id="x7q3vn"
                 Kafka
                   │
                 Topic
                   │
             ┌─────┴─────┐
             ↓           ↓
            P0          P1
             │           │
             ↓           ↓
          Consumer    Consumer
             │           │
             └─────┬─────┘
                   ↓
             Consumer Group
                   │
                   ↓
                Offsets
                   │
                   ↓
             Consumer Lag
```

---

# ⚠️ Частые ошибки на собеседовании

### ❌ «Offset — это ID сообщения во всей Kafka»

Нет.

> **Offset уникален только внутри partition.**

---

### ❌ «Consumer commit удаляет сообщение»

Нет.

```text id="q4m8xz"
Commit offset
      ↓
сохраняет прогресс consumer group
```

Сообщение продолжает храниться в Kafka согласно retention policy.

---

### ❌ «Если сообщение прочитали, offset автоматически становится committed»

Не обязательно.

Есть разница между:

```text id="n8p3yk"
прочитать
```

и:

```text id="m6v9qc"
commit offset
```

---

### ❌ «Если consumer упал, Kafka потеряла сообщение»

Обычно нет.

Если offset не был зафиксирован, сообщение может быть прочитано повторно.

---

# 🧠 Шпаргалка

```text id="c8m4vz"
Offset
→ позиция сообщения внутри partition

Committed Offset
→ сохранённый прогресс Consumer Group

Current Position
→ текущая позиция чтения consumer

Log End Offset
→ конец доступного лога partition

Consumer Lag
→ Log End Offset − Consumer Group Offset

auto.offset.reset
→ откуда начинать чтение,
  если нет подходящего offset
```

### Главная цепочка

```text id="r7x2mp"
Message
   ↓
Partition
   ↓
Offset
   ↓
Consumer читает
   ↓
Process
   ↓
Commit Offset
   ↓
Consumer Group сохраняет прогресс
   ↓
Lag показывает отставание
```

> **Offset — это механизм позиционирования consumer'а в Kafka. Consumer Group хранит прогресс по каждой partition, а Consumer Lag показывает разницу между этим прогрессом и концом лога.**
