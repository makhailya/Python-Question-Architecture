# Factory Method — Фабричный метод 🏭

## 🎯 Ответ на собеседовании

**Factory Method** — порождающий паттерн, который определяет интерфейс для создания объекта, но позволяет подклассам решать, **какой конкретно объект создать**.

Главная идея:

> Код работает с абстракцией и не обязан напрямую создавать конкретный класс через `ClassName()`.

Вместо:

```python id="x8j3a4"
if notification_type == "email":
    notification = EmailNotification()
elif notification_type == "sms":
    notification = SMSNotification()
```

создание объекта выносится в отдельный метод:

```python id="r8n5k2"
notification = creator.create_notification()
```

---

## 🎤 Суперкоротко

> Factory Method выносит создание объектов в специальный метод и позволяет подклассам определять, какой конкретный объект будет создан. Клиент работает с абстракцией и меньше зависит от конкретных классов.

---

# 1. Какую проблему решает Factory?

Представим приложение отправки уведомлений:

```text id="k2s7q1"
EmailNotification
SMSNotification
PushNotification
```

Без фабрики:

```python id="v9x1fz"
if notification_type == "email":
    notification = EmailNotification()
elif notification_type == "sms":
    notification = SMSNotification()
elif notification_type == "push":
    notification = PushNotification()
```

Проблема:

* код знает все конкретные классы;
* при добавлении нового типа нужно менять условие;
* логика создания смешивается с бизнес-логикой.

Factory Method выносит создание объекта.

---

# 2. Основная идея

```text id="u4n9c3"
          Creator
             │
      create_product()
             │
      ┌──────┴──────┐
      ↓             ↓
ConcreteCreator A  ConcreteCreator B
      │             │
      ↓             ↓
 Product A       Product B
```

Клиент знает:

```text id="f6j1p8"
Product
```

но не обязан знать:

```text id="2c5k7r"
ProductA
ProductB
```

---

# 3. Простой пример

Создадим общий интерфейс:

```python id="e2f8m1"
from abc import ABC, abstractmethod


class Notification(ABC):

    @abstractmethod
    def send(self, message: str) -> None:
        pass
```

Конкретные продукты:

```python id="r4k7v2"
class EmailNotification(Notification):

    def send(self, message: str) -> None:
        print(f"Email: {message}")


class SMSNotification(Notification):

    def send(self, message: str) -> None:
        print(f"SMS: {message}")
```

---

# 4. Creator

Создаём базовый класс:

```python id="n8q3w6"
class NotificationCreator(ABC):

    @abstractmethod
    def create_notification(self) -> Notification:
        pass

    def notify(self, message: str) -> None:
        notification = self.create_notification()
        notification.send(message)
```

Обрати внимание:

```python id="a7d2m9"
notify()
```

не знает, какой именно класс будет создан.

Он просто получает:

```text id="g3k8p1"
Notification
```

---

# 5. Concrete Creator

Для Email:

```python id="q5v2n8"
class EmailNotificationCreator(NotificationCreator):

    def create_notification(self) -> Notification:
        return EmailNotification()
```

Для SMS:

```python id="m9c4x7"
class SMSNotificationCreator(NotificationCreator):

    def create_notification(self) -> Notification:
        return SMSNotification()
```

Теперь:

```python id="z6p3k1"
email_creator = EmailNotificationCreator()
email_creator.notify("Привет")
```

Создастся:

```text id="w2n8r5"
EmailNotification
```

А:

```python id="h4q7s2"
sms_creator = SMSNotificationCreator()
sms_creator.notify("Привет")
```

создаст:

```text id="c8m1v6"
SMSNotification
```

---

# 6. Где здесь Factory Method?

Вот этот метод:

```python id="y7p2k9"
def create_notification(self) -> Notification:
    ...
```

Это и есть **Factory Method**.

Базовый класс определяет:

> «У меня есть способ создать Notification».

А конкретный creator решает:

> «Какой именно Notification создать».

```text id="e1r5t8"
NotificationCreator
        │
        └── create_notification()
                    ↑
          переопределяется
                    │
       ┌────────────┴────────────┐
       ↓                         ↓
EmailCreator               SMSCreator
       ↓                         ↓
EmailNotification          SMSNotification
```

---

# 7. Почему это лучше прямого создания?

Без Factory:

```python id="s5k2q9"
class OrderService:

    def send_notification(self):
        notification = EmailNotification()
        notification.send(...)
```

`OrderService` напрямую зависит от:

```text id="n1r6w3"
EmailNotification
```

С Factory Method:

```python id="g8v4p2"
class OrderService:

    def send_notification(self):
        notification = self.create_notification()
        notification.send(...)
```

