# 🔄 Shared Mutable State

## 🎤 Короткий ответ

**Shared Mutable State** — это **общее изменяемое состояние**, к которому могут обращаться несколько конкурентных задач: потоков, процессов или других участников выполнения.

Состоит из трёх частей:

* **Shared** — несколько задач имеют доступ к одному состоянию.
* **Mutable** — это состояние можно изменить.
* **State** — данные, от которых зависит поведение программы.

В многопоточности shared mutable state — один из основных источников **Race Condition** и **Data Race**, поэтому доступ к нему часто требует синхронизации.

---

## 🗣️ Ответ на собеседовании

> Shared Mutable State — это общее изменяемое состояние программы, доступное нескольким конкурентным задачам.
>
> Например, несколько потоков работают с одним счётчиком, балансом пользователя или общим словарём. Состояние является shared, потому что к нему обращаются несколько потоков, и mutable, потому что они могут его изменять.
>
> Проблема возникает, когда несколько потоков одновременно читают и изменяют это состояние. Тогда может появиться Race Condition или Data Race.
>
> Например, два потока одновременно увеличивают общий `counter`. Без синхронизации одно обновление может потеряться.
>
> Поэтому shared mutable state стараются минимизировать. Если он необходим, доступ к нему защищают через `Lock`, `RLock`, другие примитивы синхронизации или используют message passing через `Queue`.

---

## 🧭 Где я нахожусь

```text id="7q2m4x"
01 Python
└── 08 Concurrency и Asyncio
    │
    ├── Многозадачность
    │   ├── Конкурентность
    │   └── Параллелизм
    │
    ├── Многопоточность
    │   ├── Thread
    │   ├── Shared Memory
    │   │   └── Shared Mutable State ← Я здесь
    │   │
    │   ├── GIL
    │   ├── Race Condition
    │   └── Data Race
    │
    └── Синхронизация
        ├── Lock
        ├── RLock
        ├── Semaphore
        ├── Condition
        ├── Event
        ├── Barrier
        └── Queue
```

---

# 📚 Разбор поглубже

## 1. Что такое State

**State (состояние)** — данные, которые описывают текущее состояние программы или объекта.

Например:

```python
balance = 1000
```

`balance` — часть состояния программы.

Другие примеры:

```python
is_authenticated = True
```

```python
current_user = user
```

```python
cart = []
```

```python
connections = {}
```

---

# 2. Что значит Mutable

**Mutable** означает, что объект можно изменить после создания.

Например:

```python
items = []

items.append("Python")
```

Список остался тем же объектом, но его состояние изменилось.

```text
До:

items → []

После:

items → ["Python"]
```

Для сравнения, immutable объект нельзя изменить непосредственно:

```python
name = "Ilya"
```

Операция:

```python
name += "!"
```

создаёт новое значение строки, а не изменяет существующую строку.

---

# 3. Что значит Shared

**Shared** означает, что несколько конкурентных участников имеют доступ к одному состоянию.

Например:

```python
counter = 0
```

и:

```text
Thread 1 ──┐
Thread 2 ──┼──> counter
Thread 3 ──┘
```

Все потоки работают с одним состоянием.

Именно это создаёт потенциальную проблему.

---

# 4. Собираем понятие целиком

Получаем:

```text
Shared
  ↓
несколько задач имеют доступ

Mutable
  ↓
состояние можно изменить

State
  ↓
данные программы

        ↓

Shared Mutable State
```

Пример:

```python
counter = 0
```

Если несколько потоков изменяют этот `counter`:

```text
Thread 1 ──┐
Thread 2 ──┼──> counter
Thread 3 ──┘
```

это классический shared mutable state.

---

# 5. Почему это проблема

Представим:

```python
counter = 0
```

Два потока выполняют:

```python
counter += 1
```

Концептуально:

```text
READ
 ↓
ADD
 ↓
WRITE
```

Возможен такой порядок:

```text
Thread 1: READ 0
Thread 2: READ 0

Thread 1: WRITE 1
Thread 2: WRITE 1
```

Результат:

```text
counter = 1
```

