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

Puesto que la máquina no nos dice directamente la IP que ha cogido por DHCP, tenemos que usar arp-scan:
* _arp-scan -l_
* Arrancamos la máquina:
* _arp-scan -l_

Y obtenemos la dirección IP de Lampiao. Como en cada caso variará, a partir de ahora a esa dirección la llamaré <IP_Lampiao>. 

# Escaneo inicial

Lanzo nmap y veo qué puertos tiene abiertos la máquina:
* _nmap -p- -A <IP_Lampiao>_

El resultado: la máquina tiene el puerto del SSH, y luego un servidor web corriendo en el puerto 1898, y el puerto 80 abierto.

Lanzo nikto a ambos puertos:

* _nikto -h <IP_Lampiao>_
* _nikto -h <IP_Lampiao> -p 1898_

Nikto indica que el puerto 80 no tiene un servidor web. Así que accedo a través de Firefox para ver qué muestra, y únicamente aparece una imagen hecha en caracteres ASCII:

Imagen

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
