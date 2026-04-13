# Лекция 14: Домашнее задание

## Уровень 1 — Повторение лекции

Задеплой devops-bootcamp на EC2 через Ansible с ролями.

**Шаги:**

1. Убедись что образ backend есть на Docker Hub (с Л12):
   ```bash
   docker pull ВАШ_USERNAME/devops-bootcamp-backend:latest
   ```

2. Открой `ansible/site.yml` и замени `docker_hub_user`:
   ```yaml
   vars:
     docker_hub_user: "ВАШ_DOCKERHUB_USERNAME"
   ```

3. Проверь что inventory.ini указывает на твой EC2:
   ```bash
   ansible all -m ping
   # → SUCCESS
   ```

4. Проверь синтаксис:
   ```bash
   cd ansible
   ansible-playbook site.yml --syntax-check
   ```

5. Запусти деплой:
   ```bash
   ansible-playbook site.yml
   ```

6. Проверь результат:
   ```bash
   curl http://EC2_IP
   curl http://EC2_IP/api/hits   # → {"hits":1}
   curl http://EC2_IP/health     # → {"status":"ok"}
   ```

**Отправь в Slack:**
- Скриншот `ansible-playbook site.yml` — `failed=0`
- Скриншот браузера `http://EC2_IP` — видно приложение
- Скриншот `http://EC2_IP/api/hits` — счётчик работает

---

## Уровень 2 — Мини-проект (необязательный)

Добавь роль `monitoring` в `ansible/roles/`:

```yaml
# roles/monitoring/tasks/main.yml
---
- name: Установить htop
  dnf:
    name: htop
    state: present

- name: Настроить cron: логировать нагрузку каждые 5 минут
  cron:
    name: "Log CPU usage"
    minute: "*/5"
    job: "top -bn1 | head -5 >> /var/log/server-stats.log"

- name: Настроить cron: логировать память каждый час
  cron:
    name: "Log memory usage"
    minute: "0"
    job: "free -m >> /var/log/server-stats.log"
```

Добавь роль в `site.yml`:
```yaml
  roles:
    - docker
    - app
    - monitoring
```

Проверь что cron работает:
```bash
ansible all -m command -a "crontab -l -u root"
```

**Отправь в Slack:** скриншот `ansible-playbook site.yml` с тремя ролями, `failed=0`.
