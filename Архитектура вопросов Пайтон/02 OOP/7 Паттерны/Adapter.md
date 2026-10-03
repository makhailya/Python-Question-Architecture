# Adapter — паттерн проектирования 🔌

## 🎯 Ответ на собеседовании

**Adapter (Адаптер)** — структурный паттерн, который позволяет объектам с **несовместимыми интерфейсами работать вместе**.

Адаптер выступает посредником:

```text id="p7k3m2"
Client
  ↓
нужный интерфейс
  ↓
Adapter
  ↓
другой интерфейс
  ↓
Existing Service
```

Главная идея:

> **Адаптер преобразует интерфейс одного объекта в интерфейс, который ожидает клиент.**

---

## 🎤 Суперкоротко

> Adapter позволяет использовать существующий класс с несовместимым интерфейсом, не изменяя сам этот класс. Он преобразует вызовы клиента в формат, который понимает адаптируемый объект.

---

# 1. Какую проблему решает Adapter?

Представим, наше приложение ожидает:

```python id="q2v8n5"
class PaymentGateway:

    def pay(self, amount: int) -> None:
        ...
```

Мы хотим использовать сторонний сервис:

```python id="m6r1x9"
class StripeAPI:

    def make_payment(self, value: int) -> None:
        ...
```

Методы разные:

```text id="u8k4p2"
Наш код:
pay()

Stripe:
make_payment()
```

Изменять сторонний класс мы не можем или не хотим.

Используем Adapter:

```text id="f5n2r8"
Application
     ↓
PaymentGateway
     ↓
StripeAdapter
     ↓
StripeAPI
```

---

# 2. Простой пример

Сторонний класс:

```python id="c9m4x7"
class StripeAPI:

    def make_payment(self, value: int) -> None:
        print(f"Stripe payment: {value}")
```

Наше приложение ожидает:

```python id="r3k8p1"
class Payment:

    def pay(self, amount: int) -> None:
        raise NotImplementedError
```

Создаём адаптер:

```python id="v6q2m9"
class StripeAdapter(Payment):

    def __init__(self, stripe: StripeAPI):
        self.stripe = stripe

    def pay(self, amount: int) -> None:
        self.stripe.make_payment(amount)
```

Теперь клиент работает с привычным интерфейсом:

```python id="j8w4n3"
stripe = StripeAPI()
payment = StripeAdapter(stripe)

payment.pay(1000)
```

Внутри происходит:

```text id="s2k7p4"
payment.pay(1000)
        ↓
StripeAdapter
        ↓
stripe.make_payment(1000)
```

---

# 3. Что именно делает Adapter?

Адаптер **не меняет исходный объект**.

Он преобразует вызов:

```text id="y5r1k8"
Client
  │
  │ pay(1000)
  ↓
Adapter
  │
  │ make_payment(1000)
  ↓
Adaptee
```

Где:

* **Client** — код, который хочет использовать объект.
* **Target** — интерфейс, который ожидает Client.
* **Adapter** — преобразователь интерфейса.
* **Adaptee** — существующий объект с несовместимым интерфейсом.

---

# 4. Термины паттерна

Классическая структура:

```text id="q8m3v6"
        Client
           │
           ↓
        Target
           ↑
           │
        Adapter
           │
           ↓
        Adaptee
```

### Target

Интерфейс, который нужен клиенту.

```python id="d7x2n9"
class Payment:

    def pay(self, amount: int) -> None:
        ...
```

### Adaptee

Существующий класс с другим интерфейсом.

```python id="h4p8k1"
class StripeAPI:

    def make_payment(self, value: int) -> None:
        ...
```

### Adapter

Соединяет эти интерфейсы.

```python id="w6r2m5"
class StripeAdapter(Payment):

    def __init__(self, stripe: StripeAPI):
        self.stripe = stripe

    def pay(self, amount: int) -> None:
        self.stripe.make_payment(amount)
```

---

# 5. Почему не изменить Adaptee?

Иногда это невозможно.

Например:

```text id="c1x7q4"
StripeAPI
```

может быть:

* сторонней библиотекой;
* внешним SDK;
* legacy-кодом;
* кодом другого отдела;
* классом, который используется ещё в десяти местах.

Изменять его интерфейс рискованно.

Adapter позволяет оставить его без изменений.

```text id="m9k3p7"
Existing code
     ↓
не меняем
     ↓
Adapter
     ↓
новое приложение
```

---

# 6. Adapter в Backend

Очень распространённый сценарий — интеграции с внешними API.

