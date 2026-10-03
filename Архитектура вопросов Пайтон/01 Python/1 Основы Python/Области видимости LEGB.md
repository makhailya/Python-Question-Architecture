# 🔎 Области видимости LEGB

## 🎤 Короткий ответ

**LEGB** — это порядок, в котором Python ищет имя переменной:

```text
L → Local
E → Enclosing
G → Global
B → Built-in
```

То есть при обращении к имени Python сначала ищет его:

1. в текущей функции — **Local**;
2. во внешней функции — **Enclosing**;
3. на уровне модуля — **Global**;
4. среди встроенных имён Python — **Built-in**.

Если имя не найдено ни на одном уровне — возникает `NameError`.

---

## 🗣️ Ответ на собеседовании

> LEGB — это правило разрешения имён в Python. Оно определяет порядок поиска переменной при обращении к имени.
>
> Сначала Python ищет имя в локальной области текущей функции — Local. Если там его нет, ищет во внешней функции — Enclosing. Затем в глобальной области модуля — Global и в конце во встроенных именах — Built-in.
>
> Например, если внутри вложенной функции нет локальной переменной, Python может найти её во внешней функции. Это как раз Enclosing scope и основа механизма замыканий.
>
> При присваивании Python по умолчанию считает имя локальным внутри функции. Для изменения глобальной переменной используется `global`, а для изменения переменной внешней функции — `nonlocal`.

---

## 🧭 Где я нахожусь

```text
01 Python
└── 01 Основы Python
    ├── Переменные и имена
    ├── Объекты и ссылки
    ├── Функции
    └── Области видимости
        └── LEGB ← Я здесь
            ├── Local
            ├── Enclosing
            ├── Global
            ├── Built-in
            ├── global
            ├── nonlocal
            └── Замыкания
```

---

# 📚 Разбор поглубже

## 1. Что такое область видимости

**Scope** — это область программы, в которой определённое имя доступно для поиска.

Например:

```python
def greet():
    message = "Привет"
    print(message)
```

`message` существует в локальной области функции `greet`.

Снаружи:

```python
print(message)
```

получим:

```text
NameError
```

Потому что `message` не находится в глобальной области.

---

# 2. LEGB

Схема поиска:

```text
Local
  ↓
Enclosing
  ↓
Global
  ↓
Built-in
```

Python ищет имя сверху вниз.

Например:

```python
print(name)
```

Python концептуально проверяет:

```text
1. Local?
2. Enclosing?
3. Global?
4. Built-in?
5. Если нигде нет → NameError
```

---

# 3. Local — локальная область

**Local** — область текущей функции.

```python
def calculate():
    result = 42
    print(result)
```

Здесь `result` — локальное имя.

```text
calculate()
└── Local
    └── result = 42
```

После завершения функции имя `result` не доступно из внешнего кода.

---

## Пример

```python
def foo():
    x = 10
    print(x)


foo()
```

Работает:

```text
10
```

Но:

```python
def foo():
    x = 10


foo()
print(x)
```

даст:

```text
NameError
```

---

# 4. Enclosing — внешняя область

**Enclosing** появляется при наличии вложенных функций.

```python
def outer():
    x = 10

    def inner():
        print(x)

    inner()
```

У `inner()` нет локального `x`.

Python идёт дальше:

```text
inner Local
    ↓
outer Enclosing
    ↓
Global
    ↓
Built-in
```

И находит:

```python
x = 10
```

во внешней функции `outer`.

---

# 5. Пример с несколькими уровнями

```python
x = "global"


def outer():
    x = "enclosing"

    def inner():
        x = "local"
        print(x)

    inner()


outer()
```

Результат:

```text
local
```

Потому что первое найденное имя находится в `Local`.

---

## Если убрать Local

```python
x = "global"


def outer():
    x = "enclosing"

    def inner():
        print(x)

    inner()


outer()
```

Результат:

```text
enclosing
```

Теперь `x` найден в `Enclosing`.

---

## Если убрать Enclosing

```python
x = "global"


def outer():

    def inner():
        print(x)

    inner()


outer()
```

Результат:

```text
global
```

Теперь Python дошёл до `Global`.

---

# 6. Global — глобальная область

**Global** — область имён модуля.

```python
name = "Ilya"


def greet():
    print(name)
```

Внутри `greet()` локального `name` нет.

Python продолжает поиск:

```text
Local
 ↓
Enclosing
 ↓
Global → name
```

Поэтому код работает.

---

# 7. Built-in — встроенная область

Последний уровень — встроенные имена Python.

Например:

```python
print
len
sum
type
id
str
list
dict
```

Когда мы пишем:

```python
numbers = [1, 2, 3]

print(len(numbers))
```

Python должен найти:

```text
print
len
```

Они доступны из **Built-in scope**.

---

# 8. Полный пример LEGB

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

Поиск `name`:

```text
Local
└── name = "Local"       ← найдено
```

Поэтому Python дальше не идёт.

---

# 9. Если имя нигде не найдено

```python
def foo():
    print(x)


foo()
```

Python проверит:

```text
Local       ❌
Enclosing   ❌
Global      ❌
Built-in    ❌
```

Результат:

```text
NameError: name 'x' is not defined
```

---

# 10. `global`

По умолчанию присваивание внутри функции создаёт локальное имя.

```python
counter = 0


def increment():
    counter = counter + 1
```

Такой код вызовет:

```text
UnboundLocalError
```

Почему?

Потому что из-за присваивания:

```python
counter = ...
```

Python считает `counter` локальным именем функции.

Чтобы явно использовать глобальную переменную:

```python
counter = 0


def increment():
    global counter
    counter += 1
```

Теперь:

```python
increment()

print(counter)
```

Получим:

```text
1
```

---

# 11. `nonlocal`

