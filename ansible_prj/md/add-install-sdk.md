# Установить SDK для RUST, GO, C/C++, NodeJS, NASM,...

    Специализированный плейбук для установки инструментов разработки:

## 📦 Плейбук для установки SDK и компиляторов (playbooks/install_dev_sdks.yml)
```yaml

---
- name: Install Development SDKs and Compilers
  hosts: userver  # или группа хостов
  become: yes
  vars:
    go_version: "1.22"  # Укажите нужную версию Go
    node_version: "20"   # Укажите нужную версию Node.js

  tasks:
    # 1. ОБНОВЛЕНИЕ СИСТЕМЫ
    - name: Update apt cache
      apt:
        update_cache: yes
        cache_valid_time: 3600

    # 2. RUST (официальный метод через rustup - рекомендовано)
    - name: Install curl for rustup
      apt:
        name: curl
        state: present

    - name: Download and run rustup installer
      shell: |
        curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y
        source "$HOME/.cargo/env"
      args:
        creates: /root/.cargo/bin/rustc  # Проверка, если уже установлен
      environment:
        CARGO_HOME: /root/.cargo
        RUSTUP_HOME: /root/.rustup

    - name: Add cargo to system PATH
      lineinfile:
        path: /etc/environment
        regexp: '^PATH=.*\.cargo/bin'
        line: 'PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/root/.cargo/bin"'
        state: present

    # 3. GO (официальные бинарники)
    - name: Install Go from official repository
      apt:
        name: golang-go
        state: present
        update_cache: yes

    - name: Install specific Go version via goenv (альтернатива)
      block:
        - name: Install git for goenv
          apt:
            name: git
            state: present
        - name: Clone goenv
          git:
            repo: https://github.com/syndbg/goenv.git
            dest: /root/.goenv
        - name: Install Go {{ go_version }} via goenv
          shell: |
            export GOENV_ROOT="/root/.goenv"
            export PATH="$GOENV_ROOT/bin:$PATH"
            eval "$(goenv init -)"
            goenv install {{ go_version }}
            goenv global {{ go_version }}
          args:
            creates: /root/.goenv/versions/{{ go_version }}
      when: false  # Отключено по умолчанию, поменяйте на true для goenv

    # 4. C/C++ TOOLCHAIN
    - name: Install C/C++ development tools
      apt:
        name:
          - build-essential
          - gcc
          - g++
          - gdb
          - cmake
          - make
          - autoconf
          - automake
          - libtool
          - pkg-config
          - clang
          - clang-format
          - clang-tidy
          - lldb
          - valgrind
        state: present

    # 5. Node.js и npm (через Nodesource)
    - name: Install Node.js from Nodesource repository
      block:
        - name: Add Nodesource GPG key
          apt_key:
            url: "https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key"
            state: present

        - name: Add Nodesource repository
          apt_repository:
            repo: "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_{{ node_version }}.x nodistro main"
            state: present
            filename: nodesource

        - name: Install Node.js and npm
          apt:
            name:
              - nodejs
              - npm
            state: present
            update_cache: yes

    # 6. NASM (ассемблер)
    - name: Install NASM and assembly tools
      apt:
        name:
          - nasm
          - yasm
          - fasm
          - binutils
          - gcc-multilib  # Для 32-битной компиляции
          - g++-multilib
        state: present

    # 7. ДОПОЛНИТЕЛЬНЫЕ ИНСТРУМЕНТЫ ДЛЯ РАЗРАБОТКИ
    - name: Install additional development tools
      apt:
        name:
          - git
          - subversion
          - mercurial
          - docker.io          # Если ещё не установлен
          - docker-compose-v2
          - python3-pip
          - python3-venv
          - virtualenv
          - libssl-dev
          - libffi-dev
          - zlib1g-dev
          - libreadline-dev
          - libsqlite3-dev
          - libbz2-dev
          - libncurses5-dev
          - libncursesw5-dev
          - libgdbm-dev
          - liblzma-dev
          - tk-dev
        state: present

    # 8. УСТАНОВКА МЕНЕДЖЕРОВ ПАКЕТОВ ДЛЯ ЯЗЫКОВ
    - name: Install language package managers
      apt:
        name:
          - cargo          # Rust (дополнительно)
          - pipx           # Python isolated apps
        state: present

    # 9. ПРОВЕРКА УСТАНОВКИ
    - name: Verify installations
      block:
        - name: Check Rust installation
          command: rustc --version
          register: rust_check
          changed_when: false

        - name: Check Go installation
          command: go version
          register: go_check
          changed_when: false

        - name: Check GCC installation
          command: gcc --version | head -1
          register: gcc_check
          changed_when: false

        - name: Check Node.js installation
          command: node --version
          register: node_check
          changed_when: false

        - name: Check NASM installation
          command: nasm --version
          register: nasm_check
          changed_when: false

        - name: Display installation summary
          debug:
            msg: |
              === DEVELOPMENT SDKs INSTALLED ===
              Rust: {{ rust_check.stdout }}
              Go: {{ go_check.stdout }}
              GCC: {{ gcc_check.stdout }}
              Node.js: {{ node_check.stdout }}
              NASM: {{ nasm_check.stdout }}
  handlers:
    - name: Update system alternatives
      command: update-alternatives --config editor
      changed_when: false

🚀 Как использовать:

    Создайте файл: playbooks/install_dev_sdks.yml

    Настройте переменные в начале файла (версии Go, Node.js)

    Запустите:

bash

# Проверка
ansible-playbook playbooks/install_dev_sdks.yml -i inventory/hosts --limit userver --check --diff

# Реальный запуск
ansible-playbook playbooks/install_dev_sdks.yml -i inventory/hosts --limit userver

📝 Особенности установки:
Язык/Инструмент	Метод установки	Альтернативы
Rust	Официальный rustup	apt install rustc (устаревшая версия)
Go	Пакет Ubuntu (golang-go)	Официальный бинарник или goenv
C/C++	build-essential + дополнительные инструменты	Минимальный набор: gcc, g++, make
Node.js	Репозиторий Nodesource	apt install nodejs (старая версия)
NASM	Пакет nasm из репозитория	Также yasm, fasm
🔧 Дополнительные плейбуки для конкретных нужд:
Для Docker-разработки:
yaml

    - name: Install Docker development tools
      apt:
        name:
          - docker.io
          - docker-compose-v2
          - dive          # Анализ Docker образов
          - ctop          # Мониторинг контейнеров
          - lazydocker    # TUI для Docker

Для веб-разработки:
yaml

    - name: Install web development tools
      apt:
        name:
          - nginx
          - certbot
          - postgresql-client
          - mysql-client
          - redis-tools

Для создания IDE-окружения:
yaml

    - name: Install development editors
      snap:
        name:
          - code
          - intellij-idea-community
        classic: yes
        state: present

💡 Советы:

    Разделите на отдельные плейбуки, если нужно выборочно устанавливать

    Добавьте теги для группировки задач:

yaml

  tags:
    - rust
    - go
    - cpp
    - nodejs
    - nasm

    Используйте роли для лучшей организации при росте проекта

Нужна ли установка дополнительных фреймворков или библиотек для конкретных языков? Например, React/Vue для Node.js или популярные крейты для Rust?
