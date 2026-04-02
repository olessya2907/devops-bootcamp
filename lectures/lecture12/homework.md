# Лекция 12: Домашнее задание

## Уровень 1 — Повторение лекции

Настрой полный pipeline: Jenkins читает Jenkinsfile из GitHub и пушит образ на Docker Hub.

**Шаги:**

1. Добавь credentials в Jenkins:
   ```
   Manage Jenkins → Credentials → Global → Add
   Kind: Username with password
   Username: твой Docker Hub username
   Password: пароль или Access Token
   ID: docker-hub-creds
   ```

2. В своём форке репо поменяй в `Jenkinsfile`:
   ```groovy
   DOCKER_HUB_REPO = "ТВОЙ_USERNAME/devops-bootcamp-backend"
   ```

3. Настрой Job:
   ```
   Job → Configure → Pipeline → Pipeline script from SCM
   Repository URL: https://github.com/ТВОЙ_USERNAME/devops-bootcamp
   Branch: */main
   Script Path: Jenkinsfile
   ```

4. Запусти Build Now, дождись завершения всех 5 stages.

5. Проверь результат:
   ```bash
   docker pull ТВОЙ_USERNAME/devops-bootcamp-backend:latest
   docker run --rm -p 3000:3000 ТВОЙ_USERNAME/devops-bootcamp-backend:latest
   curl.exe http://localhost:3000/health
   # → {"status":"ok","uptime":N}
   ```

**Отправь в Slack:**
- Ссылку на образ на Docker Hub
- Скриншот Stage View — все 5 stages зелёные

---

## Уровень 2 — Мини-проект (необязательный)

Добавь в Jenkinsfile stage `Security Scan` между `Test` и `Push`:

```groovy
stage('Security Scan') {
    steps {
        sh """
            docker scout quickview ${IMAGE_NAME}:${BUILD_NUMBER} || true
        """
    }
}
```

`docker scout` — сканер уязвимостей Docker Desktop. Проверяет образ на известные CVE.

`|| true` — pipeline не падает если scout не установлен или нашёл проблемы.
Это нормально на этапе обучения — в продакшне решаем найденные уязвимости.

**Отправь в Slack:** скриншот Console Output с выводом `docker scout quickview`.