Теперь создание объекта отделено от основной логики.

---

# 8. Factory Method и Dependency Inversion

Factory Method хорошо сочетается с принципом:

**D — Dependency Inversion Principle.**

Вместо зависимости от конкретного класса:

```text id="j3q7m1"
Service
   ↓
EmailNotification
```

получаем:

```text id="x8r2k5"
Service
   ↓
Notification
   ↑
EmailNotification
```

Код работает с абстракцией.

---

# 9. Factory Method vs обычная Factory Function

Это важное различие.

### Factory Function

Обычная функция, которая создаёт объект:

```python id="p4m8s1"
def create_notification(notification_type: str) -> Notification:
    if notification_type == "email":
        return EmailNotification()

    if notification_type == "sms":
        return SMSNotification()

    raise ValueError("Unknown type")
```

Использование:

```python id="q7v3n9"
notification = create_notification("email")
```

Это часто называют **Simple Factory** или **Factory Function**.

---

### Factory Method

Создание объекта происходит через метод, который может быть переопределён в подклассе:

```python id="w6k2r8"
class Creator(ABC):

    @abstractmethod
    def create_product(self):
        pass
```

И:

```python id="m3p9x4"
class ConcreteCreator(Creator):

    def create_product(self):
        return ConcreteProduct()
```

### Главное отличие

```text id="z1f7c5"
Simple Factory
    ↓
одна функция выбирает класс

Factory Method
    ↓
подклассы определяют создание
```

---

# 10. Factory Method vs Abstract Factory

Их часто путают.

### Factory Method

Создаёт **один тип продукта**:

```text id="h4n8q2"
Creator
   ↓
create_button()
   ↓
Button
```

### Abstract Factory

Создаёт **семейство связанных продуктов**:

```text id="c7m2v9"
AbstractFactory
   ├── create_button()
   ├── create_checkbox()
   └── create_input()
```

Например:

```text id="p5x8k3"
WindowsFactory
 ├── WindowsButton
 ├── WindowsCheckbox
 └── WindowsInput

MacFactory
 ├── MacButton
 ├── MacCheckbox
 └── MacInput
```

То есть:

```text id="q9r1w6"
Factory Method
→ один продукт

Abstract Factory
→ семейство продуктов
```

---

# 11. Factory Method и Strategy — не одно и то же

Эти паттерны тоже часто путают.

### Factory

Отвечает:

> **Какой объект создать?**

```text id="a8m4p2"
Factory
   ↓
EmailNotification
```

### Strategy

Отвечает:

> **Какую стратегию поведения использовать?**

```text id="v6k1s9"
Service
   ↓
Strategy
 ┌─┴─────────┐
 ↓           ↓
Fast       Accurate
```

Упрощённо:

```text id="k3r7x1"
Factory  → создание
Strategy → поведение
```

---

# 12. Factory Method в Python

Python позволяет сделать фабрику значительно проще.

Например:

```python id="b5n8q4"
class NotificationFactory:

    @staticmethod
    def create(notification_type: str) -> Notification:
        if notification_type == "email":
            return EmailNotification()

        if notification_type == "sms":
            return SMSNotification()

        raise ValueError("Unknown notification type")
```

Использование:

```python id="t2v7m9"
notification = NotificationFactory.create("email")
notification.send("Привет")
```

Это уже скорее **Simple Factory**, а не классический GoF Factory Method.

---

# 13. Factory через словарь

В Python можно избежать большого `if/elif`:

```python id="n6q3w8"
FACTORIES = {
    "email": EmailNotification,
    "sms": SMSNotification,
    "push": PushNotification,
}


def create_notification(notification_type: str) -> Notification:
    try:
        notification_class = FACTORIES[notification_type]
    except KeyError:
        raise ValueError("Unknown notification type")

    return notification_class()
```

Использование:

```python id="r9m4k2"
notification = create_notification("email")
```

Такой подход часто проще классического GoF-паттерна.

---

# 14. Factory в реальном Backend

Представим разные способы оплаты:

```text id="y7c2m8"
Payment
├── CardPayment
├── SBPPayment
├── CryptoPayment
└── PayPalPayment
```

Можно использовать фабрику:

```python id="u5n9r3"
class PaymentFactory:

    @staticmethod
    def create(payment_type: str):
        if payment_type == "card":
            return CardPayment()

        if payment_type == "sbp":
            return SBPPayment()

        if payment_type == "crypto":
            return CryptoPayment()

        raise ValueError("Unsupported payment type")
```

Сервис:

```python id="f8q1k6"
payment = PaymentFactory.create("card")
payment.process()
```

Сервису не нужно самостоятельно создавать:

```text id="p2m7v4"
CardPayment
SBPPayment
CryptoPayment
```

