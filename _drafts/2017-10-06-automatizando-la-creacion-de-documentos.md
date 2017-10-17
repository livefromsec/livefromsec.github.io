---
layout: post
title: "Automatizando la creación de documentos"
author: "Pete"
---

Después de algunas entradas hablando de noticias de seguridad informática ([Equifax](https://livefromsec.github.io/2017-09-08/equifax-hackeado), [Yahoo](https://livefromsec.github.io/2017-10-06/mas-bases-de-datos-comprometidas)), la noticia de esta semana podía haber sido la vulnerabilidad encontrada en WPA2, [KRACK](https://www.reddit.com/r/netsec/comments/76onkk/the_krack_attack_info_will_be_available_here/). 

Pero si hablamos siempre de noticias, el blog no tendría otros contenidos. Así que voy a utilizar de nuevo el blog como [libreta para apuntar](https://livefromsec.github.io/2017-10-13/usando-alias-en-linux) las cosas importantes, y voy a hablar de automatizar el trabajo de creación de documentos utilizando Python y LibreOffice. Hay un paquete de Django que nos facilitará la tarea: templated-docs. Ésta es la página en la que su autor presenta el [funcionamiento](http://morozov.ca/django-pdf-msword-excel-templates.html) y ésta es la [documentación](https://templated-docs.readthedocs.io/en/latest/).

Django es un framework de Python para la creación de sitios web. Si estás leyendo el artículo seguro que lo conoces, si no aquí está la [documentación del proyecto](https://www.djangoproject.com/) (que por cierto, acaban de sacar la beta de su versión 2.0). Para ver mejor cómo funciona la creación automatizada de documentos, podemos utilizar el ejemplo que da el creador del paquete, está en su [Github](https://github.com/alexmorozov/templated-docs/tree/master/example). 

Nos bajamos el zip (botón de la parte superior derecha de Github, "Clone or download" -> "Download ZIP"), lo descomprimimos en nuestra máquina y nos vamos al directorio "example". Tenemos que estar en un directorio que tiene la base de datos (fichero sqlite3), el fichero manage.py y dos carpetas: example e invoices. Tenemos que fijarnos en tres archivos:
- example/urls.py -> Nos indica que para cualquier URL llame a la función invoice_view 
- invoices/views.py -> Importa el formulario y tiene la lógica para tratarlo
- invoices/forms.py -> Define el formulario para que el usuario introduzca los datos

Levantamos el servidor Django desde la consola con "python manage.py runserver" y accedemos con el navegador a "localhost:8000". El formulario que veremos es similar al siguiente:

![placeholder]({{ site.url }}/assets/img/0008_20171020_templated_docs_initial_form.png)

Para ver la lógica que hay detrás del formulario nos vamos al archivo invoices/views.py. Si no se han introducido datos en el formulario, utilizará el archivo invoices/form.html para mostrar el formulario, y una vez que lo rellenemos, por ejemplo con los siguientes datos:

![placeholder]({{ site.url }}/assets/img/0009_20171020_templated_docs_views.png)

![placeholder]({{ site.url }}/assets/img/0010_20171020_templated_docs_form_data.png)

Al hacer el "POST", la lógica entrará en la parte de código del "if form.is_valid():", pasándole los datos a la plantilla a través del diccionario form.cleaned_data. Y finalmente, Django nos devolverá el fichero, en el formato indicado, con los parámetros que hayamos rellenado en el formulario:

![placeholder]({{ site.url }}/assets/img/0011_20171020_templated_docs_document_created.png)

Si queremos modificar la plantilla es tan sencillo como introducir el campo nuevo en la plantilla (situada en invoices/templates/invoice.odt), por ejemplo a continuación he añadido un campo nuevo debajo de "BILL TO":

![placeholder]({{ site.url }}/assets/img/0012_20171020_templated_docs_new_field.png)

Ese nuevo_campo que hemos definido tendremos que pasárselo a la plantilla ahora. Obviamente lo lógico es, o añadir en el formulario (en invoices/forms.py) ese nuevo campo, o que lo coja de un modelo, pero para simplicar el ejemplo podemos añadirlo directamente al código:

![placeholder]({{ site.url }}/assets/img/0013_20171020_templated_docs_new_key_dict.png)

Y al generar el documento, ese campo se pasará a la plantilla para que lo rellene automáticamente:

![placeholder]({{ site.url }}/assets/img/0014_20171020_templated_docs_new_doc.png)

Aquí está. Obviamente, ahora el precio hemos tenido que subirlo a 15$, ya que ahora lleva el nombre del blog ;)

De esta manera podemos automatizar la generación de documentos, ya sean facturas como en el ejemplo, informes, o cualquier otro tipo de documento. En la documentación del paquete se muestra cómo introducir bucles for, para poder añadir listas de objetos, imágenes, así como cómo podemos hacer para generar hojas de cálculo o presentaciones.
