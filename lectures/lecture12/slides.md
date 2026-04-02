---
marp: true
theme: gaia
class: invert
paginate: true
---

# Лекция 12 — Jenkins + Docker

## Реальный CI Pipeline

---

## На прошлой лекции

```
Jenkins UI → вбили pipeline руками в браузере
Stages: pwd, docker --version, echo

Проблема:
  → pipeline не в git
  → нет истории изменений
  → потеряли UI — потеряли pipeline
```

---

## Сегодня

```
GitHub репо → Jenkinsfile в git
Jenkins читает Jenkinsfile из репо

git push
  → Jenkins сам забирает код
  → docker build
  → docker push на Docker Hub
  → новая версия образа готова
```

> То, что делаем сегодня — работает на реальных проектах

---

## Pipeline from SCM

```
Плохо:
  Jenkinsfile в Jenkins UI
  → у каждого своя версия
  → нет истории изменений
  → Jenkins упал — pipeline потерян

Хорошо:
  Jenkinsfile в git рядом с кодом
  → версионируется вместе с приложением
  → изменение pipeline = обычный коммит
  → Jenkins читает из репо при каждом билде
```

**SCM = Source Control Management = Git**

---

## Docker Registry

```
GitHub     — хранилище кода
Docker Hub — хранилище образов
```

Формат имени образа:
```
username/repo:tag
ivanpetrov/devops-bootcamp-backend:42
│           │                      │
аккаунт     название               версия
```

Компании используют:
- AWS ECR, GitLab Registry, Harbor
- Мы: Docker Hub (бесплатно, публично)

---

## Credentials — почему не в Jenkinsfile

```groovy
// Плохо — пароль в git (публичное репо!)
sh 'docker login -u ivanpetrov -p MySecret123'

// Хорошо — ссылка по ID, пароль в Jenkins
withCredentials([usernamePassword(
    credentialsId: 'docker-hub-creds',
    ...
)]) {
    sh 'docker login ...'
}
```

**Пароль никогда не попадает в git**

---

## Реальный кейс: утечка секрета

> Разработчик в пятницу закоммитил пароль от Docker Hub в Jenkinsfile.
> Репо публичное.
> Через 10 минут пришло письмо от GitHub:
> **"Leaked credential detected. Revoke immediately."**
>
> Пришлось в выходные менять пароль, ротировать токены,
> проверять логи на подозрительные действия.

**Правило: любой секрет — только через credentials store. Никаких исключений.**

---

## Access Token вместо пароля

```
Docker Hub → Settings → Security → Access Tokens
→ New Access Token → скопировать

В Jenkins: Password = токен (не пароль аккаунта)
```

**Почему токен лучше пароля:**
- Можно отозвать без смены пароля
- Можно ограничить права (Read Only, Read+Write)
- Видно когда и откуда использовался
- Один компрометирован — меняешь только его

---

## Структура Jenkinsfile

```groovy
pipeline {
    agent any
    environment {
        IMAGE_NAME      = "devops-bootcamp-backend"
        DOCKER_HUB_REPO = "username/devops-bootcamp-backend"
    }
    stages {
        stage('Checkout') { ... }  // git clone
        stage('Build')    { ... }  // docker build
        stage('Test')     { ... }  // smoke test
        stage('Push')     { ... }  // docker push
        stage('Cleanup')  { ... }  // docker rmi
    }
    post { success { } failure { } }
}
```

---

## stage('Build') — два тега

```groovy
docker build \
  -t ${IMAGE_NAME}:${BUILD_NUMBER} \
  -t ${IMAGE_NAME}:latest \
  ./app/backend
```

| Тег | Зачем |
|-----|-------|
| `latest` | всегда последний, удобно для быстрого старта |
| `42` | фиксированная версия, можно откатиться |

```bash
# Откат на конкретную версию
docker run myapp:41   # вместо myapp:latest
```

---

## stage('Push') — withCredentials

```groovy
withCredentials([usernamePassword(
    credentialsId: 'docker-hub-creds',
    usernameVariable: 'DOCKER_USER',
    passwordVariable: 'DOCKER_PASS'
)]) {
    sh '''
        echo $DOCKER_PASS | docker login \
          -u $DOCKER_USER --password-stdin
        docker push ${DOCKER_HUB_REPO}:${BUILD_NUMBER}
        docker push ${DOCKER_HUB_REPO}:latest
    '''
}
```