---

# 15. Factory и FastAPI

В backend фабрика может использоваться для выбора реализации зависимости.

Например:

```python id="c6x9w2"
def create_payment_service(payment_type: str):
    if payment_type == "card":
        return CardPaymentService()

    if payment_type == "sbp":
        return SBPPaymentService()

    raise ValueError("Unsupported payment type")
```

Затем конкретный сервис используется в обработчике.

Идея:

```text id="m4r8q1"
HTTP Request
     ↓
payment_type
     ↓
Factory
     ↓
конкретный Service
```

Но в FastAPI для многих подобных задач естественнее использовать встроенный механизм **Dependency Injection**, а фабрику — там, где действительно есть логика выбора реализации.

---

# 16. Factory и тестирование

Фабрика позволяет централизовать создание объектов.

Например:

```text id="s7k3p9"
Production Factory
       ↓
PostgresRepository

Test Factory
       ↓
FakeRepository
```

Это удобно для тестирования.

Но часто ещё проще передать зависимость напрямую через DI:

```python id="e2v6m8"
service = UserService(repository=fake_repository)
```

Поэтому Factory и DI могут использоваться вместе.

---

# 17. Преимущества Factory Method

### ✅ Скрывает создание объектов

Клиент не обязан знать детали создания.

### ✅ Снижает связанность

Код работает с абстракциями.

### ✅ Упрощает расширение

Можно добавить новый тип продукта через новую реализацию creator.

### ✅ Централизует создание

Логика создания находится в одном месте.

### ✅ Хорошо сочетается с SOLID

Особенно:

```text id="h5q8m2"
Dependency Inversion
Open/Closed
```

---

# 18. Недостатки

### ❌ Дополнительные классы

Простой код может превратиться в:

```text id="q3m7x9"
Creator
ConcreteCreatorA
ConcreteCreatorB
Product
ConcreteProductA
ConcreteProductB
```

Для небольшой задачи это избыточно.

### ❌ Усложнение архитектуры

Если обычной функции достаточно:

```python id="w8n2c5"
create_user()
```

не обязательно создавать несколько фабричных классов.

### ❌ Не всегда нужен GoF-паттерн

В Python часто достаточно:

* функции;
* словаря;
* `match`;
* DI;
* регистрации классов.

---

# 19. Factory Method vs прямой `Class()`

### Прямое создание

```python id="m1q7v4"
payment = CardPayment()
```

Код напрямую зависит от `CardPayment`.

### Factory

```python id="p8k3n6"
payment = PaymentFactory.create("card")
```

Создание скрыто внутри фабрики.

```text id="z4r9w2"
Client
  ↓
Factory
  ↓
CardPayment
```

---

# 20. Типичный вопрос на собеседовании

### ❓ Что такое Factory Method?

> Factory Method — порождающий паттерн, который определяет интерфейс создания объекта, позволяя подклассам выбирать конкретный тип создаваемого объекта.

### ❓ Зачем нужна фабрика?

> Чтобы отделить логику создания объектов от клиентского кода и уменьшить зависимость от конкретных реализаций.

### ❓ Factory Method и Simple Factory — одно и то же?

> Нет. Simple Factory обычно представляет собой функцию или отдельный объект, который выбирает конкретный класс. Factory Method предполагает метод создания, который переопределяется в подклассах.

### ❓ Чем Factory отличается от Abstract Factory?

> Factory Method обычно отвечает за создание одного типа продукта, а Abstract Factory — семейства связанных продуктов.

### ❓ Нужна ли классическая Factory в Python?

> Не всегда. Благодаря динамической типизации, функциям первого класса, словарям и Dependency Injection фабрику часто можно реализовать гораздо проще.

---

# 21. Схема Factory Method

```text id="x6p2m8"
                 Client
                    │
                    ▼
                Creator
                    │
             create_product()
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
 ConcreteCreator A      ConcreteCreator B
          │                   │
          ▼                   ▼
      Product A            Product B
```

Клиент работает через:

```text id="q1v7r4"
Product
```

а не напрямую через:

```text id="m8k3x6"
Product A
Product B
```

---

# 22. Главное

```text id="n4q8w2"
Factory Method
      ↓
отделяет создание
от использования объекта
      ↓
Creator
      ↓
create_product()
      ↓
Concrete Product
```

Запомнить:

```text id="r7m2k5"
Factory → создание объекта
Strategy → выбор поведения
Singleton → один экземпляр
Abstract Factory → семейство объектов
```

И важный Python-момент:

> **Не нужно использовать классический GoF Factory Method только потому, что он существует. В Python часто более простое решение — функция-фабрика, словарь зарегистрированных классов или Dependency Injection.**
