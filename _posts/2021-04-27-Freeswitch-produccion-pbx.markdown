---
title:  "Freeswitch: Instalación sobre Debian 10 Buster"
image: 
  path: /images/freeswitch/freeswitch.jpeg
  thumbnail: /images/freeswitch/freeswitch-thumbnail.jpeg
author: yoko
categories: 
  - Freeswitch
show_excerpts: true
tags: 
  - Freeswitch
  - PBX
  - VoIP
  - SIP
date: 2021-04-26 20:00:40 +0200
---
# Introducción

FreeSwitch es una plataforma de telefonía de código abierto.  
Es realmente flexible gracias a su buen rendimiento. Se puede usar como PBX, como un soft-switch. 
Podríamos usarla para distintos propósitos como por ejemplo un SBC entre otras centralitas físicas o virtuales como Alcatel, Panasonic, Avaya, Asterisk, etc. Podríamos instalar una GUI para usar como PBX. Podríamos usarlo para control de facturación entre distintos clientes y proveedores, es decir, balancear nuestras llamadas por coste de todos nuestros clientes además de muchas más cosas.

Freeswitch es muy versátil. Es posible instalar Freeswitch desde una Raspberry hasta un buen servidor físico, va a depender de la carga. Lo mejor de todo es que es *escalable*, por lo que podríamos ampliar en cualquier momento.

*[PBX]: Private Branch Exchange
*[SBC]: Session Border Controller

# Objetivos del aprendizaje
No vamos a entrar en profundidad en definiciones o aspectos técnicos sobre VoIP. En _internet_ se puede encontrar numerosos sitios donde explican conceptos de telefonía como:  
* Protocolo SIP
* Códecs
* Wireshark
* TCPdump
* Aplicaciones de PBX
* Softphone
* LUA
* XML
* Otras cosas

Vamos a instalar Freeswitch y empezar a "trastearlo" todo para entender como funciona y que ventajas nos puede ofrecer. Así pues...¡Vamos al lío!

# Instalación de Freeswitch 
En nuestro ejemplo vamos a instalar Freeswitch sobre [Debian 10 Buster](https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/debian-10.9.0-amd64-netinst.iso){:.link}, de forma local a través de una máquina virtual en red interna. 
Ya te voy avisando que si usas arquitectura i386 no te va a funcionar. Tienes el enlace a la versión 10.9 amd64.  
##### Máquina virtual:
- 4Gb Ram
- 16Gb Almacenamiento
- 2 Procesadores
- Adaptador puente _IP 192.168.0.15_
- Sin escritorio
- SSH Server

Con eso vamos a tirar de momento. Más adelante vamos a ver como podemos mover a producción este servidor local. Una vez iniciado Debian 10 actualizamos el sistemas, agregamos el repositorio e instalamos Freeswitch.
```shell
apt-get update && apt-get install -y gnupg2 wget lsb-release
wget -O - https://files.freeswitch.org/repo/deb/debian-release/fsstretch-archive-keyring.asc | apt-key add -
echo "deb http://files.freeswitch.org/repo/deb/debian-release/ `lsb_release -sc` main" > /etc/apt/sources.list.d/freeswitch.list
echo "deb-src http://files.freeswitch.org/repo/deb/debian-release/ `lsb_release -sc` main" >> /etc/apt/sources.list.d/freeswitch.list
apt-get update && apt-get install -y freeswitch-meta-all
```
Luego levantamos Freeswitch y ya que estamos habilitamos el servicio en el arranque y comprobamos que funciona correctamente:
```shell
systemctl start freeswitch
systemctl enable freeswitch.service
fs_cli
```
<cite>Evidentemente tenemos que ser root o lanzar como "sudo"</cite>

Si todo es correcto veremos lo siguiente:
![center-aligned-image](/images/freeswitch/fs-cli.jpeg){: .align-center}

## Configuración y primera llamada
Ya es posible realizar nuestra primera llamada. Es necesario tener instalado algún dispositivo SIP.  
Yo voy a usar Zoiper, además en su versión 3, que me gusta más que la 5 de momento. Os dejo un enlace por si queréis descargar la misma versión [ZOIPER](https://www.zoiper.com/en/voip-softphone/download/zoiper3/for/windows){:.link}.  
Primero vamos a comprobar que en Freeswitch no existe ningún registro:
```shell
freeswitch@debian> show registrations
```
<cite>Devolverá 0 total</cite>  

Posteriormente vamos a registrar nuestra primera extensión "1000". Así que tenemos que crear en Zoiper una `nueva cuenta SIP`:
* *Usuario:* 1000
* *Contraseña:* 1234
* *Domain:* 192.168.0.15 (tendrás que poner la IP asignada a tu Sistema)

Si tienes éxito podemos volver a lanzar el comando `show registrations` y veremos que nos muestra datos de nuestra extensión recién registrada además de _1 total._
Ahora podrías añadir por ejemplo otra extensión como la 1001 o simplemente podemos comprobar que todo funciona llamando a algunas extensiones que Freeswitch ha creado y enrutado de manera predeterminada.

| Header1 | Header2 |
|:--------:|:------:|
| 5000   | IVR   |
|4000| Voicemail |
|9196| Eco test|
|9195| Eco test con 5 segundos de retardo|
|9197| Un pitido terrible|
|9198| Tetris (wav) |
|9664| Música de espera (MOH)|

Hay muchos más números pero de momento si todo esto funciona correctamente podemos estar satisfechos.

## Conclusión
Bueno, pues aunque parezca que ya que podemos llamar tenemos un 70% del trabajo realizado, creo que con esto no llegamos ni la 1%.  
La configuración es realmente compleja y vamos a ir aprendiendo como usarlo. 
En un principio no vamos a instalar ningún programa adicional para manejar la configuración sino que todo lo vamos a realizar a mano. Ya sabes, a base de golpes se aprende.  
  
Si lo que quieres es tener una PBX simple pequeña te recomiendo que busques FreePBX u otro tipo de solución. Para dar servicio de telefonía a un pequeño negocio no necesitas Freeswitch, sino otro tipo de PBX con una buena interfaz, fácil de usar y más completa en lo referente a servicios para el usuario.  

Nosotros más adelante si vamos a conectarle otra PBX como FreePBX, vamos a crear proveedores, posiblemente instalemos Pyfreebilling, instalaremos un servidor de trazas como _Homer_ y realizar test de carga y comparativas.

¡Nos vemos en el siguiente post!

