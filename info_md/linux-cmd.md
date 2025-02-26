# Linux commands через ansible
* **Просмотр содержимого директории:**
```        
ansible u_server24 -m shell -a "ls -la /var/log" -i ansible_prj/inventory/hosts
```
* **Проверка свободного места на диске:**
```
ansible u_server24 -m shell -a "df -h" -i ansible_prj/inventory/hosts
```
* **Проверка использования памяти:**
```
ansible u_server24 -m shell -a "free -h" -i ansible_prj/inventory/hosts
```
* **Проверка запущенных процессов:**
```
ansible u_server24 -m shell -a "ps aux" -i ansible_prj/inventory/hosts
```
* **Создание директории:**
```
ansible u_server24 -m file -a "path=/path/to/dir state=directory mode=0755" -i ansible_prj/inventory/hosts
```
* **Копирование файла:**
```
ansible u_server24 -m copy -a "src=/local/file dest=/remote/path" -i ansible_prj/inventory/hosts
```
* **Перезапуск сервиса:**
```
ansible u_server24 -m service -a "name=nginx state=restarted" -i ansible_prj/inventory/hosts --become
```
* **Проверка статуса сервиса:**
```
ansible u_server24 -m service -a "name=nginx state=started" -i ansible_prj/inventory/hosts --become
```
* **Обновление пакетов:**
```
ansible u_server24 -m apt -a "update_cache=yes upgrade=yes" -i ansible_prj/inventory/hosts --become
```
* **Установка пакета:**
```
ansible u_server24 -m apt -a "name=htop state=present" -i ansible_prj/inventory/hosts --become
```
* **Просмотр системной информации:**
```
ansible u_server24 -m shell -a "uname -a" -i ansible_prj/inventory/hosts
```
* **Проверка сетевых соединений:**
```
ansible u_server24 -m shell -a "netstat -tulpn" -i ansible_prj/inventory/hosts --become
```
* **Поиск файла:**
```
ansible u_server24 -m shell -a "find / -name '*.log'" -i ansible_prj/inventory/hosts --become
```
* **Просмотр содержимого файла:**
```
ansible u_server24 -m shell -a "cat /etc/hostname" -i ansible_prj/inventory/hosts
```
* **Проверка даты и времени:**
```
ansible u_server24 -m shell -a "date" -i ansible_prj/inventory/hosts
```

## Примеры перенаправления вывода и использования grep с Ansible:
* **Сохранение вывода в файл:**
```
ansible u_server24 -m shell -a "df -h > /home/starmark/disk_space.txt" -i ansible_prj/inventory/hosts
```
* **Использование grep и сохранение результата:**
```
ansible u_server24 -m shell -a "free -h | grep 'Mem' > /home/starmark/memory_info.txt" -i ansible_prj/inventory/hosts
```
* **Сложное перенаправление с grep:**
```
ansible u_server24 -m shell -a "ps aux | grep nginx > /home/starmark/nginx_processes.txt" -i ansible_prj/inventory/hosts
```
* **Добавление в конец файла (append):**
```
ansible u_server24 -m shell -a "df -h >> /home/starmark/system_info.txt" -i ansible_prj/inventory/hosts
```
* **Сохранение с сортировкой:**
```
ansible u_server24 -m shell -a "ps aux | sort -k 3 -r | grep python > /home/starmark/python_processes.txt" -i ansible_prj/inventory/hosts
```
