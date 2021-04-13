---
title:  "Jekyll: Crea tu blog Jekyll desde una plantilla"
image: 
  path: /images/jekyll-logo.png
  thumbnail: /images/jekyll-logo-thumbnail.png
author: yoko
categories: 
  - Jekyll
show_excerpts: true
tags: 
  - Jekyll
  - Ruby
  - Github
date: 2021-04-09 10:00:40 +0200
---

# Introducción
En un [artículo anterior]({% post_url 2021-04-07-crear-un-sitio-web-estatico-gratis-jekyll %}){:.link} hemos creado nuestro primer blog/sitio.  
Ahora vamos a crear un blog partiendo desde una plantilla, por tanto como otros CMS nos va a proporcionar la mayoría del trabajo en cuanto a:
* Estilos
* Rutas
* Configuración
* SEO
* Javascript
* Tags
* Y un largo etc

## Requisitos previos
Esto son los requisitos previos impuestos para este artículo, ya que puedes hacerlo de varias maneras. Yo voy a realizar un fork sobre el repositorio de Github de la plantilla que me interesa. Podrías copiar directamente el directorio pero vas a necesitar al menos estos requisitos para el siguiente paso *publicarlo gratuitamente* en internet.

* Tener cuenta en Github
* [GIT](https://git-scm.com/downloads){:.link} instalado.

Si no estás familiarizado con git no es mayor problema, solo vamos a hacer _git clone_.  
He tomado de ejemplo [Agus Makmun](https://github.com/agusmakmun/agusmakmun.github.io){:.link}, es una plantilla sencilla y fácil de entender para un inicio. 
Si quieres seguir los mismos pasos pincha en el enlace.

### Realizar un fork
Cuando estemos en el repositorio de la plantilla que hayas elegido, en la esquina superior derecha, bajo tu foto de perfil de Github verás lo siguiente:  

![center-aligned-image](/images/fork.png){: .align-center}

Presiona sobre `FORK` y el repositorio se copiará automáticamente como un repositorio tuyo.

## Traer el proyecto a local
Ahora si vamos a necesitar git. Abre una nueva consola y situate donde quieras clonar los archivos a tu PC. Ejecuta:
```bash
git clone https://github.com/username/agusmakmun.github.io.git
```
Tras esto, nos desplazamos dentro del proyecto `agusmakmun.github.io.git` y siguiendo las instrucciones del creador tecleamos en la consola:
```bash
bundle update
jekyll serve
```
Abre en tu navegador `localhost:4000`  
¡Y ya está!, ya tenemos todo lo que hicimos en el [artículo anterior]({% post_url 2021-04-07-crear-un-sitio-web-estatico-gratis-jekyll %}){:.link} pero con las ventajas de una plantilla.  

En el siguiente artículo vamos a publicar gratis nuestro proyecto y además vamos a realizar una integración contínua.  

#### ¿Te vienes?

[Voy para alla!]({% post_url 2021-04-09-Jekyll-publicar-gratis-ci %}){: .btn .btn--primary .btn--large} 

