# ☸️ Kubernetes

## 🎤 Короткий ответ

**Kubernetes (K8s)** — это система оркестрации контейнеров, которая автоматически управляет их запуском, масштабированием, сетевым взаимодействием, обновлением и восстановлением.

Если Docker отвечает на вопрос **«как запустить контейнер?»**, то Kubernetes отвечает на вопрос **«как управлять сотнями контейнеров на нескольких серверах?»**

## 🎯 Формула для собеседования

> **Kubernetes = Cluster → Node → Pod → Container.**

А для приложения:

> **Deployment → управляет Pod'ами → Service даёт стабильный доступ → Ingress принимает внешний HTTP/HTTPS-трафик.**

---

# 🔹 Зачем нужен Kubernetes

Допустим, есть FastAPI:

```text id="7m3kqp"
FastAPI
   ↓
1 Container
```

Для небольшого проекта этого может быть достаточно.

Но в production может понадобиться:

```text id="x8q4mv"
              Load Balancer
                    ↓
        ┌───────────┼───────────┐
        ↓           ↓           ↓
     FastAPI      FastAPI      FastAPI
       Pod          Pod          Pod
```

А ещё:

* если контейнер упал → запустить новый;
* если нагрузки стало больше → увеличить количество экземпляров;
* обновить приложение без полного простоя;
* распределить контейнеры по серверам;
* проверять состояние приложения;
* хранить конфигурацию;
* управлять секретами.

Этим и занимается Kubernetes.

---

# 🔹 Kubernetes Cluster

**Cluster** — совокупность машин, которыми управляет Kubernetes.

```text id="p4n7xc"
             Kubernetes Cluster
                    │
        ┌───────────┴───────────┐
        ↓                       ↓
   Control Plane              Nodes
                              ┌──┼──┐
                              ↓  ↓  ↓
                             Pod Pod Pod
```

Кластер состоит из:

* **Control Plane** — управляет кластером;
* **Worker Nodes** — запускают приложения.

---

# 🔹 Node

**Node** — физический или виртуальный сервер, на котором Kubernetes запускает Pods.

```text id="k6m3qw"
Node
 ├── kubelet
 ├── container runtime
 └── Pods
```

Например:

```text id="v8q2mx"
Cluster
 ├── Node 1
 ├── Node 2
 └── Node 3
```

Если Node выходит из строя, Kubernetes может запустить нужные Pods на других доступных Nodes.

---

# 🔹 Pod

**Pod** — минимальная единица развертывания в Kubernetes.

Важно:

> Kubernetes запускает не контейнер напрямую, а **Pod**, внутри которого находятся один или несколько контейнеров.

Обычно:

```text id="m5r8xc"
Pod
 └── Container
      └── FastAPI
```

Но технически Pod может содержать несколько тесно связанных контейнеров:

```text id="q3n7vp"
Pod
 ├── Main container
 └── Sidecar container
```

Контейнеры одного Pod:

* используют общую сетевую namespace;
* могут обращаться друг к другу через `localhost`;
* могут использовать общие volumes.

---

# 🔹 Почему Pod, а не Container

Pod — это абстракция Kubernetes вокруг одного или нескольких контейнеров.

Например:

```text id="n4k8wp"
Pod
 ├── FastAPI
 └── Logging sidecar
```

Они должны работать совместно и поэтому размещаются вместе.

Но стандартный сценарий:

```text id="c7m2qx"
1 Pod
   ↓
1 основной контейнер
```

---

# 🔹 Deployment

**Deployment** управляет набором одинаковых Pods.

Например:

```text id="r8q3mc"
Deployment
     │
     ├── Pod 1
     ├── Pod 2
     └── Pod 3
```

Можно сказать:

> «Я хочу, чтобы постоянно работало 3 экземпляра приложения».

Deployment следит за этим состоянием.

Если Pod упал:

```text id="w5n7kx"
3 Pods
 ↓
Pod 2 ❌
 ↓
Deployment
 ↓
создаётся новый Pod
 ↓
3 Pods снова работают
```