`nonlocal` используется во вложенной функции, когда нужно изменить переменную внешней функции.

```python
def counter():
    value = 0

    def increment():
        nonlocal value
        value += 1
        return value

    return increment
```

Использование:

```python
count = counter()

print(count())
print(count())
print(count())
```

Результат:

```text
1
2
3
```

Здесь:

```text
counter()
└── value = 0          ← Enclosing
    │
    └── increment()
        └── nonlocal value
```

---

# 12. `global` vs `nonlocal`

|                  | `global`          | `nonlocal`                 |
| ---------------- | ----------------- | -------------------------- |
| Изменяет         | переменную модуля | переменную внешней функции |
| Где используется | внутри функции    | во вложенной функции       |
| Уровень LEGB     | Global            | Enclosing                  |

Пример `global`:

```python
counter = 0


def increment():
    global counter
    counter += 1
```

Пример `nonlocal`:

```python
def outer():
    counter = 0

    def inner():
        nonlocal counter
        counter += 1
```

---

# 13. Чтение и присваивание — важное различие

Посмотрим:

```python
x = 10


def foo():
    print(x)
```

Это работает.

Python ищет `x`:

```text
Local → ❌
Global → ✅
```

Но:

```python
x = 10


def foo():
    x = 20
```

глобальная `x` **не изменяется**.

Создаётся локальная:

```text
Global
└── x = 10

foo()
└── Local
    └── x = 20
```

После завершения:

```text
Global x = 10
```

---

# 14. Почему возникает `UnboundLocalError`

Классический вопрос на собеседовании:

```python
x = 10


def foo():
    print(x)
    x = 20
```

Что произойдёт?

`UnboundLocalError`.

Почему?

Потому что Python анализирует функцию и видит присваивание:

```python
x = 20
```

Следовательно, `x` считается локальным именем всей функции.

Получается:

```text
Local x
  ↓
print(x)
  ↓
попытка прочитать локальную переменную
до её присваивания
  ↓
UnboundLocalError
```

Если нужен глобальный `x`:

```python
x = 10


def foo():
    global x
    print(x)
    x = 20
```

---

# 15. Область видимости `if`, `for`, `while`

В Python `if`, `for`, `while` **не создают отдельную локальную область видимости**.

Например:

```python
if True:
    x = 10

print(x)
```

Работает:

```text
10
```

То же самое:

```python
for i in range(3):
    pass

print(i)
```

`i` остаётся доступной после цикла.

Но функция создаёт отдельный scope:

```python
def foo():
    x = 10


print(x)
```

Здесь:

```text
NameError
```

---

# 16. Класс — отдельный namespace, но не обычный enclosing scope

Классы имеют собственное пространство имён:

```python
class User:
    name = "Ilya"
```

Но важно не смешивать это с обычным `Enclosing` для функций.

Например:

```python
class User:
    name = "Ilya"

    def hello(self):
        print(name)
```

`name` здесь не будет автоматически найден как переменная класса через LEGB.

Для доступа к атрибуту класса используется:

```python
self.name
```

или:

```python
User.name
```

Это важное отличие **scope** от **attribute lookup**.

---

# 17. Scope ≠ Namespace

Эти понятия связаны, но не одинаковы.

### Scope

Определяет:

> где имя доступно.

### Namespace

Хранит:

> соответствия `имя → объект`.

Например:

```python
x = 10
```

В namespace существует связь:

```text
x → объект 10
```

А scope определяет, где это имя можно найти.

---

# 18. Замыкания и LEGB

LEGB непосредственно связан с **closures**.

```python
def make_multiplier(n):

    def multiply(x):
        return x * n

    return multiply
```

Создаём:

```python
double = make_multiplier(2)
```

После завершения `make_multiplier()` функция `multiply()` всё ещё имеет доступ к `n`.

Почему?

Потому что `n` находится в **Enclosing scope**.

```text
double()
   ↓
Local: x
   ↓
Enclosing: n = 2
   ↓
Global
   ↓
Built-in
```

---

# 19. Важная схема для запоминания

```text
                 LEGB
                  │
        ┌─────────┼─────────┐
        ↓         ↓         ↓
      Local   Enclosing   Global
                            │
                            ↓
                         Built-in
```

Или ещё проще:

```text
L → здесь
E → снаружи
G → модуль
B → Python
```

Где:

```text
L = текущая функция
E = внешняя функция
G = модуль
B = встроенные имена
```

---

# 🎤 Вопросы на собеседовании

### Что означает LEGB?

Порядок поиска имён:

**Local → Enclosing → Global → Built-in.**

### Где находится Local?

В текущей функции.

### Где находится Enclosing?

Во внешней функции при наличии вложенных функций.

### Что такое Global?

Пространство имён текущего модуля.

### Что такое Built-in?

Встроенные имена Python: `len`, `print`, `type`, `sum` и т.д.

### Что произойдёт?

```python
x = 10


def foo():
    x = 20


foo()

print(x)
```

**10**, потому что `x = 20` — локальная переменная `foo()`.

### Что произойдёт?

```python
x = 10


def foo():
    global x
    x = 20


foo()

print(x)
```

**20**, потому что `global x` указывает на глобальную переменную.

### Что делает `nonlocal`?

Позволяет вложенной функции изменять переменную из внешней функции.

### Создаёт ли `if` отдельный scope?

Нет.

### Создаёт ли `for` отдельный scope?

Нет.

### Создаёт ли функция отдельный scope?

Да.

### Почему возникает `UnboundLocalError`?

Потому что Python считает имя локальным из-за присваивания внутри функции, но попытка чтения происходит до присваивания.

### Чем scope отличается от namespace?

**Scope** определяет область доступности имени, а **namespace** хранит соответствия между именами и объектами.
