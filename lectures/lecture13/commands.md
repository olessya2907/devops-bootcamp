# Лекция 13 — Ansible: шпаргалка

> **Windows:** все команды выполняются в терминале **WSL2** (Ubuntu).
> SSH ключ доступен в WSL2 по пути `/mnt/c/Users/ВАШ_USERNAME/.ssh/key.pem`

---

## Установка Ansible

```bash
# Mac
brew install ansible

# WSL2 / Linux (Ubuntu)
sudo apt update && sudo apt install ansible -y

# Проверить
ansible --version
```

---

## Настройка проекта

```bash
cd devops-bootcamp/ansible

# Создать inventory из примера
cp inventory.ini.example inventory.ini

# Отредактировать: вставить реальный IP и путь к ключу
# [web]
# 54.x.x.x   ansible_ssh_private_key_file=~/.ssh/key.pem
```

---

## Основные команды

```bash
# Проверить что Ansible видит серверы
ansible all --list-hosts

# Проверить SSH соединение
ansible all -m ping
# → SUCCESS => {"ping": "pong"}

# Проверить синтаксис плейбука
ansible-playbook playbook.yml --syntax-check

# Запустить плейбук
ansible-playbook playbook.yml

# Посмотреть что изменится (без применения)
ansible-playbook playbook.yml --check

# Передать переменную
ansible-playbook playbook.yml -e "server_env=dev"
```

---

## Статусы задач

```
ok       — задача выполнена, ничего не менялось
changed  — задача выполнена, сервер изменился
failed   — ошибка (плейбук остановился)
skipped  — задача пропущена (условие when:)
```

---

## Ad-hoc команды

```bash
# Выполнить команду на сервере без плейбука
ansible all -m command -a "df -h"
ansible all -m command -a "systemctl status nginx"

# Посмотреть всё о сервере (CPU, RAM, ОС, IP...)
ansible all -m setup | head -80
```

---

## ✅ Самопроверка

```bash
# 1. Ansible установлен
ansible --version

# 2. Серверы в inventory
ansible all --list-hosts

# 3. SSH работает
ansible all -m ping
# → SUCCESS

# 4. Плейбук прошёл
ansible-playbook playbook.yml
# → failed=0

# 5. Nginx отвечает
curl http://EC2_IP
# → страница DevOps Bootcamp
```

---

## Почитать

| Тема | Ссылка |
|------|--------|
| Ansible — официальная документация | https://docs.ansible.com |
| Все встроенные модули (yum, service, copy...) | https://docs.ansible.com/ansible/latest/collections/ansible/builtin |
| Ansible Best Practices | https://docs.ansible.com/ansible/latest/tips_tricks/ansible_tips_tricks.html |
| Как работает inventory | https://docs.ansible.com/ansible/latest/inventory_guide/intro_inventory.html |
| YAML синтаксис | https://yaml.org/spec/1.2.2 |
