# 🔗 Замыкания и late binding

## 🎤 Короткий ответ

**Замыкание (closure)** — это функция, которая сохраняет доступ к переменным из внешней функции даже после завершения внешней функции.

```python
def make_multiplier(n):
    def multiply(x):
        return x * n

    return multiply


double = make_multiplier(2)

print(double(5))  # 10
```

`multiply()` помнит `n = 2`, хотя `make_multiplier()` уже завершилась.

**Late binding** — особенность замыканий: свободная переменная обычно разрешается **при выполнении внутренней функции**, а не в момент её создания.

Из-за этого возникает классическая проблема:

```python
funcs = []

for i in range(3):
    funcs.append(lambda: i)

print([func() for func in funcs])
```

Результат:

```text
[2, 2, 2]
```

Все функции обращаются к одной переменной `i`, а к моменту вызова цикла она уже равна `2`.

---

## 🗣️ Ответ на собеседовании

> Замыкание — это функция, которая захватывает переменные из внешней области видимости и сохраняет к ним доступ после завершения внешней функции.
>
> Например, фабрика функций может принять множитель и вернуть функцию, которая использует этот множитель. Возвращённая функция продолжает видеть переменную внешней функции.
>
> С этим связан механизм late binding. Свободная переменная в замыкании разрешается не в момент создания функции, а в момент её вызова. Поэтому при создании функций внутри цикла все они могут ссылаться на одну и ту же переменную цикла и при вызове получить её последнее значение.
>
> Классический пример — `lambda` внутри `for`: все функции возвращают последнее значение `i`. Чтобы зафиксировать значение на каждой итерации, можно использовать аргумент по умолчанию, например `lambda i=i: i`, либо `functools.partial`.

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
        ├── global
        ├── nonlocal
        └── Замыкания ← Я здесь
            ├── Free variables
            ├── Closure
            ├── __closure__
            ├── Late binding
            └── Раннее связывание
                ├── default arguments
                └── functools.partial
```

---

# 📚 Разбор поглубже

## 1. Что такое замыкание

Начнём с обычной вложенной функции:

```python
def outer():
    x = 10

    def inner():
        return x

    return inner
```

Получаем функцию:

```python
func = outer()
```

Хотя `outer()` уже завершилась, можно выполнить:

```python
print(func())
```

Результат:

```text
10
```

Почему `x` не исчезла?

Потому что `inner` сохранила доступ к переменной из внешней области.

Это и есть **замыкание**.

---

# 2. Что именно замыкается

Важно понимать:

**Замыкается не просто значение, а доступ к переменной из внешней области.**

Например:

```python
def outer():
    x = 10

    def inner():
        return x

    return inner
```

У `inner` есть свободная переменная `x`.

С точки зрения структуры:

```text
outer()
└── x = 10
    │
    └── inner()
        └── использует x
```

`x` находится в **Enclosing scope** относительно `inner`.

---

# 3. Free variable

Переменная `x` внутри `inner()` называется **свободной переменной (free variable)**.

```python
def outer():
    x = 10

    def inner():
        return x
```

Для `inner`:

```text
x
↓
не Local
↓
найдена во внешней функции
↓
free variable
```

Можно проверить:

```python
def outer():
    x = 10

    def inner():
        return x

    return inner


func = outer()

print(func.__code__.co_freevars)
```

Результат:

```text
('x',)
```

---

# 4. `__closure__`

У функции-замыкания можно посмотреть связанные с ней closure cells:

```python
def outer():
    x = 10

    def inner():
        return x

    return inner


func = outer()

print(func.__closure__)
```

`__closure__` содержит ячейки (`cell`), в которых находятся захваченные переменные.

Можно посмотреть их содержимое:

```python
for cell in func.__closure__:
    print(cell.cell_contents)
```

Получим:

```text
10
```

Упрощённо:

```text
func
├── code → код функции
└── closure
    └── cell → x = 10
```

---

# 5. Замыкание может изменять состояние

Для изменения переменной внешней функции используется `nonlocal`.

```python
def make_counter():
    count = 0

    def counter():
        nonlocal count
        count += 1
        return count

    return counter
