# Лекция 11: Jenkins — Шпаргалка команд

## Установка Jenkins в Docker

```bash
# Запустить Jenkins (с docker.sock для доступа к Docker)
docker run -d --name jenkins -p 8080:8080 -p 50000:50000 -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts

docker run -d --name jenkins -p 8080:8080 -p 50000:50000 -v jenkins_home:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock jenkins/jenkins:lts

# Установить Docker CLI внутри Jenkins
docker exec -u root jenkins bash -c "apt-get update -qq && apt-get install -y docker.io -qq"

# Получить начальный пароль
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

## Управление Jenkins

```bash
# Посмотреть логи Jenkins
docker logs jenkins
docker logs -f jenkins          # следить в реальном времени

# Войти внутрь контейнера Jenkins
docker exec -it jenkins bash

# Перезапустить Jenkins (без потери данных)
docker restart jenkins

# Остановить и удалить контейнер (данные в volume сохранятся!)
docker stop jenkins
docker rm jenkins

# Удалить контейнер И данные
docker stop jenkins
docker rm jenkins
docker volume rm jenkins_home
```

## Volume

```bash
# Список всех volumes
docker volume ls

# Информация о jenkins_home
docker volume inspect jenkins_home

# Где хранится на хосте (Windows Docker Desktop)
# C:\Users\<username>\AppData\Local\Docker\wsl\data\...
```

## Проверки

```bash
# Docker CLI работает внутри Jenkins
docker exec jenkins docker --version

# Jenkins отвечает
curl http://localhost:8080/login

# Список jobs через API
curl -s http://admin:PASSWORD@localhost:8080/api/json | python -m json.tool
```

## URL Jenkins

```
http://localhost:8080               — главная страница
http://localhost:8080/env-vars.html — все переменные окружения
http://localhost:8080/manage        — настройки
http://localhost:8080/credentials   — хранилище секретов
```

## Первый Jenkinsfile (для вставки в UI)

```groovy
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                echo 'Получаем код...'
                sh 'pwd'
                sh 'ls -la'
            }
        }

        stage('Build') {
            steps {
                echo 'Проверяем Docker...'
                sh 'docker --version'
            }
        }

        stage('Done') {
            steps {
                echo "Билд #${BUILD_NUMBER} завершён"
            }
        }
    }

    post {
        success { echo '✅ Успешно!' }
        failure { echo '❌ Ошибка!' }
    }
}
```

## Переменные окружения Jenkins

```groovy
// Использование в pipeline:
sh 'echo "Билд: ${BUILD_NUMBER}"'
sh 'echo "Job: ${JOB_NAME}"'
sh 'echo "Агент: ${NODE_NAME}"'
sh 'echo "Папка: ${WORKSPACE}"'
sh 'echo "URL: ${BUILD_URL}"'
```

## Почитать

| Тема | Ссылка |
|------|--------|
| Pipeline синтаксис | https://www.jenkins.io/doc/book/pipeline/syntax/ |
| Переменные окружения | https://www.jenkins.io/doc/book/pipeline/jenkinsfile/#using-environment-variables |
| Pipeline шпаргалка | https://www.jenkins.io/doc/book/pipeline/getting-started/ |
