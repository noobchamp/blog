---
title:  "FreeSwitch: Variables"
image: 
  path: /images/freeswitch/freeswitch.webp
  thumbnail: /images/freeswitch/freeswitch-thumbnail.webp
author: yoko
categories: 
  - FreeSwitch
show_excerpts: true
tags: 
  - FreeSwitch
  - PBX
  - VoIP
  - SIP
  - Variables
date: 2021-04-28 19:00:40 +0200
---
# Introducción
Como en cualquier otro software, las variables proporcionan poder acceder a un valor cuando se solicita sin tener asignarla contínuamente.  
Lo más importante aquí es saber que FreeSwitch carga las variables en memoria, por tanto, cuando accedes a esta variable lo estás haciendo en la memoria del sistema.

## Tipos
Básicamente son dos tipos de variables: variable `GLOBAL` (Global variables) y variable de `CANAL` (Channel variable).
En el primer caso `GLOBAL`, la variable es accesible para todos lo canales mientras en el segundo caso `CANAL` la variable es solo accesible en el contexto del canal.  

Por ejemplo, una variable global sería la IP que usa el sistema `local_ip_v4`, siempre vamos a poder acceder a ella en cualquier parte.  
Como ejemplo de una variable de canal sería por ejemplo una asignada para el número de destino al que se está llamando *destination_number*, asignada para manejar la llamada en el _dialplan_. Esta variable será reasignada en cada llamada, en cada canal, porque no va a obtener un valor fijo, eso lo tenemos claro en este punto.

## Acceder a las variables
Se accedes a las variables con formato `${variable}`. Como en el ejemplo anterior, vamos a comprobar nuestra variable `local_ip_v4`, podemos hacerlo de dos formas:  
Desde la consola de FreeSwitch con el comando `eval` + `${variable}` (ejecutando fs_cli):
```
freeswitch@debian> eval ${local_ip_v4}
192.168.0.15
```

O desde la misma shell, con `fs_cli -x` + `eval ${variable}`:
```
root@debian:/home/oliva# fs_cli -x 'eval ${local_ip_v4}'
192.168.0.15
```
<cite>¡Ojo! Debes usar las comillas simple y no doble para que no sea interpretado por bash</cite>

## Asignación y re-asignación
En los archivos de configuración será referidas con un doble $ `$${variable}`, sólo en los archivos de configuración para *sólo variables globales*.  
Para el resto como hemos visto anteriormente nos referimos con un sólo $ `${variable}`. Podemos referirnos tanto a variables globales como variables de canal.  
Entonces...¿Es posible re-asignar una variable global? Rotundamente sí.  

```
freeswitch@debian> global_getvar local_ip_v4
192.168.0.15
freeswitch@debian> global_setvar local_ip_v4=1.1.1.1
+OK
freeswitch@debian> global_getvar local_ip_v4
1.1.1.1
freeswitch@debian>
```

Esta variable global `local_ip_v4` se encuentra definida en el archivo vars.xml (en realidad no es así del todo pero luego se aclara). Le hemos dicho a FreeSwitch que su IP local es 1.1.1.1. Las consecuencias son catastróficas, nuestro teléfono ha dejado de registrar, en el anterior capítulo tienes como comprobar o simplemente el teléfono te va a devolver un error 403 - Forbidden.  

Algo interesante, si hacemos _reloadxml_ recargará la configuración de FreeSwitch pero la variable seguirá definida como 1.1.1.1. El comando debería recargar los archivos XML como el _dialplan_, además de _vars.xml_ pero no es así en caso de variables *pre-processor*. Vamos a comprobar varias cosas:

* Abrimos el archivo _/etc/freeswitch/vars.xml_ -> Aquí vamos a ver las variables como PRE-PROCCESS, un ejemplo que vas a recordar.  

```xml
<X-PRE-PROCESS cmd="set" data="default_password=1234"/>
```

* Visualiza el contenido de _/var/log/freeswitch/freeswitch.xml.fsxml_. Este archivo son todas las variables que se han cargado al iniciar FreeSwitch.

`global_setvar` ha modificado una variable "pre-cargada" por el sistema que no puede ser recargada. Ahora solo tenemos dos caminos para que nuestro FreeSwitch funcione correctamente de nuevo permitiendo registrar los dispositivos: O bien volvemos a lanzar el comando con el parámetro correcto o bien reiniciamos el servicio. Debes de tener en cuenta que en producción el reinicio va a provocar la **caída de todas las llamadas** en curso. Estamos en un desarrollo así que....alegría.

<cite>--La consola de FreeSwitch estará devolviendo un warning por este problema que hemos provocado--</cite>


Ahora vamos a darle una vuelta más. En el archivo _vars.xml_ no aparece definida 
```xml
<X-PRE-PROCESS cmd="set" data="local_ip_v4=192.168.0.15"/>
```
Si en algún momento te paraste a leer, habrás visto que indica expresamente que una serie de variables, entre ellas _local_ip_v4_ han sido calculadas por FreeSwitch dinámicamente y definidas como variables globales. 

>Repite los pasos anteriormente pero en vez de cambiar la variable _local_ip_v4_, cambia la variable _default_password_.
>Luego recarga configuración XML `reloadxml` y comprueba de nuevo el valor de la variable.
>¡Sí! En este caso la variable ha sido cargada nuevamente y ahora devuelve nuevamente _1234_ sin tener que recargar FreeSwitch

>Puede que la pregunta que te ronda ahora es para qué tanto follón y lío con esto de las variables. Bueno, según como vayas a montar tu sistema, vas a necesitar conocer si es necesario definir ciertas variables del sistema como la ip local. Puedes enrutar tanto por IP o puede que tu IP sea dinámica y vayas a registrar tus terminales mediante DNS y así no es necesario asignar un IP fija local.  

En fin, por una parte me va a obligar a indicar una IP fija en mi DNS en la interfaz local de mi FreeSwitch o bien por otra a configurar un nombre de dominio y tener la libertad de tener una ip dinámica. Cada cual tiene sus ventajas e inconvenientes y cuando se trata de interfaz local pues no es verdaderamente útil que digamos pero piensa en lo siguiente:

Estamos en un entorno local de desarrollo pero la idea es tener a usuarios en cualquier parte del mundo. ¿Es más factible que registren contra una IP pública o contra un dominio típico como voip.empresa.com  

Esto abre un mundo de posibilidades ya que tener un nuevo cliente en cualquier parte y si tuvieras algún problema con la IP no te va a importar ya que no necesitarás re-configurar los terminales. También ahorramos un pequeño coste por pagar IP fija en algunos casos.  


En el siguiente post habrá un poco de teoría sobre FreeSwitch para entender el funcionamiento un poquito mejor.  

¡Nos vemos!