```

Теперь:

```python
counter = make_counter()

print(counter())
print(counter())
print(counter())
```

Результат:

```text
1
2
3
```

Получается состояние, которое сохраняется между вызовами:

```text
counter
└── closure
    └── count
        ├── 0
        ├── 1
        ├── 2
        └── 3
```

---

# 6. Замыкание и `nonlocal`

Связь с предыдущей темой:

```text
Замыкание
└── функция использует переменную Enclosing scope
    │
    └── nonlocal
        └── позволяет изменить эту переменную
```

Без `nonlocal`:

```python
def outer():
    x = 0

    def inner():
        x += 1
```

будет `UnboundLocalError`.

С `nonlocal`:

```python
def outer():
    x = 0

    def inner():
        nonlocal x
        x += 1
```

работает.

---

# 7. Что такое late binding

Теперь самое важное.

**Late binding** означает, что свободная переменная в замыкании разрешается при выполнении функции, а не фиксируется своим текущим значением в момент создания функции.

Классический пример:

```python
funcs = []

for i in range(3):
    funcs.append(lambda: i)
```

Кажется, что мы создаём:

```text
lambda → 0
lambda → 1
lambda → 2
```

Но фактически все функции используют одну переменную:

```text
lambda 1 ──┐
lambda 2 ──┼──→ i
lambda 3 ──┘
```

После завершения цикла:

```text
i = 2
```

Поэтому:

```python
print([func() for func in funcs])
```

даст:

```text
[2, 2, 2]
```

---

# 8. Почему это происходит

Рассмотрим:

```python
funcs = []

for i in range(3):
    funcs.append(lambda: i)
```

На каждой итерации создаётся новая функция, но `i` не превращается автоматически в отдельную копию значения.

Упрощённо:

```text
i ──→ 0
 ↑
 ├── lambda 1

i ──→ 1
 ↑
 ├── lambda 1
 └── lambda 2

i ──→ 2
 ↑
 ├── lambda 1
 ├── lambda 2
 └── lambda 3
```

После цикла:

```text
i = 2
```

И все функции читают:

```text
i → 2
```

---

# 9. Как исправить через аргумент по умолчанию

Самый известный способ:

```python
funcs = []

for i in range(3):
    funcs.append(lambda i=i: i)
```

Теперь:

```python
print([func() for func in funcs])
```

получим:

```text
[0, 1, 2]
```

Почему?

Значение аргумента по умолчанию вычисляется при создании функции.

Получается:

```text
lambda i=0
lambda i=1
lambda i=2
```

То есть здесь используется механизм **раннего связывания значения через default argument**.

---

# 10. Важный нюанс: default argument и closure — разные механизмы

Сравним.

### Late binding

```python
funcs.append(lambda: i)
```

`i` — свободная переменная.

Она разрешается при вызове.

### Фиксация через default argument

```python
funcs.append(lambda i=i: i)
```

Значение `i` вычисляется при создании функции и сохраняется как default argument.

Поэтому это уже не тот же механизм closure.

---

# 11. Решение через обычную функцию

Можно сделать более явно:

```python
def make_func(value):
    def func():
        return value

    return func


funcs = []

for i in range(3):
    funcs.append(make_func(i))
```

Теперь:

```python
print([func() for func in funcs])
```

Результат:

```text
[0, 1, 2]
```

Каждый вызов `make_func(i)` создаёт своё отдельное окружение:

```text
func 1
└── value = 0

func 2
└── value = 1

func 3
└── value = 2
```

---

# 12. Решение через `functools.partial`

Ещё один вариант:

```python
from functools import partial


def multiply(x, n):
    return x * n


double = partial(multiply, n=2)
triple = partial(multiply, n=3)

print(double(5))
print(triple(5))
```

Результат:

```text
10
15
```

`partial` создаёт вызываемый объект, в котором часть аргументов уже зафиксирована.

---

# 13. Late binding особенно часто встречается в циклах

Классический вопрос на собеседовании:

```python
funcs = []

for i in range(3):
    funcs.append(lambda: i)

for func in funcs:
    print(func())
