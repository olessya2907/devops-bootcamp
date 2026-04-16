# Лекция 15 — Kubernetes: Шпаргалка

## Установка

```bash
# minikube — Mac
brew install minikube

# minikube — Windows
choco install minikube

# kubectl — Mac
brew install kubectl

# kubectl — Windows
choco install kubernetes-cli
```

## Запуск кластера

```bash
minikube start --driver=docker   # запустить кластер
minikube stop                    # остановить
minikube delete                  # удалить кластер
minikube status                  # статус
minikube ip                      # IP адрес кластера
```

## Структура проекта (что добавляем)

```
devops-bootcamp/
└── k8s/
    ├── namespace.yaml           ← Namespace bootcamp
    ├── redis-deployment.yaml    ← Redis Deployment + Service
    └── backend-deployment.yaml  ← Backend Deployment
```

## namespace.yaml

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: bootcamp
```

## backend-deployment.yaml — ключевые части

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
  namespace: bootcamp
spec:
  replicas: 1              # ← желаемое количество Pod
  selector:
    matchLabels:
      app: backend         # ← Deployment управляет Pod с этим label
  template:
    metadata:
      labels:
        app: backend       # ← label на Pod
    spec:
      containers:
      - name: backend
        image: psychodol/devops-bootcamp:latest
        env:
        - name: REDIS_URL
          value: redis://redis:6379
        livenessProbe:     # ← перезапустить Pod если нет ответа
          httpGet:
            path: /health
            port: 3000
        readinessProbe:    # ← не слать трафик пока не готов
          httpGet:
            path: /health
            port: 3000
```

## Применить манифесты

```bash
kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/redis-deployment.yaml
kubectl apply -f k8s/backend-deployment.yaml

# Применить все файлы в папке
kubectl apply -f k8s/
```

## Основные команды kubectl

```bash
# Просмотр ресурсов
kubectl get pods -n bootcamp                    # список Pod
kubectl get pods -n bootcamp -w                 # watch (обновляется)
kubectl get all -n bootcamp                     # все ресурсы
kubectl get namespaces                          # список namespace

# Подробности
kubectl describe pod <name> -n bootcamp         # детали Pod + Events
kubectl describe deployment backend -n bootcamp  # детали Deployment

# Логи
kubectl logs <pod-name> -n bootcamp             # логи Pod
kubectl logs -l app=backend -n bootcamp         # логи по label
kubectl logs -f <pod-name> -n bootcamp          # следить за логами (follow)

# Выполнить команду в Pod
kubectl exec -it <pod-name> -n bootcamp -- sh   # войти в shell
kubectl exec -it deployment/backend -n bootcamp -- sh  # через Deployment

# Удаление
kubectl delete pod <name> -n bootcamp           # удалить Pod (Deployment создаст новый)
kubectl delete deployment backend -n bootcamp   # удалить Deployment
kubectl delete -f k8s/backend-deployment.yaml   # удалить по файлу

# Масштабирование
kubectl scale deployment backend --replicas=3 -n bootcamp

# Обновить образ
kubectl set image deployment/backend backend=psychodol/devops-bootcamp:v2 -n bootcamp
```

## Port-forward (для разработки/отладки)

```bash
kubectl port-forward deployment/backend 3000:3000 -n bootcamp
# → localhost:3000 → Pod:3000

kubectl port-forward pod/<name> 3000:3000 -n bootcamp
# Ctrl+C чтобы выйти
```

## Статусы Pod

| Статус | Что означает |
|--------|-------------|
| `Pending` | Pod создан, ищет ноду / скачивает образ |
| `Running` | Pod запущен, контейнеры работают |
| `CrashLoopBackOff` | Pod падает и перезапускается снова и снова |
| `ImagePullBackOff` | Не может скачать Docker-образ |
| `Terminating` | Pod удаляется |
| `Completed` | Pod завершил работу (Job/CronJob) |

## Дебаг

```bash
# Pod не стартует — смотреть Events
kubectl describe pod <name> -n bootcamp
# ищи секцию Events внизу

# Pod в CrashLoopBackOff — смотреть логи
kubectl logs <name> -n bootcamp
kubectl logs <name> -n bootcamp --previous  # логи предыдущего запуска

# События в namespace
kubectl get events -n bootcamp --sort-by=.lastTimestamp
```

## ✅ Самопроверка

```bash
# 1. minikube запущен
kubectl get nodes
# → minikube   Ready   control-plane

# 2. Оба пода работают
kubectl get pods -n bootcamp
# → backend-xxxx   1/1   Running   0
# → redis-xxxx     1/1   Running   0

# 3. Backend отвечает
kubectl port-forward deployment/backend 3000:3000 -n bootcamp &
curl localhost:3000/health
# → {"status":"ok","uptime":...}

# 4. Redis работает (счётчик растёт)
curl localhost:3000/api/hits
# → {"hits":1}
curl localhost:3000/api/hits
# → {"hits":2}
```

## Почитать

| Тема | Ссылка |
|------|--------|
| Kubernetes | kubernetes.io/docs |
| kubectl cheatsheet | kubernetes.io/docs/reference/kubectl/cheatsheet |
| minikube | minikube.sigs.k8s.io/docs |
