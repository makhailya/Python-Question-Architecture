# 🅞 Open/Closed Principle — принцип открытости/закрытости

## 🎯 Ответ на собеседовании

**Open/Closed Principle (OCP)** — принцип открытости/закрытости, согласно которому программные сущности должны быть **открыты для расширения, но закрыты для изменения**.

Это означает, что при добавлении новой функциональности желательно **расширять существующий код**, не изменяя уже работающий код.

Например, если у нас появляются новые способы оплаты, лучше добавить новую реализацию общего интерфейса, чем постоянно изменять один класс с большим количеством `if/elif`.

---

## 🎤 Суперкоротко

```text
OCP
 ↓
Открыт для расширения
Закрыт для изменения
```

То есть:

```text
❌ Менять существующий код
✅ Добавлять новую реализацию
```

---

# 🧩 Что означает «открыт для расширения»

Систему можно расширять новой функциональностью.

Например, у нас есть:

```text
CardPayment
CashPayment
```

И появляется:

```text
CryptoPayment
```

Мы хотим добавить новый класс, а не переписывать существующую бизнес-логику.

---

# 🧩 Что означает «закрыт для изменения»

Уже работающий код желательно не изменять при добавлении нового поведения.

Например, если есть:

```python
class PaymentService:
    def pay(self, payment_type, amount):
        if payment_type == "card":
            pass
        elif payment_type == "cash":
            pass
```

Добавляется новый способ:

```text
crypto
```

И нам приходится изменять `PaymentService`.

Это признак потенциального нарушения OCP.

---

# ❌ Пример нарушения OCP

```python
class PaymentService:
    def pay(self, payment_type, amount):
        if payment_type == "card":
            print(f"Card payment: {amount}")

        elif payment_type == "cash":
            print(f"Cash payment: {amount}")

        elif payment_type == "crypto":
            print(f"Crypto payment: {amount}")
```

Добавим новый способ:

```text
Bank Transfer
```

Нужно изменить существующий класс:

```python
class PaymentService:
    def pay(self, payment_type, amount):
        if payment_type == "card":
            pass
        elif payment_type == "cash":
            pass
        elif payment_type == "crypto":
            pass
        elif payment_type == "bank_transfer":
            pass
```

Каждое новое поведение приводит к изменению старого кода.

---

# ✅ Пример с соблюдением OCP

Используем абстракцию:

```python
from abc import ABC, abstractmethod


class PaymentMethod(ABC):

    @abstractmethod
    def pay(self, amount):
        pass
```

Теперь создаём реализации:

```python
class CardPayment(PaymentMethod):

    def pay(self, amount):
        print(f"Card payment: {amount}")


class CashPayment(PaymentMethod):

    def pay(self, amount):
        print(f"Cash payment: {amount}")


class CryptoPayment(PaymentMethod):

    def pay(self, amount):
        print(f"Crypto payment: {amount}")
```

Если появляется новый способ оплаты:

```python
class BankTransferPayment(PaymentMethod):

    def pay(self, amount):
        print(f"Bank transfer: {amount}")
```

Существующие классы менять не пришлось.

```text
PaymentMethod
      │
      ├── CardPayment
      ├── CashPayment
      ├── CryptoPayment
      └── BankTransferPayment
```

---

# 🧠 Где здесь OCP?

Мы:

```text
добавили новую функциональность
        ↓
создали новый класс
        ↓
старый код не изменяли
```

То есть система:

```text
открыта для расширения
        +
закрыта для изменения
```

---

# 🐍 Полиморфизм помогает соблюдать OCP

Можно работать с абстракцией:

```python
def process_payment(
    payment_method: PaymentMethod,
    amount: float,
):
    payment_method.pay(amount)
```

Теперь функция не знает конкретный тип платежа.

Она работает с интерфейсом:

```text
PaymentMethod
     ↓
     pay()
```

Можно передать:

```python
process_payment(CardPayment(), 1000)
process_payment(CashPayment(), 1000)
process_payment(CryptoPayment(), 1000)
```

