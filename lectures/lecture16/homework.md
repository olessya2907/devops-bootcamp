# Лекция 16 — Домашнее задание

## Уровень 1 — Повторение лекции

1. Убедиться что кластер запущен и поды из Л15 работают:
   ```bash
   minikube status
   kubectl get pods -n bootcamp
   ```

2. Включить Ingress addon если ещё не включён:
   ```bash
   minikube addons enable ingress
   kubectl get pods -n ingress-nginx
   # → ingress-nginx-controller-xxxx   1/1   Running
   ```

3. Применить Service и Ingress:
   ```bash
   kubectl apply -f k8s/backend-service.yaml
   kubectl apply -f k8s/ingress.yaml
   ```

4. Добавить `bootcamp.local` в hosts-файл:

   **Mac/Linux:**
   ```bash
   grep bootcamp.local /etc/hosts   # убедиться что нет дубликата
   echo "$(minikube ip) bootcamp.local" | sudo tee -a /etc/hosts
   ```

   **Windows** (Блокнот от Администратора → `C:\Windows\System32\drivers\etc\hosts`):
   ```
   127.0.0.1  bootcamp.local
   ```

   > **Windows:** IP должен быть `127.0.0.1`. Запустить tunnel в отдельном терминале и держать открытым:
   > ```bash
   > minikube tunnel
   > ```

5. Проверить что всё работает:
   ```bash
   curl http://bootcamp.local/health
   # → {"status":"ok","uptime":...}

   curl http://bootcamp.local/api/hits
   # → {"hits":1}
   curl http://bootcamp.local/api/hits
   # → {"hits":2}
   ```

6. Сделать скриншот: вывод `kubectl get svc,ingress -n bootcamp` + результат curl — отправить в Slack

---

## 📚 Что почитать

| Тема | Ссылка |
|------|--------|
| Kubernetes Service — типы и как работает | kubernetes.io/docs/concepts/services-networking/service |
| DNS для Service и Pod | kubernetes.io/docs/concepts/services-networking/dns-pod-service |
| Kubernetes Ingress — правила маршрутизации | kubernetes.io/docs/concepts/services-networking/ingress |
| pathType: Prefix vs Exact | kubernetes.io/docs/concepts/services-networking/ingress/#path-types |
| Ingress Controllers — обзор вариантов | kubernetes.io/docs/concepts/services-networking/ingress-controllers |
