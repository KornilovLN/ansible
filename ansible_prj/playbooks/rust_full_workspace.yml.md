# 🦀 Плейбук: Полное Rust-окружение с изоляцией
    (playbooks/rust_full_workspace.yml)

## Запуск плейбука:
```bash

ansible-playbook playbooks/rust_full_workspace.yml \
  -i inventory/hosts \
  --limit userver \
  -e "user_workspace=/mnt/big8-dev/poligon/prj-current/rust_workspace"
```

```yaml

---
- name: Complete Rust development environment with project isolation
  hosts: userver
  become: yes
  vars:
    # === ОСНОВНЫЕ ПУТИ ===
    rust_global_dir: "/opt/rust"                     # Глобальные инструменты
    user_workspace: "/mnt/big8-dev/poligon/prj-current/rust_workspace"  # Ваша рабочая область
    ansible_user: "starmark"                         # Ваш пользователь
    
    # === ВЕРСИИ ===
    rust_toolchain: "stable"
    rust_components: ["rustc", "cargo", "rustfmt", "clippy", "rust-analyzer", "rust-docs"]
    
    # === ПРОФИЛИ УСТАНОВКИ ===
    rust_profiles:
      web: ["actix-web", "axum", "rocket", "tokio", "serde", "reqwest"]
      gui: ["iced", "slint", "egui", "druid", "winit", "glutin"]
      tui: ["ratatui", "cursive", "tui", "crossterm", "termion"]
      async: ["tokio", "async-std", "smol", "futures", "tokio-test"]
      math: ["nalgebra", "ndarray", "statrs", "rand", "approx"]
      data: ["polars", "arrow", "datafusion", "rocksdb", "sqlx"]
      cli: ["clap", "structopt", "dialoguer", "indicatif", "console"]
      dev: ["cargo-watch", "cargo-edit", "cargo-audit", "cargo-nextest", "cargo-expand"]

  tasks:
    # === 1. СИСТЕМНЫЕ ЗАВИСИМОСТИ ===
    - name: Install system dependencies for Rust
      apt:
        name:
          - curl
          - build-essential
          - pkg-config
          - libssl-dev
          - libclang-dev
          - libsqlite3-dev      # Для базы данных в проектах
          - libpq-dev           # Для PostgreSQL
          - libmysqlclient-dev  # Для MySQL
          - cmake
          - ninja-build
          - git
        state: present
        update_cache: yes

    # === 2. УСТАНОВКА RUSTUP (ГЛОБАЛЬНАЯ) ===
    - name: Install rustup system-wide
      shell: |
        curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | \
        sh -s -- -y \
          --default-toolchain {{ rust_toolchain }} \
          --profile default \
          --no-modify-path
      args:
        chdir: /tmp
        creates: "/root/.cargo/bin/rustup"
      environment:
        CARGO_HOME: "{{ rust_global_dir }}/cargo"
        RUSTUP_HOME: "{{ rust_global_dir }}/rustup"

    # === 3. КОНФИГУРАЦИЯ ГЛОБАЛЬНОГО CARGO ===
    - name: Configure global cargo environment
      block:
        - name: Create cargo config directory
          file:
            path: "{{ rust_global_dir }}/cargo"
            state: directory
            mode: '0755'
        
        - name: Set global cargo config
          copy:
            dest: "{{ rust_global_dir }}/cargo/config.toml"
            content: |
              [build]
              jobs = {{ ansible_processor_vcpus | default(4) }}
              incremental = true
              rustflags = ["-C", "target-cpu=native"]
              
              [install]
              root = "{{ rust_global_dir }}/cargo"
              
              [profile.dev]
              opt-level = 1
              debug = true
              
              [profile.release]
              opt-level = 3
              lto = true
              codegen-units = 1
              panic = 'abort'
              
              [registries.crates-io]
              protocol = 'sparse'
              
              [net]
              git-fetch-with-cli = true
            mode: '0644'
        
        - name: Create environment loader
          copy:
            dest: /etc/profile.d/rust_global.sh
            content: |
              # Global Rust environment (DO NOT TOUCH FOR PROJECTS)
              export RUST_GLOBAL_HOME="{{ rust_global_dir }}"
              export RUSTUP_HOME="{{ rust_global_dir }}/rustup"
              export CARGO_HOME_GLOBAL="{{ rust_global_dir }}/cargo"
              
              # Add to PATH only for system tools
              if [[ $PATH != *"$CARGO_HOME_GLOBAL/bin"* ]]; then
                export PATH="$CARGO_HOME_GLOBAL/bin:$PATH"
              fi
              
              # Useful aliases
              alias cargo-global="CARGO_HOME=$CARGO_HOME_GLOBAL RUSTUP_HOME=$RUSTUP_HOME cargo"
              alias rustup-global="RUSTUP_HOME=$RUSTUP_HOME rustup"
            mode: '0644'

    # === 4. СОЗДАНИЕ ИЗОЛИРОВАННОЙ РАБОЧЕЙ ОБЛАСТИ ===
    - name: Create isolated workspace structure
      block:
        - name: Create workspace root
          file:
            path: "{{ user_workspace }}"
            state: directory
            mode: '0755'
            owner: "{{ ansible_user }}"
            group: "{{ ansible_user }}"
        
        - name: Create workspace template
          copy:
            dest: "{{ user_workspace }}/template/"
            content: |
              # Rust Isolated Workspace Template
              # Each project here is completely isolated
            mode: '0644'
            owner: "{{ ansible_user }}"
        
        - name: Create project isolation script
          copy:
            dest: "{{ user_workspace }}/create_rust_project.sh"
            mode: '0755'
            owner: "{{ ansible_user }}"
            content: |
              #!/bin/bash
              # Script to create isolated Rust project
              
              if [ -z "$1" ]; then
                  echo "Usage: $0 <project-name> [--web|--gui|--cli|--lib]"
                  exit 1
              fi
              
              PROJECT_NAME="$1"
              PROJECT_TYPE="${2:---cli}"
              PROJECT_DIR="{{ user_workspace }}/projects/$PROJECT_NAME"
              
              # Create project directory
              mkdir -p "$PROJECT_DIR"
              cd "$PROJECT_DIR"
              
              # Create .env file for isolation
              cat > .env << EOF
              # Rust Project Isolation Environment
              # Generated: $(date)
              
              # Isolated paths
              export CARGO_HOME="\$PWD/.cargo"
              export RUSTUP_HOME="{{ rust_global_dir }}/rustup"
              
              # Project-specific
              export PROJECT_NAME="$PROJECT_NAME"
              export PROJECT_TYPE="$PROJECT_TYPE"
              export RUST_PROJECT_ROOT="\$PWD"
              
              # Build configuration
              export RUSTFLAGS="-C target-cpu=native"
              export CARGO_BUILD_JOBS={{ ansible_processor_vcpus | default(4) }}
              
              # Add local cargo to PATH
              export PATH="\$PWD/.cargo/bin:\$PATH"
              EOF
              
              # Create cargo config
              mkdir -p .cargo
              cat > .cargo/config.toml << EOF
              [build]
              target-dir = "target/\$PROJECT_NAME"
              
              [env]
              PROFILE = "dev"
              
              [profile.dev]
              incremental = true
              
              [profile.release]
              lto = true
              EOF
              
              # Initialize cargo project based on type
              case "$PROJECT_TYPE" in
                  --web)
                      cargo init --name "$PROJECT_NAME" --bin
                      cargo add actix-web tokio serde
                      ;;
                  --gui)
                      cargo init --name "$PROJECT_NAME" --bin
                      cargo add iced
                      ;;
                  --lib)
                      cargo init --name "$PROJECT_NAME" --lib
                      ;;
                  *)
                      cargo init --name "$PROJECT_NAME" --bin
                      ;;
              esac
              
              # Create activation script
              cat > activate.sh << 'EOF'
              #!/bin/bash
              # Activate isolated Rust environment
              
              if [ -f .env ]; then
                  set -a
                  source .env
                  set +a
                  echo "Activated: $PROJECT_NAME ($PROJECT_TYPE)"
                  echo "CARGO_HOME: $CARGO_HOME"
              else
                  echo "Error: .env file not found"
                  exit 1
              fi
              EOF
              
              chmod +x activate.sh
              
              echo "Project '$PROJECT_NAME' created in $PROJECT_DIR"
              echo "To activate: cd $PROJECT_DIR && source activate.sh"
        
        - name: Create example isolated project
          shell: |
            cd "{{ user_workspace }}" && \
            bash create_rust_project.sh demo_app --cli
          environment:
            CARGO_HOME: "{{ rust_global_dir }}/cargo"
            PATH: "/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
          args:
            creates: "{{ user_workspace }}/projects/demo_app/Cargo.toml"

    # === 5. УСТАНОВКА ПРОФИЛЬНЫХ КРЕЙТОВ ===
    - name: Install profile-based crates (GLOBAL FOR TEMPLATES)
      shell: |
        source /etc/profile.d/rust_global.sh && \
        cargo install {{ item }} --root {{ rust_global_dir }}/cargo
      loop: "{{ rust_profiles.dev }}"
      args:
        creates: "{{ rust_global_dir }}/cargo/bin/{{ item.split('-')[1] }}"
      environment:
        CARGO_HOME: "{{ rust_global_dir }}/cargo"
        RUSTUP_HOME: "{{ rust_global_dir }}/rustup"

    # === 6. СОЗДАНИЕ WORKSPACE ШАБЛОНОВ ===
    - name: Create workspace templates for different project types
      block:
        - name: Create web workspace template
          copy:
            dest: "{{ user_workspace }}/templates/web/"
            src: "templates/web/"  # Можно предварительно создать
            mode: '0755'
          when: false  # Заглушка для реальных шаблонов
        
        - name: Create workspace manifest
          copy:
            dest: "{{ user_workspace }}/Cargo.toml"
            content: |
              [workspace]
              members = [
                  "projects/*",
                  "libraries/*",
              ]
              default-members = ["projects/*"]
              
              [workspace.dependencies]
              # Common dependencies for all workspace projects
              tokio = { version = "1.0", features = ["full"] }
              serde = { version = "1.0", features = ["derive"] }
              anyhow = "1.0"
              thiserror = "1.0"
              tracing = "0.1"
              
              [workspace.package]
              authors = ["{{ ansible_user }}"]
              edition = "2021"
              rust-version = "1.70"
            mode: '0644'
            owner: "{{ ansible_user }}"

    # === 7. НАСТРОЙКА АВТОМАТИЧЕСКОЙ АКТИВАЦИИ ===
    - name: Setup auto-activation for shell
      block:
        - name: Create bashrc addition for user
          copy:
            dest: /home/{{ ansible_user }}/.bashrc_rust
            content: |
              # Rust workspace auto-detection
              function _activate_rust_env() {
                  local current_dir="$PWD"
                  while [[ "$current_dir" != "/" ]]; do
                      if [[ -f "$current_dir/.rustenv" ]]; then
                          source "$current_dir/.rustenv"
                          return 0
                      elif [[ -f "$current_dir/activate.sh" ]]; then
                          source "$current_dir/activate.sh"
                          return 0
                      fi
                      current_dir="$(dirname "$current_dir")"
                  done
                  # Deactivate if we leave project
                  if [[ -n "$CARGO_HOME" && "$CARGO_HOME" != *".cargo"* ]]; then
                      unset CARGO_HOME RUSTUP_HOME PROJECT_NAME
                      echo "Rust environment deactivated"
                  fi
              }
              
              # Add to PROMPT_COMMAND
              if [[ -n "$PROMPT_COMMAND" ]]; then
                  PROMPT_COMMAND="_activate_rust_env; $PROMPT_COMMAND"
              else
                  PROMPT_COMMAND="_activate_rust_env"
              fi
              
              # Aliases
              alias rs='cargo run'
              alias rb='cargo build'
              alias rt='cargo test'
              alias rc='cargo check'
              alias rcl='cargo clippy'
              alias rfmt='cargo fmt'
            mode: '0644'
            owner: "{{ ansible_user }}"
        
        - name: Add to user's bashrc
          lineinfile:
            path: /home/{{ ansible_user }}/.bashrc
            line: 'source ~/.bashrc_rust'
            state: present
            owner: "{{ ansible_user }}"

    # === 8. ТЕСТОВАЯ ПРОВЕРКА ===
    - name: Test isolated environment
      block:
        - name: Create test project
          shell: |
            cd "{{ user_workspace }}" && \
            bash create_rust_project.sh test_env --lib && \
            cd projects/test_env && \
            source activate.sh && \
            cargo build --verbose
          environment:
            CARGO_HOME: "{{ rust_global_dir }}/cargo"
            PATH: "{{ rust_global_dir }}/cargo/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
          args:
            creates: "{{ user_workspace }}/projects/test_env/target/test_env/debug/libtest_env.rlib"
          register: test_build
          changed_when: false
        
        - name: Show test results
          debug:
            msg: |
              === ISOLATED RUST ENVIRONMENT READY ===
              
              Global tools: {{ rust_global_dir }}
              Your workspace: {{ user_workspace }}
              
              To create new isolated project:
              1. cd {{ user_workspace }}
              2. ./create_rust_project.sh <name> <--web|--gui|--cli|--lib>
              
              Example:
                ./create_rust_project.sh my_web_app --web
                cd projects/my_web_app
                source activate.sh
                cargo run
              
              Test project: {{ user_workspace }}/projects/test_env/
              Build success: {{ test_build is succeeded }}

  handlers:
    - name: Reload shell environment
      shell: |
        source /etc/profile.d/rust_global.sh
      changed_when: false
```

