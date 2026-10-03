# `frozenset` — неизменяемое множество

## 🎯 Ответ на собеседовании

> **`frozenset` — встроенный неизменяемый (`immutable`) аналог `set`. Он хранит уникальные hashable-элементы, не поддерживает изменение содержимого и поэтому сам является hashable. Благодаря этому `frozenset` можно использовать как ключ словаря или элемент другого `set`.**

### Если спросят отличие от `set`

> **`set` — изменяемый и unhashable, а `frozenset` — неизменяемый и hashable. Поэтому `frozenset` можно использовать там, где требуется hashable-объект.**

```python
set([1, 2, 3])
# изменяемый

frozenset([1, 2, 3])
# неизменяемый
```

---

# Коротко

`frozenset` — это **множество, которое нельзя изменить**.

```python
numbers = frozenset([1, 2, 3])

print(numbers)
# frozenset({1, 2, 3})
```

Как и `set`, он:

* хранит только уникальные элементы;
* не сохраняет порядок элементов;
* поддерживает операции над множествами.

---

# Создание

### Из списка

```python
numbers = frozenset([1, 2, 3, 3, 2])

print(numbers)
# frozenset({1, 2, 3})
```

Дубликаты удаляются.

### Из `set`

```python
numbers = frozenset({1, 2, 3})
```

### Из строки

```python
letters = frozenset("hello")

print(letters)
# frozenset({'h', 'e', 'l', 'o'})
```

Каждый символ рассматривается как отдельный элемент.

---

# Уникальность

Как и `set`, `frozenset` автоматически убирает дубликаты:

```python
numbers = frozenset([1, 1, 2, 2, 3])

print(numbers)
# frozenset({1, 2, 3})
```

---

# Порядок не гарантируется

Нельзя рассчитывать на порядок:

```python
numbers = frozenset([3, 1, 2])

print(numbers)
```

Не следует ожидать:

```text
3, 1, 2
```

`frozenset` — **неупорядоченная коллекция**.

Если нужен порядок — используй `list` или `tuple`.

---

# Immutable

Главное отличие от `set`:

```python
numbers = frozenset([1, 2, 3])
```

Нельзя:

```python
numbers.add(4)
```

или:

```python
numbers.remove(1)
```

или:

```python
numbers.clear()
```

Таких методов у `frozenset` нет.

---

# `set` vs `frozenset`

```text
set
 ↓
mutable
 ↓
нельзя hash()
 ↓
нельзя ключом dict

frozenset
 ↓
immutable
 ↓
можно hash()
 ↓
можно ключом dict
```

---

# Hashable

Это одна из самых важных особенностей `frozenset`.

```python
numbers = frozenset([1, 2, 3])

hash(numbers)
```

Работает.

А:

```python
numbers = {1, 2, 3}

hash(numbers)
```

даст:

```text
TypeError: unhashable type: 'set'
```

---

# `frozenset` как ключ словаря

Поскольку `frozenset` hashable, его можно использовать как ключ:

```python
permissions = {
    frozenset({"read", "write"}): "editor"
}
```

Получить значение:

```python
permissions[frozenset({"write", "read"})]
# 'editor'
```

Обрати внимание: порядок элементов не имеет значения.

```python
frozenset({"read", "write"}) == frozenset({"write", "read"})
# True
```

Поэтому такой тип может быть удобен, когда ключ представляет **набор элементов**, а не последовательность.

---

# `frozenset` внутри `set`

`frozenset` можно использовать как элемент другого множества:

```python
data = {
    frozenset({1, 2}),
    frozenset({3, 4})
}
```

А обычный `set`:

```python
data = {
    {1, 2}
}
```

нельзя использовать как элемент другого `set`.

Причина — `set` является unhashable.

---

# Важный нюанс ⚠️

`frozenset` может содержать только **hashable-элементы**.

Можно:

```python
frozenset([1, 2, 3])
```

Можно:

```python
frozenset(["Python", "Java"])
```

Можно:

```python
frozenset([(1, 2), (3, 4)])
```

Но нельзя:

```python
frozenset([[1, 2], [3, 4]])
```

Потому что `list` — unhashable.

Получим:

```text
TypeError: unhashable type: 'list'
```

