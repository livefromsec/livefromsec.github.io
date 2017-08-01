---
layout: post
title: "¿De dónde vienen tus visitas?"
author: "Pete"
---

En la entrada pasada veíamos cómo podemos iniciar un (pequeño) blog con [Jekyll](https://livefromsec.github.io/2017-08-04/tu-blog-en-jekyll) alojado en Github. Si optamos por un CMS, tendremos acceso a estadísticas de navegación de manera automática, de lo contrario tendremos que configurar estas estadísticas usando, por ejemplo, Google Analytics.

Para usar Google Analytics tendremos que registrarnos en el [sitio](https://analytics.google.com), muy fácil (y gratuito, para un blog pequeño) si ya tenemos una cuenta de Google. Durante el proceso habrá que indicar la dirección del sitio para el que queremos obtener las estadísticas, y al terminar obtendremos un código similar al siguiente:

{% highlight js %}
<script>
  (function(i,s,o,g,r,a,m){i['GoogleAnalyticsObject']=r;i[r]=i[r]||function(){
  (i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),
  m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;m.parentNode.insertBefore(a,m)
  })(window,document,'script','https://www.google-analytics.com/analytics.js','ga');
  ga('create', 'UA-XXXXXXXXX-Y', 'auto');
  ga('send', 'pageview');
</script>
{% endhighlight %}

Lógicamente variará esa parte final, XXXXXXXXX-Y. Una vez que tenemos ese código, lo que hay que hacer es incluirlo automáticamente en todas las páginas del blog. ¿A mano? Sería una locura; para eso utilizaremos la carpeta "_includes" de nuestro Jekyll. Si nos fijamos, hay un archivo en dicha carpeta llamado head.html. Lo que haremos es crear otro archivo en esa misma carpeta llamado "g_analytics.html" con el código que hemos obtenido al registrarnos en Google Analytics.

Una vez que tenemos ese archivo creado, tenemos que buscar el sitio en el que se indica que queremos incluirlo. Eso se hará en la carpeta "_layouts", en el archivo default.html. Si lo editamos, vemos que al principio hay una línea que se encarga de incluir head.html; la línea en concreto es:

{% highlight js %}
 {% include head.html %}
{% endhighlight %}

Pues bien, lo que hacemos es incluir también el archivo que hemos creado. De manera que esa parte del archivo quedará:

{% highlight js %}
 {% include g_analytics.html %} 
 {% include head.html %}
{% endhighlight %}

Y ya está, no hay que hacer más. A partir de este momento se irán recogiendo estadísticas de acceso a nuestro blog, que podremos consultar a través de Google Analytics. Por ejemplo, algo que me ha resultado muy curioso ha sido acceder a las estadísticas de este sitio y encontrarme con los siguientes países que acceden:

![placeholder]({{ site.url }}/assets/img/0003_img.png)

El blog está empezando y las visitas desde España las esperaba, pero las otras es verdad que me han sorprendido :)
