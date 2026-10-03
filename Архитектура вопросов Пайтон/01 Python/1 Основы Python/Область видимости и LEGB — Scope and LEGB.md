# Область видимости и LEGB — Scope and LEGB

## Коротко

**Область видимости (Scope)** — область программы, в которой имя переменной доступно.

Python ищет имя по определённому порядку:

```text
L → Local
E → Enclosing
G → Global
B → Built-in
```

Это называется **LEGB**.

То есть при обращении к имени Python сначала ищет его:

```text
1. Local       → локальная область
2. Enclosing   → внешняя функция
3. Global      → глобальная область
4. Built-in    → встроенные имена Python
```

## На собеседовании достаточно сказать

> Scope — это область, в которой имя доступно. При поиске имени Python использует правило LEGB: Local, Enclosing, Global, Built-in. Сначала ищется локальное имя, затем имя во внешней функции, затем глобальное и в конце — встроенное.

## Что важно помнить

### Local — локальная область

Переменная, созданная внутри функции, обычно является локальной:

```python
def hello():
    name = "Ilya"
    print(name)
```

`name` существует в локальной области функции `hello`.

Снаружи функции обратиться к ней нельзя:

```python
def hello():
    name = "Ilya"

hello()

print(name)
```

Получим:

```text
NameError
```

Потому что `name` существует только внутри `hello`.

---

## Global — глобальная область

Переменная, созданная на уровне модуля, находится в глобальной области:

```python
name = "Ilya"

def hello():
    print(name)

hello()
```

Функция не имеет локальной переменной `name`, поэтому Python продолжает поиск и находит глобальную:

```text
name ───> "Ilya"
  ↑
  │
hello()
```

---

## Local имеет приоритет над Global

Если внутри функции есть переменная с таким же именем:

```python
name = "Global"

def hello():
    name = "Local"
    print(name)

hello()
```

Результат:

```text
Local
```

Python сначала ищет имя в локальной области:

```text
Local
  ↑
  │
print(name)
```

До глобальной переменной он уже не доходит.

---

## Enclosing — внешняя функция

Enclosing появляется при использовании **вложенных функций**.

Например:

```python
def outer():
    name = "Ilya"

    def inner():
        print(name)

    inner()

outer()
```

У `inner()` нет локальной переменной `name`.

Python ищет её дальше и находит в окружающей функции `outer()`.

```text
outer()
│
├── name = "Ilya"
│
└── inner()
      │
      └── print(name)
              ↓
          Enclosing
```

---

## Пример полного LEGB

```python
name = "Global"

def outer():
    name = "Enclosing"

    def inner():
        name = "Local"
        print(name)

    inner()

outer()
```

Результат:

```text
Local
```

Почему?

Потому что Python находит `name` уже на первом уровне:

```text
L → найдено
E → не проверяется
G → не проверяется
B → не проверяется
```

Если убрать локальную переменную:

```python
def outer():
    name = "Enclosing"

    def inner():
        print(name)

    inner()
```

будет:

```text
Enclosing
```

Теперь Python ищет:

```text
L → нет
E → найдено
```

---

## Built-in — встроенная область

Последний уровень LEGB — встроенные имена Python.

Например:

```python
print(len([1, 2, 3]))
```

`len` не определён локально или глобально.

Python находит его среди встроенных имён:

```text
L → нет
E → нет
G → нет
B → len
```

Другие примеры встроенных имён:

```python
print()
len()
sum()
max()
min()
type()
isinstance()
```

---

## Что происходит, если имя нигде не найдено

Например:

```python
print(username)
```

Если `username` не существует ни в одной области видимости, Python выдаст:

```text
NameError: name 'username' is not defined
```

Поиск закончился:

```text
L → нет
E → нет
G → нет
B → нет
      ↓
   NameError
```

---

## `global`

Если внутри функции нужно изменить глобальную переменную, можно использовать `global`.

Например:

