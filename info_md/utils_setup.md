## Установка программного обеспечения на сервер
* 1. Создать файл плейбука:
``` 
---
- name: Install essential software
  hosts: u_server24
  become: yes
  tasks:
    - name: Update apt cache
      apt:
        update_cache: yes

    - name: Check and install system utilities
      apt:
        name: "{{ item }}"
        state: present
      register: install_result
      with_items:
        - htop
        - mc
        - vim
        - net-tools
        - curl
        - wget
        - git
        - tree
        - ncdu
        - iotop
        - nmap
        - tcpdump
        - iftop
        - zip
        - unzip
        - screen 
        - tmux

    - name: Show installation results
      debug:
        msg: "{{ item.item }} is {{ 'already installed' if not item.changed else 'newly installed' }}"
      loop: "{{ install_result.results }}"       
```
* 2. Запустить плейбук:
```
ansible-playbook ansible_prj/playbooks/install_utils.yml -i ansible_prj/inventory/hosts --ask-become-pass
```