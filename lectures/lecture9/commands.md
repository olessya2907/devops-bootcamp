# Лекция 9 — Docker: шпаргалка

Упаковываем Node.js backend в контейнер. Всё запускается локально — AWS не нужен.

---

## Шаг 0 — Проверить установку Docker

```bash
docker --version
# Ожидаемый результат: Docker version 24.x.x или выше

docker run hello-world
# Ожидаемый результат: "Hello from Docker!"
```

Если Docker не установлен — скачай [Docker Desktop](https://www.docker.com/products/docker-desktop/).

---

## Шаг 1 — Получить файлы

```bash
# Обновить репо:
git pull upstream main

# Перейти в папку backend:
cd app/backend
```

Убедись что в папке есть `Dockerfile`:

```bash
ls
# Должны быть: app.js  Dockerfile  package.json  .dockerignore
```

---

## Шаг 2 — Собрать образ

```bash
docker build -t devops-bootcamp-backend .
```

Ожидаемый результат — последняя строка:
```
Successfully built abc123def456
```

Можно добавить тег версии:
```bash
docker build -t devops-bootcamp-backend:1.0 .
```

### ✅ Проверь себя — образ создан

```bash
docker images
```

Ожидаемый вывод:
```
REPOSITORY                  TAG       IMAGE ID       SIZE
devops-bootcamp-backend     latest    abc123def456   ~180MB
```

---

## Шаг 3 — Запустить контейнер

**Режим foreground** (видишь логи, Ctrl+C останавливает):
```bash
docker run -p 3000:3000 devops-bootcamp-backend
```

**Режим detached** (фоновый, рекомендуется):
```bash
docker run -d -p 3000:3000 --name backend devops-bootcamp-backend
```

Флаги:
- `-p 3000:3000` — порт на хосте : порт в контейнере
- `-d` — запустить в фоне
- `--name backend` — дать контейнеру имя

### ✅ Проверь себя — контейнер работает

```bash
docker ps
# Должен быть контейнер "backend" в статусе "Up"
```

Проверь приложение:

```bash
# Mac/Linux:
curl http://localhost:3000/health
# Ожидаемый ответ: {"status":"ok","uptime":0}

curl http://localhost:3000/info
# Ожидаемый ответ: {"hostname":"...", "version":"1.0.0", "node":"v18.x.x"}
```

```powershell
# Windows (PowerShell):
Invoke-WebRequest -Uri http://localhost:3000/health | Select-Object -Expand Content
```

Или просто открой в браузере: `http://localhost:3000/health`

---

## Шаг 4 — Управление контейнером

```bash
# Посмотреть запущенные контейнеры:
docker ps

# Посмотреть все контейнеры (включая остановленные):
docker ps -a

# Логи контейнера:
docker logs backend

# Следить за логами в реальном времени:
docker logs -f backend

# Остановить контейнер:
docker stop backend

# Запустить снова:
docker start backend

# Зайти внутрь контейнера:
docker exec -it backend sh
```

---

## Шаг 5 — Очистка

```bash
# Остановить и удалить контейнер:
docker stop backend
docker rm backend

# Удалить образ:
docker rmi devops-bootcamp-backend

# Посмотреть что осталось:
docker images
docker ps -a
```

---

## Troubleshooting

| Проблема | Решение |
|----------|---------|
| `port is already allocated` | Уже что-то занимает порт 3000 — `docker ps`, найди и останови |
| `Cannot connect to Docker daemon` | Docker Desktop не запущен — открой приложение |
| `no such image` | Сначала собери образ: `docker build -t devops-bootcamp-backend .` |
| Контейнер сразу останавливается | Смотри логи: `docker logs backend` |
| `permission denied` | На Linux: `sudo usermod -aG docker $USER`, перелогиниться |

---

## Почитать

| Тема | Ссылка |
|------|--------|
| Dockerfile reference | https://docs.docker.com/reference/dockerfile/ |
| docker build | https://docs.docker.com/reference/cli/docker/buildx/build/ |
| docker run | https://docs.docker.com/reference/cli/docker/container/run/ |
| .dockerignore | https://docs.docker.com/build/concepts/context/#dockerignore-files |
| Node.js на Docker Hub | https://hub.docker.com/_/node |
