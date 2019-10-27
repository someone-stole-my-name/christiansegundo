---
layout: post
title: Crea tu propio Stage4
category: Gnu-Linux
tags:
- linux
- gentoo
- funtoo
---

Los Stages son ficheros que contienen un entorno con (dependiendo
del Stage) todos los binarios, directorios y ficheros de configuración
necesarios para tener un sistema funcional. La instalación de Funtoo o Gentoo
por ejemplo se realiza mediante un Stage3. Crear nuestro propio Stage4 es útil
cuando queremos clonar todo el sistema de un equipo a otro o tener un backup
completo en caso de que suceda algo inesperado.

{% highlight code %}
# mount /boot
# cd
# tar --exclude=/run --exclude=/dev --exclude=/proc --exclude=/sys --exclude=/tmp --exclude=/root/stage4.tar.bz2 -cvjf --xattr stage4.tar.bz2 /
{% endhighlight %}
