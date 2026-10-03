# 🧩 Pod в Kubernetes

## 🎤 Короткий ответ

**Pod** — это минимальная единица развертывания и запуска приложения в Kubernetes. Pod содержит **один или несколько контейнеров**, которые работают вместе и разделяют сетевое пространство и подключённые volumes.

На практике чаще всего:

> **1 Pod → 1 основной контейнер приложения.**

## 🎯 Формула для собеседования

> **Pod = контейнер(ы) + общая сеть + общие volumes + единый lifecycle.**

```text id="7m4qkc"
Kubernetes
    ↓
   Pod
    ├── Container
    ├── Network
    └── Volume
```

---

# 🔹 Что такое Pod

Kubernetes не работает с контейнером как с основной единицей развертывания.

Его основная единица:

```text id="n8v3qx"
Pod
```

Внутри Pod находятся контейнеры:

```text id="p5m7kc"
Pod
 ├── Container 1
 └── Container 2
```

Но наиболее распространённая модель:

```text id="q4x8mn"
Pod
 └── FastAPI Container
```

---

# 🔹 Зачем нужен Pod

Pod объединяет один или несколько контейнеров, которым необходимо работать **как единое целое**.

Например:

```text id="m6q3vx"
Pod
 ├── FastAPI
 └── Logging sidecar
```

Эти контейнеры:

* запускаются на одной Node;
* имеют общий сетевой namespace;
* могут обращаться друг к другу через `localhost`;
* могут использовать общие volumes;
* имеют общий lifecycle Pod.

---

# 🔹 Pod и Container

Очень важно не путать:

```text id="x7m4qp"
Pod
  ↓
Container
```

Pod **не является контейнером**.

Pod — это Kubernetes-абстракция, внутри которой запускается один или несколько контейнеров.

Можно представить:

```text id="k3n8vc"
Kubernetes
    ↓
   Pod
    ↓
Container Runtime
    ↓
Container
```

---

# 🔹 Pod и Docker Container

Docker:

```text id="r5q9mx"
Docker
  ↓
Container
```

Kubernetes:

```text id="v8m2kc"
Kubernetes
    ↓
   Pod
    ↓
Container
```

Поэтому Kubernetes управляет **Pod**, а контейнеры внутри Pod запускаются через container runtime.

---

# 🔹 Общая сеть

Контейнеры одного Pod используют общий network namespace.

Поэтому они могут обращаться друг к другу через:

```text id="c7m4vx"
localhost
```

Например:

```text id="q8n3mp"
Pod
 ├── FastAPI :8000
 └── Sidecar :9000
```

FastAPI может обратиться к sidecar:

```text id="j5r7kc"
localhost:9000
```

Это возможно потому, что контейнеры находятся в одном Pod и разделяют сетевое пространство.

---

# 🔹 IP-адрес Pod

Pod получает **один IP-адрес**.

Например:

```text id="m3q8vx"
Pod
IP: 10.0.2.15
 ├── FastAPI :8000
 └── Sidecar :9000
```

Оба контейнера используют этот Pod IP.

Но разные контейнеры должны слушать разные порты, если они используют один network namespace.

---

# 🔹 Pod эфемерен

Pod не следует рассматривать как постоянную виртуальную машину.

Он может быть уничтожен и создан заново.

Например:

```text id="x4m8qp"
Pod
IP: 10.0.2.15
     ↓
    ❌
     ↓
New Pod
IP: 10.0.3.21
```

Поэтому **не следует привязывать приложение к конкретному Pod IP**.

Для стабильного доступа используется:

```text id="k7q3vx"
Service
   ↓
Pods
```

---

# 🔹 Pod vs Service

Pod:

> запускает приложение.

Service:

> предоставляет стабильный сетевой endpoint для доступа к группе Pods.

```text id="p6m9xc"
             Service
                │
       ┌────────┼────────┐
       ↓        ↓        ↓
     Pod 1    Pod 2    Pod 3
```

Если Pod пересоздали и его IP изменился:

```text id="v4q8mn"
Pod 1 ❌
   ↓
Pod 4 🆕
```

Service продолжает предоставлять стабильный endpoint.

---

# 🔹 Pod vs Deployment

Pod можно создать напрямую:

```python id="m8q3vx"
apiVersion: v1
kind: Pod

metadata:
  name: fastapi

spec:
  containers:
    - name: fastapi
      image: myapp:1.0
```

Но в production обычно **не управляют одиночными Pods вручную**.

Используют Deployment:

```text id="q5m7kc"
Deployment
     ↓
ReplicaSet
     ↓
┌────┼────┐
↓    ↓    ↓
Pod  Pod  Pod
```

Deployment обеспечивает:

* нужное количество Pods;
* обновление версий;
* восстановление после отказов;
* rollback.

---

# 🔹 ReplicaSet и Pod

ReplicaSet следит за количеством Pods.

Например:

```python id="x7n4qm"
replicas: 3
```

