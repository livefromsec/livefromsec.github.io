---
layout: post
title: "Usando alias en Linux"
author: "Pete"
---

Lo bueno de tener un blog es que lo puedes usar como bloc de notas, así que la gente con mala memoria podemos aprovechar para dejarnos aquí referencias a cosas que utilizamos frecuentemente y luego lo tenemos disponible a un click :)

En la entrada de hoy voy a escribir sobre los "alias" de Linux: son una manera de abreviar los comandos [Wiki: Alias](https://es.wikipedia.org/wiki/Alias_(Unix)). Con esto podemos resumir los comandos más utilizados, dejándolos en comandos mucho más manejables.

Por ejemplo, en mi caso me gusta tener las Kali Linux siempre actualizadas a la última versión disponible de cada paquete. Podría ejecutar cada vez que trabajo con ellas "apt-get update && apt-get upgrade -y && apt-get dist-upgrade -y && apt autoremove -y", pero parece un comando un poco largo para tener que escribirlo manualmente cada vez.

Una opción sería listar los comandos ya ejecutados con "history" y luego llamarlo con "!número_del_comando", pero me parece más sencillo definir un alias, por ejemplo "actualiza", que se encargue de hacer todo.

Para ello modificamos el fichero bashrc que se encuentra en nuestro home: en el caso de Kali corriendo como root, en /root. Cualquier editor nos vale, con nano por ejemplo: nano .bashrc (o si no estamos en el home, nano /etc/.bashrc). Como no voy a añadir muchos, no me hará falta utilizar un fichero aparte, un .bash_aliases; únicamente necesitaré añadir una línea en el .bashrc en la parte final, después de los alias, que diga:

alias actualiza='apt-get update && apt-get upgrade -y && apt-get dist-upgrade -y && apt autoremove -y'

y solucionado, con el alias "actualiza" tengo la máquina actualizada en 4 pulsaciones de teclado. 
