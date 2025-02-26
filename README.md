# Ansible

## https://docs.ansible.com/ansible/latest/index.html
## https://www.8host.com/blog/kak-rabotat-s-ansible-prostaya-i-udobnaya-shpargalka/

## Основные понятия
* **Control Machine (или Node)**: ведущая система, в которой установлен Ansible и откуда он может подключаться к нодам и выполнять на них команды.
* **Нода**: сервер, управляемый Ansible.
* **Inventory**: файл содержит инфо о серверах, которыми управляет Ansible
  * обычно находится в /etc/ansible/hosts
* **Playbook**: файл, с серией задач, которые нужно выполнить на удаленном сервере.
* **Роль**: коллекция плейбуков и других файлов, которые имеют отношение к цели
  * например, к установке веб-сервера
* **Play**: полный набор инструкций Ansible.
  * В play может быть несколько:
    * плейбуков и
    * ролей, включенных в один плейбук, который служит точкой входа.

## Ранее созданный venv перенести во внешнюю директорию:
* **First, move the existing venv up one level:**
```
mv ansible_prj/venv ./venv
```

* **Create a new directory for all Ansible virtual environments:**
``` 
mkdir ansible_venvs
```

* **Move the venv into the new directory:**
``` 
mv venv ansible_venvs/ansible_common_venv
```

* **Activate the moved venv using the new path:**
``` 
source ansible_venvs/ansible_common_venv/bin/activate
```

## Надо инсталлировать с офмциального сайта:
* https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html
* pip install --index-url https://pypi.org/simple ansible
* python3 -m venv ansible_venvs/ansible_common_venv_new
* source ansible_venvs/ansible_common_venv_new/bin/activate

## Как подключиться к удаленной машине:
### Условие: В invertory/hosts есть строка:
* vpn.vm.NPPTest ansible_ssh_host=192.168.88.104 ansible_user=starmark
* 
### Команда:
* С указанием запроса пароля со стороны удаленной VM:
```
ansible vpn.vm.NPPTest -m ping -i ansible_prj/inventory/hosts --ask-pass
```
* С указанием пароля в файле ansible_prj/inventory/hosts:
```
vpn.vm.NPPTest ansible_host=192.168.88.104 ansible_user=starmark nsible_ssh_pass="password"
```

## Настройте
* SSH-ключи между машинами:
```
ssh-keygen -t rsa
```
* Скопируйте публичный ключ на целевой хост:
```
ssh-copy-id starmark@192.168.88.104
```
* Проверьте права на домашнюю директорию и SSH-ключи:
```
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_rsa
chmod 644 ~/.ssh/id_rsa.pub
```

## Примеры запросов из (ansible_common_venv_new) leon@starmark-GA-890GPA-UD3H:/mnt/poligon/ansible$:
* Показать список в директории / :
```
ansible u_server24 -a "ls /" -i ansible_prj/inventory/hosts
ansible u_server24 -a "ls -al /" -i ansible_prj/inventory/hosts
ansible u_server24 -m shell -a "ls -al /" -i ansible_prj/inventory/hosts

```
* Создать алиас на удаленной машине и перезагрузить .bashrc:
```
ansible u_server24 -m lineinfile -a "path=/home/starmark/.bashrc line='alias ll=\"ls -al\"' regexp='^alias ll=' create=yes" -i ansible_prj/inventory/hosts

ansible u_server24 -m shell -a "bash -ic 'source ~/.bashrc'" -i ansible_prj/inventory/hosts
```
* Как использовать алиас на удаленной машине (напрямую "ll /" не работает):
```
ansible u_server24 -m shell -a "bash -ic 'll /'" -i ansible_prj/inventory/hosts
``` 

* Показать все хосты:
```
ansible u_server24 -a "cat /etc/hosts" -i ansible_prj/inventory/hosts
```



* Какие группы есть на удаленной машине:
```
ansible u_server24 -a "groups" -i ansible_prj/inventory/hosts
```
* Установка пакета tree на удаленной машине:
```
ansible u_server24 -m apt -a "name=tree state=present" -i ansible_prj/inventory/hosts --become --ask-become-pass
```
* Установка пакета mc на удаленной машине:
```
ansible u_server24 -a "apt install mc" -i ansible_prj/inventory/hosts
```


## Как запустить плейбук:
* Выполнить плейбук:
```
ansible-playbook -i ansible_prj/inventory/hosts -l u_server24 ansible_prj/playbooks/test.yml
```
* Текст плейбука:
```
---
- name: Read hosts file
  hosts: u_server24
  tasks:
    - name: Display content of /etc/hosts
      command: cat /etc/hosts
      register: hosts_content

    - name: Show the content
      debug:
        var: hosts_content.stdout_lines
```
* Результат:
```
TASK [Show the content] *********************************************************************
ok: [u_server24] => {
    "hosts_content.stdout_lines": [
        "127.0.0.1 localhost",
        "127.0.1.1 u_server24",
        "",
        "# The following lines are desirable for IPv6 capable hosts",
        "::1     ip6-localhost ip6-loopback",
        "fe00::0 ip6-localnet",
        "ff00::0 ip6-mcastprefix",
        "ff02::1 ip6-allnodes",
        "ff02::2 ip6-allrouters"
    ]
}

PLAY RECAP *********************************************************************
u_server24                 : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```
* Ключевые моменты:
  * ansible-playbook - запускает плейбук
  * -i ansible_prj/inventory/hosts - указывает путь к inventory файлу
  * -l u_server24 - ограничивает выполнение для хоста u_server24
  * ansible_prj/playbooks/test.yml - путь к playbook файлу