Получаем:

```text id="c3m8vx"
ReplicaSet
    │
    ├── Pod 1
    ├── Pod 2
    └── Pod 3
```

Если Pod исчез:

```text id="p8q4kc"
Pod 2 ❌
   ↓
ReplicaSet обнаруживает:
2 вместо 3
   ↓
создаёт новый Pod
```

---

# 🔹 Labels у Pod

Pods обычно имеют Labels:

```python id="r6m2vx"
metadata:
  labels:
    app: fastapi
    environment: production
```

Service может использовать Label Selector:

```python id="n9q5kc"
selector:
  app: fastapi
```

Получаем:

```text id="m4x8qp"
Service
   │
   │ app=fastapi
   ↓
┌──────┬──────┬──────┐
Pod 1  Pod 2  Pod 3
```

---

# 🔹 Container внутри Pod

В Pod можно определить несколько контейнеров:

```python id="q7m3vx"
spec:
  containers:
    - name: app
      image: myapp:1.0

    - name: sidecar
      image: logger:1.0
```

Они работают рядом:

```text id="k8q4mc"
┌──────────────────────────────┐
│ Pod                          │
│                              │
│  ┌─────────┐   ┌──────────┐  │
│  │   App   │   │ Sidecar  │  │
│  └─────────┘   └──────────┘  │
│       shared network          │
│       shared volumes          │
└──────────────────────────────┘
```

---

# 🔹 Sidecar

**Sidecar** — дополнительный контейнер в том же Pod, который выполняет вспомогательную функцию для основного контейнера.

Например:

```text id="x3m7qp"
Pod
 ├── Application
 └── Logging Agent
```

Другие варианты:

* proxy;
* service mesh sidecar;
* агент мониторинга;
* обработка логов.

Но sidecar не обязателен.

---

# 🔹 Volumes внутри Pod

Контейнеры Pod могут использовать общий Volume:

```text id="p5m8vx"
             Pod
              │
       ┌──────┴──────┐
       ↓             ↓
   Container A   Container B
       │             │
       └──────┬──────┘
              ↓
           Volume
```

Это позволяет контейнерам обмениваться файлами.

---

# 🔹 Lifecycle Pod

Pod имеет жизненный цикл.

Упрощённо:

```text id="q8m4kc"
Pending
   ↓
Running
   ↓
Succeeded
```

или:

```text id="r3n7vx"
Pending
   ↓
Running
   ↓
Failed
```

Основные состояния Pod:

### `Pending`

Pod создан, но ещё не запущен полностью.

Например:

* Scheduler ещё не выбрал Node;
* Image ещё скачивается;
* не хватает ресурсов.

### `Running`

Pod назначен на Node и хотя бы один контейнер запущен или запускается.

### `Succeeded`

Все контейнеры успешно завершились.

### `Failed`

Все контейнеры завершились, и хотя бы один завершился с ошибкой.

### `Unknown`

Kubernetes не может получить информацию о состоянии Pod.

---

# 🔹 Restart Policy

Для Pod можно задать:

```python id="v5q8mx"
restartPolicy: Always
```

Возможные значения:

```text id="m4k7qp"
Always
OnFailure
Never
```

### `Always`

Перезапускать завершившиеся контейнеры.

### `OnFailure`

Перезапускать только после ошибки.

### `Never`

Не перезапускать.

Важно:

> Restart policy относится к **контейнерам внутри Pod**, а не означает, что Kubernetes обязательно создаст новый Pod.

---

# 🔹 Pod и Self-healing

Pod тесно связан с Self-healing.

Например:

```text id="k8m3qx"
Deployment
replicas: 3
       ↓
Pod 1
Pod 2
Pod 3
       ↓
Pod 2 ❌
       ↓
ReplicaSet
       ↓
New Pod
       ↓
Pod 1
Pod 3
Pod 4
```

Таким образом Kubernetes возвращает систему к:

```text id="q5n7vc"
Desired State = 3 Pods
```

---

# 🔹 Pod и Node

Pod всегда запускается на конкретной Node.

```text id="x4m8qp"
Cluster
   │
   ├── Node 1
   │     ├── Pod
   │     └── Pod
   │
   └── Node 2
         ├── Pod
         └── Pod
```

Scheduler выбирает Node для Pod с учётом:

* ресурсов;
* affinity;
* anti-affinity;
* taints/tolerations;
* других ограничений.

---

# 🔹 Pod Requests и Limits

Контейнеры внутри Pod могут иметь ограничения ресурсов:

```python id="p7m3vx"
resources:
  requests:
    cpu: "250m"
    memory: "256Mi"

  limits:
    cpu: "500m"
    memory: "512Mi"
```

`requests` используются Kubernetes при планировании.

`limits` задают верхние ограничения потребления ресурсов.

---

# 🔹 Pod и Health Probes

Pod может иметь:

### Liveness Probe

```text id="m8q4kc"
Liveness ❌
    ↓
restart container
```