---

# 🔹 ReplicaSet

Deployment обычно управляет **ReplicaSet**, а ReplicaSet — количеством Pods.

Упрощённо:

```text id="h9m4vp"
Deployment
     ↓
ReplicaSet
     ↓
┌────┼────┐
↓    ↓    ↓
Pod  Pod  Pod
```

Deployment также отвечает за управление версиями и rolling updates.

---

# 🔹 Desired State

Одна из фундаментальных концепций Kubernetes — **Desired State**.

Мы описываем:

> Какое состояние хотим получить.

Например:

```text id="a7q2mx"
replicas: 3
image: myapp:1.0
```

Kubernetes сравнивает:

```text id="k5n8vc"
Desired State
      ↓
      ↕
Current State
```

Если состояние отличается:

```text id="p3m7qx"
Хотим: 3 Pods
Есть:  2 Pods
       ↓
Kubernetes создаёт ещё один
```

Это называется **reconciliation** — приведение текущего состояния к желаемому.

---

# 🔹 YAML-манифест

Kubernetes обычно описывают декларативными YAML-манифестами.

Пример Deployment:

```python id="w6r2nk"
apiVersion: apps/v1
kind: Deployment

metadata:
  name: fastapi

spec:
  replicas: 3

  selector:
    matchLabels:
      app: fastapi

  template:
    metadata:
      labels:
        app: fastapi

    spec:
      containers:
        - name: fastapi
          image: myapp:1.0
          ports:
            - containerPort: 8000
```

Здесь:

```text id="z8q4mx"
kind: Deployment
        ↓
replicas: 3
        ↓
3 Pods
        ↓
myapp:1.0
```

---

# 🔹 Service

Pod имеет собственный IP, но Pod может быть пересоздан:

```text id="f3m7qx"
Pod
IP: 10.0.1.15
   ↓
Pod deleted
   ↓
New Pod
IP: 10.0.2.21
```

Поэтому обращаться напрямую к Pod неудобно.

**Service** предоставляет стабильный сетевой endpoint для группы Pods.

```text id="n8k3vc"
             Service
                │
       ┌────────┼────────┐
       ↓        ↓        ↓
     Pod 1    Pod 2    Pod 3
```

Service выбирает Pods по labels.

---

# 🔹 Service и Load Balancing

Например:

```text id="q6m4xp"
Client
  ↓
Service
  ↓
┌──────┬──────┬──────┐
↓      ↓      ↓
Pod 1  Pod 2  Pod 3
```

Service распределяет сетевой трафик между подходящими Pods.

---

# 🔹 Типы Service

Основные:

| Тип            | Назначение                 |
| -------------- | -------------------------- |
| `ClusterIP`    | доступ внутри кластера     |
| `NodePort`     | публикация через порт Node |
| `LoadBalancer` | внешний Load Balancer      |
| `ExternalName` | DNS-алиас внешнего сервиса |

По умолчанию:

```text id="c9q5mw"
ClusterIP
```

То есть Service доступен внутри кластера.

---

# 🔹 Ingress

**Ingress** управляет внешним HTTP/HTTPS-доступом к сервисам.

Например:

```text id="m2v7qx"
Internet
   ↓
Ingress
   ↓
Service
   ↓
Deployment
   ↓
Pods
```

Можно маршрутизировать:

```text id="p5n8kc"
api.example.com
       ↓
   FastAPI

admin.example.com
       ↓
   Django
```

Важно:

> **Ingress — это Kubernetes API-объект для описания HTTP/HTTPS-маршрутизации. Для фактической обработки трафика нужен Ingress Controller**, например NGINX Ingress Controller или Traefik.

---

# 🔹 ConfigMap

**ConfigMap** хранит некритичную конфигурацию приложения.

Например:

```python id="v4q8mx"
apiVersion: v1
kind: ConfigMap

metadata:
  name: app-config

data:
  APP_ENV: production
  LOG_LEVEL: info
```

Приложение может получить эти значения через environment variables или файлы.

