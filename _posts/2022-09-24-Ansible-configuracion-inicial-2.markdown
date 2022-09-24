---
title:  "Ansible - Tutorial rápido completo 2/2"
author: teemo
categories: 
  - Ansible
  - Linux
show_excerpts: true
tags: 
  - Red Hat
  - CentOS
  - Ansible
date: 2022-09-24 13:45:00 +0000
---
![center-aligned-image](/images/ansible.webp){: .align-center}
# Introducción
En el [anterior post](https://noobchamp.github.io/ansible/linux/Ansible-configuracion-inicial-1/) vimos lo básico de `Ansible`. Aunque con nuestro inventario podemos lanzar prácticamente cualquier módulo por línea de comandos, lo ideal es hacer una estructura y usar todo lo disponible a nuestro alcance.

Ahora vamos a ver playbooks, handlers, tags, collections, vault, roles y ansible-galaxy. Estas cosas son las que hacen que `ansible` sea tan útil. 
Vamos a ver cada uno y entremedias veremos otras cosillas como el vault, al final va a ser un revuelto pero la cosa es ver cómo trabajan juntos puesto que por separado no es eficiente.

> Las opciones de los comandos se han explicado en el anterior post.
> Recuerda que la opción `-C` va a simular la ejecución pero no la llevará a cabo.


# Playbook
Un playbook nos sirver para ejecutar específicas tareas sobre específicos servidores. Por supuesto tiene sintaxis yamel.

Mejor vamos a explicar con un ejemplo, vamos a crear el primer playbook. Como vimos anteriormente todo lo hacemos como root pero vamos a crear el usuario ansible y copiar la _rsa_ a cada máquina. 

```yml
---
- hosts: micluster
  tasks:
    - name: Crear el usuario ansible
      user:
        name: ansible
        shell: /bin/bash
        #password:
        generate_ssh_key: yes
        ssh_key_bits: 2048
        ssh_key_file: .ssh/id_rsa
```
Y comprobamos que pasaría (lanzamos a un solo nodo):
```bash
[root@ansible miansible]# ansible-playbook crearusuario.yml -i inventory.yml -l nodo1 -C

PLAY [micluster] *******************************************************************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************************************************************
ok: [nodo1]

TASK [Crear el usuario ansible] ****************************************************************************************************************************************************************
changed: [nodo1]

PLAY RECAP *************************************************************************************************************************************************************************************
nodo1                      : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```
Como vemos en el playbook, no tenemos clave para este usuario, en Linux se puede crear el usuario pero no proporcionar contraseña.
Esa es la línea que tenemos comentada, pero como no queremos pasar contraseñas en texto plano vamos a la siguiente cuestión.

## Vault
No es más que un gestor de contraseñas (por así llamarlo). Se usa para cifrar las claves y no enviarlas en texto plano. Además te permite poder subir tu proyecto incluida las contraseñas puesto que usa una _contraseña maestra_.

Primero vamos a crear el archivo
```bash
[root@ansible miansible]# ansible-vault create vault.yml
New Vault password: 
Confirm New Vault password: 
# En mi caso he puesto secreto123, sera la contraseña maestra
```
Directamente entra en el editor vi (puedes cambiarlo), si no sabes salir pulsa _esc_ y _:wq_.  
He agregado la entrada:

```yml
ansible_usuario: ans1bl3_p4ss
```
Comprobamos que ahora está encriptado
```bash
[root@ansible miansible]# cat vault.yml 
$ANSIBLE_VAULT;1.1;AES256
34646139623266333865396234626265666437383332616538363665356539373636376630303032
3235656132326633333831356463613837386635353736340a356331646439373139366438316539
35613964643632373562343032386136313631366634353666306565663163613935313435346166
3163366363313865350a633634633464353637643233353237366164363033386537356137653466
35356530636132356431626239393663613461346165396530343666626431336632
```
Ahora si podemos añadir una contraseña, por ejemplo
```yml
---
- hosts: micluster
  tasks:
    - name: Crear el usuario ansible
      user:
        name: ansible
        shell: /bin/bash
        password: micomtraseña
        generate_ssh_key: yes
        ssh_key_bits: 2048
        ssh_key_file: .ssh/id_rsa
```
Esto nos va a reportar un problema
```bash
[root@ansible miansible]# ansible-playbook crearusuario.yml -i inventory.yml -l nodo1 -C

PLAY [micluster] *******************************************************************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************************************************************
ok: [nodo1]

TASK [Crear el usuario ansible] ****************************************************************************************************************************************************************
[WARNING]: The input password appears not to have been hashed. The 'password' argument must be encrypted for this module to work properly.
changed: [nodo1]

PLAY RECAP *************************************************************************************************************************************************************************************
nodo1                      : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

Hay varias soluciones, la más simple que sería usar _mkpasswd_ o _openssl_ para generar una clave SHA-512.
El propósito del `vault` era no depender de estas cosas, por tanto hay que modificar el playbook para indicarle que vamos a usar el playbook y que la contraseña será una variable
{% raw %}
```yml
---
- hosts: micluster
  vars_files:
    - vault.yml
  tasks:
    - name: Crear el usuario ansible
      user:
        name: ansible
        shell: /bin/bash
        password: "{{ ansible_usuario | password_hash('sha512') }}"
        generate_ssh_key: yes
        ssh_key_bits: 2048
        ssh_key_file: .ssh/id_rsa
```
{% endraw %}
> Como se aprecia, mediante una tubería hemos cifrado a SHA-512 el texto plano que teníamos en el vault.

Puesto que vamos a usar el vault ahora el comando es distinto, puesto que tenemos que indicar que la contraseña maestra para abrirlo
```bash
[root@ansible miansible]# ansible-playbook crearusuario.yml -i inventory.yml -l nodo1 -C --ask-vault-pass
Vault password: 

PLAY [micluster] 
...
...
nodo1                      : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```
Dirás que es un peñazo tener que poner la contraseña o que hace que lanzar una tarea automática sea imposible puesto que un prompt nos va a pedir la contraseña. Explora la opción `--vault-password-file ~/.vault_pass.txt` en la que podemos tener un archivo con los permisos correctos en el sitio correcto para que sea accesible por quien queremos.

# Roles
En estos momentos nos planteamos ciertas preguntas.
¿Para cada cierto tipo de acción necesitamos un playbook?
¿Qué ocurre si no quiero ejecutar un playbook completo, solo una parte?
¿Voy a juntar un montón de playbooks?

La respuesta es crear roles. A modo de hacer nuestro Ansible _modular_.
Vemos que estamos creando un cluster y vamos a tener que añadir tareas relativas a ellos. Pero la cuestión, no todo van a ser las mismas tareas, los mismos SO, los mismos elementos en general...

Lo ideal es hacer `roles` que realicen tareas generales que puedan valer para cualquier servidor. Por ejemplo: En nuestro caso haremos un rol para englobar las tareas relacionadas con SSH. Posteriormente podríamos crear otro rol para crear clúster Pacemaker, otro para servidores web...

Vamos a crear el primer rol, se puede hacer manualmente o mediante el comando `ansible-galaxy`

## Ansible Galaxy
Etiquetado para compartir con la comunidad roles y colecciones entre otros pero además nos va a permitir crear nuestras estructuras.

```bash
[oliva@ansible miansible]$ ansible-galaxy init ssh
- Role ssh was created successfully
[oliva@ansible miansible]$ ls -la ssh
total 8
drwxrwxr-x. 10 oliva oliva  154 Sep 22 20:51 .
drwxrwxr-x.  3 oliva oliva   79 Sep 22 20:51 ..
drwxrwxr-x.  2 oliva oliva   22 Sep 22 20:51 defaults
drwxrwxr-x.  2 oliva oliva    6 Sep 22 20:51 files
drwxrwxr-x.  2 oliva oliva   22 Sep 22 20:51 handlers
drwxrwxr-x.  2 oliva oliva   22 Sep 22 20:51 meta
-rw-rw-r--.  1 oliva oliva 1328 Sep 22 20:51 README.md
drwxrwxr-x.  2 oliva oliva   22 Sep 22 20:51 tasks
drwxrwxr-x.  2 oliva oliva    6 Sep 22 20:51 templates
drwxrwxr-x.  2 oliva oliva   39 Sep 22 20:51 tests
-rw-rw-r--.  1 oliva oliva  539 Sep 22 20:51 .travis.yml
drwxrwxr-x.  2 oliva oliva   22 Sep 22 20:51 vars
```
> Aqui vemos el árbol creado en la carpeta _ssh_ el cual es nuestro rol para tareas de SSH.

¿Te suena _tasks_? Son las tareas que ejecutamos en el playbook, ahora van en este archivo. Por tanto vamos a copiar el contenido del playbook, en la sección _tasks_ aquí
{% raw %}
```yml
# tasks file for ssh
- name: Crear el usuario ansible
  user:
    name: ansible
    shell: /bin/bash
    password: "{{ ansible_usuario | password_hash('sha512') }}"
    generate_ssh_key: yes
    ssh_key_bits: 2048
    ssh_key_file: .ssh/id_rsa
```
{% endraw %}
Mientras que el playbook _crearusuario.yml_ quedará asi
```yml
---
- hosts: micluster
  vars_files:
    - vault.yml
  roles:
    - ssh
```
Ejecutamos de nuevo 
```bash
[root@ansible miansible]# ansible-playbook crearusuario.yml -i inventory.yml -l nodo1 -C --ask-vault-pass 
```
### Y vemos que se completa correctamente. ¡Ya estamos trabajando con nuestro primer rol!

## TAGS
Necesitamos al menos agregar una segunda tarea para entender los tags.
{% raw %}
```yml
---
# tasks file for ssh
- name: Instalar la última versión de ssh
  yum:
    name: openssh-server, openssh-clients
    state: latest
  tags: instalar
    
- name: Crear el usuario ansible
  user:
    name: ansible
    shell: /bin/bash
    password: "{{ ansible_usuario | password_hash('sha512') }}"
    generate_ssh_key: yes
    ssh_key_bits: 2048
    ssh_key_file: .ssh/id_rsa
  tags: usuarios
```
{% endraw %}
Si volvemos a ejecutar, vamos a ver que se ejecutaran naturalmente ambas tareas.
Podemos especificar dentro del rol que tareas queremos especificar. Como sabemos que ya está instalado SSH vamos a decirle que ejecute solamente la tarea de crear el usuario ansible.
Usa el parámetro _-t_ para indicar los tags.

```bash
[root@ansible miansible]# ansible-playbook -v crearusuario.yml -i inventory.yml -l nodo1 -C -t usuarios --ask-vault-pass 
No config file found; using defaults
Vault password: 

PLAY [micluster] ************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************************************************************************************************
ok: [nodo1]

TASK [ssh : Crear el usuario ansible] ***************************************************************************************************************************************************************************************
changed: [nodo1] => {"changed": true}

PLAY RECAP ******************************************************************************************************************************************************************************************************************
nodo1                      : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```
> También puedes hacer el efecto contrario _--skip-tags_

No hay manera fácil, por así decirlo, de incluirlo en el playbook. Hay formas pero son más complejas.
Esto es una de las cosas, bajo mi punto de vista, están un poco verdes.

## Collections
Las coleciones son contenidos que pueden ser playbooks, roles, módulos y plugins.
Básicamente se ha creado una comunidad para ahorrarte tareas como las que hasta ahora estamos haciendo (muy normales, nada específico).
Toda la info se encuentra en la [página oficial de Ansible](https://galaxy.ansible.com).

Vamos a hacer una pequeña prueba, vamos a instalar un motd de [GoKEV](https://galaxy.ansible.com/gokev/motd-splash)
```bash
[root@ansible miansible]# ansible-galaxy install gokev.motd-splash -p /home/oliva/miansible/
Starting galaxy role install process
- downloading role 'motd-splash', owned by gokev
- downloading role from https://github.com/GoKEV/motd-splash/archive/master.tar.gz
- extracting gokev.motd-splash to /home/oliva/miansible/gokev.motd-splash
- gokev.motd-splash (master) was installed successfully
[root@ansible miansible]# ll gokev.motd-splash/
total 4
drwxr-xr-x.  2 root root   22 Sep 23 21:00 defaults
drwxr-xr-x.  2 root root  163 Sep 23 21:00 files
drwxr-xr-x.  2 root root   22 Sep 23 21:00 handlers
drwxr-xr-x.  2 root root   50 Sep 23 21:00 meta
drwxr-xr-x. 10 root root  135 Sep 23 21:00 motd-splash
drwxr-xr-x.  9 root root  118 Sep 23 21:00 prep-new-vm
-rw-rw-r--.  1 root root 2793 May  3 19:37 README.md
drwxr-xr-x.  2 root root   22 Sep 23 21:00 tasks
drwxr-xr-x.  2 root root  152 Sep 23 21:00 templates
drwxr-xr-x.  2 root root   39 Sep 23 21:00 tests
drwxr-xr-x.  2 root root   22 Sep 23 21:00 vars
drwxr-xr-x.  8 root root  105 Sep 23 21:00 vm-reboot
drwxr-xr-x.  8 root root  120 Sep 23 21:00 vmware-provision
```
En el _readme_ indica que puedes incluiir el rol `gokev.motd-splash` en un playbook.
Añadimos a nuestro playbook
```yml
---
- hosts: micluster
  vars_files:
    - vault.yml
  roles:
    - ssh
    - gokev.motd-splash
  vars:
    motd_template_file: templates/motd_redhat
```
Y ejecutamos con --diff para contemplar las diferencias de las plantillas
```bash
[root@ansible miansible]# ansible-playbook crearusuario.yml -i inventory.yml -l nodo1 -C --diff --ask-vault-pass 
Vault password: 

PLAY [micluster] ****************************************************************************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************************************************************
ok: [nodo1]

TASK [ssh : Instalar la última versión de ssh] **********************************************************************************************************************************************************
ok: [nodo1]

TASK [ssh : Crear el usuario ansible] *******************************************************************************************************************************************************************
changed: [nodo1]

TASK [gokev.motd-splash : Define the custom MOTD file if this is a CentOS system] ***********************************************************************************************************************
skipping: [nodo1]

TASK [gokev.motd-splash : Define the custom MOTD file if this is a RHEL system] *************************************************************************************************************************
skipping: [nodo1]

TASK [gokev.motd-splash : Define the MOTD file for any non-RHEL and non-CENT machine] *******************************************************************************************************************
skipping: [nodo1]

TASK [gokev.motd-splash : motd template] ****************************************************************************************************************************************************************
--- before: /etc/motd
+++ after: /root/.ansible/tmp/ansible-local-4924bw_vip6k/tmpvwnd3yzd/motd_redhat
@@ -0,0 +1,27 @@
+@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
+@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
+@@@@@@@@@@@###########%@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
+@@@@@@@@@@###############%@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
+@@@@@@@@@#################@@@@@@@@@@@@       @@@@@@@@@@@@@@@@@@@@  @@@@   @@@@@   @@@@@@@@@@@@@@@@@@@@
+@@@@@@@@@##################@@@@@@@@@@@   @@@   #@@@@@@@@@@@@@@@@@  @@@@   @@@@@   @@@@@@@@@@@   @@@@@@
+@@@#####   &###############@@@@@@@@@@@   @@@@   @       @@@        @@@@   @@@@@   @       &@       @@@
+@@#######      ############@@@@@@@@@@@         @   @@@   @   @@@@  @@@@           @@@@@@   @@   @@@@@@
+@@@#########                ###@@@@@@@   @@   @@     ,,,,@   @@@@  @@@@   @@@@@   @   **   @@   @@@@@@
+@@@@@###########%         ######@@@@@@   @@@   @@       @@@        @@@@   @@@@@   @        @@&    *@@@
+@@@@@@@#########################@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
+@@@@@@@@@@@####################&@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
+@@@@@@@@@@@@@@@@#############@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
+@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
+@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
+
+                              HOSTNAME  ===>  centos 
+                             IP ADDRESS ===>  10.0.0.11 
+               --------------------------------------------------------------------
+
+               KERNEL          4.18.0-348.7.1.el8_5.x86_64
+               ARCH            x86_64
+               MACHINE         innotek GmbH :: 1.2
+               COREs           2
+
+               --------------------------------------------------------------------
+

changed: [nodo1]

TASK [gokev.motd-splash : issue template] ***************************************************************************************************************************************************************
--- before: /etc/issue
+++ after: /root/.ansible/tmp/ansible-local-4924bw_vip6k/tmpytb0z_ud/issue
@@ -1,3 +1,12 @@
-\S
-Kernel \r on an \m
+  ###########################   WARNING!   ###########################
 
+              YOU ARE ABOUT TO CONNECT TO centos!!
+
+The computer you are about to use is company owned and is intended to be used
+for official company business. As such, the company reserves the right to
+monitor all activity on all company provided equipment and services. All use
+of this machine must comply with company IT policies,  available from HR.
+
+             ALL ACTIVITIES IN THIS SYSTEM ARE MONITORED!
+
+  ###########################   WARNING!   ###########################

changed: [nodo1]

PLAY RECAP **********************************************************************************************************************************************************************************************
nodo1                      : ok=5    changed=3    unreachable=0    failed=0    skipped=3    rescued=0    ignored=0   
```

## Handlers
Te va a permitir realizar una acción en cadena, es decir, si algo ha cambiado, se reinicia el servicio, pero no lo hará si no hay un cambio.

Si cambiamos el archivo de configuración de SSH necesitaremos reiniciar el servicio para que los cambios sean efectivos. Por tanto, es necesario reiniciar.

En la [página oficial](https://docs.ansible.com/ansible/latest/user_guide/playbooks_handlers.html) podemos ver un ejemplo muy claro.

¡Nos vemos en el siguiente post!

