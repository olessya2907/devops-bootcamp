# Лекция 15 — Домашнее задание

## Уровень 1 — Повторение лекции

1. Запустить кластер:
   ```bash
   minikube start --driver=docker
   ```

2. Применить манифесты:
   ```bash
   kubectl apply -f k8s/namespace.yaml
   kubectl apply -f k8s/redis-deployment.yaml
   kubectl apply -f k8s/backend-deployment.yaml
   ```

3. Убедиться что оба Pod в статусе `Running`:
   ```bash
   kubectl get pods -n bootcamp
   ```
   Ожидаемый результат: `backend-xxxx` и `redis-xxxx` со статусом `Running`, READY `1/1`

4. Проверить через port-forward:
   ```bash
   kubectl port-forward deployment/backend 3000:3000 -n bootcamp
   ```
   Во втором терминале:
   ```bash
   curl localhost:3000/health
   curl localhost:3000/info
   curl localhost:3000/api/hits
   ```

5. Сделать скриншот: вывод `kubectl get pods -n bootcamp` + ответы curl — отправить в Slack

---

## Уровень 2 — Мини-проект (необязательный, но поощряется)

Найди на Docker Hub образ `traefik/whoami` — это простой HTTP-сервер который возвращает информацию о запросе и hostname контейнера.

Напиши `whoami-deployment.yaml` для namespace `bootcamp`:
- Deployment с 1 репликой
- Image: `traefik/whoami`
- Port: 80

Задеплой и проверь через port-forward:
```bash
kubectl port-forward deployment/whoami 8080:80 -n bootcamp
curl localhost:8080
```

Затем запусти 3 реплики и посмотри: меняется ли hostname при каждом запросе?
```bash
kubectl scale deployment whoami --replicas=3 -n bootcamp
kubectl port-forward deployment/whoami 8080:80 -n bootcamp
curl localhost:8080
curl localhost:8080
curl localhost:8080
```

Меняется ли hostname? Почему?

---

## 📚 Что почитать

| Тема | Ссылка |
|------|--------|
| Pod — минимальная единица в K8s | kubernetes.io/docs/concepts/workloads/pods |
| Deployment — управление репликами и self-healing | kubernetes.io/docs/concepts/workloads/controllers/deployment |
| Labels и Selectors — как K8s связывает объекты | kubernetes.io/docs/concepts/overview/working-with-objects/labels |
| Namespace — изоляция ресурсов в кластере | kubernetes.io/docs/concepts/overview/working-with-objects/namespaces |
| kubectl — шпаргалка по командам | kubernetes.io/docs/reference/kubectl/cheatsheet |
| minikube — начало работы | minikube.sigs.k8s.io/docs/start |
