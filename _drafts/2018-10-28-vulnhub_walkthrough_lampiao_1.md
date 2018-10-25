---
layout: post
title: "Lampiao: 1 Walkthrough"
author: "Pete"
---

Bienvenido de nuevo. Aprovechando que tenía un rato libre he aprovechado para jugar un poco con la última máquina que han subido a Vulnhub ([Lampiao: 1](https://www.vulnhub.com/entry/lampiao-1,249/)).

Repetimos la metodología de las máquinas anteriores:
* [Basic pentesting](https://livefromsec.github.io/2018-06-04/vulnhub_walkthrough_basic_pentesting)
* [Bob: 101](https://livefromsec.github.io/2018-06-11/vulnhub_walkthrough_bob_101)
* [Toppo 1](https://www.vulnhub.com/entry/toppo-1,245/)

# Preparación

En este caso no hace falta que usemos _arp-scan_, ya que cuando la máquina arranca nos muestra la dirección que ha cogido por DHCP.

![placeholder]({{ site.url }}/assets/img/0046_20180716_0001.png)

# Escaneo inicial

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


# Flag

Hecho!
