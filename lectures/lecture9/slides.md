---
marp: true
theme: gaia
class: invert
paginate: true
---

# Лекция 9 — Docker

## Контейнеры с нуля

---

## "Работает у меня"

```
Сервер (Amazon Linux):    yum install nginx  ✓

Твой ноутбук (macOS):     yum install nginx  ✗
                          команда не найдена

Ноутбук коллеги (Ubuntu): yum install nginx  ✗
                          команда не найдена
```

> Docker упаковывает приложение **вместе со всем что ему нужно**

---

## VM vs Container

<style scoped>
table { width: 100%; font-size: 0.85em; }
</style>

|  | Виртуальная машина | Docker контейнер |
|--|-------------------|-----------------|
| Что внутри | Приложение + полная OS | Приложение + зависимости |
| Размер | ~10 GB | ~180 MB |
| Старт | ~60 секунд | ~1 секунда |
| Ядро OS | своё | хостовой машины |

---

## Dockerfile → Image → Container

```
Dockerfile       Image            Container
(рецепт)    →   (слепок)    →   (работающий экземпляр)
          build            run
```

```bash
docker build -t my-app .    # собрать образ
docker run -p 3000:3000 my-app  # запустить контейнер
```

Из одного образа — сколько угодно контейнеров
*(увидим на Лекции 18: Kubernetes HPA)*

---

## Слои и кеш

```dockerfile
FROM node:18-alpine        # слой 1 — базовый образ
WORKDIR /app               # слой 2
COPY package*.json ./      # слой 3 — зависимости
RUN npm install            # слой 4 — кешируется! ✓
COPY . .                   # слой 5 — код
```

**Правило:** зависимости ПЕРЕД кодом

- Изменил `app.js` → пересобирается только слой 5
- `npm install` берётся из кеша
- **4 минуты → 40 секунд** *(реальный кейс)*

---

## Ключевые команды

```bash
docker build -t my-app .       # собрать образ
docker run -d -p 3000:3000 \
  --name backend my-app        # запустить в фоне

docker ps                      # запущенные контейнеры
docker logs backend            # логи
docker exec -it backend sh     # зайти внутрь

docker stop backend            # остановить
docker rm backend              # удалить контейнер
docker rmi my-app              # удалить образ
```

```powershell
curl.exe localhost:3000/health  # Windows
curl localhost:3000/health      # macOS / Linux
```