---

# 🔹 Secret

**Secret** предназначен для конфиденциальных данных:

* пароли;
* токены;
* ключи;
* credentials.

Например:

```python id="k7m3qx"
apiVersion: v1
kind: Secret

metadata:
  name: db-secret

stringData:
  DB_PASSWORD: password
```

⚠️ Важно:

**Kubernetes Secret по умолчанию не означает шифрование секрета в etcd.**

Secret — объект для хранения/передачи чувствительных данных, а реальная защита зависит от конфигурации кластера, включая encryption at rest и RBAC.

---

# 🔹 ConfigMap vs Secret

| ConfigMap            | Secret                |
| -------------------- | --------------------- |
| Обычная конфигурация | Чувствительные данные |
| URL                  | Пароли                |
| Feature flags        | API keys              |
| Настройки приложения | Tokens                |

---

# 🔹 Namespace

**Namespace** — логическое разделение ресурсов внутри одного Kubernetes Cluster.

```text id="s4n8mx"
Cluster
 ├── namespace: dev
 │    ├── Pods
 │    └── Services
 │
 ├── namespace: staging
 │    ├── Pods
 │    └── Services
 │
 └── namespace: production
      ├── Pods
      └── Services
```

Namespaces помогают разделять:

* окружения;
* команды;
* приложения;
* права доступа;
* ресурсы.

---

# 🔹 Labels

**Labels** — пары `key=value`, которыми помечают ресурсы.

Например:

```python id="j8q2mc"
labels:
  app: fastapi
  environment: production
```

Service может искать Pods:

```python id="m5r7vx"
selector:
  app: fastapi
```

Получается:

```text id="z3n9qw"
Service
   │
   │ selector: app=fastapi
   ↓
Pod 1 → app=fastapi
Pod 2 → app=fastapi
Pod 3 → app=fastapi
```

---

# 🔹 Annotations

**Annotations** — метаданные для хранения дополнительной информации о ресурсе.

В отличие от Labels, они обычно **не используются для выбора ресурсов**.

Например:

```python id="x7m4qp"
annotations:
  description: "Production API"
```

---

# 🔹 kubelet

**kubelet** работает на каждой Worker Node.

Он отвечает за:

* получение информации о том, какие Pods должны работать на Node;
* запуск и контроль контейнеров через container runtime;
* выполнение lifecycle-операций;
* отправку информации о состоянии Node/Pods в Control Plane.

Упрощённо:

```text id="q8m3vx"
Control Plane
      ↓
   kubelet
      ↓
Container Runtime
      ↓
Container
```

---

# 🔹 Container Runtime

Kubernetes не обязан использовать именно Docker.

Современный Kubernetes взаимодействует с container runtime через **CRI (Container Runtime Interface)**.

Например:

* containerd;
* CRI-O.

Схема:

```text id="r6q2nw"
Kubernetes
    ↓
   CRI
    ↓
containerd / CRI-O
    ↓
Containers
```

Это важный момент:

> **Kubernetes и Docker — не одно и то же.**

---

# 🔹 Control Plane

Control Plane управляет кластером.

Основные компоненты:

```text id="m9k4qx"
Control Plane
├── kube-apiserver
├── etcd
├── scheduler
└── controller-manager
```

---

# 🔹 kube-apiserver

**API Server** — центральная точка взаимодействия с Kubernetes API.

Например:

```python id="v5m8qc"
kubectl get pods
```

запрашивает информацию через API Server.

Упрощённо:

```text id="p7q3mx"
kubectl
   ↓
kube-apiserver
   ↓
Kubernetes
```

---

# 🔹 etcd

**etcd** — распределённое key-value хранилище, в котором Kubernetes хранит состояние кластера и конфигурационные данные Control Plane.

Условно:

```text id="x4n8vk"
Kubernetes State
       ↓
      etcd
```

Например, Kubernetes должен знать:

```text
какие Deployment существуют
какие Pods должны работать
какие Services существуют
какое состояние ресурсов
```

