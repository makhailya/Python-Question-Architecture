# `bytearray` — изменяемая последовательность байтов

## 🎯 Ответ на собеседовании

> **`bytearray` — это встроенный изменяемый тип Python, представляющий последовательность байтов от 0 до 255. В отличие от `bytes`, его содержимое можно изменять. Он используется преимущественно для работы с изменяемыми бинарными данными.**

### Если спросят про `bytes`

> **`bytes` — неизменяемая последовательность байтов, а `bytearray` — её изменяемый аналог.**

### Если спросят про `memoryview`

> **`memoryview` предоставляет представление существующих буферных данных, например `bytearray`, без копирования этих данных.**

## Коротко

`bytearray` — это **изменяемая последовательность байтов** (`0–255`).

Главное отличие:

* `bytes` — **неизменяемый** (`immutable`)
* `bytearray` — **изменяемый** (`mutable`)

```python
data = bytearray(b"Hello")

data[0] = 74

print(data)
# bytearray(b'Jello')
```

---

## Что хранит?

Каждый элемент `bytearray` — это целое число от **0 до 255**.

```python
data = bytearray([65, 66, 67])

print(data)
# bytearray(b'ABC')

print(data[0])
# 65
```

---

## Главное свойство — изменяемость

Можно изменять отдельные элементы:

```python
data = bytearray(b"Hello")

data[0] = ord("J")

print(data)
# bytearray(b"Jello")
```

Можно изменять несколько элементов:

```python
data[0:2] = b"Hi"

print(data)
# bytearray(b"Hillo")
```

Можно добавлять и удалять данные:

```python
data.append(33)

data.pop()

data.extend(b" World")
```

---

## Создание

### Из `bytes`

```python
data = bytearray(b"Hello")
```

### Из списка чисел

```python
data = bytearray([65, 66, 67])
```

### Указав размер

```python
data = bytearray(5)

print(data)
# bytearray(b'\x00\x00\x00\x00\x00')
```

---

## Основные методы

```python
data = bytearray(b"Hello")

data.append(33)          # добавить один байт
data.extend(b" World")   # добавить несколько байтов
data.insert(0, 65)       # вставить байт
data.pop()               # удалить байт
data.remove(72)          # удалить первое вхождение
data.reverse()           # развернуть
data.clear()             # очистить
```

---

## Работа со строками

`bytearray` работает с байтами, поэтому строку сначала нужно кодировать:

```python
text = "Привет"

data = bytearray(text, encoding="utf-8")

print(data)
```

Обратно в строку:

```python
text = data.decode("utf-8")

print(text)
# Привет
```

---

## `bytes` vs `bytearray`

|                                    | `bytes`                      | `bytearray`                |
| ---------------------------------- | ---------------------------- | -------------------------- |
| Изменяемый                         | ❌ Нет                        | ✅ Да                       |
| Индексация                         | число                        | число                      |
| Срез                               | создаёт копию                | создаёт копию              |
| Можно изменить элемент             | ❌                            | ✅                          |
| Можно `append()`                   | ❌                            | ✅                          |
| Можно использовать как ключ `dict` | ✅                            | ❌                          |
| Основное назначение                | неизменяемые бинарные данные | изменяемые бинарные данные |

### Пример

```python
data = bytes(b"Hello")

data[0] = 74
# TypeError
```

А:

```python
data = bytearray(b"Hello")

data[0] = 74

print(data)
# bytearray(b'Jello')
```

---

## `bytearray` и `memoryview`

`memoryview` позволяет работать с данными `bytearray` **без создания копии**.

```python
data = bytearray(b"Hello")

view = memoryview(data)

view[0] = 74

print(data)
# bytearray(b'Jello')
```

То есть:

```text
bytearray
    ↓
данные в памяти
    ↑
memoryview — представление этих данных
```

---

## Где используется?

`bytearray` полезен при работе с:

* бинарными файлами;
* сетевыми данными;
* протоколами;
* изображениями и другими бинарными форматами;
* большими массивами байтов, которые нужно изменять;
* низкоуровневой обработкой данных.

---