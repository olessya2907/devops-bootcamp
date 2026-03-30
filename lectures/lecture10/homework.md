# Домашнее задание — Лекция 10

---

## Уровень 1 — Повторение лекции

Подними полный стек и проверь что всё работает.

**Шаги:**

```bash
git pull upstream main
docker compose up -d
```

### ✅ Проверь себя

```bash
docker compose ps
# Все три сервиса в статусе running
```

Открой браузер: `http://localhost:8080`

Нажми кнопку "Hit me!" 5 раз — счётчик должен расти.

```powershell
# Windows:
curl.exe http://localhost:8080/api/hits
# {"hits":5} или больше
```

**Поделись в Slack:**
- Скриншот: браузер на `http://localhost:8080` со счётчиком ≥ 5
- Скриншот: `docker compose ps` — все три сервиса Up

**Не забудь остановить:**
```bash
docker compose down
```

---

## Уровень 2 — Мини-проект: добавить Redis UI

Добавь в `docker-compose.yml` четвёртый сервис — веб-интерфейс для Redis.

**Задача:** запустить `redis-commander` и открыть его в браузере.

### Что добавить в docker-compose.yml:

```yaml
  redis-commander:
    image: rediscommander/redis-commander:latest
    environment:
      - REDIS_HOSTS=local:redis:6379
    ports:
      - "8081:8081"
    depends_on:
      - redis
```

### Шаги:

1. Добавить сервис в `docker-compose.yml`
2. `docker compose up -d`
3. Открыть `http://localhost:8081`
4. В интерфейсе найти ключ `hits` — увидеть текущее значение счётчика
5. Нажать "Hit me!" на `http://localhost:8080` — значение изменится в redis-commander

### ✅ Проверь себя

```bash
docker compose ps
# Четыре сервиса в статусе running
```

```powershell
curl.exe http://localhost:8081
# Открывается redis-commander
```

**Поделись в Slack:**
- Скриншот: `http://localhost:8081` с ключом `hits` и его значением
- Твой обновлённый `docker-compose.yml` (вставь текст в Slack)

**Не забудь остановить:**
```bash
docker compose down
```
