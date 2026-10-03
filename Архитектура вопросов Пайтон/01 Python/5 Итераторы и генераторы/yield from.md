# 🔗 `yield from` в Python

## 🎤 Короткий ответ

`yield from` позволяет **делегировать выдачу значений другому итерируемому объекту или генератору**.

В простом случае:

```python
def numbers():
    yield from [1, 2, 3]
```

примерно соответствует:

```python
def numbers():
    for value in [1, 2, 3]:
        yield value
```

Но `yield from` — не просто сокращение цикла. При работе с генераторами он также делегирует `send()`, `throw()` и `close()`, а после завершения подгенератора позволяет получить значение его `return`.

---

## 🗣️ Ответ на собеседовании

`yield from` используется внутри генератора для делегирования работы другому итерируемому объекту или генератору.

В простейшем случае он последовательно выдаёт все значения вложенного объекта:

```python
def child():
    yield 1
    yield 2


def parent():
    yield from child()
```

При итерации `parent()` получим `1` и `2`.

Главное отличие от обычного `for` с `yield` проявляется при работе именно с генераторами. `yield from` прозрачно делегирует взаимодействие с подгенератором: значения, `send()`, `throw()`, `close()`, а также получает значение, с которым подгенератор завершился через `return`.

Поэтому `yield from` можно рассматривать как **полноценное делегирование генератору**, а не просто как удобную запись цикла.

---

## 🧭 Где я нахожусь

```text
01 Python
└── 05 Итераторы и генераторы
    ├── Iterable
    ├── Iterator
    ├── iter()
    ├── next()
    ├── Generator
    │   ├── yield
    │   └── yield from ← Я здесь
    ├── send()
    ├── throw()
    └── close()
```

---

## 📚 Разбор поглубже

### 1. Зачем нужен `yield from`

Представим два генератора:

```python
def child():
    yield 1
    yield 2
    yield 3
```

Нужно сделать генератор, который выдаёт те же значения.

Можно написать:

```python
def parent():
    for value in child():
        yield value
```

Но Python предоставляет специальный механизм:

```python
def parent():
    yield from child()
```

Теперь:

```python
print(list(parent()))
```

получим:

```text
[1, 2, 3]
```

---

### 2. `yield from` с обычным iterable

`yield from` работает не только с генераторами.

Например:

```python
def numbers():
    yield from [1, 2, 3]
```

или:

```python
def letters():
    yield from "ABC"
```

Можно использовать:

```python
print(list(numbers()))
# [1, 2, 3]

print(list(letters()))
# ['A', 'B', 'C']
```

То есть справа от `yield from` должен находиться **итерируемый объект**.

---

### 3. Простая модель

```text
parent()
   │
   │ yield from
   ▼
child()
   │
   ├── yield 1
   ├── yield 2
   └── yield 3
```

Значения проходят через `parent()` наружу:

```text
child
 ↓
yield 1 ──────┐
yield 2 ──────┼──→ вызывающий код
yield 3 ──────┘
```

Сам `parent()` не обязан вручную делать:

```python
for value in child():
    yield value
```

---

## 4. `yield from` и обычный `for`

Для простого iterable:

```python
def parent():
    yield from [1, 2, 3]
```

концептуально близко к:

```python
def parent():
    for value in [1, 2, 3]:
        yield value
```

Но это **не полная эквивалентность** для генераторов.

У `yield from` есть дополнительная семантика делегирования.

Именно она делает его важным.

---

## 5. Делегирование `next()`

Рассмотрим:

```python
def child():
    yield 1
    yield 2


def parent():
    yield from child()
```

Когда вызываем:

```python
gen = parent()

next(gen)
```

выполнение фактически передаётся `child()`:

```text
next(parent)
     ↓
yield from
     ↓
child
     ↓
yield 1
     ↓
1
```

Следующий:

```python
next(gen)
```

продолжает `child()`:

```text
next(parent)
     ↓
yield from
     ↓
child
     ↓
yield 2
     ↓
2
```

---

## 6. `yield from` и `send()`

Здесь начинается отличие от обычного цикла.

Генератор может принимать значение через `send()`:

```python
def child():
    value = yield "готов"
    yield value * 2
```

Родитель:

```python
def parent():
    result = yield from child()
    print(result)
```

Использование:

```python
gen = parent()

print(next(gen))
```

Получим:

```text
готов
```

Теперь:

```python
print(gen.send(10))
```

Получим:

```text
20
```

`send(10)` был делегирован из `parent()` в `child()`.

То есть:

```text
caller
  │
  │ send(10)
  ▼
parent()
  │
  │ yield from
  ▼
child()
  │
  │ получает 10
  ▼
yield 20
```

---

## 7. `yield from` и `return`

Это одна из самых важных особенностей.

Подгенератор может завершиться через `return`:

```python
def child():
    yield 1
    yield 2
    return "готово"
```

Сам `return` не выдаёт `"готово"` через `yield`.

Он завершает генератор.

Но `yield from` получает это значение:

```python
def parent():
    result = yield from child()
    print(result)
```

После завершения `child()`:

```text
result == "готово"
```

То есть:

```text
child()
  │
  ├── yield 1
  ├── yield 2
  └── return "готово"
             ↓
      StopIteration("готово")
             ↓
        yield from
             ↓
       result = "готово"
```

---

## 8. Почему `return` попадает в `StopIteration`

У генератора:

```python
def child():
    yield 1
    return "done"
```

после последнего `yield` происходит завершение.

Значение `"done"` становится значением:

```python
StopIteration.value
```

`yield from` автоматически извлекает это значение и делает его результатом выражения:

```python
result = yield from child()
```

Именно поэтому можно написать:

```python
def parent():
    result = yield from child()
    return result
```

---

## 9. `yield from` — полноценное делегирование

