# DevOps Bootcamp

Учебный проект. Простое веб-приложение, которое мы постепенно обкладываем DevOps-инструментами.

## Архитектура

```
Browser → Nginx (port 8080) → Node.js Backend → Redis
```

## Endpoints

| Endpoint        | Описание                          |
|-----------------|-----------------------------------|
| `GET /`         | Frontend (статика через Nginx)    |
| `GET /api/hits` | Счётчик хитов (Redis)             |
| `GET /health`   | Healthcheck бэкенда               |
| `GET /info`     | Hostname и версия (полезно в K8s) |
| `GET /metrics`  | Prometheus метрики                |

## Структура репо

```
devops-bootcamp/
├── app/
│   ├── backend/          # Node.js API
│   │   ├── app.js
│   │   ├── package.json
│   │   └── Dockerfile         # появился на Л9
│   └── frontend/         # Nginx + статика
│       ├── index.html
│       └── nginx.conf
└── lectures/             # файлы для практики на каждой лекции
    ├── lecture8/         # Terraform: EC2 + nginx + User Data
    │   ├── main.tf
    │   ├── variables.tf
    │   ├── outputs.tf
    │   ├── terraform.tfvars.example
    │   ├── commands.md
    │   ├── homework.md
    │   └── modules/ec2/
    └── lecture9/         # Docker: сборка образа backend
        ├── commands.md
        └── homework.md
```

## Правила безопасности

- **Никогда** не коммить `.pem`, `.env`, `terraform.tfvars`
- После каждой AWS-сессии запускать `terraform destroy`
- Настроить AWS Budget Alert на $10/month
