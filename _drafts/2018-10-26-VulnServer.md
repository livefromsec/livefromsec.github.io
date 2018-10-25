---
layout: post
title: "VulnServer - Basic Exploitation"
author: "Pete"
---

En la entrada de hoy voy a describir cómo hacer la explotación básica de VulnServer. VulnServer es una aplicación diseñada para practicar la explotación de buffer overflows. 

¿Qué es un buffer overflow? Wikipedia al rescate [Buffer overflow](https://en.wikipedia.org/wiki/Buffer_overflow) :) Un buffer overflow se resume en que la aplicación no controla el tamaño de la entrada que le suministra el usuario, y llega un momento en el que desborda su espacio asignado. Al desbordarlo se sobreescriben registros y otras zonas sobre las que, a priori, no se debería poder escribir. 

El repositorio en el que se encuentra la aplicación es [GitHub](https://github.com/stephenbradshaw/vulnserver) y la página web de su creador es [The Grey Corner](http://www.thegreycorner.com/). 

Una vez hechas las presentaciones, vamos a ver cómo podemos explotar la aplicación! En este caso la metodología es diferente, ya que necesitamos una máquina de pruebas y un debugger con el que ir paso a paso.

# Preparación

Hace falta tener una máquina (Windows) y la aplicación vulnerable (que se puede descargar del GitHub que hemos visto antes). Además, hace falta un debugger (voy a usar [Inmunity](https://www.immunityinc.com/products/debugger/), y también un módulo para el debugger [Mona, en Github](https://github.com/corelan/mona).

# Escaneo inicial

Igual que hemos hecho en ejemplos anteriores:
* [Basic pentesting](https://livefromsec.github.io/2018-06-04/vulnhub_walkthrough_basic_pentesting)
* [Bob: 101](https://livefromsec.github.io/2018-06-11/vulnhub_walkthrough_bob_101)
* [Toppo](https://livefromsec.github.io/2018-07-16/vulnhub_walkthrough_toppo_1)

Empiezo lanzando Nmap y analizando los puertos abiertos. Por defecto, la aplicación escucha en el puerto 9999, así que el escaneo servirá para confirmar que está a la escucha:

![placeholder]({{ site.url }}/assets/img/0047_20180716_0002.png)

Efectivamente, el servidor está a la escucha en ese puerto.

Siguiente paso, conecto a través de netcat, para ver qué opciones me ofrece:

# Crasheo de la aplicación

En esta entrada, la explotación va a ser la del buffer overflow en el comando TRUN.

El primer paso es conseguir "crashear" la aplicación, para ello generamos un script (en mi caso, con Python ;) [¿Por qué Python?](https://livefromsec.github.io/2017-10-27/por-que-python).

<<Imagen del script>>
  
Vamos incrementando la longitud de la cadena de "A" que mandamos, hasta que desborda el búffer, sobreescribe registros que no debería... y la aplicación no sabe cómo continuar.

<<Imagen del crash>>
  
Como vemos, 

--- 

Igual que hasta ahora, comenzamos lanzando nmap y viendo qué puertos tiene abiertos la máquina:

El resultado: la máquina tiene un servidor web, un servidor ftp, y algún puerto más abierto.:

![placeholder]({{ site.url }}/assets/img/0047_20180716_0002.png)

Como habitualmente las máquinas vulnerables que tienen los puertos 80/443 suelen tener ahí los puntos de entrada, lanzamos nikto al puerto 80:

* _nikto -h 10.0.6.89_

El servidor tiene una serie de carpetas interesantes: entre ellas "admin". Al ver qué hay en la carpeta, comprobamos que el servidor permite "directory listing", por lo que vemos que tiene un fichero "notes.txt". Lo abrimos y vemos una password :)

# Consiguiendo la shell

Nikto nos muestra que hay otra carpeta interesante, con la que igual se pueden ejecutar comandos pasándolos por GET o POST... pero esto tiene tan buena pinta, que es difícil resistirse a intentar hacer login en la máquina por SSH.

Probamos el usuario root, y no funciona; y puesto que la clave tiene la cadena "ted", probamos a ver si el usuario ted existe... bingo!

![placeholder]({{ site.url }}/assets/img/0048_20180716_0003.png)

Usando el comando id vemos que somos un usuario normal:

* _id_

![placeholder]({{ site.url }}/assets/img/0049_20180716_0004.png)

Probamos qué podemos hacer con esta shell y parece que está limitada, por ejemplo _ifconfig_ no deja ejecutarlo. Y con _help_ muestra un listado bastante corto de comandos. Intentamos abrir una shell que esté menos limitada:

* _python -c 'import pty; pty.spawn("/bin/sh")'_

Funciona, y no solo eso, también vemos que ahora mismo, el EUID del usuario es 0... Vamos a ver si podemos hacer maldades :)

![placeholder]({{ site.url }}/assets/img/0050_20180716_0005.png)

Perfectamente, podemos entrar en la carpeta /root/ y listar el contenido!

# Enumeración

No hace falta :) 

# Escalada de privilegios

Tampoco hace mucha falta... Podemos ver el kernel de la máquina, etc. Pero llegados a este punto, únicamente nos falta leer el flag:

![placeholder]({{ site.url }}/assets/img/0051_20180716_0006.png)
