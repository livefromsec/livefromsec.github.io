---
layout: post
title: "Publicado el top ten OWASP 2017"
author: "Pete"
---

Bienvenid@ otra semana más a LiveFromSec. 

Para este viernes 24 (black friday!) me traigo la publicación de la versión final del OWASP top ten 2017. La versión preliminar (release candidate) fue publicada hace varios meses, antes de verano, y este lunes se ha publicado la versión final.

OWASP [wiki](https://es.wikipedia.org/wiki/Open_Web_Application_Security_Project) publica trianualmente su top ten de vulnerabilidades web. El pdf de este año lo puedes encontrar en su github [pdf](https://github.com/OWASP/Top10/blob/master/2017/OWASP%20Top%2010-2017%20(en).pdf)... y otra vez, la inyección SQL gana. Y eso que los prepared statements llevan entre nosotros mucho tiempo.

En la siguiente imagen puedes ver cómo queda el top ten, mapeado con el de la versión previa (2013):

![placeholder]({{ site.url }}/assets/img/0016_20171124_owasp_top_ten.png)

Los XSS caen 4 puntos, hasta la séptima posición; y los CSRF salen del top ten. El pódium lo completan los problemas con la autenticación (que repite en el segundo puesto) y la exposición de datos sensibles, que sube desde el sexto puesto al tercero.

![placeholder]({{ site.url }}/assets/img/0017_20171124_owasp_top_three.png)

En resumen, muy buen documento que **sintetiza en 20 páginas los principales riesgos a los que se enfrentan las aplicaciones web**.

Y aprovechando que es 24 de noviembre, pequeño offtopic: como es el black friday, te recomiendo que le eches un ojo a la página de Shodan. Shodan es un scanner de IPv4 en el que se puede encontrar muchísima información, y ha repetido su habitual promoción de una suscripción permanente por un precio irrisorio. Si te dedicas a la seguridad informática, seguro que lo conoces ;)
