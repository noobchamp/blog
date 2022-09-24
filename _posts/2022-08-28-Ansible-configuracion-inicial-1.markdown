---
title:  "Ansible - Tutorial rápido completo 1/2"
author: teemo
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
![center-aligned-image](/images/ansible.webp){: .align-center}
# Introducción
`Ansible` es un herramienta de automatización. No me extenderé en explicar lo que es pero si diré que es muy útil y cada vez es mas usado por administradores sobre todo para automatizar tareas repetitivas o hacer inspección de todos los servidores.  

En mis inicios de la informática, cada vez que instalabamos un servidor, inmediatamente pasábamos un script (con variables) para definir el hostname, la red, instalar los servicios básicos o los habituales de la organización, etc.. 
Posteriormente incluso otros para la monitorización, u otros elementos que no eran comunes, como por ejemplo si se trataba de un servidor de aplicaciones.

`Ansible` nos facilita estas tareas mediante plantillas y además nos permite controlar la ejecución, ya sea por fallos, entender qué va a ocurrir u otros factores donde antes "te la jugabas".

Está basado en `python` :heart: y en este tutorial descubrirás que además de poder personalizar cada tarea hasta el más mínimo detalle, vas a poder simplemente instalar otros "módulos" que van a evitar que tengas que picar todo el código tú solo.

## Escenario
Por el momento, como somos _gente molona_ vamos a trabajar como root. Luego ya veremos...

* VirtualBox
* Máquina virtual Ansible: Centos 8.5.2 (puedes usar Red Hat ya que Centos a muerto :disappointed:)
* IP Ansible 10.0.0.5: He creado una red interna nat. En un capítulo siguiente veremos para qué pero puedes usar el puente o el nat normal para tener acceso a internet.
* Máquina virtual nodo1, nodo2, nodo3: Centos 8.5.2 Clonadas desde una plantilla
* IP nodoX: 10.0.0.1X 
* No olvides tener todo actualizado
``` bash
yum clean all && yum update -y
```

# Instalar ansible
``` bash
[root@ansible ~]# yum install python39
[root@ansible ~]# pip3 install ansible
[root@ansible ~]# pip3 install argcomplete
# Vamos a generar ya la clave ssh para copiarla a otro servidor, cuando proceda.
# Le das a INTRO (todo por defecto)
[root@ansible ~]# ssh-keygen -t rsa -b 4096
```

# Ansible
¡Ahora si! 
Ansible por defecto te indica que el archivo de inventario es en /etc/ansible/hosts.
Vamos a pasar de esto y hacer nuestro propio ansible, nuestro propio inventario en yamel.
Para resumir, me he creado mi usuario _oliva_ y he craeado /home/oliva/miansible.yml (aun seguimos como root):
```yml
all:
  hosts:
  children:
    micluster:
      hosts:
        nodo1:
          ansible_host: 10.0.0.11
        nodo2:
          ansible_host: 10.0.0.12
        nodo3:
          ansible_host: 10.0.0.13
```
Estas entradas como se puede ver, es para crear un cluster de 3 nodos, al que invocaremos con el nombre `cluster`. Posteriormente seguiremos aumentando este inventario.

Por tanto debemos tener las entradas correspondientes en /etc/hosts:
``` bash
127.0.0.1   localhost localhost.localdomain ansible
10.0.0.11   nodo1.micluster nodo1
10.0.0.12   nodo2.micluster nodo2
10.0.0.13   nodo3.micluster nodo3
```
### Vamos a hacernos _amigos_ de los nodos, copiamos la clave ssh para que no pida contraseña
```bash
[root@ansible ~]# ssh-copy-id root@nodo2
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/root/.ssh/id_rsa.pub"
The authenticity of host 'nodo2 (10.0.0.12)' can't be established.
ECDSA key fingerprint is SHA256:VZ5ZJ4xh0PEgUe1twpgZEU0dsjtTSzWey5tVajSx1nA.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@nodo2's password:

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'root@nodo2'"
and check to make sure that only the key(s) you wanted were added.
```
Esto para los 3 nodos evidentemente...

Y ahora vamos a hacer la primera prueba, un ping a todos. He apagado el nodo2 para probar:
```bash
[root@ansible ~]# ansible all -m ping -i /home/oliva/miansible/inventory.yml
nodo1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "ping": "pong"
}
nodo3 | SUCCESS => {
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
```
* -m: modulo de ansible
* -i: inventario, recuerda que hemos especificado un inventario, por defecto sería /etc/ansible/hosts
* -u: En caso de que no estuvieramos como root especificamos -u root o el usuario con el que hayamos realizado el ssh-copy-id.
* 
Funciona con el mismo usuario en origen a no ser que se especifique bien por línea de comandos o se puede establecer en el archivo de inventario de esta forma (aunque la vamos a descartar, en la siguiente parte veremos por qué):
```yml
all:
  hosts:
  children:
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
```
Como se puede ver, la sintaxis de yame hace que según la identación, pueda establecerla para los host de `cluster`o para todos los hosts en adelante si no tenemos pensado trabajar con otro usuario que no sea root.

En la siguiente parte vamos a ver los playbooks, roles, tags, handlers...
Además empezaremos a trabajar con ansible-galaxy.

¡Nos vemos en el siguiente post!