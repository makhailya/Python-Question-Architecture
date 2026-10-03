# 🔑 SSH-ключи

## 🎤 Короткий ответ

**SSH-ключи** — это пара криптографических ключей, которая используется для аутентификации при подключении по SSH.

Пара состоит из:

```text
Private Key → хранится у клиента и никому не передаётся
Public Key  → размещается на сервере
```

Например:

```text
Ноутбук                         Linux Server
┌──────────────┐               ┌──────────────────┐
│ private key  │               │ authorized_keys  │
│              │               │                  │
│ id_ed25519   │               │ public key       │
└──────┬───────┘               └────────┬─────────┘
       │                                │
       └──────── SSH authentication ────┘
```

Классическая схема:

```bash
ssh-keygen -t ed25519
```

Публичный ключ копируется на сервер:

```bash
ssh-copy-id user@server
```

После этого можно подключаться:

```bash
ssh user@server
```

**Private key не передаётся серверу.** Сервер проверяет, что клиент действительно владеет соответствующим приватным ключом.

---

## 🗣️ Ответ на собеседовании

SSH-ключи используются для аутентификации клиента при подключении к SSH-серверу.

Создаётся асимметричная пара:

```text
private key + public key
```

Приватный ключ хранится только у клиента, а публичный помещается на сервер, обычно в:

```text
~/.ssh/authorized_keys
```

При подключении сервер проверяет, что клиент владеет соответствующим приватным ключом. Сам приватный ключ по сети не передаётся.

Обычно сегодня используют Ed25519:

```bash
ssh-keygen -t ed25519
```

После настройки подключение выглядит так:

```bash
ssh user@server
```

SSH-ключи безопаснее и удобнее паролей для серверного доступа и особенно важны в DevOps, Git, CI/CD и автоматизации.

---

## 🧭 Где я нахожусь

```text
07 DevOps
└── 06 Environment и Infrastructure
    ├── Linux
    │   ├── Файловая система
    │   ├── Процессы
    │   ├── Права доступа
    │   └── Сеть
    ├── Shell
    │   ├── Terminal
    │   └── Pipe
    ├── SSH
    │   ├── SSH-протокол
    │   ├── SSH-ключи ← Я здесь
    │   │   ├── Private Key
    │   │   ├── Public Key
    │   │   ├── authorized_keys
    │   │   ├── known_hosts
    │   │   └── ssh-agent
    │   └── Tunneling
    └── systemd
```

---

# 📚 Разбор поглубже

## 1. Зачем нужны SSH-ключи

Обычная аутентификация:

```text
user + password
```

SSH может использовать вместо этого:

```text
private key + public key
```

Преимущества:

* не нужно вводить пароль пользователя при каждом подключении;
* приватный ключ не передаётся серверу;
* удобно для автоматизации;
* удобно для Git;
* можно использовать passphrase;
* можно ограничивать доступ конкретными ключами.

---

# 2. Асимметричная криптография

SSH-ключи основаны на криптографической паре:

```text
Private Key
     │
     │ математически связанная пара
     ▼
Public Key
```

Главный принцип:

> Из публичного ключа нельзя практически получить приватный ключ при нормальных параметрах криптографического алгоритма.

Поэтому:

```text
Private Key → секрет
Public Key  → можно передавать
```

---

# 3. Private Key

Приватный ключ хранится **на клиентском компьютере**.

Например:

```text
~/.ssh/id_ed25519
```

Его нельзя публиковать или передавать другим людям.

Посмотреть содержимое технически можно:

```bash
cat ~/.ssh/id_ed25519
```

Но делать это без необходимости не следует.

Права обычно должны быть ограниченными:

```bash
chmod 600 ~/.ssh/id_ed25519
```

---

# 4. Public Key

Публичный ключ можно безопасно размещать на сервере.

Например:

```text
~/.ssh/id_ed25519.pub
```

Типичный вид:

```text
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAI... user@laptop
```

Именно **public key** добавляется на сервер.

---

# 5. `authorized_keys`

На сервере SSH обычно проверяет файл:

```text
~/.ssh/authorized_keys
```

Например:

```text
/home/ilya/.ssh/authorized_keys
```

В нём могут находиться публичные ключи:

```text
ssh-ed25519 AAAA... laptop
ssh-ed25519 BBBB... macbook
ssh-ed25519 CCCC... work
```

Каждая строка обычно соответствует отдельному разрешённому ключу.

Схема:

