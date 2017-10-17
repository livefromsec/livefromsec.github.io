---
layout: post
title: "Automatizando la creación de informes"
author: "Pete"
---

Después de algunas entradas hablando de noticias de seguridad informática (blabla equifax, blabla yahoo), la noticia de esta semana podía haber sido la vulnerabilidad encontrada en WPA2 que facilita robar la clave para poder conectarse con el AP. Pero es un rollo xD

Así que voy a utilizar de nuevo el blog como libreta para apuntar las cosas importantes, y voy a hablar de automatizar el trabajo utilizando python y LibreOffice. Hay un paquete de Django que nos facilitará la tarea: templated-docs. Esta es la página en la que su autor presenta el [funcionamiento](http://morozov.ca/django-pdf-msword-excel-templates.html) y ésta es la [documentación](https://templated-docs.readthedocs.io/en/latest/).

[Django es un framework de python para la creación de sitios web. Si estás leyendo el artículo seguro que lo conoces, si no aquí está la documentación](https://www.djangoproject.com/) (que por cierto, acaban de sacar la beta de su versión 2.0). Para ver mejor cómo funciona, podemos utilizar el ejemplo que da el creador del paquete, está en su [github](https://github.com/alexmorozov/templated-docs/tree/master/example). 

Nos bajamos el zip (botón de la parte superior derecha de github, "Clone or download" -> "Download ZIP"), los descomprimimos en nuestra máquina y nos vamos al directorio "example". Tenemos que estar en un directorio que tiene la base de datos (fichero sqlite3), el fichero manage.py y dos carpetas: example e invoices. Tenemos que fijarnos en tres archivos:
- example/urls.py -> Nos indica que para cualquier URL llame a la función invoice_view 
- invoices/views.py -> Importa el formulario y tiene la lógica para tratarlo
- invoices/forms.py -> Define el formulario para que el usuario introduzca los datos

Levantamos el servidor Django con "python manage.py runserver" y accedemos con el navegador a "localhost:8000". El formulario que veremos es similar al siguiente:



Para ver la lógica que hay detrás del formulario nos vamos al archivo views.py. Si no se han introducido datos en el formulario, utilizará el archivo invoices/form.html para mostrar el formulario, y una vez que lo rellenemos, por ejemplo con los siguientes datos:



Al hacer el "POST", la lógica entrará en la parte de código del "if form.is_valid():", pasándole los datos a la plantilla a través del diccionario form.cleaned_data. Y finalmente, Django nos devolverá el fichero, en el formato indicado, con los parámetros que hayamos rellenado en el formulario:


Si queremos modificar la plantilla es tan sencillo como introducir el campo nuevo en la plantilla (situada en invoices/templates/invoice.odt), por ejemplo:


Ese nuevo_campo que hemos definido tendremos que pasárselo a la plantilla ahora. Obviamente, lo lógico es añadir en el formulario (en invoices/forms.py) ese nuevo campo, pero para simplicar el ejemplo, podemos añadirlo directamente al código:


Y al generar el documento, ese campo se pasará a la plantilla para que lo rellene automáticamente:


Aquí está. Obviamente, ahora el precio hemos tenido que subirlo a 15$, ya que ahora lleva el nombre del blog ;)

De esta manera podemos automatizar la generación de documentos, ya sean facturas como en el ejemplo, informes, o cualquier otro tipo de documento. En  la documentación del paquete se muestra cómo introducir bucles for, para poder añadir listas de objetos, imágenes, así como cómo podemos hacer para generar hojas de cálculo o presentaciones.