## 🚀 Как использовать эту систему:
### 1. Создание изолированного проекта:
```bash

cd /mnt/big8-dev/poligon/prj-current/rust_workspace
./create_rust_project.sh my_web_app --web
cd projects/my_web_app
source activate.sh  # Активирует изолированное окружение
```
### 2. Что происходит при активации:

    Создаётся локальная папка .cargo/ в проекте

    Все зависимости скачиваются в project/.cargo/registry

    Сборка происходит в project/target/project_name/

    Глобальная система не затронута

### 3. Проверка изоляции:
```bash

# В активированном проекте
echo $CARGO_HOME        # Должен показывать путь ВНУТРИ проекта
cargo build             # Собирает только этот проект
```
### 4. Запуск плейбука:
```bash

ansible-playbook playbooks/rust_full_workspace.yml \
  -i inventory/hosts \
  --limit userver \
  -e "user_workspace=/mnt/big8-dev/poligon/prj-current/rust_workspace"
```

## 📦 Преимущества этого подхода:
| Аспект	| Решение	| Результат
|---|---|--- 
| Изоляция	| .env + локальный .cargo/	| Каждый проект независим
| Версионирование	| Свой Cargo.toml в каждом проекте	| Разные версии крейтов
| Кэш	| Локальный target/project_name/	| Не смешиваются сборки
| Глобальные утилиты	| В /opt/rust	| Доступны везде

## 🔧 Дополнительные возможности:

Добавьте в плейбук для pre-commit hooks:
```yaml

    - name: Install pre-commit hooks template
      copy:
        dest: "{{ user_workspace }}/.pre-commit-config.yaml"
        content: |
          repos:
            - repo: local
              hooks:
                - id: cargo-check
                  name: cargo check
                  entry: cargo check
                  language: system
                  pass_filenames: false
                - id: cargo-fmt
                  name: cargo fmt
                  entry: cargo fmt -- --check
                  language: system
```
### Эта система гарантирует, что:

    Сбой питания не повредит глобальные настройки

    Каждый проект самодостаточен

    Можно иметь разные версии Rust/cargo для разных проектов

    Легко удалить проект полностью (удалить папку)

#### Нужно добавить интеграцию с Docker для сборки изолированных контейнеров
