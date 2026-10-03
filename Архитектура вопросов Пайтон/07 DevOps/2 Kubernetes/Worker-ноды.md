# ☸️ Worker-ноды

## 🎤 Короткий ответ

**Worker-нода** — это сервер в Kubernetes-кластере, на котором непосредственно **запускаются Pod'ы [[Pod]]и контейнеры [[Контейнер]] приложения**.

Control Plane управляет кластером, а Worker Nodes выполняют рабочую нагрузку.

```python id="r8m4kx"
Kubernetes Cluster
│
├── Control Plane
│   └── Управляет кластером
│
├── Worker Node 1
│   ├── Pod
│   └── Pod
│
├── Worker Node 2
│   ├── Pod
│   └── Pod
│
└── Worker Node 3
    └── Pod
```

---

## 🎯 Формула для собеседования

> **Worker Node = вычислительный сервер Kubernetes, на котором kubelet управляет Pod'ами, container runtime запускает контейнеры, а kube-proxy/сетевой плагин обеспечивает сетевое взаимодействие.**

```python id="w5n2qv"
Cluster
   ↓
Worker Node
   ↓
Pod
   ↓
Container
```

---

# 🧠 Что такое Node

**Node** — это машина, на которой Kubernetes может запускать рабочие нагрузки.

Node может быть:

* физическим сервером;
* виртуальной машиной;
* облачным сервером.

Например:

```text id="m7q3pz"
Physical Server
      ↓
      VM
      ↓
Worker Node
      ↓
Pods
```

---

# ⚙️ Основные компоненты Worker Node

На Worker Node находятся ключевые компоненты:

```text id="c4k8yx"
Worker Node
│
├── kubelet
├── Container Runtime
├── kube-proxy
└── Pods
```

---

# 🔧 kubelet

**kubelet** — агент Kubernetes, работающий на каждой Worker Node.

Его задача:

> **обеспечивать запуск и состояние Pod'ов согласно тому, что требуется Kubernetes.**

Упрощённо:

```text id="s2p6mv"
Control Plane
      ↓
   API Server
      ↓
   kubelet
      ↓
   Pod
```

kubelet:

* получает информацию о Pod'ах;
* взаимодействует с container runtime;
* следит за состоянием контейнеров;
* выполняет lifecycle-операции;
* запускает probes;
* сообщает состояние Node и Pod'ов обратно в Control Plane.

---

# 🐳 Container Runtime

**Container Runtime** отвечает непосредственно за запуск и управление контейнерами.

Например:

```text id="y8v3nk"
kubelet
   ↓
Container Runtime
   ↓
Container
```

Современный Kubernetes взаимодействует с runtime через **CRI (Container Runtime Interface)**.

Распространённые runtime:

* containerd;
* CRI-O.

Важно:

> **Kubernetes не обязан использовать Docker как container runtime.**

---

# 🌐 kube-proxy

`kube-proxy` традиционно отвечает за реализацию части сетевого поведения Kubernetes Services на Node.

Упрощённо:

```text id="f5m2qc"
Client
  ↓
Service
  ↓
Network rules
  ↓
Pod
```

Он связан с маршрутизацией трафика к Pod'ам через Service.

В современных Kubernetes-сетях конкретная реализация может зависеть от используемого network plugin, поэтому `kube-proxy` не стоит представлять как единственный компонент всей Kubernetes-сети.

---

# 📦 Что запускается на Worker Node

Основная рабочая нагрузка — **Pod'ы**.

Например:

```text id="v9k3mx"
Worker Node
│
├── Pod
│   └── FastAPI
│
├── Pod
│   └── Celery Worker
│
└── Pod
    └── Nginx
```

Один Worker Node может запускать множество Pod'ов.

Количество зависит от:

* CPU;
* RAM;
* requests/limits;
* количества Pod'ов;
* сетевых и других ограничений.

---

# 🧮 Resource Requests и Limits

Kubernetes учитывает ресурсы Pod'ов.

Например:

```python id="q6w8mr"
resources:
  requests:
    cpu: "500m"
    memory: "512Mi"

  limits:
    cpu: "1"
    memory: "1Gi"
```

