# Лекция 13: Домашнее задание

## Уровень 1 — Повторение лекции

Запусти Ansible плейбук на своём EC2 и убедись что nginx работает.

**Шаги:**

1. Запусти EC2 (Amazon Linux 2023, t2.micro) и запиши публичный IP.
   Security Group: порты 22 (SSH) и 80 (HTTP) открыты.

2. Настрой inventory:
   ```bash
   cd devops-bootcamp/ansible
   cp inventory.ini.example inventory.ini
   # Открой inventory.ini и вставь IP + путь к ключу
   ```

3. Проверь соединение:
   ```bash
   ansible all -m ping
   # → SUCCESS
   ```

4. Запусти плейбук:
   ```bash
   ansible-playbook playbook.yml
   ```

5. Проверь результат:
   ```bash
   curl http://EC2_IP
   # → HTML страница с hostname сервера
   ```

**Отправь в Slack:**
- Скриншот вывода `ansible-playbook playbook.yml` — все задачи ok/changed, `failed=0`
- Скриншот браузера с открытым `http://EC2_IP` — видна страница с hostname

---

## Уровень 2 — Мини-проект (необязательный)

Добавь в `playbook.yml` проверку что nginx отвечает после установки:

```yaml
- name: Проверить что nginx отвечает
  uri:
    url: http://localhost
    status_code: 200
```

И задачу для установки `htop`:

```yaml
- name: Установить htop
  yum:
    name: htop
    state: present
```

Запусти плейбук дважды. Убедись что второй раз — `changed=0`.

**Отправь в Slack:**
- Скриншот первого запуска (`changed > 0`)
- Скриншот второго запуска (`changed=0`) — идемпотентность в деле
