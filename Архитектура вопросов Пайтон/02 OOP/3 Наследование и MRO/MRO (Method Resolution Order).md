# MRO — Method Resolution Order

## 🎯 Ответ на собеседовании

**MRO (Method Resolution Order)** — это порядок, в котором Python ищет методы и атрибуты в классе и его родителях при наследовании.

MRO особенно важен при [[Множественное наследование]], когда у класса несколько родителей.

Получить MRO можно через:

```python
ClassName.mro()
```

или:

```python
ClassName.__mro__
```

---

## 📌 Простой пример

```python
class A:
    def hello(self):
        print("A")


class B(A):
    pass


class C(B):
    pass
```

MRO:

```python
print(C.mro())
```

Результат:

```text
C → B → A → object
```

Если вызвать:

```python
C().hello()
```

Python ищет `hello` именно в таком порядке:

```text
C
↓
B
↓
A  ← нашёл
↓
object
```

---

## 🔀 Множественное наследование

```python
class A:
    def hello(self):
        print("A")


class B(A):
    pass


class C(A):
    def hello(self):
        print("C")


class D(B, C):
    pass
```

MRO класса `D`:

```python
print(D.mro())
```

Упрощённо:

```text
D → B → C → A → object
```

Поэтому:

```python
D().hello()
```

выведет:

```text
C
```

Потому что `B` не содержит `hello`, а следующий класс в MRO — `C`.

---

## 🧠 Как строится MRO?

В Python используется алгоритм **C3 Linearization**.

Он обеспечивает:

1. сохранение порядка наследования;
2. отсутствие повторного посещения класса;
3. согласованное разрешение методов при множественном наследовании.

Например:

```python
class A:
    pass


class B(A):
    pass


class C(A):
    pass


class D(B, C):
    pass
```

Получаем:

```text
D → B → C → A → object
```

А не:

```text
D → B → A → C → A
```

Потому что `A` не должен встречаться дважды.

---

## 🔑 `super()` и MRO

`super()` использует **MRO**, а не просто обращается к непосредственному родителю.

```python
class A:
    def hello(self):
        print("A")


class B(A):
    def hello(self):
        print("B")
        super().hello()


class C(B):
    def hello(self):
        print("C")
        super().hello()
```

MRO:

```text
C → B → A → object
```

Вызов:

```python
C().hello()
```

даст:

```text
C
B
A
```

То есть:

```text
C.hello()
   ↓ super()
B.hello()
   ↓ super()
A.hello()
```

---

## ⚠️ Важный момент

`super()` означает не:

> «вызови метод моего родителя».

А скорее:

> **«продолжи поиск метода со следующего класса в MRO»**.

Это особенно важно при множественном наследовании.

---

## 🧩 Diamond Problem

Классическая ситуация:

```text
      A
     / \
    B   C
     \ /
      D
```

```python
class A:
    def hello(self):
        print("A")


class B(A):
    pass


class C(A):
    pass


class D(B, C):
    pass
```

MRO:

```text
D → B → C → A → object
```

`A` вызывается только один раз.

Именно MRO + C3 Linearization позволяют Python корректно разрешать такие схемы наследования.

---

## 🔍 Как посмотреть MRO

### Через `mro()`

```python
print(D.mro())
```

### Через `__mro__`

```python
print(D.__mro__)
```

Оба варианта покажут порядок поиска.

---

## 🎤 Суперкоротко

> **MRO** — это порядок поиска методов и атрибутов в классе и его родителях. В Python он особенно важен при множественном наследовании и строится с помощью **C3 Linearization**. `super()` продолжает поиск по этому MRO.