---

# 🔹 Scheduler

**kube-scheduler** решает, на какой Worker Node разместить новый Pod.

```text id="k8m2qp"
New Pod
   ↓
Scheduler
   ↓
Node 1 / Node 2 / Node 3
```

Он учитывает различные ограничения и требования:

* доступные ресурсы;
* requests/limits;
* affinity/anti-affinity;
* taints/tolerations;
* topology constraints.

---

# 🔹 Controller Manager

Контроллеры следят за тем, чтобы фактическое состояние соответствовало желаемому.

Например:

```text id="u3q7mx"
Desired: 3 Pods
Current: 2 Pods
       ↓
Controller
       ↓
Create Pod
       ↓
Current: 3 Pods
```

Это и есть механизм **reconciliation**.

---

# 🔹 Kubernetes Architecture

Общая схема:

```text id="n7m4qx"
                    Kubernetes Cluster
                           │
              ┌────────────┴────────────┐
              │      Control Plane      │
              │                         │
              │ kube-apiserver          │
              │ scheduler               │
              │ controller-manager      │
              │ etcd                    │
              └────────────┬────────────┘
                           │
          ┌────────────────┼────────────────┐
          ↓                ↓                ↓
       Node 1           Node 2           Node 3
          │                │                │
        kubelet          kubelet          kubelet
          │                │                │
        Pods             Pods             Pods
```

---

# 🔹 Requests и Limits

Kubernetes позволяет указать ресурсы для контейнера.

Например:

```python id="c5r8mv"
resources:
  requests:
    cpu: "250m"
    memory: "256Mi"

  limits:
    cpu: "500m"
    memory: "512Mi"
```

### `requests`

Минимальный объём ресурсов, который Kubernetes учитывает при планировании Pod.

### `limits`

Ограничение потребления ресурсов контейнером.

Упрощённо:

```text id="q4n7xc"
requests → сколько нужно для размещения
limits   → сколько максимум можно использовать
```

---

# 🔹 Liveness Probe

Проверяет:

> **Процесс приложения жив?**

Если liveness probe постоянно не проходит, Kubernetes может перезапустить контейнер.

```text id="m8q3vx"
Pod
 ↓
Liveness Probe
 ↓
failure
 ↓
restart container
```

---

# 🔹 Readiness Probe

Проверяет:

> **Готово ли приложение принимать трафик?**

Если readiness не проходит:

```text id="w5r9mk"
Pod
 ↓
Readiness failure
 ↓
Pod временно исключается из Service endpoints
```

Контейнер при этом **не обязательно перезапускается**.

---

# 🔹 Liveness vs Readiness

| Liveness                         | Readiness                         |
| -------------------------------- | --------------------------------- |
| Приложение живо?                 | Готово принимать трафик?          |
| Может привести к restart         | Убирает Pod из Service endpoints  |
| Обнаруживает зависшее приложение | Обнаруживает неготовое приложение |

---

# 🔹 Rolling Update

Kubernetes может обновлять приложение постепенно.

Было:

```text id="r3m8qx"
v1
v1
v1
```

Обновляем на `v2`:

```text id="q7n4mc"
v1
v1
v2
```

Затем:

```text id="m5x8qp"
v1
v2
v2
```

И в конце:

```text id="c9r3vk"
v2
v2
v2
```

Это называется **Rolling Update**.

---

# 🔹 Rollback

Если новая версия работает неправильно, Deployment можно откатить:

```python id="x6m2qw"
kubectl rollout undo deployment fastapi
```

Получаем:

```text id="v8q4nk"
v2 ❌
 ↓
rollback
 ↓
v1 ✅
```

---

# 🔹 Horizontal Pod Autoscaler

**HPA** автоматически изменяет количество Pods в зависимости от нагрузки/метрик.

Например:

```text id="m4r7xc"
CPU ↑
 ↓
HPA
 ↓
3 Pods → 5 Pods
```

Когда нагрузка уменьшается:

```text id="k8q3mv"
CPU ↓
 ↓
HPA
 ↓
5 Pods → 3 Pods
```

---

# 🔹 PersistentVolume

Контейнеры и Pods эфемерны, поэтому для постоянных данных Kubernetes использует абстракции хранения.

Основные понятия:

```text id="p6m9qx"
Pod
 ↓
PersistentVolumeClaim
 ↓
PersistentVolume
 ↓
Storage
```

### PV

**PersistentVolume** — ресурс постоянного хранилища.

### PVC

**PersistentVolumeClaim** — запрос приложения на storage.

Упрощённо:

> **Pod говорит: «Мне нужно 10 GB диска» → PVC → Kubernetes предоставляет подходящий PV.**

---

# 🔹 StatefulSet

Для stateless-приложения:

```text id="q7m3vx"
Pod 1
Pod 2
Pod 3
```

не так важно, какой именно Pod обслуживает запрос.

Для stateful-приложений важны:

* стабильные имена;
* идентичность;
* порядок запуска;
* persistent storage.

Для этого используется **StatefulSet**.

Например:

```text id="x4n8mq"
postgres-0
postgres-1
postgres-2
```

---

# 🔹 Deployment vs StatefulSet

| Deployment                          | StatefulSet                                   |
| ----------------------------------- | --------------------------------------------- |
| Stateless приложения                | Stateful приложения                           |
| Pods взаимозаменяемы                | Pods имеют стабильную идентичность            |
| FastAPI                             | PostgreSQL/Kafka-подобные stateful workload'ы |
| Обычно без собственной идентичности | Стабильные имена и storage                    |

---

# 🔹 DaemonSet

**DaemonSet** гарантирует запуск Pod на каждой подходящей Node.

Например:

```text id="h5m8qx"
Node 1 → monitoring Pod
Node 2 → monitoring Pod
Node 3 → monitoring Pod
```

Используется для:

* логирования;
* monitoring agents;
* node-level служб.

---

# 🔹 Job и CronJob

### Job

Выполняет задачу до успешного завершения:

```text id="u8q3mk"
Job
 ↓
Pod
 ↓
Task
 ↓
Completed
```

Например:

```text id="w5m7qx"
database migration
```

### CronJob

Запускает Job по расписанию:

```text id="c4n8vp"
CronJob
   ↓
каждый день 03:00
   ↓
Job
   ↓
Pod
```

---

# 🔹 `kubectl`

Основной CLI для управления Kubernetes.

Посмотреть Pods:

```python id="r7m3qx"
kubectl get pods
```

Посмотреть Nodes:

```python id="k4n8vc"
kubectl get nodes
```

Посмотреть Deployment:

```python id="x6m2qp"
kubectl get deployments
```

Описание ресурса:

```python id="m8q5vr"
kubectl describe pod my-pod
```

Логи:

```python id="p3k7xc"
kubectl logs my-pod
```

Выполнить команду:

```python id="v9m4qx"
kubectl exec -it my-pod -- bash
```

Применить YAML:

```python id="j5n8mc"
kubectl apply -f deployment.yaml
```

---

# 🔹 Docker Compose vs Kubernetes

| Docker Compose                         | Kubernetes                             |
| -------------------------------------- | -------------------------------------- |
| Обычно локальная разработка            | Production/кластерная оркестрация      |
| Несколько контейнеров                  | Большие распределённые системы         |
| Простая конфигурация                   | Более сложная декларативная система    |
| Один хост/небольшие окружения          | Много Nodes                            |
| `docker compose up`                    | `kubectl apply`                        |
| Простое масштабирование                | Автоматическое масштабирование         |
| Простая сеть                           | Service/Ingress/Network Policies и др. |
| Нет полноценного cluster orchestration | Cluster orchestration                  |

Важно:

> Kubernetes не является «улучшенным Docker Compose». Это значительно более сложная система оркестрации.

---

# 🔹 Типичная архитектура Python Backend

