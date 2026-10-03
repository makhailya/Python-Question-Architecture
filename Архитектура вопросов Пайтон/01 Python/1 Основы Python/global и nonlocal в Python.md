# 🔗 `global` и `nonlocal` в Python

## 🎤 Короткий ответ

`global` и `nonlocal` нужны, чтобы **изменять переменные за пределами локальной области текущей функции**.

* `global` — обращается к переменной **глобальной области модуля**.
* `nonlocal` — обращается к переменной **внешней функции** во вложенной функции.

```python
counter = 0


def increment():
    global counter
    counter += 1
```

Здесь `global` говорит: `counter` находится не локально, а на уровне модуля.

Для `nonlocal`:

```python
def outer():
    counter = 0

    def increment():
        nonlocal counter
        counter += 1

    return increment
```

Здесь `counter` находится в `outer()`, а `increment()` получает возможность изменить его.

---

## 🗣️ Ответ на собеседовании

> `global` и `nonlocal` управляют тем, в какой области видимости Python должен искать переменную при присваивании.
>
> По умолчанию присваивание внутри функции создаёт локальную переменную. `global` позволяет явно указать, что мы работаем с переменной глобальной области модуля.
>
> `nonlocal` используется во вложенной функции и позволяет изменять переменную из ближайшей внешней функции.
>
> Главное отличие: `global` работает с Global scope, а `nonlocal` — с Enclosing scope.
>
> Например, `global` можно использовать для изменения глобального счётчика, а `nonlocal` — для реализации состояния замыкания.
>
> На практике чрезмерное использование `global` обычно ухудшает структуру программы, поэтому состояние чаще передают явно или инкапсулируют в объекте.

---

## 🧭 Где я нахожусь

```text
01 Python
└── 01 Основы Python
    ├── Переменные и имена
    ├── Объекты и ссылки
    ├── Функции
    └── Области видимости
        ├── LEGB
        ├── global ← Я здесь
        ├── nonlocal ← Я здесь
        └── Замыкания
```

---

# 📚 Разбор поглубже

## 1. Почему вообще нужны `global` и `nonlocal`

Рассмотрим обычное присваивание:

```python
x = 10


def foo():
    x = 20
```

После вызова:

```python
foo()

print(x)
```

получим:

```text
10
```

Почему?

Потому что:

```python
x = 20
```

внутри функции создаёт **локальное имя `x`**.

Получается:

```text
Global
└── x → 10

foo()
└── Local
    └── x → 20
```

Локальная `x` не изменяет глобальную.

---

# 2. `global`

`global` сообщает Python:

> Это имя относится к глобальной области текущего модуля.

```python
counter = 0


def increment():
    global counter
    counter += 1
```

Теперь:

```python
increment()
increment()

print(counter)
```

Получим:

```text
2
```

Схема:

```text
Global
└── counter = 0
       ↑
       │
increment()
└── global counter
       │
       └── изменяет Global counter
```

---

# 3. Что происходит без `global`

Рассмотрим:

```python
counter = 0


def increment():
    counter += 1
```

Кажется логичным, что Python должен взять глобальный `counter` и увеличить его.

Но будет:

```text
UnboundLocalError
```

Почему?

Оператор:

```python
counter += 1
```

с точки зрения Python подразумевает чтение и последующее присваивание:

```python
counter = counter + 1
```

А из-за присваивания Python считает `counter` **локальным именем функции**.

Получается:

```text
Local counter
    ↓
counter + 1
    ↓
пытаемся прочитать локальную переменную
до её значения
    ↓
UnboundLocalError
```

---

# 4. `global` не нужен для чтения

Это важный нюанс.

Если мы только читаем глобальную переменную:

```python
name = "Ilya"


def greet():
    print(name)
```

`global` не нужен.

Python найдёт `name` через LEGB:

```text
Local       ❌
Enclosing   ❌
Global      ✅
```

`global` нужен, когда внутри функции необходимо **присваивание этому глобальному имени**.

---

# 5. `global` не делает объект глобальным

