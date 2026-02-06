# Администрирование Ubuntu 24.04 LTS через Ansible

## 🛠️ Поддержка языков и сред в Ubuntu 24.04

Ubuntu 24.04 имеет отличную поддержку современных технологий:
### ✅ Готовы к использованию (в официальных репозиториях):

    Go: Версия 1.21+ (можно установить golang-go или golang-1.22)

    Python: Python 3.12 по умолчанию (Ansible работает на Python)

    Java: OpenJDK 21 (именно то, что вы упомянули)

    Node.js: Доступен через репозитории NodeSource

    Rust: rustc 1.75 (как и указано)

    .NET: .NET 8.0 SDK и runtime

    PHP: PHP 8.3

    Ruby: Ruby 3.2

    Docker: Поддержка через официальные репозитории Docker

## 📦 Пакеты для разработки и администрирования:
```text

# Установка базовых инструментов разработки
sudo apt install build-essential git curl wget

# Установка языков
sudo apt install golang-go openjdk-21-jdk python3-pip
sudo apt install rustc cargo dotnet-sdk-8.0
sudo apt install nodejs npm php8.3-cli ruby

# Контейнеризация
sudo apt install docker.io docker-compose-v2 podman
```

## 🚀 Ansible для Ubuntu 24.04: лучшие практики
### 1. Улучшите ваш inventory файл:
```ini

[proxmox_home]
vm-debian ansible_host=192.168.0.243
userver ansible_host=192.168.0.166

[proxmox_home:vars]
ansible_user=starmark
ansible_ssh_pass="!18leon28"
ansible_ssh_common_args='-o StrictHostKeyChecking=no'
ansible_python_interpreter=/usr/bin/python3
```
### 2. Базовый playbook для настройки (initial_setup.yml):
```yaml

---
- name: Initial setup for Ubuntu 24.04 servers
  hosts: userver  # или proxmox_home для всех
  become: yes
  
  tasks:
    - name: Update apt cache
      apt:
        update_cache: yes
        cache_valid_time: 3600
    
    - name: Install essential packages
      apt:
        name:
          - build-essential
          - git
          - curl
          - wget
          - htop
          - ncdu
          - net-tools
          - ufw
          - fail2ban
        state: present
    
    - name: Install development languages
      apt:
        name:
          - golang-go
          - python3-pip
          - python3-venv
          - openjdk-21-jdk
          - rustc
          - cargo
          - dotnet-sdk-8.0
        state: present
    
    - name: Configure firewall (allow SSH)
      ufw:
        rule: allow
        port: '22'
        proto: tcp
      notify: Enable UFW
    
    - name: Create developer user
      user:
        name: developer
        groups: sudo
        shell: /bin/bash
        create_home: yes
        password: "{{ 'your_encrypted_password' | password_hash('sha512') }}"
    
    - name: Set up SSH key for Ansible user
      authorized_key:
        user: starmark
        key: "{{ lookup('file', '~/.ssh/id_rsa.pub') }}"
        state: present
  
  handlers:
    - name: Enable UFW
      ufw:
        state: enabled
        policy: deny
```
### 3. Специализированный playbook для IDE/редакторов:
```yaml

---
- name: Install development environments
  hosts: userver
  become: yes
  
  tasks:
    - name: Install VS Code (через snap)
      snap:
        name: code
        classic: yes
        state: present
    
    - name: Install IntelliJ IDEA Community
      snap:
        name: intellij-idea-community
        classic: yes
        state: present
    
    - name: Install Docker
      include_role:
        name: geerlingguy.docker
    
    - name: Install Kubernetes tools
      apt:
        name:
          - kubectl
          - helm
          - minikube
        state: present
    
    - name: Set up Python virtualenv for projects
      pip:
        name:
          - virtualenv
          - pipenv
          - poetry
        executable: pip3
```

## 🔧 Особенности Ubuntu 24.04 для Ansible:
### Изменения в системе:

    Python 3.12 по умолчанию - Ansible работает отлично

    systemd 255 - улучшенная производительность и логирование

    Netplan для сетей - используйте модуль community.general.netplan

    Snap по умолчанию - для установки некоторых приложений (VS Code, IDEA)

###Рекомендуемые роли Ansible:
```bash

# Установите роли из Ansible Galaxy
ansible-galaxy role install geerlingguy.docker
ansible-galaxy role install geerlingguy.kubernetes
ansible-galaxy role install geerlingguy.nodejs
ansible-galaxy collection install community.general
```
Проверка соединения:
```bash

# Проверьте, что Ansible видит вашу VM
ansible userver -i inventory -m ping

# Проверьте информацию о системе
ansible userver -i inventory -m setup -a "filter=ansible_distribution*"
```

