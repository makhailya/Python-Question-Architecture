# `str` — строки

## 🎯 Ответ на собеседовании

> **`str` — встроенный неизменяемый (`immutable`) тип Python для работы с текстовыми данными. Строки являются последовательностями Unicode-символов, поддерживают индексацию, срезы, поиск, форматирование и большое количество строковых методов.**

### Если спросят про `immutable`

> **`str` — immutable. Нельзя изменить отдельный символ существующей строки. При операции изменения создаётся новая строка.**

```python
text = "Hello"

text = text + " World"
```

Здесь создаётся новый объект `str`.

### Если спросят про Unicode

> **Python 3 хранит строки как Unicode, поэтому `str` может содержать символы разных языков, emoji и специальные символы.**

```python
text = "Привет 🌍"
```

---

# Коротко

`str` используется для хранения **текста**:

```python
name = "Илья"
language = "Python"
message = "Hello, world!"
```

Проверить тип:

```python
type("Python")
# <class 'str'>
```

---

# Создание строк

### Одинарные кавычки

```python
name = 'Python'
```

### Двойные кавычки

```python
name = "Python"
```

Для Python разницы по смыслу нет.

### Многострочная строка

```python
text = """Первая строка
Вторая строка
Третья строка"""
```

---

# Строка — это последовательность

Строку можно представить как последовательность символов:

```text
Python
012345
```

Каждый символ имеет свой индекс.

```python
text = "Python"

text[0]
# 'P'

text[1]
# 'y'

text[5]
# 'n'
```

Индексация начинается с **нуля**.

---

# Отрицательная индексация

Можно обращаться к символам с конца:

```python
text = "Python"

text[-1]
# 'n'

text[-2]
# 'o'
```

```text
P y t h o n
0 1 2 3 4 5
-6 -5 -4 -3 -2 -1
```

---

# Срезы

Можно получать часть строки:

```python
text = "Python"

text[0:3]
# 'Pyt'
```

Формат:

```text
строка[start:stop:step]
```

`stop` не включается.

Например:

```python
text[2:5]
# 'tho'
```

---

## Шаг

```python
text[::2]
# 'Pto'
```

Каждый второй символ.

Развернуть строку:

```python
text[::-1]
# 'nohtyP'
```

Это популярный вопрос на собеседованиях.

---

# Строки immutable

Нельзя изменить отдельный символ:

```python
text = "Python"

text[0] = "J"
```

Получим:

```text
TypeError: 'str' object does not support item assignment
```

Правильно создать новую строку:

```python
text = "J" + text[1:]

print(text)
# Jython
```

---

# Длина строки

Используется `len()`:

```python
text = "Python"

len(text)
# 6
```

Важно: `len()` считает **Unicode-кодовые точки**, а не обязательно количество визуально воспринимаемых символов.

---

# Unicode

Python 3 использует Unicode для `str`.

Поэтому:

```python
text = "Привет мир 🌍"

print(text)
```

работает без специальных настроек.

Можно получить код символа:

```python
ord("A")
# 65
```

И обратно:

```python
chr(65)
# 'A'
```

---

# `str` и `bytes`

Это очень важное отличие.

### `str`

Текст:

```python
text = "Привет"
```

### `bytes`

Байтовые данные:

```python
data = b"Hello"
```

Условно:

```text
str
 ↓
текст / Unicode

bytes
 ↓
байты 0–255
```

Для преобразования:

```python
text = "Привет"

data = text.encode("utf-8")
```

Получаем `bytes`.

Обратно:

```python
text = data.decode("utf-8")
```

Получаем `str`.

---

# Основные строковые методы

## `lower()`

Перевод в нижний регистр:

```python
"PYTHON".lower()
# 'python'
```

## `upper()`

```python
"python".upper()
# 'PYTHON'
```

## `strip()`

Удаляет пробелы с начала и конца:

```python
"  hello  ".strip()
# 'hello'
```

Также есть:

```python
lstrip()
rstrip()
```

---

# Поиск

## `find()`

```python
text = "Hello Python"

text.find("Python")
# 6
```

Если подстрока не найдена:

```python
text.find("Java")
# -1
```

## `index()`

Похож на `find()`, но если строка не найдена — вызывает `ValueError`.

---

# Проверка наличия

Очень часто используется оператор `in`:

```python
"Python" in "I love Python"
# True
```

```python
"Java" in "I love Python"
# False
```

---

# Замена

```python
text = "I love Java"

text = text.replace("Java", "Python")

print(text)
# I love Python
```

Важно: `replace()` **не изменяет исходную строку**, а возвращает новую.

---

# Разделение строки

`split()` превращает строку в список:

```python
text = "Python Java Go"

languages = text.split()

print(languages)
# ['Python', 'Java', 'Go']
```

Можно указать разделитель:

```python
text = "apple,banana,orange"

text.split(",")
# ['apple', 'banana', 'orange']
```

---

# Объединение строк

Метод `join()` работает наоборот:

```python
languages = ["Python", "Java", "Go"]

result = ", ".join(languages)

print(result)
# Python, Java, Go
```

Важно понимать:

```text
split() → str → list

join()  → list → str
```

---

# Проверка содержимого

Есть полезные методы:

```python
"123".isdigit()
# True

"abc".isalpha()
# True

"abc123".isalnum()
# True

"hello".islower()
# True

"HELLO".isupper()
# True
```

---

# Форматирование строк

## f-string — основной современный способ

```python
name = "Илья"
age = 31

text = f"Меня зовут {name}, мне {age} лет."
```

Получим:

```text
Меня зовут Илья, мне 31 лет.
```

Можно выполнять выражения:

```python
a = 10
b = 20

text = f"Сумма: {a + b}"
```

---

# Старые способы форматирования

### `.format()`

```python
name = "Илья"

text = "Привет, {}!".format(name)
```

### `%`

```python
name = "Илья"

text = "Привет, %s!" % name
```

В современном Python обычно предпочтительнее **f-string**.

---

# Конкатенация

Строки можно объединять через `+`:

```python
first = "Hello"
second = "World"

result = first + " " + second
```

Получим:

```text
Hello World
```

Но при большом количестве строк постоянное использование `+` может быть неэффективным.

Для объединения множества строк обычно используют:

```python
" ".join(words)
```

---

# Повторение строки

Можно умножать строку на целое число:

```python
"ha" * 3
# 'hahaha'
```

Например:

```python
"-" * 20
```

Получим строку из 20 дефисов.

---

# Сравнение строк

Строки можно сравнивать:

```python
"abc" == "abc"
# True

"abc" != "def"
# True
```

Также возможны сравнения:

```python
"abc" < "abd"
# True
```

Сравнение выполняется **лексикографически**, то есть на основе Unicode-кодов символов.

---

# Escape-последовательности

Специальные символы можно записывать через `\`.

```python
text = "Hello\nWorld"
```

`\n` → новая строка.

Другие:

```text
\n → перенос строки
\t → табуляция
\\ → обратный слэш
\' → одинарная кавычка
\" → двойная кавычка
```

---

# Raw string

Префикс `r` отключает обычную обработку большинства escape-последовательностей:

```python
path = r"C:\Users\Ilya\Documents"
```

Особенно полезно при работе с:

* Windows-путями;
* регулярными выражениями.

---

# Immutable и строки

Следует понимать важный момент:

```python
text = "Hello"

text.upper()
```

Исходный объект не меняется.

Метод возвращает новую строку:

```python
text = text.upper()
```

Теперь переменная `text` ссылается на новый объект.

---

# Часто используемые методы

```text
lower()       → нижний регистр
upper()       → верхний регистр
strip()       → убрать пробелы по краям
replace()     → заменить
find()        → найти подстроку
split()       → строка → список
join()        → последовательность → строка
startswith()  → начинается ли с...
endswith()    → заканчивается ли на...
isdigit()     → состоит ли из цифр
isalpha()     → состоит ли из букв
isalnum()     → буквы и цифры
```

---

# `str` и память

Как и другие immutable-типы, строку нельзя изменить на месте:

```python
text = "Hello"
```

Операция:

```python
text += " World"
```

создаёт новый объект строки.

Это важно учитывать при создании очень большого количества строк.

---

# 🧠 Шпаргалка

```text
str
│
├── текст
├── Unicode
├── immutable
│
├── индексация
│     ├── text[0]
│     └── text[-1]
│
├── срезы
│     └── text[start:stop:step]
│
├── len()
│
├── методы
│     ├── lower()
│     ├── upper()
│     ├── strip()
│     ├── replace()
│     ├── find()
│     ├── split()
│     └── join()
│
├── encode() → bytes
├── decode() ← bytes
│
└── форматирование
      └── f-string
```

## ⭐ Что обязательно помнить

```python
text = "Python"

type(text)
# str

text[0]
# 'P'

text[-1]
# 'n'

text[::-1]
# 'nohtyP'

len(text)
# 6

"Py" in text
# True

"  hello  ".strip()
# 'hello'

"hello".upper()
# 'HELLO'
```

### Главная мысль

**`str` = immutable последовательность Unicode-символов для работы с текстом.**

Связка, которую стоит держать в голове:

```text
str      → текст / Unicode
bytes    → неизменяемые байты
bytearray → изменяемые байты
memoryview → представление буферной памяти без копирования
```
