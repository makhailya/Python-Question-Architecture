# `bytes` — последовательность байтов

## 🎯 Ответ на собеседовании

> **`bytes` — встроенный неизменяемый (`immutable`) тип Python для представления последовательности байтов. Каждый элемент имеет значение от 0 до 255. `bytes` используется для работы с бинарными данными: файлами, сетевыми протоколами, изображениями и другими данными в байтовом представлении.**

### Если спросят отличие от `str`

> **`str` хранит текст в виде Unicode-символов, а `bytes` — последовательность числовых значений от 0 до 255. Для преобразования между ними используются `encode()` и `decode()` с указанием кодировки.**

```python
text = "Привет"

data = text.encode("utf-8")  # str → bytes
text = data.decode("utf-8")  # bytes → str
```

### Если спросят отличие от `bytearray`

> **`bytes` — immutable, а `bytearray` — mutable. Поэтому содержимое `bytes` нельзя изменить после создания, а `bytearray` можно.**

---

# Коротко

`bytes` — это **неизменяемая последовательность байтов**.

```python
data = b"Hello"

print(data)
# b'Hello'
```

Каждый байт имеет значение:

```text
0 ≤ byte ≤ 255
```

Например:

```python
data = bytes([65, 66, 67])

print(data)
# b'ABC'
```

---

# Как создать `bytes`

## Через `b"..."`

Самый простой вариант:

```python
data = b"Hello"
```

Префикс `b` означает:

> создать объект `bytes`.

---

## Через `bytes()`

Из списка чисел:

```python
data = bytes([65, 66, 67])

print(data)
# b'ABC'
```

Важно: значения должны быть от `0` до `255`.

```python
bytes([255])   # OK
bytes([256])   # ValueError
```

---

## Из строки

Нельзя просто сделать:

```python
bytes("Hello")
```

Нужно указать кодировку:

```python
data = bytes("Hello", "utf-8")
```

Или чаще используют:

```python
data = "Hello".encode("utf-8")
```

---

# `str` → `bytes`

Главный способ:

```python
text = "Привет"

data = text.encode("utf-8")
```

Теперь:

```python
type(text)
# str

type(data)
# bytes
```

---

# `bytes` → `str`

Используем `decode()`:

```python
data = b"Hello"

text = data.decode("utf-8")

print(text)
# Hello
```

Для русского:

```python
text = "Привет"

data = text.encode("utf-8")

result = data.decode("utf-8")

print(result)
# Привет
```

---

# Кодировка

Это важная тема.

Строка:

```python
text = "А"
```

является Unicode-текстом.

После:

```python
data = text.encode("utf-8")
```

получаем конкретное байтовое представление.

Например:

```python
"А".encode("utf-8")
```

даст:

```text
b'\xd0\x90'
```

Поэтому можно представить преобразование так:

```text
str
 │
 │ encode("utf-8")
 ▼
bytes
 │
 │ decode("utf-8")
 ▼
str
```

---

# Индексация

У `bytes` есть индексация:

```python
data = b"Hello"

print(data[0])
# 72
```

Обрати внимание:

```python
data[0]
```

возвращает **число**, а не `bytes`.

```python
type(data[0])
# int
```

Например:

```python
data[1]
# 101
```

Потому что `e` имеет код 101 в ASCII.

---

# Срезы

Срез `bytes` возвращает **новый объект `bytes`**:

```python
data = b"Hello"

data[0:2]
# b'He'
```

```python
type(data[0:2])
# bytes
```

---

# Immutable

`bytes` нельзя изменить.

```python
data = b"Hello"

data[0] = 74
```

Получим:

```text
TypeError: 'bytes' object does not support item assignment
```

Если нужно изменять байты — используем `bytearray`.

```python
data = bytearray(b"Hello")

data[0] = 74

print(data)
# bytearray(b'Jello')
```

---

# `bytes` vs `bytearray`

|                 | `bytes` | `bytearray` |
| --------------- | ------- | ----------- |
| Байтовые данные | ✅       | ✅           |
| Immutable       | ✅       | ❌           |
| Mutable         | ❌       | ✅           |
| Индексация      | ✅       | ✅           |
| `append()`      | ❌       | ✅           |
| `decode()`      | ✅       | ✅           |
| Hashable        | ✅       | ❌           |

Главная формула:

```text
bytes     → байты + immutable
bytearray → байты + mutable
```

---

# `bytes` vs `str`

Это один из самых важных вопросов.

|                        | `str`         | `bytes`    |
| ---------------------- | ------------- | ---------- |
| Что хранит             | Unicode-текст | байты      |
| Immutable              | ✅             | ✅          |
| Элемент при индексации | `str` длины 1 | `int`      |
| Пример                 | `"Hello"`     | `b"Hello"` |
| Для бинарных данных    | ❌             | ✅          |
| `encode()`             | ✅             | ❌          |
| `decode()`             | ❌             | ✅          |

