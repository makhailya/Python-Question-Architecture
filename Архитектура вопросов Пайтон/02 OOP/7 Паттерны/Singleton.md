# Singleton — паттерн проектирования 🧩

## 🎯 Ответ на собеседовании

**Singleton (Одиночка)** — порождающий паттерн проектирования, который гарантирует, что у класса существует **только один экземпляр**, и предоставляет глобальную точку доступа к нему.

Идея:

```text
Singleton
    │
    ├── instance
    │
    ├── instance
    │
    └── instance
         ↓
      один объект
```

Все обращения к Singleton получают **один и тот же экземпляр**.

---

## 🎤 Суперкоротко

> Singleton гарантирует наличие единственного экземпляра класса и предоставляет глобальную точку доступа к нему.

В Python Singleton часто реализуют через:

* [[__new__]];
* метакласс;
* модуль;
* [[Dependency Injection]].

При этом в Python полноценный классический Singleton часто вообще не нужен — **модуль уже создаётся один раз на процесс и может выполнять ту же роль**.

---

# 1. Какую проблему решает Singleton?

Допустим, приложение должно иметь только один объект конфигурации:

```python
config1 = Config()
config2 = Config()
```

Обычный класс создаст:

```text
config1 → объект A
config2 → объект B
```

То есть:

```python
config1 is config2
# False
```

Singleton должен обеспечить:

```text
config1 ─┐
         ├──→ один объект
config2 ─┘
```

```python
config1 is config2
# True
```

---

# 2. Основная идея

Обычный класс:

```python
class Config:
    pass


a = Config()
b = Config()

print(a is b)
# False
```

Singleton:

```python
class Singleton:
    _instance = None

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)

        return cls._instance
```

Использование:

```python
a = Singleton()
b = Singleton()

print(a is b)
# True
```

---

# 3. Как работает `__new__`?

Важно понимать разницу:

```text
__new__
   ↓
создаёт экземпляр

__init__
   ↓
инициализирует экземпляр
```

Singleton контролирует создание объекта через `__new__`.

```python
class Singleton:
    _instance = None

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)

        return cls._instance
```

Первый вызов:

```python
a = Singleton()
```

Происходит:

```text
_instance == None
        ↓
создать объект
        ↓
сохранить его
        ↓
вернуть объект
```

Второй:

```python
b = Singleton()
```

Происходит:

```text
_instance != None
        ↓
вернуть существующий объект
```

---

# 4. Проверка идентичности

```python
a = Singleton()
b = Singleton()
c = Singleton()

print(a is b)
print(b is c)
print(a is c)
```

Результат:

```text
True
True
True
```

Все переменные указывают на один экземпляр.

---

# 5. Singleton и `__init__`

Есть важный нюанс.

`__new__` возвращает существующий объект, но `__init__` при каждом вызове конструктора всё равно может выполняться.

Например:

```python
class Singleton:
    _instance = None

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)

        return cls._instance

    def __init__(self):
        print("init")
```

```python
a = Singleton()
b = Singleton()
```

Может вывести:

```text
init
init
```

Хотя объект один:

```python
a is b
# True
```

Поэтому если инициализацию нужно выполнить только один раз, это необходимо учитывать отдельно.

---

# 6. Пример с одноразовой инициализацией

```python
class Singleton:
    _instance = None

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
            cls._instance._initialized = False

        return cls._instance

    def __init__(self):
        if self._initialized:
            return

        self.value = 42
        self._initialized = True
```

Теперь:

```python
a = Singleton()
b = Singleton()

print(a is b)
# True
```

Инициализация выполняется только один раз.

---

# 7. Singleton через модуль — Python-подход

В Python часто нет необходимости создавать специальный Singleton-класс.

Например:

```python
# config.py

DATABASE_URL = "postgresql://localhost/db"
```

Другой файл:

```python
from config import DATABASE_URL
```

Модуль импортируется и кэшируется в рамках процесса Python.

Поэтому модуль часто используется как естественный способ предоставить единое состояние.

---

# 8. Singleton через функцию

Можно использовать кэширование:

```python
from functools import cache


@cache
def get_config():
    return Config()
```

Теперь:

```python
a = get_config()
b = get_config()

print(a is b)
# True
```

Функция возвращает закэшированный объект.

Это уже не классический GoF Singleton, но решает похожую задачу.

---

# 9. Где Singleton может использоваться?

Классические примеры:

```text
Configuration
Logger
Registry
Cache
Connection manager
```

Например:

```text
Application
   │
   ├── Service A ──┐
   ├── Service B ──┼──→ Logger
   └── Service C ──┘
```

Все компоненты используют один объект.

---

# 10. Singleton и Logger

