# 📍 Current Offset в Apache Kafka

## 🎯 Ответ на собеседовании

**Current Offset** — это текущая позиция консьюмера в партиции Kafka, то есть offset следующего сообщения, которое консьюмер будет читать.

Он показывает, **насколько далеко консьюмер продвинулся при чтении партиции**.

Важно отличать его от **Committed Offset**: current offset находится в текущем состоянии консьюмера, а committed offset — сохранённая Kafka позиция, с которой можно продолжить работу после перезапуска.

---

## 🎤 Суперкоротко

```python
Current Offset = текущая позиция чтения консьюмера
```

Например:

```text
Partition:

Offset:   0   1   2   3   4   5   6   7   8   9
          ↑               ↑
        начало       Current Offset
```

Если `Current Offset = 5`, консьюмер находится на позиции **5**.

Следующим он будет читать сообщение с offset `5`.

---

# 🔹 Как это работает

Представим партицию:

```text
0 → 1 → 2 → 3 → 4 → 5 → 6 → 7 → 8 → 9
```

Консьюмер начал читать:

```text
Current Offset = 0
```

Прочитал сообщения `0`, `1`, `2`:

```text
Current Offset = 3
```

Почему `3`, а не `2`?

Потому что offset обычно обозначает **позицию следующего сообщения для чтения**.

То есть:

```text
прочитано:       0  1  2
Current Offset:              3
                              ↓
                         следующее чтение
```

---

# 🔹 Current Offset vs Committed Offset

Это одна из важных вещей для собеседования.

| Понятие                  | Что означает                             |
| ------------------------ | ---------------------------------------- |
| **Current Offset**       | текущая позиция консьюмера               |
| **Committed Offset**     | сохранённая Kafka позиция consumer group |
| **Log End Offset (LEO)** | позиция конца лога партиции              |

Пример:

```text
Partition:

0  1  2  3  4  5  6  7  8  9
            ↑           ↑
         Current      LEO
          Offset
```

Допустим:

```text
Current Offset   = 5
Committed Offset = 4
LEO              = 10
```

Это означает:

* консьюмер сейчас находится на позиции `5`;
* Kafka сохранила позицию `4`;
* в конце партиции находится позиция `10`.

---

# 🔹 Почему Current Offset может отличаться от Committed Offset

Представим:

```text
Committed Offset = 5
Current Offset   = 5
```

Консьюмер получил сообщение `5` и обработал его:

```text
Current Offset = 6
```

Но commit ещё не сделал:

```text
Committed Offset = 5
Current Offset   = 6
```

Если в этот момент консьюмер упадёт, после перезапуска он может снова начать с:

```text
Offset = 5
```

Получается повторная обработка сообщения.

Именно поэтому Kafka часто работает по модели:

```text
at-least-once
```

и приложение должно учитывать возможность **дубликатов**.

---

# 🔹 Current Offset и Consumer Lag

Связь можно представить так:

```text
Current Offset
      ↓
      5
      │
      │      Consumer Lag
      ↓         ↓
0  1  2  3  4  5  6  7  8  9
                         ↑
                        LEO
                       10
```

Упрощённо:

```python
lag ≈ LEO - Current/Committed Offset
```

Но на практике при мониторинге Kafka обычно смотрят на **consumer group's committed/current position относительно Log End Offset**, в зависимости от конкретного инструмента и метрики.

В нашем примере:

```text
LEO = 10
Current Offset = 5

Lag ≈ 10 - 5 = 5
```

---

# 🔹 Current Offset не хранится как отдельное сообщение

Kafka не говорит:

> «Вот это сообщение является Current Offset».

Это **позиция чтения**, которой управляет consumer.

Условно:

```text
Kafka partition
      │
      ├── 0
      ├── 1
      ├── 2
      ├── 3
      ├── 4
      ├── 5  ← current position
      ├── 6
      └── ...
             ↑
          Consumer
```

---

# 🔹 Что происходит после перезапуска

Здесь важен **Committed Offset**.

Допустим:

```text
Current Offset   = 100
Committed Offset = 95
```

Консьюмер упал.

После восстановления Kafka может восстановить позицию с:

```text
95
```

Поэтому сообщения:

```text
95
96
97
98
99
```

могут быть обработаны повторно.

Это нормальное поведение для **at-least-once delivery**.

Отсюда появляется необходимость в:

* idempotency;
* deduplication;
* уникальных ограничениях в БД;
* корректной обработке повторных сообщений.

---

# 🔹 Current Offset и Commit

Типичный цикл:

```text
1. Consumer читает сообщение
          ↓
2. Current Offset двигается
          ↓
3. Consumer обрабатывает сообщение
          ↓
4. Consumer делает commit
          ↓
5. Committed Offset сохраняется
```

Например:

```text
Current Offset   = 10
Committed Offset = 9

        ↓ обработка сообщения 10

Current Offset   = 11
Committed Offset = 9

        ↓ commit

Current Offset   = 11
Committed Offset = 11
```

---

# 🔹 Важный нюанс

**Current Offset не обязательно означает, что сообщение уже успешно обработано бизнес-логикой.**

Консьюмер мог получить сообщение, продвинуть свою текущую позицию, а затем упасть во время обработки.

Поэтому нельзя автоматически считать:

```text
Current Offset = успешно обработанные сообщения
```

Это разные понятия.

Надёжная схема обычно строится вокруг:

```text
прочитал
   ↓
обработал
   ↓
commit
```

и правильного порядка этих операций.

---

# 🔹 Current Offset vs Log End Offset

Это особенно важно после изучения LEO.

### Current Offset

Показывает:

> Где сейчас находится consumer при чтении.

### Log End Offset

Показывает:

> Где заканчивается текущий лог партиции.

Например:

```text
0  1  2  3  4  5  6  7  8  9
            ↑                 ↑
         Current              LEO
         Offset
            5                 10
```

Консьюмер отстаёт:

```text
10 - 5 = 5
```

---

# 🔹 Три offset, которые нужно различать

```text
             Kafka Partition
                  │
0  1  2  3  4  5  6  7  8  9
            ↑                 ↑
            │                 │
       Current Offset         LEO
            │
       ┌────┘
       │
Committed Offset
```

На практике:

```text
Current Offset
    ↓
текущая позиция consumer

Committed Offset
    ↓
сохранённая позиция consumer group

LEO
    ↓
конец partition log
```

---

# 🧠 Главное

```text
Current Offset
        ↓
текущая позиция чтения

Committed Offset
        ↓
сохранённая позиция

Log End Offset
        ↓
конец лога
```

Пример:

```text
Current Offset   = 50
Committed Offset = 48
LEO              = 60
```

Значит:

```text
Consumer сейчас → 50
Сохранённая позиция → 48
Конец партиции → 60
```

Именно поэтому при падении consumer может вернуться к `48` и повторно обработать сообщения.

---

## 🎯 Формула для собеседования

```text
Current Offset
= текущая позиция чтения consumer

Committed Offset
= последняя сохранённая позиция consumer group

LEO
= offset следующей записи в конце partition
```

А упрощённо:

```text
Consumer Lag ≈ LEO - Consumer Offset
```
