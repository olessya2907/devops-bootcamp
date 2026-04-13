# Лекция 14 — Ansible Deploy: шпаргалка

> **Windows:** все команды выполняются в терминале **WSL2** (Ubuntu).
> SSH ключ доступен в WSL2 по пути `/mnt/c/Users/ВАШ_USERNAME/.ssh/key.pem`

> **Важно для WSL2:** если `ansible.cfg` лежит на Windows-диске — добавь в терминале:
> ```bash
> export ANSIBLE_CONFIG=/mnt/c/Users/ВАШ_USERNAME/path/to/devops-bootcamp/ansible/ansible.cfg
> ```
> Или один раз добавь в `~/.bashrc` чтобы не вводить каждый раз.

---

## Структура папки ansible/

```
ansible/
  ansible.cfg                 ← настройки
  inventory.ini               ← серверы (в .gitignore)
  inventory.ini.example       ← шаблон
  playbook.yml                ← Л13: базовый nginx
  site.yml                    ← Л14: полный деплой
  roles/
    docker/tasks/main.yml     ← установка Docker
    app/tasks/main.yml        ← деплой приложения
    app/handlers/main.yml     ← перезапуск по событию
    app/templates/docker-compose.yml.j2   ← шаблон compose
    app/files/nginx.conf      ← конфиг nginx
    app/files/index.html      ← фронтенд
```

---

## Перед запуском

```bash
cd devops-bootcamp/ansible

# Создать inventory из примера (если ещё не создан)
cp inventory.ini.example inventory.ini

# Отредактировать: вставить реальный IP и путь к ключу
# [web]
# 54.x.x.x   ansible_ssh_private_key_file=~/.ssh/key.pem

# Открыть site.yml и вставить свой Docker Hub username:
#   docker_hub_user: "ВАШ_USERNAME"
```

---

## Деплой

```bash
# Проверить синтаксис
ansible-playbook site.yml --syntax-check

# Полный деплой (установка Docker + запуск приложения)
ansible-playbook site.yml

# Задеплоить конкретную версию образа
ansible-playbook site.yml -e "app_version=7"

# Только деплой (Docker уже установлен)
ansible-playbook site.yml --tags deploy

# Dry run — что изменится, ничего не применять
ansible-playbook site.yml --check
```

---

## Обновление и откат

```bash
# Jenkins собрал образ :8 — деплоим
ansible-playbook site.yml -e "app_version=8"

# Откат на предыдущую версию
ansible-playbook site.yml -e "app_version=7"
```

`-e` имеет наивысший приоритет — перекрывает всё что написано в `vars:` плейбука.

---

## Переменные

```bash
# Передать одну переменную
ansible-playbook site.yml -e "app_version=7"

# Передать несколько
ansible-playbook site.yml -e "app_version=7" -e "docker_hub_user=myname"
```

Приоритет (от высшего к низшему):
```
1. -e "var=value"           ← командная строка (всегда побеждает)
2. vars: в плейбуке
3. group_vars/all.yml
4. group_vars/ИМЯ_ГРУППЫ.yml
5. host_vars/IP.yml
6. defaults/ в роли         ← значения по умолчанию (проигрывает всем)
```

---

## Установка и удаление пакетов

```yaml
# Установить пакет
- name: Установить htop
  dnf:
    name: htop
    state: present

# Удалить пакет
- name: Удалить htop
  dnf:
    name: htop
    state: absent
```

```bash
# Применить изменения
ansible-playbook site.yml
```

---

## Проверка на сервере

```bash
# Контейнеры запущены
ansible all -m shell -a "docker ps" --become

# Логи backend
ansible all -m shell -a "docker logs devops-bootcamp-backend-1" --become

# Свободное место на диске
ansible all -m command -a "df -h"
```

---

## ✅ Самопроверка

```bash
# 1. Деплой прошёл без ошибок
ansible-playbook site.yml
# → failed=0

# 2. Приложение отвечает
curl http://EC2_IP
curl http://EC2_IP/api/hits
# → {"hits": 1}
curl http://EC2_IP/health
# → {"status": "ok"}

# 3. Контейнеры работают
ansible all -m shell -a "docker ps" --become
# → видим backend, nginx, redis
```

---

## Почитать

| Тема | Ссылка |
|------|--------|
| Ansible Roles — документация | https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse_roles.html |
| Ansible переменные и приоритет | https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html |
| Handlers | https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_handlers.html |
| Jinja2 шаблоны в Ansible | https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_templating.html |
| Все встроенные модули | https://docs.ansible.com/ansible/latest/collections/ansible/builtin |
