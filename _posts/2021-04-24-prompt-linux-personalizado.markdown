---
title:  "Prompt: Personaliza el prompt de tu Linux"
author: pancho
categories: 
  - Linux
show_excerpts: true
tags: 
  - Linux
  - Prompt
  - Debian
date: 2021-04-22 20:06:40 +0200
---

# Prompt
A menudo tenemos varios terminales abiertos y la Shell de Linux suele ser siempre la misma por norma general.  
```
username@hostname$
```
Varía un poco según la distro o según si el usuario que usamos es `root` u otro distinto.  

## ¿Cuál es el propósito
En muchas ocasiones nos movemos contínuamente por ejemplo de servidores de producción a desarrollo o simplemente tenemos varios terminales en los que solemos trabajar a lo largo del día.  
Durante algunos años estuve trabajando como administrador de sistemas realizando contínuos despliegues. Llega el momento en el que te confundes a la hora de reiniciar un servicio, realizar copias y consultas varias entre el servidor de producción y el servidor pre-producción puesto que suelen ser idénticos y la información se suele desplazar del segundo al primero una vez se ha verificado todo.  

##### _¿A nadie le ha ocurrido reiniciar un proceso en producción creyendo que se trataba del servidor de pre-producción o desarrollo?_  
A mí unas cuantas, así que empecé a personalizar el prompt para evitar este tipo de errores.  
Como idea, puedes incluirlo en un script para cuando recibes acceso a un servidor. A lo largo del tiempo verás que es rentable ya que siempre vas a realizar tareas iniciales como esta además de otras como configurar una clave ssh o hacer un diagnóstico inicial del sistema.

## Vamos al grano
Hay decenas de páginas web para hacer esta tarea más sencilla. Si quieres seguir mi ejemplo entra en [bashrcgenerator](http://bashrcgenerator.com).  

![center-aligned-image](/images/bashrcgenerator.webp){: .align-center}


 
## Personalizando el prompt de Debian
Ahora tienes que copiar el resultado de la casilla 4.  
Como puedes observar `*?*` no nos dice nada, eso vamos a cambiarlo por `PRODUCCIÓN`, `DESARROLLO` o la etiqueta que quieras poner para identificarlo en todo momento.
En la Shell, edita el archivo `.bashrc` de tu home:
```shell
vi ~/.bashrc
```
<cite>Si no sabes usar el editor de VI, usa nano o el que prefieras</cite>
Pega el código copiado anteriormente y sustituye `?` como indicaba anteriormente. 
Tras esto tienes que salir y volver a entrar o simplemente usa el comando:

```
source ~/.bashrc
```
![right-aligned-image](/images/bashrc.webp){: .align-right}
¡Espero que te sirva de algo!
Saludos.