```

Ответ:

```text
2
2
2
```

Причина:

> Все функции замыкают одну переменную `i`, а не отдельную копию её значения на каждой итерации.

---

# 14. Более практический пример

Например:

```python
handlers = []

for event in ["create", "update", "delete"]:
    handlers.append(lambda: print(event))
```

Может показаться, что получим:

```text
create
update
delete
```

Но при вызове:

```python
for handler in handlers:
    handler()
```

получим:

```text
delete
delete
delete
```

Потому что все lambda используют одно и то же `event`.

Исправление:

```python
handlers = []

for event in ["create", "update", "delete"]:
    handlers.append(
        lambda event=event: print(event)
    )
```

Теперь:

```text
create
update
delete
```

---

# 15. Late binding и async

Эта проблема особенно неприятна в асинхронном коде, callback и обработчиках.

Например, концептуально:

```python
callbacks = []

for i in range(3):
    callbacks.append(lambda: print(i))
```

Позже callbacks могут выполняться уже после изменения `i`.

Поэтому при создании callback внутри цикла часто нужно явно фиксировать значение:

```python
lambda i=i: print(i)
```

или использовать отдельную фабрику функций.

---

# 16. Замыкание vs default argument

| Closure                                | Default argument                          |
| -------------------------------------- | ----------------------------------------- |
| Использует свободную переменную        | Использует параметр                       |
| Значение обычно разрешается при вызове | Значение фиксируется при создании функции |
| Может демонстрировать late binding     | Позволяет избежать late binding           |
| Работает через enclosing scope         | Работает через `__defaults__`             |

Например:

```python
lambda: i
```

против:

```python
lambda i=i: i
```

---

# 17. Как увидеть разницу

```python
def make():
    x = 10

    def closure():
        return x

    return closure


func = make()
```

Здесь:

```python
print(func.__code__.co_freevars)
```

покажет:

```text
('x',)
```

А при default argument:

```python
func = lambda x=10: x
```

значение находится среди default arguments:

```python
print(func.__defaults__)
```

Результат:

```text
(10,)
```

То есть механизмы действительно разные.

---

# 18. Важная формула

```text
Замыкание
    ↓
функция + доступ к Enclosing scope
    ↓
free variable
    ↓
значение обычно ищется при вызове
    ↓
Late Binding
```

А для фиксации:

```text
lambda x=x: x
        ↓
default argument
        ↓
значение фиксируется при создании функции
```

---

# 🎤 Вопросы на собеседовании

### Что такое замыкание?

Функция, которая сохраняет доступ к переменным из внешней области видимости после завершения внешней функции.

### Что такое free variable?

Переменная, используемая функцией, но не определённая в её локальной области. Например, переменная из enclosing scope.

### Что такое late binding?

Механизм, при котором свободная переменная в замыкании разрешается при выполнении функции, а не фиксируется значением в момент создания.

### Что выведет код?

```python
funcs = []

for i in range(3):
    funcs.append(lambda: i)

print([f() for f in funcs])
```

Ответ:

```text
[2, 2, 2]
```

### Почему `[2, 2, 2]`, а не `[0, 1, 2]`?

Все lambda ссылаются на одну переменную `i`. При вызове цикла уже завершён, и `i == 2`.

### Как исправить?

Через default argument:

```python
funcs = []

for i in range(3):
    funcs.append(lambda i=i: i)
```

Или через фабрику функций:

```python
def make_func(value):
    def func():
        return value

    return func
```

### Что делает `nonlocal` в замыкании?

Позволяет изменять переменную внешней функции:

```python
def outer():
    count = 0

    def inner():
        nonlocal count
        count += 1
```

### Как проверить, есть ли у функции closure?

Можно посмотреть:

```python
func.__closure__
```

А имена свободных переменных:

```python
func.__code__.co_freevars
```

### В чём разница между

```python
lambda: i
```

и

```python
lambda i=i: i
```

В первом случае `i` — свободная переменная, и возникает late binding. Во втором значение текущего `i` используется как default argument и фиксируется при создании функции.

### Связь с LEGB

```text
LEGB
└── Enclosing
    └── Замыкания
        └── Free variables
            └── Late binding
```
