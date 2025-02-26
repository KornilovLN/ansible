# Как удаленно подключиться к серверу по SSH и выполнить команду
### 1. Простую команду можно выполнить с помощью команды ssh.
```
$ ssh starmark@192.168.88.107 "ls -al ~/"
```
* **Ответ:**
``` 
total 76
drwxr-x--- 7 starmark starmark  4096 Feb  9 13:23 .
drwxr-xr-x 3 root     root      4096 Feb  8 12:41 ..
drwx------ 3 starmark starmark  4096 Feb  8 12:47 .ansible
-rw------- 1 starmark starmark   108 Feb  9 13:32 .bash_history
-rw-r--r-- 1 starmark starmark   220 Mar 31  2024 .bash_logout
-rw-r--r-- 1 starmark starmark  3770 Feb  8 14:45 .bashrc
drwx------ 3 starmark starmark  4096 Feb  8 13:15 .cache
drwx------ 4 starmark starmark  4096 Feb  9 13:24 .config
-rw-rw-r-- 1 starmark starmark   457 Feb  8 15:32 disk_space.txt
drwxrwxr-x 3 starmark starmark  4096 Feb  8 12:43 .local
-rw-rw-r-- 1 starmark starmark    81 Feb  8 15:36 memory_info.txt
-rw-rw-r-- 1 starmark starmark   213 Feb  8 15:38 nginx_processes.txt
-rw-r--r-- 1 starmark starmark   807 Mar 31  2024 .profile
-rw-rw-r-- 1 starmark starmark 14056 Feb  9 13:23 res.txt
drwx------ 2 starmark starmark  4096 Feb  8 12:41 .ssh
-rw-r--r-- 1 starmark starmark     0 Feb  8 13:15 .sudo_as_admin_successful
-rw------- 1 starmark starmark   556 Feb  8 17:02 .viminfo
```

### 2. Запустить терминальную команду на удаленном сервере и получить вывод
```
ssh -t starmark@192.168.88.107 htop
```
* **Ответ:**
```
Запустится htop на удаленном сервере с показом на хосте.
После закрытия htop на удаленном сервере, закроется терминал на хосте.
Connection to 192.168.88.107 closed.
```
