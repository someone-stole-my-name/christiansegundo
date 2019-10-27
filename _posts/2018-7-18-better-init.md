---
layout: post
title: better-initramfs (Dropbear + LVM + LUKS) setup
category: Gnu-Linux
tags:
- initramfs
- linux
- dropbear
- lvm
- luks
---

Desde hace años que uso better-initramfs en todos los sistemas que necesiten un initramfs y es que como su nombre anuncia, es mejor que todas las alternativas IMO (mkinit, genkernel, dracut...etc).
Es rápido, tiene soporte para lvm, luks, dropbear! y encima es totalmente independiente del kernel!, una vez 'compilado' te puedes olvidar (cuidado con los 0days de Dropbear)... todo ventajas. Esta es mi configuración para 
un servidor con LVM on LUKS en un SSD.

<hr>

# Instalación desde git
{% highlight code %}
# cd /opt
# git clone https://bitbucket.org/piotrkarbowski/better-initramfs.git
{% endhighlight %}

*La compilación la hace en un chroot por lo que hace falta hacerlo como root!*
{% highlight code %}
# cd better-initramfs
# bootstrap/bootstrap-all
# make prepare
{% endhighlight %}

## Shell remota (Opcional)
Antes de crear la imagen tenemos que añadir las claves públicas que tendrán acceso.
{% highlight code %}
# cp /rutal/a/.ssh/authorized_keys /opt/better-initramfs/sourceroot
{% endhighlight %}

# Crear e instalar la imagen
{% highlight code %}
# make image
# cp /opt/better-initramfs/output/initramfs.cpio.gz /boot
{% endhighlight %}

# Entrada de GRUB
better-initramfs es muy flexible y soporta muchísimas configuraciones, en la [página oficial](https://bitbucket.org/piotrkarbowski/better-initramfs#rst-header-parameters) están explicadas al detalle todas las opciones.
Es importante consultar la documentación oficial pues algunos parámetros pueden cambiar con el tiempo o dejar de existir (eg `luks_trim` ya no existe).

Editamos el fichero `/etc/grub.d/40_custom` y añadimos una nueva entrada, en mi caso queda así:
{% highlight code %}
#!/bin/sh
exec tail -n +3 $0
# This file provides an easy way to add custom menu entries.  Simply type the
# menu entries you want to add after this comment.  Be careful not to change
# the 'exec tail' line above.

menuentry 'Gentoo Linux 4.17.7 (better-init)' --class gentoo --class gnu-linux --class gnu --class os {
	load_video
	insmod gzio
	insmod part_msdos
	insmod fat
	set root='hd2,msdos1'
	search --no-floppy --fs-uuid --set=root 2ACF-01EF
	echo	'Loading Linux 4.17.7 ...'
	linux	/vmlinuz-4.17.7-gentoo \
	root=/dev/mapper/vg1-root ro lvm luks sshd\
	binit_net_if=eth3 binit_net_addr=10.0.0.242/24\
	binit_net_gw=10.0.0.1\
	enc_root=UUID=ca098a60-f942-43be-9138-bcb06922211f\
	rootfstype=ext4 
	echo	'Loading initial ramdisk ...'
	initrd	/initramfs.cpio.gz
}
{% endhighlight %}

*La mayoria de líneas se pueden copiar descaradamente de las entradas que genera GRUB automáticamente!* 

Básicamente las únicas dos líneas en las que estamos interesados son la del kernel (`linux ...`), a la que se le pasan todos los parámetros y 
el initrd que debe apuntar a la imagen creada anteriormente (aparte del título, los `echo` y demás pijadas).

`root=` : partición con la raiz (`/`).  
`lvm` : escanea los discos en busca de volúmenes.  
`enc_root=UUID=...` : especifica el UUID de la partición cifrada.  
`luks` : indica que queremos hacer `luksOpen` en `enc_root`.  
`ro` : monta la raiz en read-only.  
`binit_net_if` `binit_net_addr` `binit_net_gw` : interfaz, dirección y gateway que queremos usar.  
`sshd` : ejecutar el servidor ssh para poder introducir la clave de forma remota.  
`rootfstype` : el formato de la partición raíz.

Montamos `/boot` si está en otra partición y regeneramos el `grub.cfg`.

{% highlight code %}
# mount /boot
# grub-mkconfig -o /boot/grub/grub.cfg
{% endhighlight %}

## Cambiar el orden de las entradas de GRUB (Opcional)

El número delante de los ficheros en `/etc/grub.d` especifica el orden en el que son procesados, para que la entrada personalizada sea seleccionada por defecto, 
basta con mover el fichero `40_custom` editado anteriormente y regenerar `grub.cfg`.

{% highlight code %}
# mv /etc/grub.d/40_custom /etc/grub.d/05_better-initramfs
# grub-mkconfig -o /boot/grub/grub.cfg
{% endhighlight %}

----

## Fuentes:

[better-initramfs](https://bitbucket.org/piotrkarbowski/better-initramfs#rst-header-build-from-source)
