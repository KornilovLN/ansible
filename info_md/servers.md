## Какие хосты прописаны на машине оператора:
```
# cloud gitlab.ivl.ua
192.168.196.21  gitlab.ivl.ua

# Office gitlab.office.ivl.ua
192.168.196.21  gitlab.office.ivl.ua
192.168.196.112 registry.master.cns
192.168.196.112 brama.master.cns
192.168.196.112 pypi.master.cns
192.168.196.112 vavilon

192.168.196.112 brama.master.cns.atom
192.168.196.112 brama.znpp.cns.atom

# VPN server proXmoX 192.168.88.15:8006
192.168.88.15:8006 vpn.proXmoX
192.168.88.101     vpn.vm.develop
192.168.88.102     vpn.vm.ubuntu18
192.168.88.103     vpn.vm.VM3
192.168.88.104     vpn.vm.NPPTest

192.168.88.105     brama.sunpp.cns.atom
192.168.88.105     srv_sunpp

192.168.88.106     uDesktop24
192.168.88.107     u_server24
```

## После преобразования адресов в хостах на формат inventory:
```
# Office hosts
[office]
192.168.196.21 ansible_host=gitlab.office.ivl.ua
192.168.196.112 ansible_host=registry.master.cns
192.168.196.112 ansible_host=brama.master.cns
192.168.196.112 ansible_host=pypi.master.cns
192.168.196.112 ansible_host=vavilon
192.168.196.112 ansible_host=brama.master.cns.atom
192.168.196.112 ansible_host=brama.znpp.cns.atom

# VPN server
[vpn_servers]
192.168.88.15 ansible_host=vpn.proXmoX
192.168.88.101 ansible_host=vpn.vm.develop
192.168.88.102 ansible_host=vpn.vm.ubuntu18
192.168.88.103 ansible_host=vpn.vm.VM3
192.168.88.104 ansible_host=vpn.vm.NPPTest

192.168.88.106 ansible_host=uDesktop24
192.168.88.107 ansible_host=u_server24

[brama_servers]
192.168.88.105 ansible_host=srv_sunpp
192.168.88.105 ansible_host=brama.sunpp.cns.atom

[mongodb_servers]
192.168.88.104 ansible_host=vpn.vm.NPPTest
```

## Выяснить, какие хосты прописаны в invertory - по команде:
```
ansible-inventory -i /mnt/poligon/ansible/ansible_prj/inventory/hosts --list
```
#### можно получить такой вывод:
```
{
    "_meta": {
        "hostvars": {
            "192.168.196.112": {
                "ansible_host": "brama.znpp.cns.atom"
            },
            "192.168.196.21": {
                "ansible_host": "gitlab.office.ivl.ua"
            },
            "srv_sunpp": {
                "ansible_ssh_host": "192.168.88.105",
                "ansible_ssh_pass": "!18leon28",
                "ansible_user": "starmark"
            },
            "uDesktop24": {
                "ansible_ssh_host": "192.168.88.106",
                "ansible_ssh_pass": "!18leon28",
                "ansible_user": "starmark"
            },
            "u_server24": {
                "ansible_ssh_host": "192.168.88.107",
                "ansible_ssh_pass": "!18leon28",
                "ansible_user": "starmark"
            },
            "vpn.vm.NPPTest": {
                "ansible_ssh_host": "192.168.88.104",
                "ansible_ssh_pass": "!18leon28",
                "ansible_user": "starmark"
            },
            "vpn.vm.VM3": {
                "ansible_ssh_host": "192.168.88.103",
                "ansible_ssh_pass": "!18leon28",
                "ansible_user": "starmark"
            },
            "vpn.vm.develop": {
                "ansible_ssh_host": "192.168.88.101",
                "ansible_ssh_pass": "!18leon28",
                "ansible_user": "starmark"
            },
            "vpn.vm.ubuntu18": {
                "ansible_ssh_host": "192.168.88.102",
                "ansible_ssh_pass": "!18leon28",
                "ansible_user": "starmark"
            }
        }
    },
    "all": {
        "children": [
            "ungrouped",
            "office",
            "vpn_servers",
            "mongodb_servers"
        ]
    },
    "mongodb_servers": {
        "hosts": [
            "vpn.vm.NPPTest"
        ]
    },
    "office": {
        "hosts": [
            "192.168.196.21",
            "192.168.196.112"
        ]
    },
    "vpn_servers": {
        "hosts": [
            "vpn.vm.develop",
            "vpn.vm.ubuntu18",
            "vpn.vm.VM3",
            "vpn.vm.NPPTest",
            "srv_sunpp",
            "uDesktop24",
            "u_server24"
        ]
    }
}
```

## Команды для просмотра плейбуков:
**1. Посмотреть список задач в конкретном плейбуке:**
```
ansible-playbook ansible_prj/playbooks/commands4.yml --list-tasks -i ansible_prj/inventory/hosts
```
**2. Посмотреть список хостов для плейбука:**
```
ansible-playbook ansible_prj/playbooks/commands4.yml --list-hosts -i ansible_prj/inventory/hosts
```
**3. Посмотреть список тегов в конкретном плейбуке:**
```
ansible-playbook ansible_prj/playbooks/commands4.yml --list-tags -i ansible_prj/inventory/hosts
```
**4. Для просмотра всех плейбуков в директории:**
```
ls -la ansible_prj/playbooks/*.yml
```