Например, приложение хочет иметь единый объект логирования:

```python
logger = Logger()
```

Разные части приложения обращаются к одному экземпляру:

```text
Service A ──┐
Service B ──┼──→ Logger
Service C ──┘
```

Это гарантирует единое состояние логгера.

Однако современные библиотеки логирования сами предоставляют механизмы управления logger instances, поэтому вручную писать Singleton часто не требуется.

---

# 11. Главный недостаток Singleton

Singleton создаёт **глобальное состояние**.

Например:

```python
Singleton.config = "production"
```

Любая часть приложения может изменить его:

```python
Singleton.config = "test"
```

И это изменение увидят другие компоненты.

Получается скрытая зависимость:

```text
Service A
   ↓
Singleton
   ↑
Service B
```

Service A и Service B косвенно зависят от общего состояния.

---

# 12. Singleton нарушает принцип Dependency Injection

Вместо:

```python
class Service:
    def __init__(self, config):
        self.config = config
```

можно сделать:

```python
class Service:
    def __init__(self):
        self.config = Singleton()
```

Но второй вариант создаёт **скрытую зависимость**.

Лучше:

```python
config = Config()

service = Service(config)
```

Зависимость явно передаётся:

```text
Config
  ↓
Service
```

Это делает код:

* проще тестировать;
* проще заменять зависимости;
* проще понимать;
* проще переиспользовать.

---

# 13. Singleton и тестирование

Singleton может усложнить тесты.

Например:

```text
Test A
 ↓
Singleton.state = "A"

Test B
 ↓
Singleton.state
 ↓
"A"
```

Состояние одного теста может повлиять на другой.

Поэтому Singleton с изменяемым состоянием часто считается плохим решением для тестируемого приложения.

Dependency Injection обычно проще:

```python
class Service:
    def __init__(self, repository):
        self.repository = repository
```

В production:

```python
service = Service(PostgresRepository())
```

В тесте:

```python
service = Service(FakeRepository())
```

---

# 14. Singleton и многопоточность

Наивная реализация:

```python
class Singleton:
    _instance = None

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)

        return cls._instance
```

может быть проблемной при конкурентном доступе.

Теоретически:

```text
Thread A                 Thread B

check instance           check instance
instance == None         instance == None
create object            create object
```

Можно получить более одного объекта без дополнительной синхронизации.

Поэтому в многопоточной среде реализация Singleton требует отдельного внимания к race condition.

---

# 15. Singleton и процессы

Особенно важно для Python backend.

Singleton обычно гарантирует один экземпляр **в пределах одного процесса**.

Например:

```text
Gunicorn
│
├── Worker 1
│    └── Singleton A
│
├── Worker 2
│    └── Singleton B
│
└── Worker 3
     └── Singleton C
```

Получается:

```text
1 Singleton
на каждый процесс
```

а не обязательно один Singleton на всё приложение.

Это очень важный момент на Backend-собеседовании.

---

# 16. Singleton ≠ глобальный объект во всей системе

Singleton:

```text
Process 1 → Object A
```

не означает:

```text
весь сервер
   ↓
один объект
```

При нескольких процессах:

```text
Process 1 → A
Process 2 → B
Process 3 → C
```

Если нужен общий state между процессами, обычно используются внешние системы:

```text
Redis
PostgreSQL
Kafka
```

или другие механизмы межпроцессного взаимодействия.

---

# 17. Singleton vs Dependency Injection

| Singleton                                                 | Dependency Injection           |
| --------------------------------------------------------- | ------------------------------ |
| Глобальная точка доступа                                  | Явная передача зависимости     |
| Скрытая зависимость                                       | Явная зависимость              |
| Сложнее тестировать                                       | Проще тестировать              |
| Общее состояние                                           | Состояние можно контролировать |
| Может быть удобно                                         | Обычно лучше для архитектуры   |
| Часто считается anti-pattern при чрезмерном использовании | Хорошо сочетается с SOLID      |

---

# 18. Singleton vs Module

В Python:

```text
Singleton class
      ↓
явно ограничиваем количество экземпляров
```

Модуль:

```text
module.py
      ↓
импортируется и кэшируется
```

Поэтому для многих задач Python-подход проще:

```python
# config.py

settings = {
    "debug": False,
}
```

И использовать:

```python
from config import settings
```

Не нужно искусственно создавать Singleton-класс.

---

# 19. Singleton в контексте GoF

Singleton относится к **порождающим паттернам**.

```text
GoF
│
├── Порождающие
│   ├── Singleton
│   ├── Factory Method
│   ├── Abstract Factory
│   ├── Builder
│   └── Prototype
│
├── Структурные
│
└── Поведенческие
```

Singleton отвечает на вопрос
