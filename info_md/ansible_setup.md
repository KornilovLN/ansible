# Подготовка окружения для работы с Ansible

## Установка Ansible
**1. Обновляем списки пакетов и устанавливаем пакет software-properties-common, необходимый для работы с репозиториями apt:**
```
sudo apt update && sudo apt -y install software-properties-common
```
**2. Добавляем репозиторий Ansible:**
```
sudo add-apt-repository --yes --update ppa:ansible/ansible
```
**3. Обновляем список пакетов и устанавливаем пакет ansible:**
```
sudo apt update && sudo apt -y install ansible
```
**4. Проверка версии Ansible с помощью команды:**
```
ansible --version
```

## Предварительная настройка Ansible
**1. Редактирование файла ansible.cfg:**
```
sudo nano /etc/ansible/ansible.cfg
```
**2. Установка параметра host_key_checking в значение False:**
```
[defaults]
host_key_checking = False
```
**3. Пояснение на выключение host key verification:**
```
Выключить проверку ключей (host key verification) при подключении по протоколу SSH.
Когда она включена, удаленный сервер просит подтвердить, что хост верный.
При этом приходится вводить yes при подключении к каждому серверу.
Для удобства выключим эту опцию.
```
**4. Подготовка файла ansible_prj/inventory/hosts:**
* Указание хостов, к которым будет подключаться Ansible по имени и паролю:
```
[all]
uDesktop24 ansible_ssh_host=192.168.88.106 ansible_user=starmark ansible_ssh_pass="!18leon28"
u_server24 ansible_ssh_host=192.168.88.107 ansible_user=starmark ansible_ssh_pass="!18leon28"
```
* Указание хостов, к которым будет подключаться Ansible по имени и файлу с закрытым ключом:
```
[all]
192.168.88.106 ansible_ssh_private_key_file=/home/starmark/.ssh/id_rsa ansible_user=starmark
192.168.88.107 ansible_ssh_private_key_file=/home/starmark/.ssh/id_rsa ansible_user=starmark
```
**5. Проверка подключения к хостам:**
```
ansible all -m ping -i ansible_prj/inventory/hosts
```
**6. Ответ:**
```
Если в выводе отобразилось SUCCESS, то подключение по SSH успешно установлено.
Если при подключении к хостам - ошибки, отправить открытый ключ на требуемые серверы.
(команда ssh-copy-id). 
```
**7. Отправка открытого ключа на удаленный сервер:**
* Напрямую через ssh-copy-id: 
```
ssh-copy-id starmark@192.168.88.107
```
* Через командную строку для конкретного сервера с указанием порта:
```
ssh-copy-id -i ~/.ssh/id_rsa.pub -p 22 starmark@192.168.88.107
```
* Через Ansible для нескольких серверов сразу:
``` 
---
- name: Copy SSH key to remote servers
  hosts: all
  tasks:
    - name: Install public ssh key
      authorized_key:
        user: starmark
        state: present
        key: "{{ lookup('file', '~/.ssh/id_rsa.pub') }}"
```
* Массовая отправка через цикл:
``` 
for host in server1 server2 server3; do ssh-copy-id starmark@$host; done
```
* Посмотреть список ключей на удаленном сервере:
```
ansible u_server24 -m shell -a "cat ~/.ssh/authorized_keys" -i ansible_prj/inventory/hosts
```
* Посмотреть права и структуру SSH директории:
``` 
ls -la ~/.ssh/
```
* Вывести количество установленных ключей:
```
wc -l ~/.ssh/authorized_keys
```

## Запуск нескольких команд сразу на удаленных серверах:
* ZB:
``` 
ansible u_server24 -m shell -a "ls /" -i ansible_prj/inventory/hosts
ansible u_server24 -m shell -a "ls -la ~/" -i ansible_prj/inventory/hosts
ansible u_server24 -m shell -a "wc -l ~/.ssh/authorized_keys" -i ansible_prj/inventory/hosts
ansible u_server24 -m shell -a "ls -la ~/.ssh/" -i ansible_prj/inventory/hosts
```
* Создаем playbook с названием commands.yml:
```
---
- name: Execute multiple commands
  hosts: u_server24
  tasks:
    - name: List root directory
      shell: ls /

    - name: List home directory
      shell: ls -la ~/

    - name: Count SSH keys
      shell: wc -l ~/.ssh/authorized_keys

    - name: List SSH directory
      shell: ls -la ~/.ssh/
```
* Запускаем playbook:
```
ansible-playbook ansible_prj/playbooks/commands.yml -i ansible_prj/inventory/hosts
```