Допустим, приложение работает с единым интерфейсом:

```python id="n5q8r2"
class NotificationProvider:

    def send(self, recipient: str, message: str) -> None:
        raise NotImplementedError
```

У нас есть разные внешние сервисы:

```text id="a7m2v9"
Telegram API
Email API
SMS API
```

Но у каждого собственный интерфейс.

Adapter приводит их к одному:

```text id="p4x8k1"
                    NotificationProvider
                            ↑
               ┌────────────┼────────────┐
               │            │            │
               ↓            ↓            ↓
        TelegramAdapter  EmailAdapter  SMSAdapter
               │            │            │
               ↓            ↓            ↓
          Telegram API   Email API    SMS API
```

Код приложения работает одинаково:

```python id="y3r7m6"
provider.send(
    recipient="user@example.com",
    message="Hello",
)
```

Неважно, какой конкретно провайдер находится внутри.

---

# 7. Adapter и Dependency Injection

Adapter отлично сочетается с **Dependency Injection**.

Например:

```python id="q6n2w8"
class OrderService:

    def __init__(self, payment: Payment):
        self.payment = payment
```

В production:

```python id="m8r4p1"
payment = StripeAdapter(StripeAPI())

service = OrderService(payment)
```

В тесте:

```python id="v2k7x5"
payment = FakePayment()

service = OrderService(payment)
```

Получается:

```text id="h9c3m6"
OrderService
      ↓
   Payment
      ↑
      │
 ┌────┴─────┐
 ↓          ↓
Adapter    Fake
```

Это снижает связанность и упрощает тестирование.

---

# 8. Adapter для legacy-кода

Представим старый класс:

```python id="w4p8n2"
class LegacyUserRepository:

    def get_user_by_id(self, user_id: int):
        ...
```

А новый код ожидает:

```python id="j7m3q9"
class UserRepository:

    def get(self, user_id: int):
        ...
```

Вместо переписывания legacy-кода:

```python id="r5x1k8"
class LegacyUserRepositoryAdapter(UserRepository):

    def __init__(self, repository: LegacyUserRepository):
        self.repository = repository

    def get(self, user_id: int):
        return self.repository.get_user_by_id(user_id)
```

Теперь новый код работает через:

```python id="n2v6m4"
repository.get(user_id)
```

---

# 9. Adapter может преобразовывать не только название метода

Он может преобразовать:

* название методов;
* параметры;
* типы данных;
* структуру объектов;
* формат ответа;
* ошибки.

Например:

```python id="c8q2r7"
class ExternalPaymentAdapter:

    def __init__(self, client):
        self.client = client

    def pay(self, amount: int) -> bool:
        response = self.client.create_charge(
            amount_cents=amount * 100,
        )

        return response.status == "success"
```

Клиент работает:

```python id="x4m9p1"
payment.pay(1000)
```

А внешний API получает:

```text id="v7k3n5"
create_charge(
    amount_cents=100000
)
```

Adapter преобразовал интерфейс и данные.

---

# 10. Object Adapter

В Python наиболее естественный вариант — **объектный адаптер через композицию**.

```python id="k5r8w2"
class Adapter:

    def __init__(self, adaptee):
        self.adaptee = adaptee

    def request(self):
        return self.adaptee.specific_request()
```

Схема:

```text id="n3q7m1"
Adapter
   │
   │ содержит
   ↓
Adaptee
```

Это **composition**.

---

# 11. Class Adapter

Классический вариант некоторых языков использует наследование:

```text id="z8p4c6"
Adapter
   ↓
наследуется от
Adaptee
```

Но в Python чаще используется композиция:

```python id="s1v6x9"
class Adapter:

    def __init__(self, adaptee):
        self.adaptee = adaptee
```

Почему?

Потому что композиция:

* уменьшает связанность;
* позволяет адаптировать объект во время выполнения;
* не требует наследования от конкретного класса.

---

# 12. Adapter vs Decorator

Очень важно не путать.

### Adapter

**Меняет интерфейс.**

```text id="b7m2q9"
Old Interface
      ↓
   Adapter
      ↓
New Interface
```

### Decorator

**Добавляет поведение, сохраняя интерфейс.**

```text id="k4x8p1"
Object
  ↓
Decorator
  ↓
тот же интерфейс
+ дополнительное поведение
```

Например:

```text id="v6n3r5"
Adapter:
pay() → make_payment()

Decorator:
pay() → logging → pay()
```

---

# 13. Adapter vs Facade

Т
