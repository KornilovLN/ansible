# Отличный детальный плейбук!

## 📊 Анализ и рекомендации
Что уже хорошо:

    Полная установка Docker с репозитория

    Широкая подборка системных утилит

    Создание Python venv с полезными пакетами

### Что нужно исправить/улучшить:
Проблема	Решение	Почему важно
Нет проверки дистрибутива для Docker repo	Использовать ansible_distribution_release	Ubuntu 24.04 имеет кодовое имя, которое нужно подставить
Устаревший метод apt_key	Использовать apt_key с keyring или загружать через get_url	Старый метод deprecated в Ansible
Прямой download Weave без проверки	Добавить проверку существования	Идемпотентность - не качать если уже есть
Неименованные задачи	Добавить name к каждой задаче	Лучшая читаемость и отладка
Нет обработки ошибок	Добавить ignore_errors и failed_when	Плейбук не должен падать на мелочах

### 🔧 Улучшенная версия плейбука

Вот совмещенный и улучшенный вариант, который включает мои рекомендации по безопасности и структуре:
```yaml

---
- name: Install essential software and Docker
  hosts: u_server24
  become: yes
  vars:
    docker_ce_version: "5:27.1.1-1~ubuntu.24.04~noble"
    weave_version: "2.8.1"

  tasks:
    # 1. Обновление системы
    - name: Update apt cache and upgrade system
      apt:
        update_cache: yes
        upgrade: dist
        cache_valid_time: 3600

    # 2. Подготовка к установке Docker (безопасный метод)
    - name: Install Docker prerequisites
      apt:
        name:
          - ca-certificates
          - curl
          - gnupg
          - lsb-release
        state: present

    - name: Create Docker keyring directory
      file:
        path: /etc/apt/keyrings
        state: directory
        mode: '0755'

    - name: Download Docker GPG key
      get_url:
        url: "https://download.docker.com/linux/ubuntu/gpg"
        dest: "/etc/apt/keyrings/docker.asc"
        mode: '0644'
      register: docker_key
      until: docker_key is succeeded
      retries: 3
      delay: 5

    - name: Add Docker repository (совместимый с Ubuntu 24.04)
      apt_repository:
        repo: "deb [arch={{ ansible_architecture }} signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu {{ ansible_distribution_release }} stable"
        state: present
        update_cache: yes
      when: ansible_distribution == "Ubuntu"

    # 3. Установка системных утилит (оптимизированный список)
    - name: Install system utilities (core)
      apt:
        name: "{{ item }}"
        state: present
      loop:
        - htop
        - mc
        - neovim
        - net-tools
        - git
        - tree
        - ncdu
        - tcpdump
        - zip
        - unzip
        - tmux
        - rsync
      register: core_utils_install

    - name: Install text processing tools
      apt:
        name:
          - jq
          - yq
          - bc
          - moreutils
          - ripgrep
          - fd-find
        state: present

    # 4. Установка Docker и зависимостей
    - name: Install Docker Engine
      apt:
        name:
          - docker-ce={{ docker_ce_version }}
          - docker-ce-cli={{ docker_ce_version }}
          - containerd.io
          - docker-buildx-plugin
          - docker-compose-plugin
        state: present
        update_cache: yes
      notify:
        - Enable docker service
        - Add user to docker group

    # 5. Установка Weave Net с проверкой
    - name: Check if Weave already exists
      stat:
        path: /usr/local/bin/weave
      register: weave_exists

    - name: Download Weave Net
      get_url:
        url: "https://github.com/weaveworks/weave/releases/download/v{{ weave_version }}/weave"
        dest: /usr/local/bin/weave
        mode: '0755'
        force: no  # Не перезаписывать если уже существует
      when: not weave_exists.stat.exists

    - name: Verify Weave installation
      command: /usr/local/bin/weave version
      register: weave_version_check
      changed_when: false
      ignore_errors: yes

    # 6. Настройка Python окружения
    - name: Ensure Python3 and venv are installed
      apt:
        name:
          - python3
          - python3-pip
          - python3-venv
          - python3-dev
          - build-essential
          - libssl-dev
          - libffi-dev
        state: present

    - name: Create virtual environment if not exists
      file:
        path: /opt/venv
        state: directory
      changed_when: false  # Только проверка, не создание

    - name: Create/update Python virtual environment
      pip:
        requirements: /tmp/requirements.txt
        virtualenv: /opt/venv
        virtualenv_command: python3 -m venv
      environment:
        VIRTUAL_ENV: /opt/venv

    - name: Create requirements file
      copy:
        dest: /tmp/requirements.txt
        content: |
          numpy>=1.24.0
          pandas>=2.0.0
          flask>=3.0.0
          sqlalchemy>=2.0.0
          requests>=2.31.0
          pytest>=7.4.0
          python-dotenv>=1.0.0
          docker>=6.1.0
          ansible>=8.0.0
        mode: '0644'

    # 7. Установка инструментов мониторинга (опционально)
    - name: Install monitoring tools
      apt:
        name:
          - prometheus-node-exporter
          - htop
          - iotop
          - iftop
          - nethogs
        state: present
      when: false  # Отключено по умолчанию, поменяйте на true если нужно

    # 8. Финальная проверка и вывод
    - name: Display Docker version
      command: docker --version
      register: docker_version
      changed_when: false

    - name: Display Python packages in venv
      command: /opt/venv/bin/pip list
      register: pip_list
      changed_when: false

    - name: Show installation summary
      debug:
        msg: |
          === УСТАНОВКА ЗАВЕРШЕНА ===
          Docker: {{ docker_version.stdout }}
          Weave: {{ weave_version_check.stdout_lines[0] if weave_version_check is succeeded else 'Не установлен' }}
          Python venv: /opt/venv ({{ pip_list.stdout_lines | length }} пакетов)
          Core utils: {{ core_utils_install.results | selectattr('changed') | list | length }} новых установок

  handlers:
    - name: Enable docker service
      systemd:
        name: docker
        state: started
        enabled: yes
        daemon_reload: yes

    - name: Add user to docker group
      user:
        name: "{{ ansible_user }}"
        groups: docker
        append: yes
      notify:
        - Reload user session

    - name: Reload user session
      # Для применения изменений группы docker без перезагрузки
      shell: "newgrp docker"
      changed_when: false
```
### 🚀 Как запустить и что проверить

    Создайте файл playbooks/deploy_server.yml с кодом выше

    Запустите с проверкой (dry-run):

```bash

ansible-playbook playbooks/deploy_server.yml --check --diff
```
    Выполните:

```bash

ansible-playbook playbooks/deploy_server.yml
```
    Проверьте результаты:

```bash

# Проверить Docker
ansible u_server24 -m command -a "docker ps"

# Проверить Python venv
ansible u_server24 -m command -a "/opt/venv/bin/python --version"

# Проверить установленные утилиты
ansible u_server24 -m command -a "which tmux htop ncdu"
```
## 📝 Ключевые улучшения в этой версии:

    Идемпотентность - каждая задача проверяет состояние перед изменением

    Безопасность - правильная работа с GPG ключами через keyring

    Переменные - версии вынесены в vars для легкого обновления

    Обработчики - автоматический запуск Docker и добавление пользователя в группу

    Уведомления - понятный финальный отчет об установке

    Обработка ошибок - retries, until, ignore_errors где нужно

Хотите добавить установку каких-то специфичных инструментов или настроить что-то дополнительно? Например, настройку firewall, SSH hardening, или установку мониторинга?
