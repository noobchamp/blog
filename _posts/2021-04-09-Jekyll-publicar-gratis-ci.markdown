---
title:  "Jekyll: Publicar tu blog gratis en Github e integración contínua (CI)"
image: 
  path: /images/jekyll-logo.webp
  thumbnail: /images/jekyll-logo-thumbnail.webp
author: bowie
categories: 
  - Jekyll
show_excerpts: true
tags: 
  - Jekyll
  - Ruby
  - Github
date: 2021-04-09 12:00:40 +0200
---
# Introducción 
En el [artículo anterior]({% post_url 2021-04-08-crear-blog-jekyll-desde-plantilla %}){:.link} vimos como crear una un blog haciendo un fork en el repositorio del creador.  Además iniciamos el sitio web en un entorno local e hicimos algunos ajustes, además creamos un nuevo post.  
Veremos que cuando creemos el sitio, cada vez que creemos un nuevo post no se va a actualizar en Github. Para ello contamos con un script que va a reconstruir el sitio para cada `push`que lancemos.

## Requisitos
* Tener cuenta en Github
* Conocimientos básicos de Git
* Un poco de ilusión

## Pasos previos.
Dentro la raíz de nuestro repositorio `agusmakmun.github.io` (en nuestro caso puesto que creamos la plantilla a partir de este repositorio) vamos a realizar `git push`.  
Comprobamos que en nuestro proyecto LOCAL, en el archivo _config.yml situado en la raíz tiene la `URL` configurada como `username.github.io`.  
Ahora vamos a actualizar nuestro repositorio, simplemente en el terminal ejecutamos los comandos: 
```bash
git add .
git commit -m "primer commint"  
git push
```
Y comprobamos en nuestra cuenta que los cambios que hemos realizado en nuestro proyecto local como nuestro primer post `¡Adiós al Covid!`.

## Crear una nueva rama (branch)
![right-aligned-image](/images/new-branch.webp){: .align-right}
En la pestaña "Master" hacemos click y ponemos en el cuadro de diálogo `gh-pages`, luego a "crear rama a partir de Master". Esta rama es la que vamos a indicar a Github que vamos a publicar.  
Así es, no vas a publicar rama principal sino una rama creada a partir de la "Master" y en los siguientes pasos vamos a ver como actualizarla cada vez que realizamos un push.    

## Configuración
Ahora vamos a la _configuración del proyecto_ y lo renombramos cono "usuario.github.io", es decir, cambiamos el nombre de "agusmakmun" a tu nombre de usuario. 

## Envioroments
Ve a la pestaña "envioroments" y crea uno nuevo. Puedes ponerle de nombre "gh-page"

## Pages
![right-aligned-image](/images/pages.webp){: .align-left}
Por último nos vamos a la pestaña "pages" y configuramos la rama a usar `gh-pages`.  

## Test
En estos momentos tu sitio web debe estar actualizado y deberías poder acceder a la URL `username.github.io`.  

_Éxito_ -> Puedes ver blog creado con Jekyll publicado en internet al alcance de cualquier interesado.
_Fracaso_ -> Puede que no hayas realizado un paso bien. Repito los pasos.
_Ni blanco ni negro_ -> Tu sitio web se visualiza pero no tiene formato [CSS]. En este caso comprueba que en el archivo `_config.yml` no tienes configurado la opción `baseurl`. A no ser que tu proyecto esté en un subdirectorio como "/blog".  
*[CSS]: Cascading Style Sheets

# Integración contínua
En estos momentos si creamos un nuevo post en nuestro proyecto local y hacemos "push" para actualizar nuestro repositorio. Vamos a comprobar que nuestro sitio `username.github.io` no se actualiza.  

Tienes que crear una carpeta dentro de tu proyecto llamada `.github` --<cite>con el "." punto delante</cite>. Dentro de ella creamos otra carpeta llamada `workflos`y dentro de esta carpeta creamos el archivo `main.yml`.  

```liquid {% raw %}
on:
  push:
    branches:
      - master
jobs:
  build_and_deploy:
    runs-on: ubuntu-latest
    steps:
      - name: GitHub Checkout
        uses: actions/checkout@v1
      - name: Bundler Cache
        uses: actions/cache@v1
        with:
          path: vendor/bundle
          key: ${{ runner.os }}-gems-${{ hashFiles('**/Gemfile.lock') }}
          restore-keys: |
            ${{ runner.os }}-gems-
      - name: Build & Deploy to GitHub Pages
        uses: joshlarsen/jekyll4-deploy-gh-pages@master
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          GITHUB_REPOSITORY: ${{ secrets.GITHUB_REPOSITORY }}
          GITHUB_ACTOR: ${{ secrets.GITHUB_ACTOR }}   
```{% endraw %}  
Al realizar nuevamente los comandos git y subir el archivo podremos ver los cambios en la pestaña "actions" de nuestro proyecto.  
Y no me voy a enrollar más sobre como funciona el archivo, si estás interesado házmelo saber mediante comentarios y poder subir otro artículo más dedicado a la integración contínua.  

También podéis dirigiros al repositorio original [joshlarsen](https://github.com/joshlarsen/jekyll4-deploy-gh-pages/blob/master/readme.md){: .link} para leer la documentación completa.

Espero que te haya servido de ayuda.