Удобно запомнить модель:

```text
yield from generator
        │
        ├── значения yield
        ├── next()
        ├── send()
        ├── throw()
        ├── close()
        └── return-value
```

То есть `yield from` делегирует управление подгенератору до тех пор, пока тот не завершится.

---

## 10. `throw()`

Генератор может получить исключение через:

```python
generator.throw(...)
```

Например:

```python
def child():
    try:
        yield 1
    except ValueError:
        yield 100
```

Родитель:

```python
def parent():
    yield from child()
```

Если исключение направляется в `parent()` в точку `yield from`, оно может быть передано делегируемому генератору `child()`.

Это позволяет строить композицию генераторов, сохраняя обработку исключений внутри подгенератора.

---

## 11. `close()`

Аналогично работает закрытие генератора.

Если генератор делегирует выполнение через:

```python
yield from child()
```

закрытие внешнего генератора корректно передаётся делегируемому генератору.

Это важно, если у подгенератора есть очистка ресурсов:

```python
def child():
    try:
        yield 1
    finally:
        print("cleanup")
```

---

## 12. Практический пример: объединение генераторов

Допустим, есть несколько источников данных:

```python
def users():
    yield "Ilya"
    yield "Anna"


def admins():
    yield "Admin1"
    yield "Admin2"
```

Можно объединить их:

```python
def all_users():
    yield from users()
    yield from admins()
```

Теперь:

```python
print(list(all_users()))
```

Результат:

```text
['Ilya', 'Anna', 'Admin1', 'Admin2']
```

Получается простой pipeline:

```text
users()
   ↓
yield from
   ↓
admins()
   ↓
yield from
   ↓
all_users()
```

---

## 13. Рекурсивный пример

`yield from` особенно удобно использовать с рекурсивными генераторами.

Например, обход дерева:

```python
def walk(node):
    yield node.value

    for child in node.children:
        yield from walk(child)
```

Здесь `yield from` означает:

> «отдай наружу все значения, которые выдаст рекурсивный вызов `walk(child)`».

Без него пришлось бы вручную перебирать каждый вложенный генератор.

---

## 14. `yield from` и рекурсивная структура

Концептуально:

```text
root
├── A
│   ├── A1
│   └── A2
└── B
    ├── B1
    └── B2
```

Генератор:

```python
def walk(node):
    yield node.value

    for child in node.children:
        yield from walk(child)
```

Получает поток:

```text
root
A
A1
A2
B
B1
B2
```

Каждый рекурсивный вызов делегирует свои значения наверх.

---

## 15. `yield from` не создаёт промежуточный список

Сравним:

```python
def parent():
    values = list(child())

    for value in values:
        yield value
```

Здесь сначала создаётся список.

С `yield from`:

```python
def parent():
    yield from child()
```

можно продолжать ленивую обработку.

Это особенно полезно для больших потоков данных.

---

## 16. Частая ошибка на собеседовании

❌ Неполный ответ:

> `yield from` — это сокращение `for` + `yield`.

Для простого списка это действительно похоже:

```python
yield from [1, 2, 3]
```

Но для генераторов такое объяснение неполное.

Правильнее:

> `yield from` — это механизм делегирования генератору или другому iterable. Помимо выдачи значений, он обеспечивает передачу управления и взаимодействия `send()`, `throw()`, `close()` и позволяет получить значение, возвращённое подгенератором через `return`.

---

## 17. `yield` vs `yield from`

| `yield`                              | `yield from`                           |
| ------------------------------------ | -------------------------------------- |
| Выдаёт одно значение                 | Делегирует выдачу другому iterable     |
| `yield value`                        | `yield from iterable`                  |
| Сам управляет точкой выдачи          | Передаёт управление другому генератору |
| Можно использовать самостоятельно    | Используется внутри генератора         |
| Не извлекает `return` подгенератора  | Получает `return` подгенератора        |
| Не является механизмом делегирования | Является механизмом делегирования      |

---

## 18. `yield from` vs `for + yield`

Простой случай:

```python
def parent():
    for x in child():
        yield x
```

и:

```python
def parent():
    yield from child()
```

дают одинаковый поток значений.

Но:

```text
for + yield
    ↓
ручное управление итерацией

yield from
    ↓
делегирование протоколу генератора
    ├── next
    ├── send
    ├── throw
    ├── close
    └── return value
```

Поэтому второй вариант семантически богаче.

---

## 🎤 Вопросы на собеседовании

**Что такое `yield from`?**

`yield from` делегирует выдачу значений другому iterable или генератору. При работе с генераторами он также делегирует `send()`, `throw()` и `close()` и получает значение, с которым подгенератор завершился через `return`.

**Чем `yield from` отличается от `yield`?**

`yield` выдаёт конкретное значение, а `yield from` передаёт управление другому iterable или генератору.

**Чем `yield from` отличается от `for + yield`?**

Для простого перебора значения будут одинаковыми, но `yield from` дополнительно делегирует взаимодействие с подгенератором и получает его `return`-значение.

**Что произойдёт с `return` внутри подгенератора?**

Подгенератор завершится, а значение `return` попадёт в `StopIteration.value`. `yield from` извлечёт его:

```python
result = yield from child()
```

**Можно ли использовать `yield from` со списком?**

Да:

```python
def numbers():
    yield from [1, 2, 3]
```

**Что произойдёт при `send()` в генератор с `yield from`?**

Если делегируется именно генератор, `send()` передаётся ему.

**Зачем `yield from` нужен в рекурсии?**

Он позволяет делегировать выдачу значений рекурсивному генератору без ручного перебора его результатов.

**Главная формула для собеседования:**

> **`yield` — выдаю значение и приостанавливаюсь. `yield from` — передаю управление другому генератору и делегирую ему работу до завершения.**