```text
Client
private key
    │
    │ authentication
    ▼
SSH Server
    │
    ▼
authorized_keys
    │
    └── matching public key
```

---

# 6. Как создать SSH-ключ

Современный распространённый вариант:

```bash
ssh-keygen -t ed25519
```

Можно указать имя файла:

```bash
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519
```

В результате обычно получаются:

```text
~/.ssh/id_ed25519
~/.ssh/id_ed25519.pub
```

То есть:

```text
id_ed25519
    → private key

id_ed25519.pub
    → public key
```

---

# 7. Passphrase

При создании ключа SSH обычно предлагает указать **passphrase**.

Получается дополнительный уровень защиты:

```text
Private Key
     ↓
зашифрован passphrase
     ↓
хранится на диске
```

Если злоумышленник украдёт файл приватного ключа, passphrase может защитить его от непосредственного использования.

Поэтому:

> SSH private key без passphrase и SSH private key с сильной passphrase — разные уровни защиты.

---

# 8. Как установить public key на сервер

Удобный способ:

```bash
ssh-copy-id user@server
```

Команда добавляет публичный ключ в:

```text
~/.ssh/authorized_keys
```

После этого:

```bash
ssh user@server
```

может использовать ключевую аутентификацию.

На системах, где `ssh-copy-id` отсутствует, public key можно добавить вручную.

---

# 9. Что происходит при SSH-подключении

Упрощённо:

```text
Client
  │
  │ ssh user@server
  ▼
SSH Server
  │
  │ authentication
  ▼
authorized_keys
```

Но важно не представлять это как:

```text
Client → отправил private key → Server
```

Такого не происходит.

Приватный ключ **не отправляется серверу**.

Клиент использует его для доказательства владения соответствующим ключом в процессе аутентификации.

---

# 10. Почему public key можно хранить на сервере

Потому что public key предназначен для проверки.

```text
Private Key
    ↓
создаёт криптографическое доказательство владения
    ↓
Server
    ↓
проверяет Public Key
```

Серверу не нужно знать приватный ключ.

Это фундаментальная идея асимметричной криптографии.

---

# 11. Private Key нельзя отправлять на сервер

Никогда не следует делать:

```bash
scp ~/.ssh/id_ed25519 user@server:/tmp/
```

Это передаст секретный ключ на сервер.

Правильная схема:

```text
Client
├── private key 🔒
└── public key
       │
       ▼
Server
└── authorized_keys
```

---

# 12. Права на SSH-файлы

SSH очень чувствителен к правам.

Например:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
chmod 600 ~/.ssh/authorized_keys
```

Типичная схема:

```text
~/.ssh/
├── id_ed25519          600
├── id_ed25519.pub      644
├── authorized_keys      600
└── known_hosts         600/644
```

Точные требования могут зависеть от конфигурации SSH и ОС.

---

# 13. `known_hosts`

Не путать:

```text
authorized_keys
```

и:

```text
known_hosts
```

### `authorized_keys`

На **сервере**:

```text
какие client public keys разрешены
```

### `known_hosts`

На **клиенте**:

```text
какие server host keys уже известны
```

Схема:

```text
Client                         Server

private key                    authorized_keys
     │                              │
     │ authentication               │
     └─────────────────────────────►

known_hosts
     ▲
     │
server host key
```

Это принципиально разные вещи.

---

# 14. Host Key

SSH-сервер тоже имеет криптографические ключи — **host keys**.

Они используются для идентификации сервера.

При первом подключении:

```bash
ssh user@server
```

SSH может показать:

```text
The authenticity of host 'server' can't be established.
```

После подтверждения информация о host key сохраняется в:

```text
~/.ssh/known_hosts
```

---

# 15. Зачем нужен `known_hosts`

Он помогает обнаружить ситуацию, когда вместо ожидаемого сервера мы подключаемся к другому.

Например:

```text
Первое подключение
       ↓
Server Host Key
       ↓
known_hosts
```

При следующем подключении:

```text
Server Host Key
       ↓
сравнение
       ↓
known_hosts
```

Если ключ неожиданно изменился, SSH предупреждает пользователя.

---

# 16. SSH и Man-in-the-Middle

Проверка host key помогает защититься от **MITM (Man-in-the-Middle)** атак.

Упрощённо:

```text
Client
   │
   ├──── ожидаемый server key ────┐
   │                             │
   │                         Server
```

Если посредник пытается выдать себя за сервер:

```text
Client
   │
   ▼
Attacker
   │
   ▼
