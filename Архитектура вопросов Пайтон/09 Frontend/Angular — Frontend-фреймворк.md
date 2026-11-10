# 🅰️ Angular — Frontend-фреймворк

## 🎤 Короткий ответ для собеседования

**Angular** — это полноценный Frontend-фреймворк с открытым исходным кодом, разработанный Google.

Используется для создания сложных веб-приложений, особенно крупных корпоративных систем.

Основные особенности:

- **TypeScript** — основной язык разработки.
- **Components** — компонентная архитектура.
- **Templates** — HTML-шаблоны интерфейса.
- **Dependency Injection (DI)** — внедрение зависимостей.
- **Services** — организация бизнес-логики и работы с API.
- **Routing** — навигация между страницами.
- **RxJS и Signals** — инструменты реактивного программирования.

## 🗣️ Ответ на собеседовании

Angular — это полноценный JavaScript-фреймворк для разработки клиентской части веб-приложений.

В отличие от React, который является библиотекой интерфейсов, Angular предоставляет готовую архитектуру и набор инструментов: маршрутизацию, HTTP-клиент, внедрение зависимостей и работу с формами.

Приложение строится из компонентов и сервисов. Компоненты отвечают за интерфейс, а сервисы могут содержать логику взаимодействия с Backend.

С Python Backend, например FastAPI или Django, Angular взаимодействует через REST API, отправляя HTTP-запросы и получая JSON.

---

## ⚙️ Как Angular взаимодействует с Backend

```text
       Пользователь
            |
            v
     Angular Frontend
            |
            | HTTP / JSON
            v
     FastAPI / Django
            |
            v
        PostgreSQL
            |
            v
       JSON Response
            |
            v
   Angular обновляет UI
```

## 🧩 Основные компоненты Angular

| Компонент | Назначение |
|---|---|
| Component | Часть пользовательского интерфейса |
| Template | HTML-разметка компонента |
| Service | Переиспользуемая логика, работа с API |
| Dependency Injection | Передача зависимостей компонентам и сервисам |
| Router | Навигация между страницами |
| HttpClient | Выполнение HTTP-запросов |
| Signals | Реактивное управление состоянием |
| RxJS | Работа с асинхронными потоками данных |

## 🔧 Пример Angular-компонента

```typescript
import { Component, signal } from '@angular/core';

@Component({
  selector: 'app-counter',
  standalone: true,
  template: `
    <button (click)="increment()">
      Нажали: {{ count() }}
    </button>
  `
})
export class CounterComponent {
  count = signal(0);

  increment() {
    this.count.update(value => value + 1);
  }
}
```

### Разбор

- `@Component` — декоратор, объявляющий Angular-компонент.
- `selector` — имя HTML-элемента компонента.
- `template` — разметка интерфейса.
- `signal(0)` — создаёт реактивное состояние.
- `count()` — получает текущее значение.
- `(click)` — обработчик нажатия.
- `count.update()` — обновляет состояние.

При изменении сигнала Angular обновляет соответствующее отображение.

## 💉 Dependency Injection (DI)

**Dependency Injection** — механизм передачи объекту необходимых зависимостей извне.

Например, компоненту нужен сервис для получения пользователей.

```typescript
import { Component, inject } from '@angular/core';
import { UserService } from './user.service';

@Component({
  selector: 'app-users',
  standalone: true,
  template: `<h1>Пользователи</h1>`
})
export class UsersComponent {
  private userService = inject(UserService);
}
```

Angular предоставляет экземпляр `UserService` через систему внедрения зависимостей.

**Аналогия с FastAPI:**

```python
from fastapi import Depends

def get_users(service = Depends(get_user_service)):
    return service.get_users()
```

В обоих случаях зависимости предоставляются специальным механизмом, а не создаются вручную внутри основной логики.

## 🌐 Angular и REST API

Для HTTP-запросов Angular предоставляет `HttpClient`.

Пример сервиса:

```typescript
import { Injectable, inject } from '@angular/core';
import { HttpClient } from '@angular/common/http';

@Injectable({
  providedIn: 'root'
})
export class UserService {
  private http = inject(HttpClient);

  getUsers() {
    return this.http.get('/api/users');
  }
}
```

### Что происходит?

1. `@Injectable` — объявляет класс доступным для DI.
2. `providedIn: 'root'` — регистрирует сервис на уровне корневого инжектора.
3. `inject(HttpClient)` — получает HTTP-клиент.
4. `http.get()` — создаёт HTTP-запрос.
5. Метод возвращает `Observable`, на который можно подписаться.

Для работы примера `HttpClient` должен быть зарегистрирован в конфигурации приложения, например через `provideHttpClient()`.

## ⚖️ Angular vs React vs Vue

| Критерий | Angular | React | Vue |
|---|---|---|---|
| Тип | Фреймворк | Библиотека | Фреймворк |
| Основной язык | TypeScript | JS / TS | JS / TS |
| Компоненты | Да | Да | Да |
| Маршрутизация | Встроенный пакет | Внешние решения | Vue Router |
| Dependency Injection | Встроенная система | Нет аналогичной обязательной системы | `provide` / `inject` |
| Частое применение | Крупные приложения | Разные веб-приложения | Разные веб-приложения |

**Главное отличие:** Angular предлагает более комплексную архитектуру из коробки, тогда как React оставляет больше решений на выбор разработчика.

---

## ❓ Частые вопросы на собеседовании

**1. Что такое Angular?**

Frontend-фреймворк для разработки веб-приложений, преимущественно на TypeScript.

**2. Чем Angular отличается от React?**

Angular — полноценный фреймворк с маршрутизацией, DI и HTTP-клиентом. React — библиотека для построения интерфейсов.

**3. Что такое Component?**

Независимая часть интерфейса со своей логикой и шаблоном.

**4. Что такое Service?**

Класс с переиспользуемой логикой, например для обращения к Backend API.

**5. Что такое Dependency Injection?**

Механизм предоставления объекту необходимых зависимостей.

**6. Что такое RxJS?**

Библиотека реактивного программирования для работы с потоками асинхронных событий через Observable.

**7. Что такое Signals?**

Реактивные значения, изменение которых Angular может отслеживать для обновления интерфейса.

**8. Как Angular взаимодействует с FastAPI?**

Через HTTP-запросы с помощью `HttpClient`, обычно используя JSON.

**9. Что такое Angular Router?**

Механизм навигации между представлениями приложения без обязательной полной перезагрузки страницы.

**10. Нужен ли Angular Python Backend-разработчику?**

Глубокое знание необязательно, но важно понимать взаимодействие с REST API, CORS, авторизацию, HTTP-статусы и формат JSON.

---

## 🧠 Запомнить

```text
Angular
 ├── Components → Интерфейс
 ├── Templates  → HTML-разметка
 ├── Services   → Переиспользуемая логика
 ├── DI         → Внедрение зависимостей
 ├── HttpClient → Работа с Backend
 ├── Router     → Навигация
 ├── Signals    → Реактивное состояние
 └── RxJS       → Асинхронные потоки
```

**Главная формула для собеседования:**

Angular — полноценный TypeScript-фреймворк для создания Frontend-приложений. Использует компонентную архитектуру, Dependency Injection, сервисы и реактивное программирование, а с Backend взаимодействует через HTTP API.