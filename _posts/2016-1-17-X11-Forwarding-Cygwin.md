---
layout: post
title: X11 Forwarding con Cygwin
category: Windows
tags:
- X11
- forwarding
- cygwin
- Windows
---

A veces es necesario ejecutar aplicaciones gráficas de forma remota, una de las formas de conseguirlo es 'redirigiendo las X' a través de un tunel SSH. Para esta tarea necesitamos el Servidor X en Windows, los mas conocidos son Xming y Cygwin.

 * Si no tenemos instalado Cygwin, lo descargamos e instalamos, durante la instalación marcamos también los paquetes xorg-server, xinit y xhost.

![alt text](/images/cygwin-setup.png "Cygwin Setup")

 * Terminada la instalación creamos un acceso directo a XWin Server en el escritorio.

![alt text](/images/cygwin-shortcut.png "Cygwin Shortcut")

 * Editamos el acceso directo y añadimos dos nuevas opciones. La primera es la importante, con ella le indicamos al servidor que ha de escuchar las conexiones TCP/IP.

![alt text](/images/cygwin-prop.png "Shortcut Properities")

 * Abrimos XWin con el acceso directo modificado y deberían de aparecer dos nuevos iconos en el area de notificación. A continuación abrimos el cliente SSH, en este caso PuTTY, introducimos los datos del servidor y marcamos la opción de X11 forwarding antes de conectarnos.

![alt text](/images/putty-x11.png "Putty X11")

 * Una vez conectados al servidor comprobamos la salida de la variable DISPLAY.

{% highlight bash %}
$ echo $DISPLAY
localhost:10.0
{% endhighlight %}

 * Abrimos ahora Cygwin Terminal y hacemos lo mismo, en caso de estar vacía le damos el valor del display que nos marque el icono de Cygwin (en mi caso :0.0) en el area de notificaciones. Por último también nos añadimos a la lista de control del servidor X.

![alt text](/images/cygwin-terminal.png "Cygwin Terminal")

{% highlight bash %}
$ export DISPLAY=:0
$ xhost +localhost
{% endhighlight %}

![alt text](/images/x11-forwarding.png "X11 Forwarding Funcionando")
