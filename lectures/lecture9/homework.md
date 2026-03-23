# Домашнее задание — Лекция 9

---

## Уровень 1 — Повторение лекции

Собери образ backend-а и запусти контейнер самостоятельно.

**Шаги:**

```bash
git pull upstream main
cd app/backend
docker build -t devops-bootcamp-backend .
docker run -d -p 3000:3000 --name backend devops-bootcamp-backend
```

### ✅ Проверь себя

```bash
docker ps
# Контейнер "backend" в статусе "Up"

curl http://localhost:3000/health
# {"status":"ok"}

curl http://localhost:3000/info
# {"hostname":"...", "version":"1.0.0"}
```

**Поделись в Slack:**
- Скриншот: `docker images` (образ создан)
- Скриншот: браузер открыт на `http://localhost:3000/health`

**Не забудь остановить контейнер:**
```bash
docker stop backend
docker rm backend
```

---

## Уровень 2 — Мини-проект: упаковать чужое приложение

Найди на GitHub любое простое приложение и упакуй его в контейнер.

**Задача:** написать Dockerfile с нуля для приложения которое ты нашёл сам.

### Откуда брать приложение

Найди репо с простым веб-приложением — достаточно чтобы оно запускалось и отвечало на запросы.

Примеры для поиска на GitHub:
- `flask hello world` — простой Python веб-сервер
- `express starter` — Node.js приложение
- `fastapi example` — Python API
- `go http server` — Go веб-сервер

### Что нужно сделать

1. Выбрать репо и прочитать README — понять как приложение запускается
2. Клонировать к себе: `git clone ...`
3. Написать `Dockerfile` и `.dockerignore`
4. Собрать образ: `docker build -t my-app .`
5. Запустить контейнер: `docker run -p ПОРТ:ПОРТ my-app`
6. Убедиться что приложение отвечает

### Пример для Python/Flask

```dockerfile
FROM python:3.11-alpine
WORKDIR /app
COPY requirements.txt ./
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 5000
CMD ["python", "app.py"]
```

### ✅ Проверь себя

```bash
docker images       # твой образ в списке
docker ps           # контейнер запущен
curl http://localhost:ПОРТ   # приложение отвечает
```

**Поделись в Slack:**
- Ссылка на GitHub репо которое использовал
- Скриншот: `docker build` завершился успешно
- Скриншот: приложение отвечает в браузере или curl
- Твой Dockerfile (вставь текст прямо в Slack)