`--password-stdin` — пароль не виден в `ps aux`

---

## stage('Cleanup') — зачем

```groovy
stage('Cleanup') {
    steps {
        sh """
            docker rmi ${IMAGE_NAME}:${BUILD_NUMBER} || true
            docker rmi ${IMAGE_NAME}:latest           || true
        """
    }
}
```

Без cleanup: за неделю активной разработки
агент накапливает **десятки гигабайт** образов

`|| true` — если образ не найден, pipeline не падает

---

## Плохо / Хорошо: checkout

```groovy
// Плохо — хардкод URL репо прямо в скрипте
stage('Checkout') {
    steps {
        git 'https://github.com/user/repo'
    }
}

// Хорошо — Jenkins сам знает откуда читать
stage('Checkout') {
    steps {
        checkout scm
    }
}
```

`scm` = репо из настроек Job. Один Jenkinsfile работает
в любом форке без изменений.

---

## Триггеры — как Jenkins узнаёт о коммите

```
Manual       → Build Now (кнопка)
Poll SCM     → Jenkins спрашивает git каждые N минут
Webhook      → GitHub сам уведомляет Jenkins при push
Schedule     → по расписанию (cron)
```

---

## Poll SCM vs Webhook

```
Poll SCM:                    Webhook:
Jenkins → "Есть новое?"      GitHub → "Вот новый коммит!"
  каждые 5 минут               мгновенно при push
  работает с localhost          нужен публичный URL
  проще настроить               быстрее, сложнее
```

**На localhost → только Poll SCM**
GitHub не может достучаться до нашего ноутбука

На EC2 с публичным IP → webhook (покажем на Л24)

---

## Синтаксис cron

```
H/5 * * * *
│   │ │ │ └── день недели (0-7)
│   │ │ └──── месяц (1-12)
│   │ └────── день месяца (1-31)
│   └──────── час (0-23)
└──────────── минуты

H/5 = каждые 5 минут
H   = Jenkins выбирает случайную минуту
      (все job-ы не стартуют одновременно в :00)
```

---

## Jenkins на EC2

```bash
# User Data (Л8 + Л9 + Л11 = всё вместе)
docker run -d \
  --name jenkins \
  --restart unless-stopped \   # ← перезапуск после reboot
  -p 8080:8080 \
  -v jenkins_home:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  jenkins/jenkins:lts
```

`--restart unless-stopped` — Jenkins живёт сам,
не падает при перезагрузке EC2

---

## Полная картина

```
Разработчик:
  git push → GitHub

Jenkins (каждые 5 мин или webhook):
  git pull → checkout scm
  docker build -t myapp:42 .
  docker run --rm myapp:42 echo "OK"  ← smoke test
  docker push username/myapp:42
  docker push username/myapp:latest
  docker rmi myapp:42

Docker Hub:
  username/myapp:42  ✓
  username/myapp:latest  ✓
```

---

## Домашка — Уровень 1

1. Добавь credentials `docker-hub-creds` в Jenkins
2. В `Jenkinsfile` замени `YOUR_DOCKERHUB_USERNAME`
3. Job → Pipeline from SCM → твой форк репо
4. Build Now

**Отправь в Slack:**
- Ссылку на образ на Docker Hub
- Stage View — все пять stages зелёные

---

## Домашка — Уровень 2

```groovy
stage('Security Scan') {
    steps {
        sh """
            docker scout quickview \
              ${IMAGE_NAME}:${BUILD_NUMBER} || true
        """
    }
}
```

`docker scout` — сканер уязвимостей (CVE) от Docker Desktop.
Добавь между `Test` и `Push`.

Отправь: скриншот вывода из Console Output.

---

## Следующая лекция — Л13

**Ansible**

- Конфигурируем сервер автоматически
- Вместо: зашёл по SSH, руками поставил nginx
- Плейбук: описываешь желаемое состояние сервера

```yaml
- name: Install nginx
  apt:
    name: nginx
    state: present
```

> Terraform создаёт сервер. Ansible его настраивает.