```python
counter = 0

def increment():
    global counter
    counter += 1

increment()

print(counter)
```

Результат:

```text
1
```

`global counter` говорит Python:

> Используй глобальное имя `counter`, а не создавай локальное.

Без `global`:

```python
counter = 0

def increment():
    counter += 1
```

возникнет ошибка:

```text
UnboundLocalError
```

Потому что Python воспринимает `counter` внутри функции как локальное имя.

### Важно

`global` существует, но использовать его без необходимости обычно не стоит.

Чаще лучше передавать значения через аргументы и возвращать результат через `return`.

---

## `nonlocal`

`nonlocal` используется во вложенной функции, когда нужно изменить переменную внешней функции.

```python
def counter():
    count = 0

    def increment():
        nonlocal count
        count += 1
        return count

    return increment


increment = counter()

print(increment())
print(increment())
```

Результат:

```text
1
2
```

Здесь:

```text
counter()
│
└── count = 0
       ↑
       │
    nonlocal
       │
   increment()
```

`nonlocal` означает:

> Используй переменную из ближайшей внешней функции.

---

## `global` vs `nonlocal`

| Ключевое слово | Где ищет переменную |
|---|---|
| `global` | Глобальная область |
| `nonlocal` | Внешняя функция |
| ничего | Обычный поиск LEGB |

Например:

```python
x = 10

def outer():
    x = 20

    def inner():
        nonlocal x
        x = 30

    inner()

    print(x)

outer()
```

Результат:

```text
30
```

`nonlocal` изменил `x` из `outer()`.

---

## Важный нюанс с присваиванием

Рассмотрим:

```python
x = 10

def test():
    print(x)
    x = 20

test()
```

Можно подумать, что Python сначала возьмёт глобальный `x`.

Но будет ошибка:

```text
UnboundLocalError
```

Почему?

Потому что наличие присваивания:

```python
x = 20
```

делает `x` локальным именем внутри функции.

Python фактически рассматривает его как локальную переменную:

```text
test()
│
└── x → Local
```

А затем:

```python
print(x)
```

пытается получить значение локального `x`, которому ещё ничего не присвоили.

---

## Простая схема LEGB

Запомнить можно так:

```text
        имя
         │
         ▼
┌─────────────────┐
│ L — Local       │
└────────┬────────┘
         │ нет
         ▼
┌─────────────────┐
│ E — Enclosing   │
└────────┬────────┘
         │ нет
         ▼
┌─────────────────┐
│ G — Global      │
└────────┬────────┘
         │ нет
         ▼
┌─────────────────┐
│ B — Built-in    │
└────────┬────────┘
         │ нет
         ▼
    NameError
```

---

## Простой пример

```python
name = "Global"

def outer():
    name = "Enclosing"

    def inner():
        name = "Local"
        print(name)

    inner()

outer()
```

Результат:

```text
Local
```

Если убрать:

```python
name = "Local"
```

получим:

```text
Enclosing
```

Если убрать и:

```python
name = "Enclosing"
```

получим:

```text
Global
```

Если убрать и глобальный `name`, Python будет искать дальше — среди встроенных имён.

---

## Главное

**Scope** — область видимости имени.

**LEGB** — порядок поиска имени:

```text
L → Local
E → Enclosing
G → Global
B → Built-in
```

Python ищет имя сверху вниз:

```text
Local
  ↓
Enclosing
  ↓
Global
  ↓
Built-in
  ↓
NameError
```

Также нужно знать:

```python
global
```

→ обращение к глобальной переменной.

```python
nonlocal
```

→ обращение к переменной внешней функции.

Главная формула:

> **LEGB = Local → Enclosing → Global → Built-in**

## Связи

- [[Переменные и присваивание — Variables and Assignment]]
- [[Объект и ссылка — Object and Reference]]
- [[Функции — Functions]]
- [[Замыкания — Closures]]
- [[NoneType]]
- [[Truthy и Falsy — Truthy and Falsy]]