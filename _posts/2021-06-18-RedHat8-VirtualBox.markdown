---
title:  "VirtualBox Guest Addittions en Red Hat 8 y CentOS 8"
image: 
  path: /images/redhat8.webp
author: pancho
categories: 
  - RHEL
  - CentOS
show_excerpts: true
tags: 
  - Red Hat
  - CentOS
  - VirtualBox
  - Guest Additions
date: 2021-06-18 20:30:40 +0200
---
# Introducción
Cuando instalamos Red Hat 8 y/o CentOS 8 en VirtualBox siempre tenemos el inconveniente de que la vista no se ajusta a la ventana.  
El propósito de este mini tutorial es instalar rápidamente Guest Addiction en RHEL 8 rápidamente.

## Comandos
```bash
yum groupinstall "Development Tools"
yum install -y kernel-devel elfutils-libelf-devel
```

## Instalar Guest Additions
Simplemente, en la ventana de VirtualBox de la máquina te vas a la pestaña `Dispositivos` y luego presiona `Insertar imagen de CD de las "Guest Additions"`.  
Saldrá un diálogo en la máquina, presiona `run`.
Si el diálogo no te aparece puedes buscarlo en el directorio o si el caso es que no instalaste entorno gráfico "_bravo_", tienes que buscar el punto de montaje del cd y lanzar el script `VBoxLinuxAdditions.run`.

#### Reinicia la máquina... y ya está

¡Nos vemos!