вместо:

```text
counter = 2
```

Проблема возникла именно потому, что:

```text
Shared Mutable State
        +
Concurrent Access
        ↓
Race Condition
```

---

# 6. Shared Mutable State и Data Race

Связь можно представить так:

```text
Shared Mutable State
        │
        ↓
Concurrent Access
        │
        ↓
┌─────────────────────┐
│ нужен контроль?     │
└─────────┬───────────┘
          ↓
         Да
          │
          ↓
   Synchronization
```

Если конкурентный доступ к памяти не синхронизирован и выполняются условия Data Race:

```text
Shared Memory
      +
Concurrent Access
      +
Write
      +
No synchronization
      ↓
   Data Race
```

---

# 7. Shared Mutable State не всегда означает ошибку

Сам факт наличия shared mutable state **не является автоматически ошибкой**.

Например:

```python
cache = {}
lock = threading.Lock()
```

Общий изменяемый cache может быть вполне нормальной архитектурой.

Проблема начинается, когда конкурентный доступ к нему неправильно организован.

Например:

```python
with lock:
    cache[key] = value
```

Здесь состояние общее и изменяемое, но доступ к нему контролируется.

---

# 8. Как защитить Shared Mutable State

## `Lock`

Самый простой вариант:

```python
import threading


counter = 0
lock = threading.Lock()


def increment():
    global counter

    with lock:
        counter += 1
```

Теперь изменение общего состояния происходит внутри критической секции.

---

## `RLock`

Если один поток может повторно войти в защищённую область:

```python
lock = threading.RLock()
```

---

## `Semaphore`

Если нужно ограничить количество одновременно работающих участников:

```python
semaphore = threading.Semaphore(5)
```

---

## `Queue`

Можно вообще отказаться от прямого совместного изменения состояния.

Вместо:

```text
Thread 1 ──┐
Thread 2 ──┼──> Shared State
Thread 3 ──┘
```

использовать:

```text
Thread 1 ──┐
Thread 2 ──┼──> Queue ──> Worker
Thread 3 ──┘
```

Это называется **message passing**.

---

# 9. Уменьшение Shared Mutable State

Один из хороших принципов конкурентного программирования:

> **Чем меньше общего изменяемого состояния, тем меньше проблем с синхронизацией.**

Например, вместо:

```text
5 потоков
   ↓
общий список
   ↓
общий словарь
   ↓
общий счётчик
   ↓
10 Lock
```

можно проектировать систему так:

```text
Thread 1 → Task
Thread 2 → Task
Thread 3 → Task
       ↓
     Queue
       ↓
    Worker
```

Это уменьшает количество shared state.

---

# 10. Shared Mutable State vs Immutable State

### Mutable

```python
users = []
```

Можно изменить:

```python
users.append(user)
```

Если список общий для нескольких потоков, нужен контроль конкурентного доступа.

### Immutable

Если состояние неизменяемое:

```text
Thread 1 ── READ ──┐
Thread 2 ── READ ──┼──> immutable data
Thread 3 ── READ ──┘
```

нет конкурентной записи в этот объект.

Поэтому immutable state существенно упрощает concurrency.

---

# 11. Shared Mutable State и `asyncio`

Shared mutable state существует не только в threading.

Он может возникнуть и в `asyncio`:

```python
counter = 0
```

Несколько `Task` могут работать с одной переменной.

Например:

```text
Event Loop
   │
   ├── Task 1 ──┐
   ├── Task 2 ──┼──> shared state
   └── Task 3 ──┘
```

Но механизм конкурентности другой.

В `asyncio` задачи переключаются кооперативно через точки вроде:

```python
await something()
```

Поэтому анализ shared state всё равно нужен.

---

# 12. Shared Mutable State и multiprocessing

У обычных процессов память изолирована:

```text
Process 1 → Memory 1
Process 2 → Memory 2
Process 3 → Memory 3
```

Поэтому обычная Python-переменная не является общей между процессами.

Но можно специально создать shared memory:

```text
Process 1 ──┐
            ├── Shared Memory
Process 2 ──┘
```