Пример:

```python
text = "Hello"
data = b"Hello"

print(text[0])
# 'H'

print(data[0])
# 72
```

---

# `bytes` и ASCII

ASCII использует значения от `0` до `127`.

Например:

```python
ord("A")
# 65
```

Поэтому:

```python
b"A"[0]
# 65
```

Для символов за пределами ASCII часто используется UTF-8 или другая кодировка.

---

# Полезные методы

`bytes` поддерживает многие строковые методы:

```python
data = b"Hello World"
```

### `upper()`

```python
data.upper()
# b'HELLO WORLD'
```

### `lower()`

```python
data.lower()
# b'hello world'
```

### `replace()`

```python
data.replace(b"World", b"Python")
# b'Hello Python'
```

### `find()`

```python
data.find(b"World")
# 6
```

Но методы работают именно с `bytes`, а не со `str`.

Например:

```python
data.replace("World", "Python")
```

❌ Ошибка типов.

Нужно:

```python
data.replace(b"World", b"Python")
```

---

# `startswith()` и `endswith()`

```python
data = b"Hello Python"

data.startswith(b"Hello")
# True

data.endswith(b"Python")
# True
```

---

# `split()`

Можно разделить `bytes`:

```python
data = b"one,two,three"

data.split(b",")
```

Результат:

```python
[b'one', b'two', b'three']
```

---

# `join()`

```python
parts = [b"Hello", b"Python"]

result = b" ".join(parts)

print(result)
# b'Hello Python'
```

---

# `len()`

```python
data = b"Hello"

len(data)
# 5
```

Для UTF-8 важно понимать:

```python
len("Привет")
# 6
```

Но:

```python
len("Привет".encode("utf-8"))
# 12
```

Почему?

Потому что один Unicode-символ не обязательно занимает один байт.

---

# Почему `bytes` нужен?

Компьютер и сеть в конечном итоге работают с **байтами**.

Например, HTTP-запросы, файлы, изображения и TCP-соединения работают с бинарными данными.

Условно:

```text
текст
 ↓ encode
bytes
 ↓
сеть / файл / бинарный протокол
```

---

# Работа с файлами

При бинарном режиме файл читается как `bytes`:

```python
with open("image.jpg", "rb") as file:
    data = file.read()
```

Здесь:

```text
rb
│
├── r → read
└── b → binary
```

`data` будет иметь тип:

```python
bytes
```

Запись:

```python
with open("image.jpg", "wb") as file:
    file.write(data)
```

---

# Работа с сетью

При низкоуровневой работе с сокетами данные обычно передаются в `bytes`:

```python
sock.send(b"Hello")
```

Полученные данные:

```python
data = sock.recv(1024)
```

будут `bytes`.

Если это текст:

```python
text = data.decode("utf-8")
```

---

# `memoryview` и `bytes`

`memoryview` может предоставить представление байтов без необходимости создавать дополнительную копию.

```python
data = bytearray(b"Hello")

view = memoryview(data)
```

Можно работать с частью буфера:

```python
part = view[1:4]
```

Главная идея:

```text
bytes / bytearray
       ↓
   buffer
       ↓
 memoryview
       ↓
доступ без копирования
```

---

# Hashable

`bytes` является hashable:

```python
data = b"Hello"

hash(data)
```

Поэтому `bytes` можно использовать как ключ словаря:

```python
data = {
    b"hello": "value"
}
```

И элементом `set`:

```python
data = {
    b"hello",
    b"world"
}
```

---

# `bytes` как ключ `dict`

```python
cache = {
    b"user_id": 123
}

print(cache[b"user_id"])
# 123
```

Это возможно благодаря immutable/hashable природе `bytes`.

---

# `bytes` и `bytearray` + `memoryview`

Эту тройку удобно запомнить вместе:

```text
bytes
 ↓
неизменяемые байты

bytearray
 ↓
изменяемые байты

memoryview
 ↓
представление существующего буфера
без копирования данных
```

---

# 🧠 Шпаргалка

```text
bytes
│
├── последовательность байтов
├── каждый элемент 0–255
├── immutable
├── hashable
│
├── b"Hello"
├── bytes([...])
│
├── encode()
│     str → bytes
│
├── decode()
│     bytes → str
│
├── индексация → int
├── срез → bytes
│
├── бинарные файлы
├── сеть
├── протоколы
└── бинарные данные
```

## ⭐ Самое главное

```python
text = "Привет"

data = text.encode("utf-8")
# str → bytes

text = data.decode("utf-8")
# bytes → str
```

Запомнить связку:

```text
str
→ текст / Unicode

bytes
→ неизменяемые байты

bytearray
→ изменяемые байты

memoryview
→ представление буферной памяти без копирования
```

### Финальная формула

**`bytes = immutable последовательность байтов от 0 до 255, используемая для работы с бинарными данными; `str`↔`bytes`преобразуются через`encode()`и`decode()`.**
