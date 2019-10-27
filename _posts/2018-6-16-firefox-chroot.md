---
layout: post
title: Firefox en una jaula chroot
category: Gnu-Linux
tags:
- firefox
- chroot
- gentoo
- container
---
<strong>El propósito de esta chroot no es la seguridad</strong>, es tener múltiples versiones instaladas simultáneamente y aisladas del sistema principal. Este mismo procedimiento se puede aplicar a cualquier jaula para aplicaciones gráficas (eg Steam;).

### Contenido

  * [Chroot](#chroot)
    * [Aceleración gráfica](#aceleración-gráfica)
	* [PulseAudio](#pulseaudio)
  * [Script](#script)
  * [urxvt y perl-matcher](#urxvt-y-perl-matcher)

<hr>

# Chroot

Creamos el directorio del chroot.

{% highlight code %}
# mkdir /usr/local/firefox
# cd /usr/local/firefox
{% endhighlight %}

Descargamos el tarball y lo extraemos.

{% highlight code %}
# wget http://distfiles.gentoo.org/releases/amd64/autobuilds/20180624T214502Z/stage3-amd64-20180624T214502Z.tar.xz
# tar xvJpf stage3-amd64-20180624T214502Z.tar.xz
{% endhighlight %}

Copiamos los DNS

{% highlight code %}
# cp -L /etc/resolv.conf etc 
{% endhighlight %}

Creamos el directorio de portage y montamos los sistemas de fichero.

{% highlight code %}
# chroot_dir="/usr/local/firefox"
# mkdir "${chroot_dir}/usr/portage"
# mount -R /dev "${chroot_dir}/dev"
# mount -R /run "${chroot_dir}/run"
# mount -R /sys "${chroot_dir}/sys"
# mount -t proc proc "${chroot_dir}/proc"
# mount -R /usr/portage "${chroot_dir}/usr/portage"
{% endhighlight %}

Cambiamos de root y creamos un nuevo usuario.

{% highlight code %}
# chroot .
# env-update && source /etc/profile
# export PS1="(chroot) ${PS1}"
# useradd -m firefox
{% endhighlight %}

Se modifica el fichero `make.conf` según el handbook y por último se instala Firefox.

## Aceleración gráfica

Esto es opcional y solo necesario si queremos aceleración gráfica en la chroot. Se instala la misma versión de los drivers compilados contra el mismo kernel del host(`mount -R /usr/src/linux "${chroot_dir}/usr/src/linux"`).

Se añade firefox al grupo video.
{% highlight code %}
# usermod -aG video firefox
{% endhighlight %}

## PulseAudio

Este paso es también completamente opcional y solo necesario si se quiere tener audio. Necesitamos <strong>instalar</strong> PulseAudio si no se instaló como dependencia de Firefox y PulseAudio en el <strong>host</strong>.

### Configuración en el host [^1]

[^1]: [PulseAudio configuration](https://wiki.gentoo.org/wiki/PulseAudio#Configuration)

ACLs en el kernel.

{% highlight code %}
File systems  --->
   Pseudo filesystems  --->
      [*] Tmpfs virtual memory file system support (former shm fs)
      [*]    Tmpfs POSIX Access Control Lists
{% endhighlight %}

ConsoleKit (sys-auth/consolekit) instalado con la USE `acl` y PAM (sys-auth/pambase) con `consolekit`. Añadimos consolekit al inicio y lo iniciamos.

{% highlight code %}
# rc-update add consolekit default
# rc-service consolekit start
{% endhighlight %}

<strong>Es importante que ningún usuario pertenezca al grupo `audio`.</strong>
{% highlight code %}
# gpasswd -d <user> audio
{% endhighlight %}


Configuramos `sudo` para que conserve el `XDG_RUNTIME_DIR`. [^2]

[^2]: [How do I play sounds from my chroot to my host's Pulseaudio daemon?](https://www.freedesktop.org/wiki/Software/PulseAudio/FAQ/#index37h3)

{% highlight code %}
$ sudo visudo

Defaults env_keep +="XDG_RUNTIME_DIR"
{% endhighlight %}


# Script

{% highlight code %}
#!/bin/sh

# chroot directory
chroot_dir="/usr/local/gentoo-64_Firefox"
chroot_user="firefox"

# check if firefox is already running
if pgrep -x "firefox" > /dev/null; then
	chroot "${chroot_dir}" su ${chroot_user} -c "dbus-launch firefox ${@}"
	exit
fi

# use this if running multiple instances in different roots
# for ROOT in /proc/*/root; do
#	LINK=$(readlink $ROOT)
#    if [ "x$LINK" != "x" ]; then
#        if [ "x${LINK:0:${#chroot_dir}}" = "x$chroot_dir" ]; then
#			PID=$(basename $(dirname "$ROOT"))
#			NAME=$(cat /proc/${PID}/status |grep Name|cut -d':' -f2 |sed 's/ //g')
#			if [ $NAME = "firefox" ]; then
#				chroot "${chroot_dir}" su ${chroot_user} -c "dbus-launch firefox ${@}"
#				exit
#			fi
#        fi
#    fi
# done

# mount directories
mount -R /dev "${chroot_dir}/dev"
mount -R /home/christian/.config/pulse "${chroot_dir}/home/${chroot_user}/.config/pulse"
mount -R /run "${chroot_dir}/run"
mount -R /sys "${chroot_dir}/sys"
mount -R /usr/portage "${chroot_dir}/usr/portage"
mount -R /var/lib/dbus "${chroot_dir}/var/lib/dbus"
mount -t proc proc "${chroot_dir}/proc"

# check dbus session
if [ -z "${DBUS_SESSION_BUS_ADDRESS}" ] ; then
  chroot "${chroot_dir}" dbus-launch
fi
# chroot
chroot "${chroot_dir}" su ${chroot_user} -c "dbus-launch firefox ${@}"

# kill any running process
for ROOT in /proc/*/root; do
	LINK=$(readlink $ROOT)
    if [ "x$LINK" != "x" ]; then
        if [ "x${LINK:0:${#chroot_dir}}" = "x$chroot_dir" ]; then
            PID=$(basename $(dirname "$ROOT"))
            kill -9 "$PID"
        fi
    fi
done

# unmount directories
umount -l "${chroot_dir}/dev"
umount -l "${chroot_dir}/home/${chroot_user}/.config/pulse"
umount -l "${chroot_dir}/proc"
umount -l "${chroot_dir}/run"
umount -l "${chroot_dir}/sys"
umount -l "${chroot_dir}/usr/portage"
umount -l "${chroot_dir}/var/lib/dbus"
{% endhighlight %}

## sudo

Creamos un enlace al script.

{% highlight code %}
# ln -s /path/to/script.sh /usr/local/bin/firefox
{% endhighlight %}

Permitimos que el grupo `wheel` pueda ejecutarlo como root.

{% highlight code %}
# visudo

%wheel ALL=(ALL) NOPASSWD: /usr/local/bin/firefox
{% endhighlight %}

# urxvt y perl-matcher

La extensión `perl-matcher` (y otros programas que usen XDG) no funciona a no ser que tengamos un MIME registrado.

Copiamos el fichero firefox.desktop que se debería haber instalado en la chroot a `~/.local/share/applications` y modificamos `Exec=` para que inicie el script con privilegios.

{% highlight code %}
$ cp /usr/local/firefox/usr/share/applications/firefox.desktop ~/.local/share/applications/
$ vim ~/.local/share/applications/firefox.desktop
{% endhighlight %}
{% highlight code %}
Exec=sudo firefox %u 
{% endhighlight %}

Creamos un enlace al script y añadimos firefox.desktop como navegador por defecto.[^3]

[^3]: [[SOLVED] Change default web browser](https://bbs.archlinux.org/viewtopic.php?id=140028)

{% highlight code %}
$ vim ~/.local/share/applications/mimeapps.list

[Default Applications]
x-scheme-handler/http=firefox.desktop
x-scheme-handler/https=firefox.desktop
{% endhighlight %}

----

# Fuentes:
