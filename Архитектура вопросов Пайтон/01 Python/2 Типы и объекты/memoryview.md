# `memoryview` — представление данных в памяти

## 🎯 Ответ на собеседовании

> **`memoryview` — встроенный тип Python, который предоставляет представление буферных данных другого объекта без копирования. Он позволяет обращаться к существующей области памяти и, если исходный буфер изменяемый, изменять данные через `memoryview`. Это используется для эффективной работы с большими бинарными данными.**

### Если спросят: зачем нужен `memoryview`?

> **Чтобы работать с частью бинарного буфера без создания копии данных. Это позволяет экономить память и уменьшать количество операций копирования.**

### Если спросят отличие от `bytes` / `bytearray`

> **`bytes` хранит неизменяемые байты, `bytearray` — изменяемые байты, а `memoryview` сам данные не хранит — он предоставляет представление существующего буфера.**

---

# Коротко

`memoryview` можно представить так:

```text id="8b3n2m"
bytearray
┌──────────────────────────────┐
│  данные в памяти              │
└──────────────────────────────┘
              ↑
              │
         memoryview
              │
       "смотрит" на данные
```

Главная идея:

**`memoryview` → доступ к существующей памяти без копирования.**

---

# Простой пример

```python id="m8w2k4"
data = bytearray(b"Hello")

view = memoryview(data)

print(view)
# <memory at 0x...>
```

Теперь `view` является представлением `data`.

Можно получить отдельный байт:

```python id="p3x7q1"
print(view[0])
# 72
```

`72` — это код символа `H`.

---

# Изменение данных через memoryview

Поскольку `bytearray` изменяемый:

```python id="c5v9r2"
data = bytearray(b"Hello")

view = memoryview(data)

view[0] = 74

print(data)
# bytearray(b'Jello')
```

То есть:

```text id="k1d6s8"
view[0] = 74
       ↓
изменяется исходный bytearray
```

Копии данных не создаётся.

---

# Важный нюанс ⚠️

Если исходный объект immutable, изменить данные через `memoryview` нельзя.

Например:

```python id="f7q2w5"
data = b"Hello"

view = memoryview(data)

view[0] = 74
```

Будет ошибка, потому что `bytes` неизменяемый.

```text id="a9k4m1"
bytes
 ↓
immutable
 ↓
memoryview не может изменить данные
```

А:

```text id="j3r8v6"
bytearray
 ↓
mutable
 ↓
memoryview может изменить данные
```

---

# Индексация

```python id="n6p2x9"
data = bytearray(b"Python")
view = memoryview(data)

print(view[0])
# 80

print(view[1])
# 121
```

При индексации получаем числовое значение байта.

---

# Срезы

Можно получить представление части буфера:

```python id="w4m7c1"
data = bytearray(b"Hello Python")

view = memoryview(data)

part = view[6:12]
```

`part` — тоже `memoryview`.

```python id="t8q3y5"
type(part)
# <class 'memoryview'>
```

Главное преимущество — такой срез **не обязан создавать копию исходных данных**.

---

# Изменение части буфера

```python id="v2h6k9"
data = bytearray(b"Hello Python")

view = memoryview(data)

view[6:12] = b"World!"

print(data)
# bytearray(b"Hello World!')
```

Изменился исходный `bytearray`.

---

# Какие объекты поддерживает?

`memoryview` работает с объектами, которые поддерживают **Buffer Protocol**.

Например:

```python id="e5r1s7"
bytearray
bytes
array.array
```

На практике чаще всего можно встретить:

```python id="q9m3k2"
memoryview(bytearray(...))
```

---

# Buffer Protocol

Это важная часть понимания `memoryview`.

**Buffer Protocol** — механизм Python, который позволяет одним объектам предоставлять другим объектам прямой доступ к своим бинарным данным в памяти.

Условно:

```text id="u7c4p8"
Объект
   ↓
Buffer Protocol
   ↓
memoryview
   ↓
доступ к памяти
```

Поэтому `memoryview` не является альтернативой `bytes` или `bytearray`.

Он работает **поверх существующего буфера**.

---

# `memoryview` не копирует данные

Представим большой буфер:

```python id="h3n8q5"
data = bytearray(10_000_000)
```

Если сделать:

```python id="k6p1s4"
part = data[1000:5000]
```

Python создаст новый `bytearray` с копией данных.

А:

```python id="r8w2m6"
view = memoryview(data)

part = view[1000:5000]
```

создаёт представление части существующего буфера.

Условно:

```text id="b5t9x2"
data
┌──────────────────────────────────┐
│       10 MB данных               │
└──────────────────────────────────┘
              ↑
              │
       memoryview[1000:5000]
              │
       только "окно" на данные
```

---

# Проверка размера

У `memoryview` есть `nbytes`:

```python id="c2v7k4"
data = bytearray(b"Hello")

view = memoryview(data)

print(view.nbytes)
# 5
```

`nbytes` показывает количество байтов, занимаемых представляемыми данными.

---

# `len()` и `nbytes`