Важно понимать, что `global` относится именно к **имени**, а не к объекту.

Например:

```python
items = []


def add_item():
    items.append(1)
```

Здесь `global` не нужен.

Почему?

Мы не переназначаем имя `items`.

Мы изменяем объект, на который оно ссылается:

```text
Global
└── items ──→ []

              ↓ append()

Global
└── items ──→ [1]
```

А вот это уже требует `global`:

```python
items = []


def reset():
    global items
    items = []
```

Здесь происходит **перепривязка имени** `items`.

---

# 6. `nonlocal`

`nonlocal` работает с **переменной внешней функции**.

Например:

```python
def outer():
    counter = 0

    def inner():
        nonlocal counter
        counter += 1

    inner()
    print(counter)
```

Получим:

```text
1
```

Здесь:

```text
outer()
└── counter = 0       ← Enclosing
    │
    └── inner()
        └── nonlocal counter
```

`inner()` изменяет `counter` из `outer()`.

---

# 7. Почему без `nonlocal` не работает так, как ожидаем

Рассмотрим:

```python
def outer():
    counter = 0

    def inner():
        counter += 1

    inner()
```

Получим:

```text
UnboundLocalError
```

Причина такая же, как с `global`.

Из-за:

```python
counter += 1
```

Python считает `counter` локальным для `inner()`.

Получается:

```text
outer
└── counter = 0

inner
└── Local counter
    ↓
    counter += 1
    ↓
    читаем Local counter до присваивания
    ↓
    UnboundLocalError
```

`nonlocal` говорит:

> Не создавай локальный `counter`. Используй переменную из внешней функции.

---

# 8. `nonlocal` работает не с Global

Например:

```python
counter = 0


def outer():

    def inner():
        nonlocal counter
```

Так нельзя.

Будет:

```text
SyntaxError
```

Почему?

Потому что `nonlocal` ищет переменную в **Enclosing scope**, а глобальная переменная находится в **Global scope**.

Для глобальной переменной нужен:

```python
global counter
```

---

# 9. `global` vs `nonlocal`

Главная таблица:

|                           | `global`             | `nonlocal`                 |
| ------------------------- | -------------------- | -------------------------- |
| Область                   | Global               | Enclosing                  |
| Что изменяет              | переменную модуля    | переменную внешней функции |
| Требует вложенную функцию | Нет                  | Да                         |
| Типичный сценарий         | глобальное состояние | замыкание                  |
| Пример                    | глобальный счётчик   | счётчик внутри closure     |

Запомнить можно так:

```text
global
   ↓
G — Global

nonlocal
   ↓
E — Enclosing
```

---

# 10. Пример с LEGB

```python
x = "global"


def outer():
    x = "enclosing"

    def inner():
        x = "local"
        print(x)

    inner()
```

Здесь:

```text
L → x = "local"       ← найдено здесь
E → x = "enclosing"
G → x = "global"
B → builtins
```

Если написать:

```python
def inner():
    print(x)
```

тогда:

```text
L → ❌
E → x = "enclosing"   ← найдено здесь
```

Если и `outer()` не содержит `x`:

```text
L → ❌
E → ❌
G → x = "global"      ← найдено здесь
```

---

# 11. `nonlocal` и замыкания

`nonlocal` особенно часто встречается в **closures**.

```python
def make_counter():
    count = 0

    def counter():
        nonlocal count
        count += 1
        return count

    return counter
```

Создаём:

```python
counter = make_counter()
```

Теперь:

```python
counter()
counter()
counter()
```

Результат:

```text
1
2
3
```

Функция `counter` сохраняет доступ к `count`, хотя `make_counter()` уже завершилась.

```text
make_counter()
└── count = 0
    │
    └── counter()
         │
         └── nonlocal count
```

Это классический пример **замыкания с изменяемым состоянием**.

---

# 12. Несколько замыканий имеют независимое состояние

```python
def make_counter():
    count = 0

    def counter():
        nonlocal count
        count += 1
        return count

    return counter


counter1 = make_counter()
counter2 = make_counter()
```

