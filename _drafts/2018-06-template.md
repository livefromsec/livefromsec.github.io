---
layout: post
title: "Bob Walkthrough"
author: "Pete"
---

Después del post previo con la resolución de una máquina, aprovechando que tenía un poco de tiempo, me puse con otra máquina de Vulnhub. Con cada máquina se aprende algo nuevo! 

La máquina en cuestión es Bob [Vulnhub](https://www.vulnhub.com/entry/bob-101,226/).

Seguimos los mismos pasos que en la [entrada anterior](https://livefromsec.github.io/2018-06-04/vulnhub_walkthrough_basic_pentesting).

# Preparación

Tenemos ya nuestra máquina Kali Linux, y tenemos que conocer la dirección de la máquina vulnerable. En el post anterior lo hacíamos comparando visualmente las salidas del comando arp-scan -l, pero en caso de que haya muchas máquinas en la red, tendremos que pasar un rato haciendo esa comparación, así que podemos hacerlo más fácil:

* _arp-scan -l > lista_antes_
* [Encendemos la máquina vulnerable]
* _arp-scan -l > lista_despues_
* _comm lista_antes lista_despues_

Así nos saldrá resaltada la máquina con su IP.

# Escaneo inicial

Una vez que conocemos la dirección IP, lanzamos nmap y vemos qué puertos/servicios tiene la máquina. Usamos los mismos flags que en post previo: -A, --reason y --top-ports.

El resultado: la máquina tiene un servidor web, y el archivo robots.txt ya nos da alguna pista de por dónde empezar a mirar.

![placeholder]({{ site.url }}/assets/img/0016_20171124_owasp_top_ten.png)   -> la imagen de la salida del nmap.

Pues nada, vamos a echarle un ojo a los sitios incluidos en el robots.txt

login.php, dev_shell.php, lat_memo.html y passwords.html

Miro primero en passwords.html... era muy bonito para ser real, los admin nos cuentan que antes alguien se despistó y puso las passwords, pero que ya se ha quitado.

![placeholder]({{ site.url }}/assets/img/0016_20171124_owasp_top_ten.png)   -> la imagen del passwords.

Miro en login.php, que da error, y en login.html que nos informa que está desactivada la página de login.

![placeholder]({{ site.url }}/assets/img/0016_20171124_owasp_top_ten.png)   -> la imagen de la salida del nmap.

Pues nada, en principio no habrá que forzar la inyección SQL en el formulario de login ;) Nos vamos a mirar la consola de desarrollo.

![placeholder]({{ site.url }}/assets/img/0016_20171124_owasp_top_ten.png)   -> la imagen de la consola de dev.

# Consiguiendo la shell

Puede ejecutar comandos, así que vamos a ver si podemos abrirnos una shell con netcat...

![placeholder]({{ site.url }}/assets/img/0016_20171124_owasp_top_ten.png)   -> viendo el troleo

Hummmm... parece que el admin se lo ha tomado en serio. Pruebo todos los trucos de la [entrada pasada](blabla), pero está todo filtrado.

Vamos a ver si con wget podemos subir una shell en .php...

![placeholder]({{ site.url }}/assets/img/0016_20171124_owasp_top_ten.png)   -> imagen diciendo que no.

También está restringido. Pues nada, a ver si se pueden ejecutar dos comandos:
* pruebo separándolos con ";", y también está bloqueado
* pruebo concatenándolos, con "&&".

Ahora sí, concatenamos id con un nc y tenemos nuestra shell 

![placeholder]({{ site.url }}/assets/img/0016_20171124_owasp_top_ten.png)   -> imagen mostrando la shell.

Sacamos una shell con python:

![placeholder]({{ site.url }}/assets/img/0016_20171124_owasp_top_ten.png)   -> imagen mostrando la shell.

y nos ponemos a enumerar.

# Enumeración

En la descripción que daban en Vulnhub, se recomienda buscar ficheros ocultos, así que a la hora de listar ficheros usaré:
* _ls -la_

Le echo un ojo al /etc/passwd con 
* _cat /etc/passwd_

y veo que hay varios usuarios en el sistema: c0rruptedb1t, bob, jc, seb y elliot.

Nos vamos al /home y vamos a ir enumerando.
* seb no tiene nada destacable en su home
* jc no tiene nada destacable en su home
* elliot tiene un fichero llamado theadminisdumb.txt xD Este fichero nos da las credenciales de james (presumiblemente jc) y las suyas.
* bob es el que más cosas tiene: un fichero oculto llamado old_passwordfile.html, que tiene las credenciales de seb y las de jc. Además tiene 2 scripts en python (Hello_Again.py y Whell_Of_Fortune.py), un proftpd1.3.3 (versión backdorizada xD) y en su carpeta Documents tiene un documento poniendo a caldo a los demás (staff.txt), un fichero cifrado (login.txt.gpg) y un script que muestra unas notas en el escritorio.


Entonces, recopilando, tenemos 3 credenciales (las de elliot, jc y seb); un documento diciendo que bob es el administrador y unos cuantos ficheros de bob.