Тогда снова появляются проблемы конкурентного доступа.

---

# 13. Примеры Shared Mutable State

### Счётчик

```python
counter = 0
```

### Общий список

```python
tasks = []
```

### Общий словарь

```python
cache = {}
```

### Состояние объекта

```python
class Account:
    def __init__(self):
        self.balance = 1000
```

Если несколько потоков работают с одним экземпляром:

```text
Thread 1 ──┐
Thread 2 ──┼──> account.balance
Thread 3 ──┘
```

`balance` становится shared mutable state.

### Singleton

Если singleton содержит изменяемое состояние и доступен из нескольких потоков:

```text
Thread 1 ──┐
Thread 2 ──┼──> Singleton
Thread 3 ──┘
```

его внутреннее состояние также становится shared mutable state.

---

# 14. Хорошая архитектурная идея

Можно мыслить так:

```text
             Shared State
                  │
          ┌───────┴───────┐
          ↓               ↓
      Immutable         Mutable
          │               │
       проще          нужен контроль
                          │
                  ┌───────┴───────┐
                  ↓               ↓
                Lock           Queue
```

Цель не в том, чтобы **никогда не иметь** shared mutable state.

Цель:

> **контролировать его количество и доступ к нему.**

---

# 15. Связь со всеми предыдущими темами

Это один из центральных узлов всей темы concurrency:

```text
                    Concurrency
                         │
                 ┌───────┴───────┐
                 ↓               ↓
             Threading        Asyncio
                 │               │
                 └───────┬───────┘
                         ↓
                 Shared Mutable State
                         │
                         ↓
                Concurrent Access
                         │
              ┌──────────┴──────────┐
              ↓                     ↓
        Race Condition          Data Race
              │                     │
              └──────────┬──────────┘
                         ↓
                   Synchronization
                         │
       ┌─────────┬───────┼────────┬────────┐
       ↓         ↓       ↓        ↓        ↓
     Lock      RLock  Semaphore  Queue  Condition
```

---

# 🎤 Вопросы на собеседовании

### Что такое Shared Mutable State?

Общее изменяемое состояние, доступное нескольким конкурентным задачам.

### Из чего состоит термин?

**Shared** — общий доступ.

**Mutable** — состояние можно изменить.

**State** — данные, определяющие текущее состояние программы.

### Почему Shared Mutable State опасен?

Потому что конкурентный доступ к нему может привести к Race Condition, Data Race и некорректному состоянию.

### Shared Mutable State всегда является ошибкой?

Нет. Он может быть необходим, но доступ к нему должен быть корректно синхронизирован.

### Как уменьшить проблемы?

* минимизировать shared mutable state;
* использовать immutable data;
* использовать message passing;
* защищать критические секции;
* применять `Lock`, `Queue` и другие подходящие механизмы.

### Как связаны Shared Mutable State и Race Condition?

Shared mutable state создаёт потенциальную точку конфликта. Если несколько задач конкурентно взаимодействуют с ним без необходимой координации, возникает Race Condition.

### Как связаны Shared Mutable State и Data Race?

Если конкурентный доступ к общему состоянию включает конфликтующие операции чтения/записи или записи без необходимой синхронизации, может возникнуть Data Race.

### Есть ли Shared Mutable State в `asyncio`?

Да. Несколько `Task` могут работать с одним изменяемым объектом.

### Есть ли Shared Mutable State между обычными процессами?

Обычно нет, поскольку процессы имеют отдельную память. Но он появляется при использовании механизмов shared memory.

---

## 🧠 Формула для собеседования

> **Shared Mutable State = общий + изменяемый + доступный нескольким конкурентным задачам state.**
>
> Например:
>
> ```text
> Thread 1 ──┐
> Thread 2 ──┼──> counter
> Thread 3 ──┘
> ```
>
> Если `counter` изменяется конкурентно:
>
> **Shared Mutable State → Concurrent Access → Race Condition / Data Race**
>
> Поэтому:
>
> **минимизируем shared mutable state → используем immutable state или message passing → если общий state необходим, синхронизируем доступ.**
