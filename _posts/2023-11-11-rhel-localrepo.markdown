---
title:  "Red Hat - Instalar un repositorio local"
author: teemo
categories: 
  - RHEL
  - Linux
show_excerpts: true
tags: 
  - Red Hat
date: 2023-11-11 11:45:00 +0000
---
![center-aligned-image](/images/redhat8.webp){: .align-center}
# Introducción
Tener un servidor conectado a Red Hat, no siempre es posible o no queremos subscribirlo por varios motivos:
- Nos encontramos en una red sin acceso a internet.
- Se trata de una máquina virtual o pruebas.
- La velocidad de red tiene poco caudal.
- Nos gusta la aventura...

Podemos instalar un repositorio local, o bien teniendo un servidor que `SI` esté subscrito o desde un `CD oficial`.
¿Cuál es la diferencia? Evidentemente, teniendo un cd como repositorio, no tenemos actualizaciones de paquetes, sin embargo, no será necesario hacer un pago en Red Hat.
El inconveniente de todo esto, será la cantidad de espacio que vamos a necestar, que dependerá de cuántas versiones querramos tener.

<cite>Desarrollado a través de [Red Hat](https://access.redhat.com/solutions/7019225).</cite>


## Preparando el terreno
### Subscripción a Red Hat
Por supuesto, tienes que tener cuenta en RHEL. No es necesario previo pago, puede obtener una licencia de pruebas o un sadbox.
```shell
# Aunque la versión no tiene por qué ser la última, podríamos tener un 8.1 y descargar cualquier versión 8.X
root@repository:~#
cat /etc/os-release
NAME="Red Hat Enterprise Linux"
VERSION="8.8 (Ootpa)"
ID="rhel"
ID_LIKE="fedora"
VERSION_ID="8.8"
PLATFORM_ID="platform:el8"
PRETTY_NAME="Red Hat Enterprise Linux 8.8 (Ootpa)"
ANSI_COLOR="0;31"
CPE_NAME="cpe:/o:redhat:enterprise_linux:8::baseos"
HOME_URL="https://www.redhat.com/"
DOCUMENTATION_URL="https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8"
BUG_REPORT_URL="https://bugzilla.redhat.com/"
REDHAT_BUGZILLA_PRODUCT="Red Hat Enterprise Linux 8"
REDHAT_BUGZILLA_PRODUCT_VERSION=8.8
REDHAT_SUPPORT_PRODUCT="Red Hat Enterprise Linux"
REDHAT_SUPPORT_PRODUCT_VERSION="8.8"

# Registro en RHEL
root@repository:~#
subscription-manager register
Registering to: subscription.rhsm.redhat.com:443/subscription
Username: oliva_rh
Password: ******
The system has been registered with ID: 3aa74c4f-2e84-439e-aa37-xxxxxxxxxxxx
The registered system name is: repository

# Aquí vemos el acceso a cualquier versión de la 8
root@repository:~#
subscription-manager release --list
+-------------------------------------------+
          Available Releases
+-------------------------------------------+
8
8.0
8.1
8.2
8.3
8.4
8.5
8.6
8.7
8.8

# Por el momento no hemos definido ninguna
root@repository:~#
subscription-manager release --show
Release not set
```
Por otro lado, tampoco hemos habilitado ningún repositorio.
Podemos listar todos los `repositorios` disponibles.
Habilitaremos los que nos interese...
En mi caso, pondré dos básicos:
- rhel-8-for-x86_64-baseos-rpms
- rhel-8-for-x86_64-appstream-rpms

```shell
root@repository:~#
subscription-manager repos --list | more
+----------------------------------------------------------+
    Available Repositories in /etc/yum.repos.d/redhat.repo
+----------------------------------------------------------+
Repo ID:   rhel-atomic-7-cdk-3.10-rpms
Repo Name: Red Hat Container Development Kit 3.10 /(RPMs)
Repo URL:  https://cdn.redhat.com/content/dist/rhel/atomic/7/7Server/$basearch/cdk/3.10/os
Enabled:   0

Repo ID:   rhocp-4.9-for-rhel-8-x86_64-source-rpms
Repo Name: Red Hat OpenShift Container Platform 4.9 for RHEL 8 x86_64 (Source RPMs)
Repo URL:  https://cdn.redhat.com/content/dist/layered/rhel8/x86_64/rhocp/4.9/source/SRPMS
Enabled:   0
# Y un largo etc ...

root@repository:~#
subscription-manager repos --enable=rhel-8-for-x86_64-baseos-rpms
Repository 'rhel-8-for-x86_64-baseos-rpms' is enabled for this system.
root@repository:~#
subscription-manager repos --enable=rhel-8-for-x86_64-appstream-rpms
Repository 'rhel-8-for-x86_64-appstream-rpms' is enabled for this system.
root@repository:~#
subscription-manager repos --list
--list           --list-disabled  --list-enabled

# Listar los repositorios habilitados
root@repository:~#
subscription-manager repos --list-enabled
+----------------------------------------------------------+
    Available Repositories in /etc/yum.repos.d/redhat.repo
+----------------------------------------------------------+
Repo ID:   rhel-8-for-x86_64-appstream-rpms
Repo Name: Red Hat Enterprise Linux 8 for x86_64 - AppStream (RPMs)
Repo URL:  https://cdn.redhat.com/content/dist/rhel8/$releasever/x86_64/appstream/os
Enabled:   1

Repo ID:   rhel-8-for-x86_64-baseos-rpms
Repo Name: Red Hat Enterprise Linux 8 for x86_64 - BaseOS (RPMs)
Repo URL:  https://cdn.redhat.com/content/dist/rhel8/$releasever/x86_64/baseos/os
Enabled:   1
```

### Servidor http/https
Vamos a necesitar un servidor web, en mi caso, voy a usar `Caddy`.
Puedes usar el mítico httpd o el que te de la gana.
Lo voy a colocar todo en la ruta _/var/www/repo_.

```shell
root@repository:~#
dnf install caddy

# Habilitar y iniciar el servicio Caddy
systemctl enable caddy
systemctl start caddy

# Verificar el estado del servicio Caddy
systemctl status caddy

# Configurar sitio
mkdir -p /var/www/repo
echo ':8080 {
>   root  * /var/www/repo
>   file_server {
      browse
    }
> }' > /etc/caddy/Caddyfile.d/repos.caddyfile
systemctl reload caddy
```

## Descarga de repositorios
Ahora simplemente vamos a establecer la versión con `subscription-manager` y descargar a nuestra ruta local _/var/www/repo_.
```shell
subscription-manager release --set=8.7 && rm -rf /var/cache/dnf
# Vamos a crear una carpeta para la versión 8.8 y otra para la versión 8.7
mkdir -p /var/www/repo/8.8
mkdir -p /var/www/repo/8.7

# Instalar reposync
yum install -y yum-utils
#Empezamos a descargar para la versión 8.8
subscription-manager release --set=8.8 && rm -rf /var/cache/dnf 
reposync -p /var/www/repo/8.8 --download-metadata --repoid=rhel-8-for-x86_64-baseos-rpms
reposync -p /var/www/repo/8.8 --download-metadata --repoid=rhel-8-for-x86_64-appstream-rpms
# Para la versión 8.7
subscription-manager release --set=8.7 && rm -rf /var/cache/dnf 
reposync -p /var/www/repo/8.7 --download-metadata --repoid=rhel-8-for-x86_64-baseos-rpms
reposync -p /var/www/repo/8.7 --download-metadata --repoid=rhel-8-for-x86_64-appstream-rpms
```
Entonces nos quedará algo así
![center-aligned-image](/images/repo.webp){: .align-center}


## CLientes
A la hora de configurar otros servidores para que puedan descargar paquetes desde este repositorio, crearemos el archivo _/etc/yum.repos.d/mi_repositorio.repo
```shell
[repo-id]
baseurl=http://X.X.X.X/version
gpgcheck=0
enabled=1
# Example
[rhel-8-for-x86_64-baseos-rpms]
baseurl=http://192.168.1.2/8.8
gpgcheck=0
enabled=1
# etc...
```
# Automatización
La forma más sencilla, es tener una tarea en cron que lo haga cada cierto periodo.
Simplemente es agregar a tu cron, los comandos que hemos visto anteriormente.

¡Adiós!

