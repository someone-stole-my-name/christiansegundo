---
layout: post
title: Bubblewrap y Firefox
category: Gnu-Linux
tags:
- firefox
- sandbox
- container
---

Si hay algo que está presente en todos los ordenadores personales es el navegador. Un programa normalmente gigantesco, lleno de fallos, que recibe y ejecuta código igualmente grande y no auditado. ¿Hacen falta mas razones para ejecutarlo en un entorno lo mas reducido posible?

<hr>

`$ bwrap [OPTION...] [COMMAND]`

***Importante**, las opciones se interpretan de izquierda a derecha!*

Ejecutar Firefox en la sandbox es tan sencillo como:

  * Crear bind mounts como read-only a los binarios, librerias...etc
  
  ```
  --ro-bind /usr /usr
  --ro-bind /bin /bin
  --ro-bind /lib64 /lib64
  ```
  * Crear bind mounts como read-only a la configuración DNS, fuentes y la cache de ld.

  ```
  --ro-bind /etc/resolv.conf /etc/resolv.conf
  --ro-bind /etc/fonts /etc/fonts
  --ro-bind /etc/ld.so.cache /etc/ld.so.cache
  ```
  * Crear bind mounts al profile existente y al directorio Downloads.
  
  ```
  --bind ~/.cache/mozilla ~/.cache/mozilla
  --bind ~/.mozilla ~/.mozilla
  --bind ~/Downloads ~/Downloads
  ```
  * PulseAudio

  ```
  --ro-bind ~/.config/pulse ~/.config/pulse
  --bind /run/user/$UID/pulse /run/user/$UID/pulse
  --setenv XDG_RUNTIME_DIR "/run/user/$UID"
  ```
  * Montar procfs, un nuevo devtmpfs y un tmpfs en `/tmp`.

  ```
  --proc /proc
  --dev /dev
  --tmpfs /tmp
  ```
  * Crear nuevos namespaces y mantener unicamente el de red.
  
  ```
  --unshare-all
  --share-net
  ```
  * CVE-2017-5226 [^1]

  [^1]: [CVE-2017-5226 -- bubblewrap escape via TIOCSTI ioctl](https://github.com/projectatomic/bubblewrap/issues/142)
  
  ```
  --new-session
  ```

<hr>

{% highlight code %}
$ bwrap \
--ro-bind /usr /usr \
--ro-bind /bin /bin \
--ro-bind /lib64 /lib64 \
--ro-bind /etc/resolv.conf /etc/resolv.conf \
--ro-bind /etc/fonts /etc/fonts \
--ro-bind /etc/ld.so.cache /etc/ld.so.cache \
--bind ~/.cache/mozilla ~/.cache/mozilla \
--bind ~/.mozilla ~/.mozilla \
--bind ~/Downloads ~/Downloads \
--ro-bind ~/.config/pulse ~/.config/pulse \
--bind /run/user/$UID/pulse /run/user/$UID/pulse \
--setenv XDG_RUNTIME_DIR "/run/user/$UID" \
--proc /proc \
--dev /dev \
--tmpfs /tmp \
--unshare-all \
--share-net \
--new-session \
firefox
{% endhighlight %}

<hr>

## xdg-open

La extensión perl-matcher de urxvt (y otros programas que usen XDG) no funciona a no ser que tengamos un MIME registrado. Lo primero es crear un script para abrir Firefox.

{% highlight code %}
#!/bin/bash

bwrap \
--ro-bind /usr /usr \
--ro-bind /bin /bin \
--ro-bind /lib64 /lib64 \
--ro-bind /etc/resolv.conf /etc/resolv.conf \
--ro-bind /etc/fonts /etc/fonts \
--ro-bind /etc/ld.so.cache /etc/ld.so.cache \
--bind ~/.cache/mozilla ~/.cache/mozilla \
--bind ~/.mozilla ~/.mozilla \
--bind ~/Downloads ~/Downloads \
--ro-bind ~/.config/pulse ~/.config/pulse \
--bind /run/user/$UID/pulse /run/user/$UID/pulse \
--setenv XDG_RUNTIME_DIR "/run/user/$UID" \
--proc /proc \
--dev /dev \
--tmpfs /tmp \
--unshare-all \
--share-net \
--new-session \
firefox "${@}"
{% endhighlight %}

Copiamos el fichero firefox.desktop a `~/.local/share/applications` y modificamos Exec= para que apunte al script anterior.

{% highlight code %}
$ cp /usr/share/applications/firefox.desktop ~/.local/share/applications/sanfox.desktop
$ vim ~/.local/share/applications/sandfox.desktop
{% endhighlight %}

{% highlight code %}
Exec=/path/to/firefox/script %u
{% endhighlight %}

Añadimos sandfox.desktop como navegador por defecto.[^2]

[^2]: [[SOLVED] Change default web browser](https://bbs.archlinux.org/viewtopic.php?id=140028)

{% highlight code %}
$ vim .local/share/applications/mimeapps.list

[Default Applications]
text/html=sandfox.desktop
x-scheme-handler/http=sandfox.desktop
x-scheme-handler/https=sandfox.desktop
{% endhighlight %}


----

# Fuentes:

[Bubblewrap -- Unprivileged sandboxing tool](https://github.com/projectatomic/bubblewrap)