Server
```

и host key не соответствует сохранённому, SSH может предупредить:

```text
WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!
```

Такое изменение не всегда означает атаку — например, сервер действительно могли переустановить или заменить его SSH host key. Но предупреждение нельзя бездумно игнорировать.

---

# 17. `ssh-agent`

`ssh-agent` — процесс, который может хранить приватные ключи в памяти и выполнять операции аутентификации от их имени.

Без agent:

```text
SSH
 ↓
Private Key
 ↓
Passphrase
```

С agent:

```text
Private Key
     ↓
ssh-agent
     ↓
SSH
```

Ключ загружается в agent:

```bash
ssh-add ~/.ssh/id_ed25519
```

Посмотреть загруженные ключи:

```bash
ssh-add -l
```

---

# 18. Зачем нужен ssh-agent

Представим, что private key защищён passphrase.

Без agent можно было бы вводить passphrase при необходимости использования ключа.

С agent:

```text
1. Один раз вводим passphrase
2. Ключ загружается в agent
3. SSH использует agent
4. Private key не приходится постоянно указывать/расшифровывать вручную
```

На современных macOS и Linux управление ключами может дополнительно интегрироваться с системным credential/keychain механизмом.

---

# 19. SSH Config

SSH позволяет хранить настройки в:

```text
~/.ssh/config
```

Например:

```sshconfig
Host myserver
    HostName 192.168.1.100
    User app
    IdentityFile ~/.ssh/id_ed25519
```

После этого вместо:

```bash
ssh app@192.168.1.100
```

можно:

```bash
ssh myserver
```

SSH сам подставит:

```text
HostName
User
IdentityFile
```

---

# 20. Несколько SSH-ключей

Например, можно иметь разные ключи:

```text
~/.ssh/
├── id_ed25519_personal
├── id_ed25519_personal.pub
├── id_ed25519_work
└── id_ed25519_work.pub
```

В `~/.ssh/config`:

```sshconfig
Host github-personal
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_personal

Host github-work
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_work
```

Это удобно для разделения окружений.

---

# 21. SSH-ключи и Git

SSH используется не только для серверов.

Например:

```bash
git clone git@github.com:username/project.git
```

Схема:

```text
Git
 ↓
SSH
 ↓
Private Key
 ↓
GitHub/GitLab
```

Публичный ключ добавляется в аккаунт GitHub/GitLab.

Private key остаётся на локальном компьютере.

---

# 22. SSH-ключи в CI/CD

В CI/CD SSH-ключи могут использоваться для:

* подключения к серверу;
* deployment;
* доступа к приватным Git-репозиториям;
* взаимодействия между системами.

Но private keys в CI/CD должны храниться как **секреты**, а не в Git-репозитории.

Упрощённо:

```text
CI/CD
  ↓
Secret Store
  ↓
Private Key
  ↓
SSH
  ↓
Server
```

---

# 23. Типичный deployment

Например:

```text
GitLab CI
    │
    │ SSH
    ▼
Production Server
    │
    ▼
systemd
    │
    ▼
FastAPI
```

CI runner использует SSH-ключ:

```text
Private Key → CI secret
Public Key  → ~/.ssh/authorized_keys на сервере
```

---

# 24. `ssh` и `scp`

SSH:

```bash
ssh user@server
```

используется для удалённой shell-сессии.

`scp`:

```bash
scp app.py user@server:/opt/myapp/
```

используется для копирования файлов через SSH.

Также существует SFTP:

```text
SSH
├── remote shell
├── SCP
└── SFTP
```

---

# 25. Типичная структура `~/.ssh`

```text
~/.ssh/
├── config
├── known_hosts
├── authorized_keys
├── id_ed25519
├── id_ed25519.pub
└── ...
```

Но не все эти файлы обязательно присутствуют на каждой машине.

Главное различать:

```text
id_ed25519
→ private key

id_ed25519.pub
→ public key

authorized_keys
→ разрешённые client public keys на сервере

known_hosts
→ известные server host keys на клиенте

