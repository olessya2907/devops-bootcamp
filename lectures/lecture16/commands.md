# Лекция 16 — Kubernetes Services & Ingress: Шпаргалка

## Что добавляем в проект

```
devops-bootcamp/
└── k8s/
    ├── backend-service.yaml   ← NodePort Service
    └── ingress.yaml           ← Ingress → bootcamp.local
```

## Типы Services

| Тип | Доступен | Когда использовать |
|-----|----------|--------------------|
| `ClusterIP` | Только внутри кластера | Базы данных, внутренние сервисы |
| `NodePort` | Снаружи через Node IP:Port | Разработка, простой доступ |
| `LoadBalancer` | Через внешний LB провайдера | Production в облаке (AWS/GCP) |

## backend-service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend
  namespace: bootcamp
spec:
  type: NodePort
  selector:
    app: backend        # ← находит Pod по label
  ports:
  - port: 3000          # ← порт Service (внутри кластера)
    targetPort: 3000    # ← порт в контейнере
    nodePort: 30080     # ← порт на ноде (снаружи, 30000-32767)
```

## ingress.yaml

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: bootcamp-ingress
  namespace: bootcamp
spec:
  ingressClassName: nginx       # ← какой Ingress Controller
  rules:
  - host: bootcamp.local        # ← Host заголовок запроса
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: backend
            port:
              number: 3000
```

## Включить Ingress в minikube

```bash
minikube addons enable ingress

# Проверить Ingress Controller
kubectl get pods -n ingress-nginx
# → ingress-nginx-controller-xxxx   Running
```

## Применить манифесты

```bash
kubectl apply -f k8s/backend-service.yaml
kubectl apply -f k8s/ingress.yaml
```

## Команды для Services

```bash
# Список Services
kubectl get svc -n bootcamp
kubectl get services -n bootcamp   # то же самое

# Подробности Service
kubectl describe svc backend -n bootcamp

# URL для NodePort (только minikube)
minikube service backend --url -n bootcamp

# Все Services во всех namespace
kubectl get svc --all-namespaces
```

## Команды для Ingress

```bash
# Список Ingress
kubectl get ingress -n bootcamp

# Подробности
kubectl describe ingress bootcamp-ingress -n bootcamp

# IP кластера (для /etc/hosts)
minikube ip
```

## Настройка /etc/hosts

**Mac/Linux:**
```bash
echo "$(minikube ip) bootcamp.local" | sudo tee -a /etc/hosts
```

**Windows** (Блокнот от Администратора → `C:\Windows\System32\drivers\etc\hosts`):
```
127.0.0.1  bootcamp.local
```

> **Windows:** запустить `minikube tunnel` в отдельном терминале и держать открытым.

## Проверка через Ingress

```bash
curl http://bootcamp.local/health
curl http://bootcamp.local/info
curl http://bootcamp.local/api/hits
```

## Подключиться к Redis без NodePort (правильный способ)

```bash
kubectl exec -it deployment/redis -n bootcamp -- redis-cli
> ping
PONG
> keys *
> get hits
```

## Добавить второй сервис в Ingress

```yaml
spec:
  rules:
  - host: bootcamp.local
    http:
      paths:
      - path: /              # первый сервис
        pathType: Prefix
        backend:
          service:
            name: backend
            port:
              number: 3000
      - path: /whoami        # второй сервис
        pathType: Prefix
        backend:
          service:
            name: whoami
            port:
              number: 80
```

## ✅ Самопроверка

```bash
# 1. Services работают
kubectl get svc -n bootcamp
# → redis     ClusterIP   ...   6379/TCP
# → backend   NodePort    ...   3000:30080/TCP

# 2. Ingress Controller запущен
kubectl get pods -n ingress-nginx
# → ingress-nginx-controller-xxxx   1/1   Running

# 3. Ingress создан
kubectl get ingress -n bootcamp
# → bootcamp-ingress   nginx   bootcamp.local   192.168.x.x   80

# 4. Приложение доступно через Ingress
curl http://bootcamp.local/health
# → {"status":"ok","uptime":...}

# 5. Redis работает (счётчик растёт)
curl http://bootcamp.local/api/hits && curl http://bootcamp.local/api/hits
# → {"hits":1}
# → {"hits":2}
```

## Почитать

| Тема | Ссылка |
|------|--------|
| Kubernetes Service | kubernetes.io/docs/concepts/services-networking/service |
| Kubernetes Ingress | kubernetes.io/docs/concepts/services-networking/ingress |
| Ingress Controllers | kubernetes.io/docs/concepts/services-networking/ingress-controllers |