---

# Операции над множествами

Несмотря на immutable, `frozenset` поддерживает операции над множествами.

```python
a = frozenset([1, 2, 3])
b = frozenset([3, 4, 5])
```

### Объединение

```python
a | b
# frozenset({1, 2, 3, 4, 5})
```

### Пересечение

```python
a & b
# frozenset({3})
```

### Разность

```python
a - b
# frozenset({1, 2})
```

### Симметрическая разность

```python
a ^ b
# frozenset({1, 2, 4, 5})
```

---

# Проверка принадлежности

```python
numbers = frozenset([1, 2, 3])

2 in numbers
# True

5 in numbers
# False
```

Операция `in` для множества обычно выполняется очень эффективно — в среднем **O(1)**.

---

# Методы

У `frozenset` нет методов изменения содержимого.

Но есть методы для работы с множествами:

```python
a = frozenset([1, 2, 3])
b = frozenset([2, 3, 4])
```

```python
a.union(b)
# frozenset({1, 2, 3, 4})

a.intersection(b)
# frozenset({2, 3})

a.difference(b)
# frozenset({1})

a.symmetric_difference(b)
# frozenset({1, 4})
```

Также:

```python
a.issubset(b)
a.issuperset(b)
a.isdisjoint(b)
```

---

# `len()`

Количество элементов:

```python
numbers = frozenset([10, 20, 30])

len(numbers)
# 3
```

---

# `tuple` vs `frozenset`

Оба типа immutable, но предназначены для разных задач.

|                     | `tuple` | `frozenset` |
| ------------------- | ------- | ----------- |
| Immutable           | ✅       | ✅           |
| Упорядоченный       | ✅       | ❌           |
| Индексация          | ✅       | ❌           |
| Уникальные элементы | ❌       | ✅           |
| Дубликаты           | ✅       | ❌           |
| Hashable            | ✅*      | ✅*          |
| Ключ `dict`         | ✅*      | ✅*          |

`*` При условии, что все вложенные элементы hashable.

Главное отличие:

```text
tuple     → важен порядок
frozenset → порядок не важен
```

Например:

```python
(1, 2) != (2, 1)
```

Но:

```python
frozenset({1, 2}) == frozenset({2, 1})
```

---

# `set` vs `frozenset`

|                     | `set` | `frozenset` |
| ------------------- | ----- | ----------- |
| Уникальные элементы | ✅     | ✅           |
| Упорядоченный       | ❌     | ❌           |
| Mutable             | ✅     | ❌           |
| Hashable            | ❌     | ✅           |
| `.add()`            | ✅     | ❌           |
| `.remove()`         | ✅     | ❌           |
| `.union()`          | ✅     | ✅           |
| `.intersection()`   | ✅     | ✅           |
| Ключ `dict`         | ❌     | ✅           |
| Элемент `set`       | ❌     | ✅           |

---

# Где используется?

`frozenset` нужен, когда необходимо представить **неизменяемый набор уникальных объектов**.

Например:

### Права пользователя

```python
permissions = frozenset({
    "read",
    "write",
    "delete"
})
```

### Комбинации элементов

```python
combination = frozenset(["A", "B"])
```

### Ключ словаря

```python
cache = {
    frozenset({"python", "backend"}): "result"
}
```

Особенно полезен, когда **порядок элементов не имеет значения**, но сам набор нужно сделать hashable.

---

# 🧠 Шпаргалка

```text
frozenset
│
├── множество
├── immutable
├── уникальные элементы
├── порядок не гарантируется
│
├── hashable
│    ├── ключ dict
│    └── элемент set
│
├── только hashable-элементы внутри
│
├── union()
├── intersection()
├── difference()
├── symmetric_difference()
│
└── нет add/remove/clear
```

## ⭐ Самое главное

```python
set        → mutable → unhashable
frozenset  → immutable → hashable
```

И ещё одна важная связка:

```text
tuple
→ порядок важен
→ immutable
→ hashable*

frozenset
→ порядок НЕ важен
→ immutable
→ hashable*
```

**Главная мысль: `frozenset` — это неизменяемое множество уникальных hashable-элементов, которое можно использовать как ключ `dict` или элемент другого `set`.**