Теперь:

```python
counter1()
counter1()
counter2()
```

Получим:

```text
1
2
1
```

Почему?

Потому что каждый вызов `make_counter()` создаёт своё окружение:

```text
counter1
└── count = 0 → 1 → 2

counter2
└── count = 0 → 1
```

---

# 13. `global` и `nonlocal` не нужны для изменения mutable-объекта

Это очень частый вопрос.

```python
def outer():
    items = []

    def inner():
        items.append(1)

    inner()

    print(items)
```

Работает без `nonlocal`.

Почему?

Потому что мы **не переназначаем имя `items`**.

Мы вызываем метод объекта:

```python
items.append(1)
```

А вот:

```python
def outer():
    items = []

    def inner():
        items = [1]
```

создаст новую локальную `items` внутри `inner()`.

Если хотим изменить именно `items` из `outer()`:

```python
def outer():
    items = []

    def inner():
        nonlocal items
        items = [1]
```

---

# 14. `global` vs изменение mutable-объекта

Аналогично:

```python
items = []


def add():
    items.append(1)
```

Работает.

Но:

```python
items = []


def reset():
    items = []
```

не изменит глобальную переменную.

Нужно:

```python
items = []


def reset():
    global items
    items = []
```

Главное правило:

> `global` и `nonlocal` нужны для **перепривязки имени**, а не просто для изменения объекта, на который имя ссылается.

---

# 15. Когда `global` и `nonlocal` использовать не стоит

Они являются нормальными возможностями Python, но чрезмерное использование усложняет код.

Например:

```python
counter = 0


def increment():
    global counter
    counter += 1
```

Функция теперь зависит от внешнего глобального состояния.

Часто лучше:

```python
def increment(counter):
    return counter + 1
```

или использовать объект:

```python
class Counter:
    def __init__(self):
        self.value = 0

    def increment(self):
        self.value += 1
```

Так состояние становится более явным.

---

# 16. Частая ловушка на собеседовании

Что выведет код?

```python
x = 10


def foo():
    x = 20

    def bar():
        print(x)

    bar()


foo()
```

Ответ:

```text
20
```

Почему?

`bar()` ищет `x`:

```text
Local      → ❌
Enclosing  → 20 ✅
```

---

А здесь:

```python
x = 10


def foo():
    x = 20

    def bar():
        global x
        print(x)

    bar()


foo()
```

Ответ:

```text
10
```

Потому что `global x` заставляет `bar()` искать `x` в Global, пропуская `foo()` как Enclosing scope.

---

# 🎤 Вопросы на собеседовании

### Чем `global` отличается от `nonlocal`?

`global` обращается к переменной глобальной области модуля, `nonlocal` — к переменной ближайшей внешней функции.

### Нужен ли `global` для чтения глобальной переменной?

Нет.

```python
x = 10


def foo():
    print(x)
```

Работает благодаря LEGB.

### Когда нужен `global`?

Когда внутри функции нужно **перепривязать глобальное имя**:

```python
x = 10


def foo():
    global x
    x = 20
```

### Когда нужен `nonlocal`?

Когда вложенная функция должна изменить переменную внешней функции:

```python
def outer():
    x = 10

    def inner():
        nonlocal x
        x = 20
```

### Можно ли использовать `nonlocal` для глобальной переменной?

Нет. `nonlocal` работает только с переменными Enclosing scope.

### Нужен ли `nonlocal` для `items.append()`?

Нет:

```python
def outer():
    items = []

    def inner():
        items.append(1)
```

Мы изменяем объект, а не перепривязываем имя.

### Почему возникает `UnboundLocalError`?

Если внутри функции есть присваивание имени, Python считает его локальным. При попытке прочитать это локальное имя до присваивания возникает `UnboundLocalError`.

### Что запомнить одной формулой?

```text
global   → Global
nonlocal → Enclosing
```

И ещё одна:

```text
изменить объект      → global/nonlocal обычно не нужны
перепривязать имя    → может понадобиться global/nonlocal
```
