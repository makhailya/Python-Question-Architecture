# Неизменяемое множество — Frozenset

## Коротко

`frozenset` — это **неизменяемая версия `set`**.

Как и `set`, он хранит только уникальные хешируемые элементы, но после создания его нельзя изменить.

Главное отличие:

- `set` — mutable;
- `frozenset` — immutable.

## На собеседовании достаточно сказать

> `frozenset` — это неизменяемое множество уникальных хешируемых элементов.
>
> В отличие от `set`, его нельзя изменить после создания. Благодаря неизменяемости `frozenset` является хешируемым и может использоваться, например, как ключ `dict` или элемент другого `set`.

## Что важно помнить

Обычный `set` можно изменять:

```python
numbers = {1, 2, 3}

numbers.add(4)
numbers.remove(2)
```

frozenset изменить нельзя:

```python
numbers = frozenset({1, 2, 3})

numbers.add(4)
# AttributeError
```

При этом frozenset можно использовать как ключ словаря:

```python
permissions = frozenset({"read", "write"})

data = {
    permissions: "admin"
}
```

Обычный set использовать как ключ нельзя:

```python
permissions = {"read", "write"}

data = {
    permissions: "admin"
}

# TypeError: unhashable type: 'set'
```

## Простой пример

Создаём обычное множество:

```python
numbers = {1, 2, 3}

numbers.add(4)

print(numbers)
# {1, 2, 3, 4}
```

Создаём frozenset:

```python
numbers = frozenset({1, 2, 3})

print(numbers)
# frozenset({1, 2, 3})
```

Попытка изменить его:

```python
numbers.add(4)

# AttributeError
```

## Главное

Запомни простую связь:

```
set
 ↓
mutable
 ↓
unhashable
 ↓
нельзя использовать как ключ dict


frozenset
 ↓
immutable
 ↓
hashable
 ↓
можно использовать как ключ dict
```

При этом оба типа:

* хранят уникальные элементы;
* не поддерживают дубликаты;
* используют хеширование;
* позволяют быстро проверять наличие элемента в среднем за O(1).

## Связи

* [[set]]
* [[dict]]
* [[Хешируемый объект — Hashable Object]]
* [[1.1 Изменяемые и неизменяемые типы данных — Mutable vs Immutable]]
* [[4. Хеш-таблица — Hash Table]]
* [[Словарь и множество — Dict vs Set]]