```text id="e8m3qx"
                       Internet
                          │
                          ↓
                       Ingress
                          │
                          ↓
                       Service
                          │
              ┌───────────┼───────────┐
              ↓           ↓           ↓
            Pod         Pod         Pod
          FastAPI     FastAPI     FastAPI
              │           │           │
              └───────────┼───────────┘
                          ↓
                      PostgreSQL
                          │
                         PVC
```

Deployment при этом управляет Pods:

```text id="q6m8vx"
Deployment
     ↓
ReplicaSet
     ↓
┌────┼────┐
↓    ↓    ↓
Pod  Pod  Pod
```

---

# 🔹 Частые вопросы на собеседовании

### Что такое Kubernetes?

> Платформа оркестрации контейнеров, которая управляет их запуском, масштабированием, сетевым взаимодействием, обновлением и восстановлением.

### Что такое Cluster?

> Совокупность Control Plane и Worker Nodes, которыми управляет Kubernetes.

### Что такое Node?

> Машина, на которой Kubernetes запускает Pods.

### Что такое Pod?

> Минимальная единица развертывания Kubernetes, содержащая один или несколько контейнеров.

### Что такое Deployment?

> Ресурс, который управляет ReplicaSet и обеспечивает нужное количество Pods и обновление их версий.

### Что такое Service?

> Стабильный сетевой endpoint для доступа к группе Pods.

### Зачем Ingress?

> Для HTTP/HTTPS-маршрутизации внешнего трафика к Kubernetes Services.

### Что такое ConfigMap?

> Ресурс для хранения некритичной конфигурации приложения.

### Что такое Secret?

> Ресурс для хранения чувствительных конфигурационных данных, например паролей и токенов.

### Что такое Namespace?

> Логическое разделение ресурсов внутри Kubernetes Cluster.

### Что делает kube-scheduler?

> Выбирает подходящую Node для нового Pod.

### Что делает kubelet?

> На Node следит за состоянием Pods и управляет запуском контейнеров через container runtime.

### Что такое etcd?

> Распределённое key-value хранилище состояния и конфигурации Kubernetes.

### Что такое Desired State?

> Желаемое состояние кластера, которое Kubernetes постоянно пытается поддерживать.

### Что такое reconciliation?

> Процесс приведения фактического состояния ресурсов к желаемому.

### Liveness vs Readiness?

> Liveness отвечает «жив ли контейнер?», Readiness — «готов ли он принимать трафик?».

### Что такое HPA?

> Механизм автоматического изменения количества Pods на основе метрик нагрузки.

### Kubernetes использует Docker?

> Kubernetes не требует Docker как container runtime. Современный Kubernetes использует CRI и может работать, например, с containerd или CRI-O.

---

## 🔑 Главное

```text id="q8m4vx"
                    Kubernetes Cluster
                           │
              ┌────────────┴────────────┐
              ↓                         ↓
        Control Plane                Nodes
              │                         │
       ┌──────┼──────┐                  │
       ↓      ↓      ↓                  ↓
      API  Scheduler etcd             Pods
                                      │
                                ┌─────┼─────┐
                                ↓     ↓     ↓
                              App   Sidecar ...
                                │
                                ↓
                            Container
```

А для обычного Backend:

```text id="m7q3xc"
Deployment
    ↓
ReplicaSet
    ↓
Pods
    ↓
Containers

Service
    ↓
доступ к Pods

Ingress
    ↓
внешний HTTP/HTTPS трафик

ConfigMap
    ↓
конфигурация

Secret
    ↓
секреты

HPA
    ↓
масштабирование

PVC
    ↓
постоянное хранилище
```

### 🎤 Финальная формула

> **Kubernetes — это оркестратор контейнеров. Cluster состоит из Control Plane и Worker Nodes. Kubernetes запускает приложения в Pods, Deployment управляет количеством и версиями Pods, Service предоставляет стабильный доступ к ним, Ingress маршрутизирует внешний HTTP/HTTPS-трафик, ConfigMap и Secret хранят конфигурацию, а контроллеры постоянно приводят фактическое состояние к Desired State.**