### ⚠️ Важные моменты:

    Безопасность паролей: Лучше использовать SSH-ключи вместо паролей:

```bash

# Генерация ключа
ssh-keygen -t ed25519

# Копирование на сервер
ssh-copy-id starmark@192.168.0.166
```
    Vault для секретов: Шифруйте пароли в Ansible:

```bash

ansible-vault create secrets.yml
```
    AppArmor в Ubuntu 24.04: Может блокировать некоторые приложения. При необходимости отключайте для тестов:

```bash

sudo aa-complain /usr/bin/docker
```

### Что примерно предполагается:
    Веб-фреймворки (Django, Rails, Express.js)

    БД (PostgreSQL, MySQL, Redis)

    Мониторинг (Prometheus, Grafana)

    CI/CD (Jenkins, GitLab Runner)

## Сервер ubuntu без GUI.

    Поэтому работать только в консоли и в программах TUI интерфейса. Поддержка docker необходима.
    Какие ide с tui интерфейсом можно применять для работы в Bash, Forth, Assembler, C/C++, RUST, GO, Python, C#, NodeJS,... если понадобится работать удаленно ?


### Для работы исключительно в консоли Ubuntu Server (без GUI) подходят несколько типов инструментов.

| Категория / Язык	| Редакторы (TUI)	| IDE (TUI / Тонкий клиент)	| Облачные IDE (Web)
|---|---|---|--- 
| Универсальные	| Vim, Neovim (с плагинами rust-analyzer, gopls, coc.nvim), Emacs, Helix	| JetBrains Fleet (удалённый режим)
| code-server (VS Code в браузере)
| C/C++	| Vim/Neovim + clangd, GNU Global, Cmake	| CLion (удалённый режим) | code-server с расширениями
| Go	| Vim/Neovim + gopls, delve	| GoLand (удалённый режим) | code-server с расширением Go
| Rust	| Vim/Neovim + rust-analyzer	| CLion с плагином Rust, Fleet | code-server с rust-analyzer
| Python	| Vim/Neovim + pyright/jedi	| PyCharm Professional (удалённый режим) | code-server с расширением Python
| C# (.NET)	| OmniSharp для Vim/Neovim	| Rider (удалённый режим)	| code-server с расширением C#
| Node.js/JS	| Vim/Neovim + tsserver	| WebStorm (удалённый режим) | code-server с Node.js-расширениями
| Assembler	| Vim/Neovim, nasm/gas (с синтаксисом)	| (Специализированные IDE редки)	| (Менее удобно)

## 🔧 Какой инструмент выбрать для сценария

    Для работы прямо в SSH-консоли сервера: Установите и настройте Vim или Neovim.
    Это стандартные инструменты, идеальные для прямого редактирования файлов на сервере.

    Для полноценной разработки с автодополнением и отладкой: Настройте code-server (VS Code в браузере)
    или используйте удалённый режим IDE JetBrains через JetBrains Gateway

    . Это позволит работать с мощной IDE, в то время как все вычисления будут выполняться на сервере.

    Для быстрой правки конфигураций или скриптов: Встроенных редакторов nano или micro часто бывает достаточно.

## 🚀 Быстрый старт с Vim/Neovim

Если вы остановились на классическом консольном редакторе:

    Установка: sudo apt install neovim (или vim).

    Базовая настройка: Создайте файл конфигурации ~/.config/nvim/init.vim (для Neovim).

    Установка менеджера плагинов: Рекомендуется vim-plug. Добавьте в конфиг:
```vim

call plug#begin('~/.vim/plugged')
Plug 'neoclide/coc.nvim', {'branch': 'release'} " Ядро автодополнения
Plug 'rust-lang/rust.vim' " Для Rust
call plug#end()
```

    Настройка LSP: После установки плагинов в init.vim настройте coc.nvim для поддержки нужных языков (например, :CocInstall coc-rust-analyzer coc-go).

## 🖥️ Быстрый старт с Code-Server (VS Code в браузере)

Это отличный компромисс между функциональностью и простотой удалённого доступа

:

    Установка: Скачайте последнюю версию с GitHub.

    Запуск: code-server --bind-addr 0.0.0.0:8080 --auth password.

    Доступ: Откройте в браузере http://<ip-адрес-вашего-сервера>:8080, введите пароль.

    Установка расширений: В интерфейсе Code-Server перейдите в Marketplace и установите расширения для нужных языков (Go, Rust, Python и т.д.).

🔍 Дальнейшие шаги

    Для Docker: Все перечисленные инструменты (Vim, code-server, удалённые IDE) прекрасно работают внутри контейнеров. Code-server и JetBrains Gateway можно развернуть как Docker-образы.

    Исследование: Посмотрите обзоры на Helix (современный редактор на Rust) и Lapce.

