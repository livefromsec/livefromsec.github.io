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

Hace falta tener una máquina (Windows) y la aplicación vulnerable (que se puede descargar del GitHub que hemos visto antes). Además, hace falta un debugger (voy a usar [Inmunity](https://www.immunityinc.com/products/debugger/)), y también un módulo para el debugger [Mona](https://github.com/corelan/mona).

# Escaneo inicial

Igual que hemos hecho en ejemplos anteriores:
* [Basic pentesting](https://livefromsec.github.io/2018-06-04/vulnhub_walkthrough_basic_pentesting)
* [Bob: 101](https://livefromsec.github.io/2018-06-11/vulnhub_walkthrough_bob_101)
* [Toppo](https://livefromsec.github.io/2018-07-16/vulnhub_walkthrough_toppo_1)

Empiezo lanzando Nmap y analizando los puertos abiertos. Por defecto, la aplicación escucha en el puerto 9999, así que el escaneo servirá para confirmar que está a la escucha:

* _nmap -A -p 9999 <IP_Windows_VulnServer>_

Aparecerá "open", así que efectivamente, el servidor está a la escucha en ese puerto.

Siguiente paso, conecto a través de netcat, para ver qué opciones me ofrece:

Imagen 001

# Crasheo de la aplicación

En esta entrada, la explotación va a ser la del buffer overflow en el comando TRUN.

El primer paso es conseguir "crashear" la aplicación, para ello generamos un script (si has leído más entradas, puedes imaginar que con Python ([¿Por qué Python?](https://livefromsec.github.io/2017-10-27/por-que-python)).

<<Imagen del script>>
  
Vamos incrementando la longitud de la cadena de "A" que mandamos, hasta que desborda el búffer, sobreescribe registros que no debería, la aplicación no sabe cómo continuar... y se produce el crash.

<<Imagen del crash>>
  
Como vemos, al enviar una cadena de XXX "A" se ha producido el error, sobrescribiendo el registro EIP y también llenando ESP con "A".

# Control de EIP

Mirando en Inmunity, se ve que ESP se ha llenado con "A", que empiezan a partir de la dirección: MIRAR. Y que EIP también tiene "A".

EIP es un registro que indica la siguiente instrucción a ejecutar por lo que si apuntase a ESP, y en ESP pongo algo que quiero que se ejecute, efectivamente conseguiré su ejecución: el procesador accederá a EIP, de ahí irá a ESP y empezará a ejecutar.

Empezamos por controlar el contenido de EIP. Ahora mismo tiene "41" que es la representación en hexadecimal de "A", pero si en lugar de XXX "A" envío una cadena que no se repita, podré identificar el punto en el que se escribe en EIP.

Para generar esta cadena, voy a usar el módulo Mona:
* _!mona pattern_create 2150_

Se abre el log de Inmunity e indica que el fichero se ha creado correctamente, con el nombre pattern.txt. Abro el fichero, y sustituyo la cadena de "A" por la nueva cadena.

Ejecuto de nuevo la prueba de concepto y esta vez, en lugar de 41414141, EIP contendrá otra cadena (33445566, es este caso.)

De nuevo uso Mona, para calcular el offset que hay:
* _!mona pattern_offset 386F4337_

Ahora que ya conozco el offset, puedo definir la variable buffer en la prueba de concepto de la siguiente manera:

* EIP = “\x42\x42\x42\x42” #BBBB
* buffer = "TRUN /.:/" + "A" * 2003 + EIP + "C" * (XXX - 2003 - 4)

Al ejecutar, si todo ha ido bien, en EIP aparecerán 4 "B"; y ESP estará lleno de "C".

# Salto a ESP

Una vez que puedo poner el valor que quiera en EIP, el siguiente paso es redigirir la ejecución a ESP. Para ello, lo más sencillo es buscar una instrucción que haga ese salto, como un "JMP ESP".

De nuevo recurrimos a Mona:
* _!mona jmp -r esp_

En el listado que tenemos, buscamos un salto que no tenga las medidas de protección activas (sin ASLR, sin SafeSEH, etc.). Por ejemplo, en este caso, cojo el primero.

Como las arquitecturas habituales son little endian los bytes más pequeños se almacenan primero, así que hay que invertir el orden de los bytes al indicar la dirección:
* _AAAA_

Confirmo que es un "JMP ESP" (hexadecimal FFE4), y vuelvo a confirmar, añadiendo un punto de parada. ¿Por qué? Porque de lo contrario la ejecución funcionará, se realizará el salto a ESP correctamente, y el error se producirá más adelante, por lo que no veré en EIP la dirección que me interesa.

Ejecuto de nuevo el exploit, y compruebo que la ejecución se para al llegar a esa instrucción, con la dirección XXX en EIP.

# Comprobación de los badcharsCreación de la shellcode

Una vez 
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
