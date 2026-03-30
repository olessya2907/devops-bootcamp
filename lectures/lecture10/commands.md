# Лекция 10 — Docker Compose: шпаргалка

Запускаем полный стек: Nginx + Node.js backend + Redis — одной командой.

---

## Шаг 0 — Проверить установку

```bash
docker compose version
# Ожидаемый результат: Docker Compose version v2.x.x
```

---

## Шаг 1 — Получить файлы

```bash
git pull upstream main
ls
# Должен быть: docker-compose.yml
```

### ✅ Проверь себя

```bash
cat docker-compose.yml
# Должны быть три сервиса: backend, nginx, redis
```

---

## Шаг 2 — Запустить стек

```bash
docker compose up -d
```

Флаги:
- `-d` — detached, запустить в фоне

### ✅ Проверь себя

```bash
docker compose ps
```

Ожидаемый вывод:
```
NAME                SERVICE   STATUS    PORTS
...-backend-1       backend   running
...-nginx-1         nginx     running   0.0.0.0:8080->80/tcp
...-redis-1         redis     running
```

Все три сервиса в статусе `running`.

---

## Шаг 3 — Проверить приложение

Открой в браузере: `http://localhost:8080`

Или через терминал:

```powershell
# Windows:
curl.exe http://localhost:8080/health
# Ожидаемый ответ: {"status":"ok","uptime":N}

curl.exe http://localhost:8080/api/hits
# Ожидаемый ответ: {"hits":1}

curl.exe http://localhost:8080/api/hits
# Ожидаемый ответ: {"hits":2}
```

```bash
# macOS / Linux:
curl http://localhost:8080/health
curl http://localhost:8080/api/hits
```

---

## Шаг 4 — Управление стеком

```bash
# Логи всех сервисов:
docker compose logs

# Логи конкретного сервиса:
docker compose logs backend
docker compose logs -f nginx      # -f = следить в реальном времени

# Зайти внутрь контейнера:
docker compose exec backend sh
# Внутри контейнера:
env | grep REDIS
exit

# Перезапустить сервис:
docker compose restart backend
```

---

## Шаг 5 — Остановка и очистка

```bash
# Остановить стек (данные в Redis сохраняются):
docker compose down

# Остановить и удалить данные (счётчик сбросится):
docker compose down --volumes
```

### ✅ Проверь себя

```bash
docker compose ps
# Список пустой — все сервисы остановлены
```

---

## Troubleshooting

| Проблема | Решение |
|----------|---------|
| `port is already allocated` | Что-то занимает порт 8080 — `docker ps`, найди и останови |
| `Cannot connect to Docker daemon` | Docker Desktop не запущен |
| `/api/hits` возвращает ошибку | Подожди 5 сек после `docker compose up` — Redis стартует |
| Счётчик не сбрасывается | `docker compose down --volumes` — удалить данные Redis |
| `no such service` | Проверь имя сервиса в `docker compose ps` |

---

## Почитать

| Тема | Ссылка |
|------|--------|
| Docker Compose overview | https://docs.docker.com/compose/ |
| Compose file reference | https://docs.docker.com/compose/compose-file/ |
| docker compose CLI | https://docs.docker.com/reference/cli/docker/compose/ |
| Networking in Compose | https://docs.docker.com/compose/how-tos/networking/ |
