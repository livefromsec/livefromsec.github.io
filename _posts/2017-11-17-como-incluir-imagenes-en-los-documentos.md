---
layout: post
title: "Como incluir imágenes en los documentos"
author: "Pete"
---

Después de la entrada de hace unas semanas describiendo cómo podíamos automatizar la [creación de documentos desde un Django](https://livefromsec.github.io/2017-10-20/automatizando-la-creacion-de-documentos) tenía ganas de continuar escribiendo acerca del tema. Me pareció bastante útil la idea, y sería completamente genial si pudiésemos generar esos documentos incluyendo imágenes... De eso va el post de hoy.

Una vez que tenemos nuestra plantilla funcionando (post previo), si nos vamos a la [documentación del proyecto](https://templated-docs.readthedocs.io/en/latest/templatetags.html) podemos ver que también se pueden incrustar imágenes dentro de los documentos (cómo generarlas es otra historia, aunque se puede empezar por mirar [pygal](http://pygal.org/en/stable/) o plotly).

Por tanto, partimos de que tenemos nuestra imagen (por ejemplo, jpg o png) en un campo ImageField del modelo. Este campo guarda el nombre del fichero, por lo que si el fichero se llama imagen.jpg, el contenido de dicho campo será "imagen.jpg".

Empezamos por crear nuestro modelo; por ejemplo un modelo Pingüino, que tendrá varios campos, entre ellos un precio y una imagen. Creamos un nuevo objeto Pingüino, que tendrá un coste 1$ y una imagen de Tux :)

En la plantilla rellenamos los campos; en algún punto tendremos que tener un penguin.price y un penguin.photo. Ojo que este último es una imagen, y en lugar de ponerlo entre dos llaves tiene una sintaxis especial: {% image penguin.photo %}

![placeholder]({{ site.url }}/assets/img/0015_20171117_templated_docs_plantilla.png)

Hecho. Ahora generamos nuestro documento... y no funcionará ;) El ejemplo que hay en la página del creador no funciona de serie, de hecho al tener el bloqueador de javascript (del que hablé en la entrada anterior) no aparecen los comentarios y fue complicado dar con la solución. Visitando la página desactivé NoScript y en los comentarios alguien se había dado cuenta del mismo bug, y se lo habían avisado y habían puesto cómo resolverlo. Es necesario modificar el archivo "templated_docs_tags.py" (en mi caso estaba en mi home, en .local/lib/python3.5/site-packages/templated_docs/templatetags/templated_docs_tags.py, pero se puede buscar con "locate templated_docs_tags.py"). 

Lo editamos con nano, y en la línea
> images = context.dicts[0].setdefault('ootemplate_imgs', {})

tenemos que sustituir ootemplate_imgs por _templated_docs_imgs

> images = context.dicts[0].setdefault('_templated_docs_imgs', {})

Ahora sí podemos generar el fichero, y esta vez nos devolverá un fichero con la imagen integrada:

![placeholder]({{ site.url }}/assets/img/0016_20171117_templated_docs_documento_final.png)


