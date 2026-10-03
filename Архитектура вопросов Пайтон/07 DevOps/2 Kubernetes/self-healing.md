# 🩹 Self-healing в Kubernetes

## 🎤 Короткий ответ

**Self-healing** в Kubernetes — это способность кластера **автоматически восстанавливать желаемое состояние приложения**, если что-то вышло из строя.

Например, если Pod с FastAPI упал, Kubernetes обнаруживает несоответствие между **Desired State** и текущим состоянием и создаёт новый Pod.

## 🎯 Формула для собеседования

> **Pod упал → Kubernetes обнаружил отклонение → Controller создал новый Pod → Desired State восстановлен.**

```text id="7q3m8x"
Desired State:
3 Pods
   ↓
Current State:
2 Pods
   ↓
Controller
   ↓
создаёт новый Pod
   ↓
Current State:
3 Pods
```

---

# 🔹 Что означает Self-healing

Kubernetes постоянно сравнивает:

```text id="m8k4vp"
Desired State
     ↕
Current State
```

Например, в Deployment указано:

```python id="x5q7nc"
replicas: 3
```

Это означает:

> Kubernetes должен поддерживать 3 экземпляра приложения.

Если один Pod погиб:

```text id="p3n9qx"
До сбоя:

Pod 1 ✅
Pod 2 ✅
Pod 3 ✅
```

После сбоя:

```text id="k7m2vw"
Pod 1 ✅
Pod 2 ❌
Pod 3 ✅
```

Контроллер обнаруживает:

```text id="r4q8mx"
Desired = 3
Current = 2
```

и создаёт новый Pod:

```text id="v6n3kp"
Pod 1 ✅
Pod 2 ❌
Pod 3 ✅
Pod 4 🆕
```

После запуска:

```text id="q8m5xc"
Pod 1 ✅
Pod 3 ✅
Pod 4 ✅
```

Снова:

```text id="d3k7vq"
Desired = 3
Current = 3
```

---

# 🔹 Кто отвечает за Self-healing

Self-healing — это не отдельный сервис Kubernetes.

За него работают **controllers** и другие компоненты Kubernetes.

Например:

```text id="n5q8mx"
Deployment
    ↓
ReplicaSet
    ↓
следит за количеством Pods
    ↓
Pod упал
    ↓
создаётся новый Pod
```

В более общем виде:

```text id="c7m3vx"
Control Plane
     ↓
Controllers
     ↓
сравнение Desired / Current State
     ↓
исправление отклонений
```

---

# 🔹 Self-healing при падении контейнера

Если приложение внутри Pod завершилось, Kubernetes может перезапустить контейнер.

Например:

```text id="p8r4km"
Pod
 └── FastAPI Container
          ↓
        crash
          ↓
    restart container
```

Здесь важна разница:

> **Перезапуск контейнера и создание нового Pod — не одно и то же.**

Если контейнер умер, kubelet может попытаться его перезапустить.

Если сам Pod исчез или ReplicaSet обнаружил, что экземпляров недостаточно, создаётся новый Pod.

---

# 🔹 Liveness Probe

Self-healing тесно связан с **Liveness Probe**.

Например:

```python id="m4q9vx"
livenessProbe:
  httpGet:
    path: /health/live
    port: 8000
  periodSeconds: 10
```

Kubernetes периодически проверяет приложение.

Если приложение зависло:

```text id="x7n3kc"
Liveness Probe
      ↓
    failure
      ↓
Kubernetes
      ↓
restart container
```

Например:

```text id="w8m2qp"
FastAPI
   ↓
deadlock / зависание
   ↓
процесс формально существует
   ↓
Liveness Probe ❌
   ↓
container restart
```

Это один из механизмов Self-healing.

---

# 🔹 Readiness Probe — немного другое

**Readiness Probe** определяет, готов ли Pod принимать трафик.

Если проверка не проходит:

```text id="f5q8mx"
Readiness Probe ❌
       ↓
Pod исключается из Service endpoints
       ↓
трафик больше не направляется на него
```

При этом контейнер **не обязательно перезапускается**.

Поэтому:

```text id="k3m7vx"
Liveness
   ↓
"Нужно ли перезапустить?"

Readiness
   ↓
"Можно ли отправлять трафик?"
```

---

# 🔹 Node failure

Self-healing работает не только на уровне Pod.

Предположим:

```text id="a8q4mc"
Node 1 ❌
   │
   ├── Pod 1
   ├── Pod 2
   └── Pod 3
```

Если кластер и его контроллеры определяют, что Node недоступна, Kubernetes может запланировать необходимые Pods на других Nodes, если это допускает конфигурация и ресурсы кластера.

```text id="q6m8vp"
Node 1 ❌
   ↓
Pods потеряны
   ↓
Controller / Scheduler
   ↓
Node 2 / Node 3
   ↓
новые Pods
```

---

# 🔹 Self-healing ≠ Backup

Это очень важное различие.

Self-healing восстанавливает **работу приложения**, но не обязательно **данные**.

Например:

```text id="r9m3kx"
PostgreSQL Pod
      ↓
     ❌
      ↓
новый PostgreSQL Pod
```

Новый Pod может запуститься, но если база не использовала persistent storage, данные могли быть потеряны.

Поэтому:

```text id="v4q7nc"
Self-healing
    ↓
восстановление сервисов

Backup
    ↓
восстановление данных
```

Для stateful-приложений нужны соответствующие механизмы хранения и резервного копирования.

---

# 🔹 Self-healing ≠ Autoscaling

Их часто путают.

### Self-healing

Восстанавливает нужное количество экземпляров после сбоя.

