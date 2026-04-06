# Лекция 12 — Jenkins + Docker: шпаргалка

Jenkins читает Jenkinsfile из git → собирает образ → пушит на Docker Hub.

---

## Шаг 0 — Jenkins должен работать

```bash
# Проверить что Jenkins запущен
docker ps
# → строка с jenkins   Up ...

# Если не запущен — поднять из Л11:
docker start jenkins
```

### ✅ Проверь себя

Открой в браузере: `http://localhost:8080`
Должна открыться страница Jenkins — не ошибка подключения.

---

## Шаг 1 — Добавить credentials в Jenkins

```
Manage Jenkins → Credentials → System → Global credentials → Add Credentials

  Kind:     Username with password
  Username: YOUR_DOCKERHUB_USERNAME
  Password: YOUR_DOCKERHUB_PASSWORD
  ID:       docker-hub-creds
```

### ✅ Проверь себя

```
Credentials → список → видна запись docker-hub-creds
```

---

## Шаг 3 — Поправить Jenkinsfile

В `Jenkinsfile` в корне репо заменить одну строку:

```groovy
DOCKER_HUB_REPO = "YOUR_DOCKERHUB_USERNAME/devops-bootcamp-backend"
//                  ↑ поменять на свой Docker Hub username
```

Закоммитить и запушить:

```bash
git add Jenkinsfile
git commit -m "Set Docker Hub username"
git push
```

---

## Шаг 4 — Создать Job (Pipeline from SCM)

```
Jenkins → New Item → название: devops-bootcamp → Pipeline → OK

Pipeline:
  Definition:     Pipeline script from SCM
  SCM:            Git
  Repository URL: https://github.com/YOUR_USERNAME/devops-bootcamp
  Credentials:    - none - (репо публичное)
  Branch:         */main
  Script Path:    Jenkinsfile

→ Save
```

---

## Шаг 5 — Запустить билд

```
Job → Build Now
```

Следи за прогрессом: **Stage View** — пять этапов должны позеленеть.

### ✅ Проверь себя

```
Stage View:
  Checkout ✅  Build ✅  Test ✅  Push ✅  Cleanup ✅
```

Если какой-то stage красный — нажми на него → **Console Output** → читай ошибку.

---

## Шаг 6 — Проверить образ на Docker Hub

```
hub.docker.com → YOUR_USERNAME/devops-bootcamp-backend → Tags

Должны быть два тега:
  latest
  1  (или номер билда)
```

Скачать и запустить локально:

```bash
docker pull YOUR_DOCKERHUB_USERNAME/devops-bootcamp-backend:latest

docker run --rm -p 3000:3000 \
  YOUR_DOCKERHUB_USERNAME/devops-bootcamp-backend:latest
```

```powershell
# Windows:
curl.exe http://localhost:3000/health
# → {"status":"ok","uptime":N}
```

```bash
# macOS / Linux:
curl http://localhost:3000/health
```

---

## Шаг 7 — Poll SCM (автоматический триггер)

```
Job → Configure → Build Triggers → ✅ Poll SCM

Schedule: H/5 * * * *
```

После сохранения: любой `git push` в репо → Jenkins сам запустит билд через ≤5 минут.

Cron шпаргалка:
```
H/5 * * * *   → каждые 5 минут
H * * * *     → каждый час
H 2 * * 1-5  → будни в 2 ночи
H 9 * * 1     → каждый понедельник в 9 утра
```

---

## Troubleshooting

| Проблема | Решение |
|----------|---------|
| `docker: not found` внутри Jenkins | `docker exec -u root jenkins bash -c "apt-get update -qq && apt-get install -y docker.io -qq"` |
| `unauthorized` при push | Проверь credentials в Jenkins: ID = `docker-hub-creds`, пароль = токен |
| Stage View не появился | Установи плагин Pipeline: Manage Jenkins → Plugins → Pipeline |
| `cannot connect to Docker daemon` | Docker Desktop не запущен |
| Билд не стартует по расписанию | Проверь синтаксис cron, нажми Build Now вручную — должен работать |

---

## Структура Jenkinsfile (напоминалка)

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

    post {
        success { echo '✅ Успешно!' }
        failure { echo '❌ Ошибка!' }
    }
}
```

---

## Почитать

| Тема | Ссылка |
|------|--------|
| Jenkinsfile синтаксис | https://www.jenkins.io/doc/book/pipeline/syntax/ |
| withCredentials | https://www.jenkins.io/doc/pipeline/steps/credentials-binding/ |
| Docker Hub токены | https://docs.docker.com/security/for-developers/access-tokens/ |
| Cron триггеры | https://www.jenkins.io/doc/book/pipeline/syntax/#triggers |