Важно различать:

```python id="m9q3w6"
len(view)
```

и:

```python id="s4h8k1"
view.nbytes
```

Для простого `bytearray` они часто совпадают:

```python id="a7p2d5"
data = bytearray(b"Hello")

view = memoryview(data)

len(view)
# 5

view.nbytes
# 5
```

Но для многомерных или других форматов буфера они могут отличаться.

---

# Формат данных

`memoryview` умеет работать не только с отдельными байтами.

Можно узнать формат:

```python id="x6m1r9"
view.format
```

Например, для обычного `bytearray`:

```text id="k4v8p2"
'B'
```

Также существуют свойства:

```python id="d9q5w3"
view.itemsize
view.ndim
view.shape
view.strides
```

Они особенно актуальны при работе с массивами и структурированными бинарными данными.

---

# Основные свойства

У `memoryview` есть несколько полезных атрибутов:

```python id="n2f7c8"
view.format
view.itemsize
view.nbytes
view.ndim
view.shape
view.strides
view.readonly
```

Например:

```python id="q5m8x1"
data = bytearray(b"Hello")
view = memoryview(data)

print(view.readonly)
# False
```

Для `bytes`:

```python id="r3k6p9"
data = b"Hello"
view = memoryview(data)

print(view.readonly)
# True
```

---

# `readonly`

Очень полезное свойство.

```python id="w8c2v5"
data = bytearray(b"Hello")
view = memoryview(data)

view.readonly
# False
```

Можно изменять.

Для `bytes`:

```python id="j4n7q3"
data = b"Hello"
view = memoryview(data)

view.readonly
# True
```

Изменять нельзя.

---

# Освобождение представления

У `memoryview` есть метод:

```python id="p6x1m8"
view.release()
```

Он освобождает доступ `memoryview` к исходному буферу.

После этого использовать `view` для доступа к данным нельзя.

Обычно вручную `release()` нужен редко, но он важен при работе с ресурсами и буферами.

---

# `memoryview` и `bytes`

```python id="h7k3s2"
data = b"Hello"

view = memoryview(data)
```

Получаем представление неизменяемых данных:

```text id="m4q8v1"
bytes
 ↓
immutable buffer
 ↓
memoryview
 ↓
read-only
```

---

# `memoryview` и `bytearray`

```python id="z2c6n9"
data = bytearray(b"Hello")

view = memoryview(data)
```

Получаем:

```text id="r5w1k7"
bytearray
 ↓
mutable buffer
 ↓
memoryview
 ↓
можно читать и изменять
```

---

# `memoryview` vs `bytes` vs `bytearray`

|                               | `bytes`         | `bytearray`      | `memoryview`                    |
| ----------------------------- | --------------- | ---------------- | ------------------------------- |
| Бинарные данные               | ✅               | ✅                | Представление                   |
| Immutable                     | ✅               | ❌                | Зависит от исходного буфера     |
| Mutable                       | ❌               | ✅                | Если исходный буфер mutable     |
| Хранит собственные данные     | ✅               | ✅                | ❌                               |
| Копирование при создании view | —               | —                | ❌                               |
| Индексация                    | ✅               | ✅                | ✅                               |
| Hashable                      | ✅               | ❌                | ❌                               |
| Основная задача               | хранение байтов | изменение байтов | доступ к буферу без копирования |

---

# Пример с большим буфером

Представим получение сетевого пакета:

```python id="f9k2m5"
data = bytearray(10_000_000)

view = memoryview(data)

header = view[:100]
body = view[100:]
```

Здесь можно работать с разными частями большого буфера, не создавая отдельные копии всех данных.

Именно поэтому `memoryview` может быть полезен в высокопроизводительном коде.

---

# Где используется?

`memoryview` особенно полезен при:

* обработке больших бинарных данных;
* сетевом программировании;
* работе с файлами;
* бинарных протоколах;
* обработке изображений;
* работе с буферами;
* высокопроизводительных операциях;
* взаимодействии Python с C/C++ расширениями.

В обычном backend-коде встречается редко, но концепция важна для понимания производительности Python.

---

# 🧠 Шпаргалка

```text
memoryview
│
├── представление существующего буфера
├── НЕ копирует данные
│
├── Buffer Protocol
│
├── работает с bytes / bytearray / array
│
├── bytes
│    └── readonly
│
├── bytearray
│    └── можно изменять через view
│
├── .readonly
├── .nbytes
├── .format
├── .itemsize
├── .shape
├── .strides
│
└── .release()
```

## ⭐ Главная связка

```text
str
  ↓ encode()
bytes
  ↓
неизменяемые байты


bytearray
  ↓
изменяемые байты


bytes / bytearray
  ↓ Buffer Protocol
memoryview
  ↓
доступ к существующей памяти
без копирования
```

### Самая короткая версия для собеседования

**`memoryview` — это представление буферных данных без копирования. Оно позволяет эффективно работать с существующей областью памяти, а если исходный буфер изменяемый, например `bytearray`, то данные можно изменять через `memoryview`.**
