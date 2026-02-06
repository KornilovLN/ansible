# 🔧 Как улучшить конфигурацию

    Добавьте явное указание Python в inventory
    Отредактируйте файл inventory/hosts:
```ini

[proxmox_home]
vm-debian ansible_host=192.168.0.243
userver ansible_host=192.168.0.166

[proxmox_home:vars]
ansible_user=starmark
ansible_ssh_pass="!18leon28"
# Явно укажите интерпретатор Python для всей группы
ansible_python_interpreter=/usr/bin/python3
```

Это уберёт предупреждение и зафиксирует путь.

Создайте ansible.cfg для удобства (в корне ansible_prj/)
```ini

[defaults]
inventory = ./inventory/hosts
host_key_checking = False
# Можно задать interpreter_python здесь глобально
# interpreter_python = /usr/bin/python3
```

После этого команды можно будет запускать без -i:
```bash

ansible userver -m ping
```

## 🚀 Следующие шаги: проверка и первый простой плейбук

Теперь, когда соединение работает, можно проверить сбор фактов (информации о системе) и выполнить первую реальную задачу.

    Собрать информацию о системе (факты Ansible):
    bash

ansible userver -m setup -a "filter=ansible_distribution*"

Вы увидите точную версию ОС на userver.

Создайте и запустите простой тестовый плейбук, например, playbooks/first_steps.yml:
```yaml

---
- name: My First Playbook on Ubuntu Server
  hosts: userver
  become: yes  # Запускать задачи с привилегиями sudo

  tasks:
    - name: Ensure cron service is running
      ansible.builtin.service:
        name: cron
        state: started
        enabled: yes

    - name: Install a useful system tool
      ansible.builtin.apt:
        name: htop
        state: present
        update_cache: yes

    - name: Create a test directory for future projects
      ansible.builtin.file:
        path: /home/starmark/test_ansible
        state: directory
        mode: '0755'
```
Запустите его:
```bash

ansible-playbook playbooks/first_steps.yml
```

## 💡 Рекомендация по безопасности

Поскольку подключение работает, самое время перейти с пароля на SSH-ключи — это стандартная и более безопасная практика для автоматизации.

    Сгенерируйте ключ (если ещё нет):
```bash

ssh-keygen -t ed25519 -f ~/.ssh/ansible_proxmox -N ""
```

Скопируйте публичный ключ на сервер:
```bash

ssh-copy-id -i ~/.ssh/ansible_proxmox.pub starmark@192.168.0.166
# Введите ваш пароль !18leon28
```
Обновите inventory, указав путь к приватному ключу и убрав пароль:
```ini

[proxmox_home:vars]
ansible_user=starmark
ansible_ssh_private_key_file=~/.ssh/ansible_proxmox
# ansible_ssh_pass="!18leon28" # Закомментируйте или удалите эту строку
ansible_python_interpreter=/usr/bin/python3
```

Проверьте подключение по ключу:
```bash

ansible userver -m ping
```