```text id="m7q2vx"
Хотим: 3 Pods
Есть:  2 Pods
      ↓
Self-healing
      ↓
3 Pods
```

### Autoscaling

Изменяет количество экземпляров **из-за изменения нагрузки**.

```text id="p8n4kc"
CPU ↑
 ↓
HPA
 ↓
3 Pods → 6 Pods
```

То есть:

> **Self-healing восстанавливает состояние после отказа, Autoscaling изменяет масштаб в зависимости от нагрузки.**

---

# 🔹 Self-healing и Deployment

Deployment:

```python id="c5m8qx"
spec:
  replicas: 3
```

означает:

```text id="j3q7vn"
Deployment
     ↓
ReplicaSet
     ↓
3 Pods
```

Если один Pod исчез:

```text id="x8m4kp"
3 → 2
   ↓
ReplicaSet
   ↓
создаёт новый Pod
   ↓
3
```

Именно поэтому Deployment является одним из основных механизмов Self-healing для stateless-приложений.

---

# 🔹 Self-healing и Service

Service сам по себе **не восстанавливает Pod**.

Он обеспечивает сетевой доступ:

```text id="f6q9mx"
Service
   ↓
Pods
```

Если Pod становится NotReady:

```text id="n4m7vc"
Pod
 ↓
Readiness Probe ❌
 ↓
Service
 ↓
не отправляет ему новый трафик
```

А Deployment/ReplicaSet может обеспечить необходимое количество Pods.

То есть механизмы работают вместе:

```text id="w3q8mx"
Probe
 ↓
определяет состояние

Service
 ↓
управляет доступностью трафика

Controller
 ↓
восстанавливает Desired State
```

---

# 🔹 Пример для FastAPI

Допустим:

```text id="r5m9kx"
Deployment
replicas: 3
```

И есть:

```text id="v7q2mc"
FastAPI Pod 1
FastAPI Pod 2
FastAPI Pod 3
```

Pod 2 завис:

```text id="p3m8qx"
Liveness Probe ❌
```

Kubernetes перезапускает контейнер.

Если Pod полностью потерян:

```text id="x6n4vk"
Deployment
   ↓
ReplicaSet
   ↓
обнаруживает только 2 Pods
   ↓
создаёт новый Pod
```

В результате приложение снова имеет нужное количество экземпляров.

---

# 🔹 Self-healing и Desired State

Это ключевая концепция, которую стоит связать на собеседовании.

```text id="q8m3vx"
YAML
 ↓
Desired State
 ↓
"replicas: 3"
 ↓
Kubernetes Controllers
 ↓
сравнение с Current State
 ↓
отклонение?
 ↓
исправление
```

Например:

```text id="k7q4mp"
Desired: 3
Current: 3
→ ничего не делать

Desired: 3
Current: 2
→ создать Pod

Desired: 3
Current: 4
→ привести к 3
```

---

# 🔹 Какие механизмы участвуют

Self-healing Kubernetes складывается из нескольких механизмов:

| Механизм        | Роль                                            |
| --------------- | ----------------------------------------------- |
| Deployment      | Управление желаемым количеством и версиями Pods |
| ReplicaSet      | Поддержание нужного количества Pods             |
| kubelet         | Контроль контейнеров на Node                    |
| Liveness Probe  | Обнаружение неработоспособного приложения       |
| Readiness Probe | Определение готовности принимать трафик         |
| Scheduler       | Размещение новых Pods на подходящих Nodes       |
| Controllers     | Reconciliation Desired/Current State            |
| Service         | Направление трафика к готовым Pods              |

---

# 🔹 Частые вопросы на собеседовании

### Что такое Self-healing?

> Способность Kubernetes автоматически восстанавливать желаемое состояние приложения после сбоев.

### Что произойдёт, если Pod из Deployment упадёт?

> ReplicaSet обнаружит, что Pods стало меньше требуемого количества, и Kubernetes создаст новый Pod.

### Что произойдёт, если контейнер завис, но процесс не завершился?

> При настроенном Liveness Probe Kubernetes может обнаружить проблему и перезапустить контейнер.

### Перезапустит ли Kubernetes Pod, если Readiness Probe не проходит?

> Нет, сама по себе Readiness Probe не предназначена для restart. Она указывает, что Pod пока не должен получать трафик через Service.

### Self-healing и Autoscaling — это одно и то же?

> Нет. Self-healing восстанавливает состояние после отказа, а Autoscaling изменяет количество ресурсов/Pods в соответствии с нагрузкой.

### Self-healing восстанавливает данные?

> Нет. Он восстанавливает работу workload'ов, но для сохранности данных нужны persistent storage и backup-механизмы.

### Кто отвечает за Self-healing?

> В основном Kubernetes Controllers, kubelet и связанные с ними механизмы reconciliation, scheduling и health probes.

---

## 🔑 Главное

```text id="m4q8vx"
                 Desired State
                     │
                 replicas: 3
                     │
                     ↓
                Controller
                     │
             ┌───────┴───────┐
             ↓       ↓       ↓
            Pod     Pod     Pod
             │       │       │
             ↓       ↓       ↓
             ✅      ❌      ✅
                     │
                     ↓
                 обнаружение
                     │
                     ↓
              новый Pod / restart
                     │
                     ↓
                3 Pods снова
```

### 🎤 Финальная формула

> **Self-healing в Kubernetes — это автоматическое восстановление Desired State. Если контейнер или Pod выходит из строя, Kubernetes с помощью kubelet, probes, controllers и scheduler может перезапустить контейнер или создать новый Pod на подходящей Node. При этом Self-healing отвечает за восстановление работы, а не за резервное копирование данных.**
