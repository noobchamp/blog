---
title:  "Jekyll: Crea tu web/blog en pocos minutos con Jekyll"
image: 
  path: /images/jekyll-logo.webp
  thumbnail: /images/jekyll-logo-thumbnail.webp
author: pancho
categories: 
  - Jekyll
show_excerpts: true
tags: 
  - Jekyll
  - Ruby
  - Github
date: 2021-04-08 19:00:40 +0200
---

# ¿Qué es Jekyll?
[Ir al grano](#instalación){: .btn .btn--primary .btn--small}  
[Jekyllrb](https://jekyllrb.com/){:.link} se define como un generador de sitios web estáticos.  
Principalmente se tratan de blogs o páginas webs en las que no es necesario tener una base de datos. 
Por ejemplo, si una persona quiere una página web para su restaurante, peluquería, taller o cualquier otro negocio en la que simplemente se usa para tener una "_imagen de empresa_" en internet. Webs en las que se publicitan servicios, precios, horarios, contacto, etc...  

En mi caso particular (y para lo que habitualmente se usa) he construido un blog. En pocos minutos y sin necesidad de ningún tipo de inversión económica...`GRATIS`, la palabra favorita de cualquiera.

Cada post o contenido agregado por el usuario está escrito en archivos Markdown o md. El formato viene dado por plantillas HTML y CSS.  
Jekyll está escrito en Ruby pero prácticamente no es necesario saber nada sobre este particular lenguaje de programación. Es necesario modificar algunos parámetros de configuración pero el sistemas de plantillas de Jekyll hace que sea realmente fácil para cualquier persona incluso con un conocimiento básico de informática.

Por supuesto este blog está hecho con Jekyll y voy a enseñarte en pocos paso como puedes crear el tuyo.
En mi caso, lo uso para propósito de aprendizaje y modo de documentación donde de paso otras personas puedan leer y aprender. Alguno que otro se pasará a criticarlo. Todos son bienvenidos.

Si quieres saber más sobre Jekyll puedes pasarte por su [página oficial](https://jekyllrb.com/){:.link} y echar un vistazo. Esta todo en Inglés, si no estás familiarizado seguro que encuentras decenas de tutoriales por la red, en Youtube y otras plataformas en castellano.

*[CSS]: Cascading Style Sheets
*[HTML]: HyperText Markup Language

# ¿Por qué Jekyll y no un CMS?
A mi modo de ver casi nunca nada es mejor que otra cosa o al menos en este mundillo con tecnologías similares. Siempre vas a oir cosas como _este lenguaje de programación es mejor que este.... Esta plataforma es mejor que esta..._ No es mi caso.  
Elegí Jekyll antes que otro CMS por varias razones:  
1. Rápida instalación y configuración.
2. No es necesario manejar una base de datos.
3. Estoy empezando a odiar los backends.
4. Para mí, al menos ahora es novedad (aunque lleva muchos años existiendo).
5. Me permite publicarlo gratis mediante [GITHUB](https://github.com/){:.link}
6. Decenas de plantillas gratuitas optimizadas en cuanto a SEO y texto enriquecido.  

Yo te diría que al menos le des una oportunidad e investigues un poco si alguno de los puntos anteriores encajan en tu carácter.


# Instalación
### Requisitos
##### Tener instalado Ruby 
`Windows:` Descarga e instala Ruby+Devkit desde la página de descargas de [RubyInstaller](https://rubyinstaller.org/downloads/){:.link} (instalaciones de Windows siempre es presionar _Sí_ a todo). En mi caso usé Ruby+Devkit 2.7.2-1 (x64). Al final de la instalación se abrirá un terminal en el que te pide elegir entre 3 opciones. Elige la 1 y presiona _Enter_ para finalizar la instalación.  

`Linux:` [Ubuntu](https://jekyllrb.com/docs/installation/ubuntu/){:.link} o según tu [distro](https://jekyllrb.com/docs/installation/other-linux/){:.link}  

##### Tener instalado Jekyll y Bundler
Abre un nuevo terminal y teclea (En la barra de búsqueda de Windows busca "Símbolo del sistema"):  
```bash
gem install jekyll bundler  
```
Finalmente comprueba que todo es correcto abriendo una nueva terminal y tecleando  
```bash
 jekill -v
 ```
En Windows tal vez es necesario reiniciar (ya conocemos a Windows) para aplicar cambios. 

Si no te aclaras o necesitas más información te dejo el **[link oficial](https://jekyllrb.com/docs/installation/)**{:.link} para la instalación de [Jekyll](https://jekyllrb.com/docs/installation/){:.link}. Es bastante completa, clara y concisa.

# Crear el sitio
En este momento ya disponemos de lo necesario para crear nuestro primer blog/sitio web.  
Para los siguientes pasos puedes usar editor de código (A mi en particular me gusta[ Visual Code Studio](https://code.visualstudio.com/download)){:.link} o la misma terminal de Windows. 
Para el ejemplo vamos a iniciar un nuevo terminal de windows, por defecto se sitúa en el directorio del usuario actual "C:\Users\Usuario>" y tecleamos:  
```bash
jekyll new nombre-sitio
```
<cite>nombre-sitio o lo que quieras llamarle (blog, miblog, misitio....)</cite>  

Se va a crear el sistema de archivos correspondiente y tenemos que desplazarnos a ese directorio:  
```bash
cd nombre-sitio
```

##### Levantar el servicio en local
Ahora simplemente vamos a arrancar un servidor local para visualizar nuestro primer sitio con Jekyll
```bash
bundle exec jekyll serve
```
Te responderá con algo como:
`Server address: http://127.0.0.1:4000`

Pinchamos sobre el enlace o abrimos nuestro navegador con al url `localhost:4000`

### ¡Listo!, ya tienes creado tu sitio

<figure class="align-center">
  <a href="#"><img src="{{ '/images/new-site.webp' }}" alt="Jekyll"></a>
  <figcaption>127.0.0.1:4000</figcaption>
</figure>  

# Primer post y configuración inicial
![right-aligned-image](/images/tree-directory-jekyll.webp){: .align-right}
En ese punto ya tenemos corriendo el servidor en local, con un primer post de ejemplo. 
El árbol de directorio debe ser similar al de la imagen de la derecha.  
En principio vamos a fijarnos en el directorio `_post`, el archivo `_config.yml` y el archivo `about.markdown`. Estos dos últimos están ubicados en la raíz del proyecto.  

# _post
En este directorio se almacenan los post a publicar. Hay que seguir una sintáxis ordenada, en cada nuevo post será guardado con el formato:  
```
formato: AAAA-mm-dd-nombre-articulo.markdown
ejemplo: 2021-12-31-pandemia-covid-19-erradicada.markdown
```  

Si compruebas el archivo de ejemplo creado `2021-04-08-welcome-to-jekyll.markdown` podrás ver que comienza con la siguiente cabecera:
```yml
---
layout: post
title:  "Welcome to Jekyll!"
date:   2021-04-08 14:18:40 +0200
categories: jekyll update
---
```

Debes respetar los 3 guiones al inicio y al final. Esta cabecera te puede guiar como plantilla, añade tu título, fecha de creación, categoría(s) y escribe el cuerpo del artículo o rellena con un poco de texto. Guarda los cambios y vuelve a tu página [localhost](http://127.0.0.1:4000) (¡Puedes hacer click en este enlace!).
```yml
---
layout: post
title:  "¡Adiós al Covid!"
date:   2020-12-31 00:00:00 +0200
categories: Covid
---

# Al fin!!
El COVID-19 es historia.
```
<figcaption >Yo he escrito este artículo. Haz el tuyo propio o copia este código y visualiza los cambios en tu entorno.</figcaption>  

Cuidado si al guardar los cambios o relanzar el comando `bundle exec jekyll serve` nos devuelve algún error, por ejemplo, si la fecha es posterior a la actual.  

# _config.yml
```yml
title: Your awesome title
email: your-email@example.com
description: >- # this means to ignore newlines until "baseurl:"
  Write an awesome description for your new site here. You can edit this
  line in _config.yml. It will appear in your document head meta (for
  Google search results) and in your feed.xml site description.
baseurl: "" # the subpath of your site, e.g. /blog
url: "" # the base hostname & protocol for your site, e.g. http://example.com
twitter_username: jekyllrb
github_username:  jekyll

# Build settings
theme: minima
plugins:
  - jekyll-feed
```
Esta parte no es un poema....no es muy difícil entender, por el momento puedes cambiar el título de tu sitio, configurar tu email, hacer una descripción y agregar si quieres tu usuario de Twitter y Github (si quieres).  
Al tratarse de una archivo de configuración tenemos que volver a compilar. Lanzamos el comando `bundle` para recompilar nuestro sitio y luego nuevamente levantamos el servidor local. 

# about.markdown
Se trata de la página "about" de nuestro sitio. Tienes una pequeña pista de como utilizar formato para enlaces:  
```markdown
[jekyllrb.com](https://jekyllrb.com/)
```
<cite>Se trata de un enlace a [jekyllrb.com](https://jekyllrb.com/){:.link}</cite>


## ¿Esto es todo?
En este punto ya tienes un nuevo sitio, con una configuración inicial, un nuevo post original y una página "about". 
Pues... de momento no es muy atractivo que digamos. Hay que cambiar el estilo, agregar nuevas páginas como "post, tags, etc", aún hay que publicarlo en internet `GRATIS` como se había prometido....
No te preocupes, no era un _clip bait_ cuando al principio te indicaba que era rápido de iniciar y que podrías publicarlo totalmente gratis pero este artículo se está alargando demasiado y mi intención era familiarizarte con Jekyll y entender el funcionamiento.  

#### ¿Y si te dijera que todo lo anterior no vale absolutamente para nada?
Porque hay maneras (y muchas) más sencillas de crear el sitio/blog con su estilo, pestañas, configuración adicional...etc. Podemos hacer nuestro sitio partiendo desde una plantilla lo que nos va a permitir ahorrarnos todo esto que hemos visto. Por tanto, es hora de elegir plantilla. Desde el sitio oficial de [jekyll](https://jekyllrb.com/docs/themes/){:.link} nos ofrecen varios enlaces donde podremos encontrar cientos de plantillas `GRATIS` ... otra vez ... `GRATIS`. Los enlaces son los siguientes:  

* [jamstackthemes](jamstackthemes.dev){:.link}
* [jekyllthemes](jekyllthemes.org){:.link}
* [jekyllthemes](jekyllthemes.io){:.link}
* [jekyll-themes](jekyll-themes.com){:.link}

Elige una de las plantillas gratuitas para proseguir.  

#### Te espero en el siguiente artículo. 
[Te sigo]({% post_url 2021-04-08-crear-blog-jekyll-desde-plantilla %}){: .btn .btn--primary .btn--large}  