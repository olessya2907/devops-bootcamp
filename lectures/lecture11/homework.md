# Лекция 11: Домашнее задание

## Уровень 1 — Повторение лекции

Запусти Jenkins в Docker и создай первый pipeline.

**Шаги:**

1. Запусти Jenkins с docker.sock:
```bash
docker run -d \
  --name jenkins \
  -p 8080:8080 \
  -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  jenkins/jenkins:lts

docker exec -u root jenkins \
  bash -c "apt-get update -qq && apt-get install -y docker.io -qq"
```

2. Получи пароль и настрой Jenkins:
```bash
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```
Открой http://localhost:8080, вставь пароль, установи плагины, создай пользователя.

3. Создай Pipeline job с именем `devops-bootcamp` и тремя stages:
   - `Checkout` — вывести `pwd` и `ls`
   - `Build` — вывести `docker --version`
   - `Done` — вывести `"Билд #${BUILD_NUMBER} завершён"`

4. Запусти pipeline кнопкой **Build Now**

**Отправь в Slack:**
- Скриншот Stage View (все три колонки зелёные)
- Скриншот Console Output с последней строкой `Finished: SUCCESS`

---

## Уровень 2 — Мини-проект (необязательный)

Добавь в pipeline stage `Notify` который выводит информацию о билде:

```groovy
stage('Notify') {
    steps {
        echo "✅ Билд #${BUILD_NUMBER} завершён"
        echo "Job: ${JOB_NAME}"
        echo "Агент: ${NODE_NAME}"
        sh 'date'
    }
}
```

**Дополнительно:** открой http://localhost:8080/env-vars.html и найди ещё 3 переменные,
которых мы не использовали на лекции. Добавь их в stage `Notify`.

**Отправь в Slack:** скриншот Console Output с выводом всех переменных.