И добавить:

```python
process_payment(BankTransferPayment(), 1000)
```

без изменения `process_payment`.

---

# 🔗 OCP и полиморфизм

Обычно OCP хорошо реализуется через:

```text
Абстракция
    ↓
Полиморфизм
    ↓
Разные реализации
```

Например:

```text
PaymentMethod
      ↓
      ├── Card
      ├── Cash
      ├── Crypto
      └── BankTransfer
```

Код работает с `PaymentMethod`, а не с каждым конкретным способом оплаты.

---

# ⚠️ OCP не означает «никогда ничего не изменять»

Это важный нюанс.

OCP не говорит:

> «Существующий код вообще никогда нельзя менять».

Принцип означает:

> **При расширении функциональности стараться не изменять стабильный существующий код, если архитектура позволяет этого избежать.**

Если требования изменились настолько, что сама абстракция стала неправильной, её иногда необходимо изменить.

---

# 🧩 OCP и `if/elif`

Большое количество условий по типам часто является сигналом возможного нарушения OCP:

```python
if payment_type == "card":
    ...
elif payment_type == "cash":
    ...
elif payment_type == "crypto":
    ...
elif payment_type == "bank":
    ...
```

Но важно:

**`if/elif` сам по себе не означает нарушение OCP.**

В маленьком и стабильном коде такое решение может быть вполне нормальным.

Проблема появляется, когда каждый новый тип требует постоянного изменения большого количества существующего кода.

---

# 🆚 SRP и OCP

Их легко перепутать.

### SRP

Спрашивает:

> **«Сколько ответственностей у класса?»**

```text
SRP
 ↓
Одна ответственность
```

### OCP

Спрашивает:

> **«Как добавить новую функциональность?»**

```text
OCP
 ↓
Расширить существующий код,
не изменяя стабильную часть системы
```

---

# 🧠 Пример из backend

Представим уведомления:

```text
Email
SMS
Telegram
Push
```

Плохой вариант:

```python
class NotificationService:

    def send(self, notification_type, message):
        if notification_type == "email":
            pass
        elif notification_type == "sms":
            pass
        elif notification_type == "telegram":
            pass
```

Добавление Push требует изменения класса.

Лучше:

```python
from abc import ABC, abstractmethod


class NotificationSender(ABC):

    @abstractmethod
    def send(self, message):
        pass


class EmailSender(NotificationSender):

    def send(self, message):
        print(f"Email: {message}")


class SmsSender(NotificationSender):

    def send(self, message):
        print(f"SMS: {message}")


class TelegramSender(NotificationSender):

    def send(self, message):
        print(f"Telegram: {message}")
```

Добавляем:

```python
class PushSender(NotificationSender):

    def send(self, message):
        print(f"Push: {message}")
```

Основная логика при этом не меняется.

---

# 🎯 Как ответить, если попросят пример

> **Например, есть сервис оплаты, который содержит `if/elif` для каждого способа оплаты. При добавлении нового способа приходится изменять этот класс. Это может нарушать OCP. Я бы выделил общий интерфейс `PaymentMethod` и сделал отдельные реализации для каждого способа оплаты. Тогда новый способ можно добавить через новый класс, не изменяя существующую бизнес-логику.**

---

# 🧠 Главное

```text
OCP
 │
 ├── Open
 │     ↓
 │   расширяем
 │
 └── Closed
       ↓
     не изменяем стабильный код
```

Практически:

```text
❌ Добавили новую функциональность
   → переписали старый класс

✅ Добавили новую функциональность
   → создали новую реализацию
```

OCP часто достигается с помощью:

```text
абстракций
интерфейсов
полиморфизма
наследования
композиции
dependency injection
```

---

# 🧠 Формула для запоминания

**OCP = открыт для расширения, закрыт для изменения.**

> **Новое поведение добавляем через расширение существующей системы, а не через постоянное изменение уже работающего кода.**


# Связанные темы:

[[SOLID|SOLID]]