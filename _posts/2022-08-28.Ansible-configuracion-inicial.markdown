---
title:  "Ansible - Configuración Inicial"
author: yoko
categories: 
  - Ansible
  - Linux
show_excerpts: true
tags: 
  - Red Hat
  - CentOS
  - Ansible
date: 2022-08-28 9:00:00 +0000
---
instalar python3
# oliva@ansible [~]$ sudo yum install python39


oliva@ansible [~/ansible]$ cat ../.bashrc | grep alias
# User specific aliases and functions
alias python='python3.9'
alias pip='pip3.9'

oliva@ansible [~/ansible]$ pip install ansible
oliva@ansible [~/ansible]$ pip install argcomplete


ssh-keygen -t rsa -b 4096
oliva@ansible [/home/oliva]# ssh-copy-id root@10.0.0.11

oliva@ansible [~]$ ansible all -u root -m ping
funciona con el mismo usuario en origen y destino a no ser que se especifique

oliva@ansible [~/ansible]$ vim inventory.yml
oliva@ansible [~/ansible]$ ansible micluster -m ping -i inventory.yml
nodo1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "ping": "pong"
}
nodo2 | UNREACHABLE! => {
    "changed": false,
    "msg": "Failed to connect to the host via ssh: ssh: connect to host 10.0.0.12 port 22: No route to host",
    "unreachable": true
}
nodo3 | UNREACHABLE! => {
    "changed": false,
    "msg": "Failed to connect to the host via ssh: ssh: connect to host 10.0.0.13 port 22: No route to host",
    "unreachable": true
}
oliva@ansible [~/ansible]$ ansible nodo1 -m ping -i inventory.yml
nodo1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "ping": "pong"
}
oliva@ansible [~/ansible]$ vim inventory.yml
micluster:
  hosts:
    nodo1:
      ansible_host: 10.0.0.11
    nodo2:
      ansible_host: 10.0.0.12
    nodo3:
      ansible_host: 10.0.0.13

  vars:
    ansible_user: root