### Requests

Минимальный объём ресурсов, который Pod **запрашивает для планирования**.

Scheduler использует requests при выборе Node.

### Limits

Максимальный лимит ресурса для контейнера, в зависимости от ресурса и настроек.

Упрощённо:

```text id="z4n7kp"
Worker Node
│
├── 4 CPU
├── 8 GB RAM
│
├── Pod A → requests 1 CPU
├── Pod B → requests 1 CPU
└── Pod C → requests 1 CPU
```

Scheduler должен учитывать доступные ресурсы Node.

---

# 🧭 Кто решает, на какую Worker Node попадёт Pod

Этим занимается **kube-scheduler** в Control Plane.

Схема:

```text id="j3x8qm"
Deployment
    ↓
Pod
    ↓
Scheduler
    ↓
выбирает Node
    ↓
Worker Node
    ↓
kubelet
    ↓
Container Runtime
    ↓
Container
```

Scheduler учитывает, среди прочего:

* requests ресурсов;
* ограничения;
* affinity/anti-affinity;
* taints/tolerations;
* topology constraints;
* доступность подходящих Node.

---

# ❤️ Self-Healing Worker Node

Если контейнер внутри Pod падает, kubelet может перезапустить контейнер согласно его политике и состоянию Pod.

```text id="w6q2mx"
Container ❌
     ↓
kubelet
     ↓
restart
     ↓
Container ✅
```

Если сам Pod исчезает, например из-за действий Node/контроллера, **Deployment/ReplicaSet** может создать новый Pod.

```text id="r9m3kp"
Pod ❌
 ↓
ReplicaSet
 ↓
New Pod
```

Если Node выходит из строя:

```text id="n4x7vc"
Worker Node ❌
       ↓
Pods недоступны
       ↓
Controller / Scheduler
       ↓
Pods могут быть размещены на другой Node
```

Это зависит от конфигурации, доступных ресурсов и типа workload.

---

# 🩺 Состояние Worker Node

Kubernetes отслеживает состояние Node.

Например:

```python id="p7m2qx"
kubectl get nodes
```

Можно получить:

```text id="n8q4vz"
NAME       STATUS   ROLES
worker-1   Ready    <none>
worker-2   Ready    <none>
worker-3   Ready    <none>
```

`Ready` означает, что Node считается готовой принимать нагрузку.

---

# 🔍 Информация о Node

```python id="k3v8mx"
kubectl describe node worker-1
```

Можно посмотреть:

* CPU;
* RAM;
* Pod'ы;
* Conditions;
* labels;
* taints;
* allocated resources;
* события.

---

# 🏷️ Labels

Worker Node можно пометить label:

```python id="x6q2pw"
kubectl label nodes worker-1 disk=ssd
```

После этого можно попросить Kubernetes размещать Pod на Node с таким label:

```python id="v4m9ks"
nodeSelector:
  disk: ssd
```

Схема:

```text id="a7p3qn"
Pod
 ↓
nodeSelector: disk=ssd
 ↓
Worker Node с disk=ssd
```

---

# 🚧 Taints и Tolerations

Можно запретить обычным Pod'ам размещаться на определённой Node.

Например:

```text id="e8m4zx"
Worker Node
   ↓
Taint
   ↓
обычный Pod не размещается
```

Pod должен иметь соответствующий **toleration**.

Это удобно для специализированных Node:

```text id="h5q7mk"
GPU Node
Database Node
System Node
```

---

# 🧩 Worker Nodes и Control Plane

Основное различие:

| Control Plane       | Worker Node                  |
| ------------------- | ---------------------------- |
| Управляет кластером | Выполняет workload           |
| API Server          | kubelet                      |
| Scheduler           | Container Runtime            |
| Controller Manager  | kube-proxy/сетевой компонент |
| etcd                | Pods                         |
| Принимает решения   | Запускает приложения         |

Упрощённо:

