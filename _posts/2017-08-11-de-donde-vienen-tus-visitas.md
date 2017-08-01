---
layout: post
title: "¿De dónde vienen tus visitas?"
author: "Pete"
---

En la entrada pasada veíamos cómo podemos iniciar un (pequeño) blog con [Jekyll](https://livefromsec.github.io/2017-08-04/tu-blog-en-jekyll) alojado en Github. Si optamos por un CMS, tendremos acceso a estadísticas de navegación de manera automática, de lo contrario tendremos que configurar estas estadísticas usando, por ejemplo, Google Analytics.

Para usar Google Analytics tendremos que registrarnos en el [sitio](https://analytics.google.com), muy fácil (y gratuito, para un blog pequeño) si ya tenemos una cuenta de Google. Durante el proceso habrá que indicar la dirección del sitio para el que queremos obtener las estadísticas, y al terminar aparecerá un código similar al siguiente:

{% highlight js %}
// Sample javascript code
<script>
  (function(i,s,o,g,r,a,m){i['GoogleAnalyticsObject']=r;i[r]=i[r]||function(){
  (i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),
  m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;m.parentNode.insertBefore(a,m)
  })(window,document,'script','https://www.google-analytics.com/analytics.js','ga');
  ga('create', 'UA-XXXXXXXXX-Y', 'auto');
  ga('send', 'pageview');
</script>
{% endhighlight %}

Lógicamente variará ese número final, el XXXXXXXXX-Y. Una vez que tenemos ese código, lo que hay que hacer es incluirlo automáticamente en todas las páginas del blog. ¿A mano? Sería una locura; para eso utilizaremos la carpeta "_includes" de nuestro Jekyll. Si nos fijamos, hay un archivo en dicha carpeta llamado head.html. 

Estuve dándole vueltas al tema del blog y, de momento, he empezado con él utilizando [Jekyll](e  
https://en.wikipedia.org/wiki/Jekyll_(software)).

Valoraba dos opciones: por una parte decantarme por utilizar un CMS (un Wordpress, por ejemplo), alojado en un proveedor que gestionase el hosting y el registro del nombre de dominio; o sino, utilizar github para alojarlo, con Jekyll.

¿Las ventajas que tiene el CMS? Sobre todo, el hecho de que el blog sea dinámico. De lo contrario nos limitamos a páginas estáticas, por lo que cosas básicas para un blog (como comentarios!) no van a estar disponibles.

¿A cambio qué se gana? Por una parte, la sencillez. El hecho de no contar con un CMS que esté detrás de todo hace que la estructura quede muy sencilla: una carpeta "_posts" para los posts, el resto viene con la plantilla, y si queremos cambiar algo, en el layout o el archivo de configuración. Y hay que reconocerlo, el toque "geek" que le da optar por esta opción, siempre es bien recibido :) Además, no hay que preocuparse por el hosting ya se que puede alojar en GitHub (como éste!).

No obstante, me parece que se queda algo corto... Más adelante escribiré sobre cómo montar el blog con Jekyll, y seguramente más adelante nos movamos a páginas dinámicas con un gestor de contenidos.