config
→ настройки SSH-клиента
```

---

# 26. Что делать при `Permission denied (publickey)`

Если:

```bash
ssh user@server
```

возвращает:

```text
Permission denied (publickey).
```

проверяем:

### 1. Есть ли ключ

```bash
ls -la ~/.ssh/
```

### 2. Используется ли нужный ключ

```bash
ssh -v user@server
```

### 3. Есть ли public key на сервере

```text
~/.ssh/authorized_keys
```

### 4. Правильный ли пользователь

```bash
ssh correct_user@server
```

### 5. Права доступа

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

### 6. Загружен ли ключ в agent

```bash
ssh-add -l
```

---

# 27. `ssh -v`

Для диагностики:

```bash
ssh -v user@server
```

Более подробный вариант:

```bash
ssh -vvv user@server
```

Можно увидеть:

* какие ключи пробуются;
* какой конфиг используется;
* этапы аутентификации;
* почему конкретный ключ не подошёл.

Это очень полезно при настройке SSH.

---

# 28. SSH-ключи и безопасность

Основные правила:

```text
🔒 Private key → никому не передавать
📤 Public key → можно размещать на сервере
🔑 Private key → желательно защищать passphrase
📁 ~/.ssh → ограниченные права
🖥️ known_hosts → проверять неожиданные изменения
```

Особенно опасно:

```text
id_ed25519
id_rsa
*.pem
```

если это действительно приватные ключи.

Их нельзя коммитить в Git.

---

# 29. RSA vs Ed25519

SSH поддерживает разные типы ключей.

Исторически широко использовался:

```text
RSA
```

Современный распространённый вариант:

```text
Ed25519
```

Создание:

```bash
ssh-keygen -t ed25519
```

Для новых конфигураций Ed25519 обычно является удобным выбором, если его поддерживает используемый SSH stack.

---

# 30. Главная схема SSH-аутентификации

Нужно запомнить архитектуру:

```text
                 CLIENT
        ┌────────────────────┐
        │                    │
        │ Private Key 🔒     │
        │                    │
        └─────────┬──────────┘
                  │
                  │ SSH
                  ▼
              SERVER
        ┌────────────────────┐
        │ ~/.ssh/            │
        │ authorized_keys    │
        │                    │
        │ Public Key         │
        └────────────────────┘
```

И отдельно:

```text
CLIENT
~/.ssh/known_hosts
       ↑
       │
Server Host Key
```

То есть существуют **две разные пары/роли ключей**:

```text
Client authentication:
Private Key → Public Key
     client      server authorized_keys

Server identity:
Server Host Key → known_hosts
     server          client
```

---

## 🎤 Вопросы на собеседовании

### Что такое SSH-ключи?

Пара криптографически связанных ключей — private и public — используемая для аутентификации и других криптографических операций SSH.

### Где хранится private key?

На клиентской машине, например:

```text
~/.ssh/id_ed25519
```

### Где хранится public key клиента на сервере?

Обычно:

```text
~/.ssh/authorized_keys
```

### Передаётся ли private key серверу?

**Нет.** Клиент использует private key для доказательства владения соответствующим ключом.

### Что такое `authorized_keys`?

Файл на SSH-сервере со списком публичных ключей, которым разрешена аутентификация для соответствующего пользователя.

### Что такое `known_hosts`?

Файл на клиенте, содержащий известные server host keys и используемый для проверки идентичности SSH-сервера.

### Чем `authorized_keys` отличается от `known_hosts`?

```text
authorized_keys
→ какие клиенты могут войти

known_hosts
→ какие серверы клиент уже знает
```

### Зачем нужен passphrase?

Для защиты приватного ключа на диске. Если файл private key будет украден, passphrase создаёт дополнительный барьер для его использования.

### Что делает `ssh-agent`?

Хранит ключи в памяти и выполняет операции аутентификации от их имени, чтобы не вводить passphrase для каждого подключения.

### Как создать Ed25519-ключ?

```bash
ssh-keygen -t ed25519
```

### Как добавить ключ на сервер?

Обычно:

```bash
ssh-copy-id user@server
```

или вручную добавить содержимое `.pub` в:

```text
~/.ssh/authorized_keys
```

### Что такое SSH host key?

Ключ, принадлежащий SSH-серверу и используемый клиентом для проверки идентичности сервера.

### Что делать при `Permission denied (publickey)`?

Проверить:

```text
1. правильный ли пользователь;
2. существует ли private key;
3. используется ли нужный ключ;
4. есть ли соответствующий public key в authorized_keys;
5. права ~/.ssh и authorized_keys;
6. ssh-agent;
7. SSH-конфигурацию;
8. подробный лог ssh -vvv.
```

### Почему SSH-ключи важны для Backend-разработчика?

Они используются при работе с Linux-серверами, GitHub/GitLab, deployment, CI/CD, Docker/инфраструктурой и автоматизированным доступом к серверам.