```text id="c9w2mv"
             Kubernetes Cluster
                     │
          ┌──────────┴──────────┐
          ↓                     ↓
    Control Plane          Worker Nodes
          │                     │
     принимает решения       выполняют
          │                     │
          └──────────┬──────────┘
                     ↓
                   Pods
```

---

# 📊 Несколько Worker Nodes

Для отказоустойчивости используют несколько Node:

```text id="r3k8xp"
             Cluster
                │
       ┌────────┼────────┐
       ↓        ↓        ↓
    Worker1  Worker2  Worker3
       │        │        │
      Pods     Pods     Pods
```

Например, Deployment:

```python id="m8q5vz"
replicas: 3
```

может получить:

```text id="x2n7kc"
Worker 1 → Pod A
Worker 2 → Pod B
Worker 3 → Pod C
```

Но конкретное размещение определяет scheduler с учётом ограничений и доступных ресурсов.

---

# 🔄 Масштабирование

Если нагрузки становится больше:

```text id="q6w3pm"
Worker 1
Worker 2
      ↓
  недостаточно ресурсов
      ↓
Worker 3
      ↓
дополнительные Pods
```

Добавление Worker Nodes — это **горизонтальное масштабирование инфраструктуры кластера**.

При этом Kubernetes также может масштабировать сами приложения через:

```text id="s7m4qx"
Deployment
    ↓
replicas

или

HPA
    ↓
количество Pods
```

Важно различать:

> **Scaling Nodes ≠ Scaling Pods.**

---

# 🏗️ Типичная архитектура Python Backend

Например:

```text id="n2x8vc"
                  Kubernetes Cluster
                         │
                 ┌───────┴───────┐
                 ↓               ↓
            Control Plane     Workers
                                │
                    ┌───────────┼───────────┐
                    ↓           ↓           ↓
                 Worker 1    Worker 2    Worker 3
                    │           │           │
                 FastAPI      FastAPI     Celery
                    │           │           │
                    └───────┬───┴───────────┘
                            ↓
                         Service
                            ↓
                          Ingress
```

---

# 🎤 Как рассказать на собеседовании

> **Worker Node — это сервер Kubernetes, на котором запускаются Pod'ы с рабочими нагрузками. На Node работает kubelet, который взаимодействует с Control Plane и container runtime, а также отслеживает состояние Pod'ов. Container runtime непосредственно запускает контейнеры. Scheduler выбирает подходящую Worker Node для Pod с учётом ресурсов и ограничений. Несколько Worker Nodes позволяют распределять нагрузку и повышать отказоустойчивость.**

---

# ❓ Частые вопросы

### Что такое Worker Node?

Сервер Kubernetes, на котором запускаются Pod'ы приложения.

### Кто запускает контейнеры?

Непосредственно **container runtime**, которым управляет kubelet.

### Кто выбирает Node для Pod?

**kube-scheduler**.

### Что делает kubelet?

Управляет жизненным циклом Pod'ов на конкретной Node и сообщает их состояние Kubernetes.

### Что будет, если Worker Node упадёт?

Pod'ы на ней станут недоступны. Для управляемых workload'ов Kubernetes может создать/разместить новые Pod'ы на других подходящих Node, если это возможно.

### Можно ли запускать несколько Pod на одной Node?

Да. Количество ограничено ресурсами и другими ограничениями.

### Чем Worker Node отличается от Control Plane?

**Control Plane управляет кластером, Worker Node выполняет workload.**

### Можно ли использовать виртуальную машину как Worker Node?

Да. Worker Node может быть VM или физическим сервером.

---

## 🔑 Главное

```python id="f8m3qw"
             Kubernetes Cluster
                    │
        ┌───────────┴───────────┐
        ↓                       ↓
  Control Plane           Worker Node
  "управляет"              "выполняет"
                                │
                     ┌──────────┼──────────┐
                     ↓          ↓          ↓
                    Pod        Pod        Pod
                     │
                 Container
```

**Worker Node = место, где реально выполняется приложение.**

```python id="u4k9mx"
Scheduler → выбирает Node
kubelet → управляет Pod
Runtime → запускает Container
Pod → содержит Container
Worker Node → предоставляет ресурсы
```
