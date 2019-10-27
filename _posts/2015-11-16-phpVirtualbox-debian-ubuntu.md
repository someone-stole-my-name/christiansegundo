---
layout: post
title: phpVirtualbox en Debian
category: Gnu-Linux
tags: 
- phpvirtualbox
- debian
- ubuntu
- vm
---

phpVirtualbox es una aplicación en PHP que permite administrar VirtualBox gráficamente de forma remota (y local) mediante una interfaz muy similar a la GUI original.

### Contenido

  * [Instalación y configuración](#instalación-y-configuración)
    * [Instalar VirtualBox](#instalar-virtualbox)
  * [vboxwebsrv](#vboxwebsrv)
    * [Iniciar vboxwebsrv en el arranque](#iniciar-vboxwebsrv-en-el-arranque)
      * [systemd](#systemd)
      * [init](#init)
  * [Autenticación externa](#autenticación-externa)
  * [De forma remota](#de-forma-remota)
    * [Un solo servidor](#un-solo-servidor)
    * [Múltiples servidores](#múltiples-servidores)

# Instalación y configuración

Instalamos los paquetes `libapache2-mod-php5` y `apache2`, los nombres lo dicen todo uno es el servidor web y el otro para dar soporte a php.

{% highlight code %}
# aptitude install unzip libapache2-mod-php5 apache2
{% endhighlight %}

Bajamos phpVirtualBox desde la [página oficial](https://sourceforge.net/projects/phpvirtualbox/). Lo descomprimimos y movemos al directorio raíz de la página por defecto, en un subdirectorio o creamos un nuevo virtualhost.

{% highlight code %}
# unzip phpvirtualbox-X.X-X.zip
# mv phpvirtualbox-X.X-X/* /var/www/html
{% endhighlight %}

El paquete viene con un fichero de configuración genérico que debemos utilizar de plantilla para el nuestro.

{% highlight code %}
# cd /var/www/html
# cp config.php-example config.php
{% endhighlight %}

Se edita el nuevo fichero y se cambia el valor de las variables `$username` y `password` por el nombre de usuario que ejecuta VirtualBox.

{% highlight code %}
# nano config.php

var $username = 'dummyuser';
var $password = 'dummypass';
{% endhighlight %}

Por último cambiamos los permisos del directorio raíz de forma recursiva.

{% highlight code %}
# chown -R www-data:www-data /var/www/html
{% endhighlight %}

## Instalar VirtualBox

Si aun no tenemos instalado VirtualBox debemos instalarlo antes de continuar. Podemos descargar el paquete específico para nuestro sistema desde la [página oficial](https://www.virtualbox.org) o añadir el repositorio e instalarlo con apt. Si estamos instalando phpVirtualBox en un servidor distinto al que corre las máquinas virtuales no es necesario.

{% highlight code %}
/etc/apt/sources.list

deb http://download.virtualbox.org/virtualbox/debian codename contrib
{% endhighlight %}

{% highlight code %}
# wget -q https://www.virtualbox.org/download/oracle_vbox.asc -O- | apt-key add -
# aptitude update
# aptitude install virtualbox-5.0
{% endhighlight %}

# vboxwebsrv

phpVirtualbox funciona con la API Web `vboxwebsrv`, podemos comprobar si funciona iniciando `vboxwebsrv` con el usuario definido anteriormente y entrando en `127.0.0.1` o de forma remota con la IP local.

{% highlight code %}
# su vboxuser # Usuario especificado en config.php

$ vboxwebsrv -H 127.0.0.1
{% endhighlight %}

El usuario y contraseña de phpVirtualbox por defecto es admin:admin.

<center><img src="/images/phpVirtualBox-login.png"></center>

<center><img src="/images/phpVirtualBox.png"></center>

## Iniciar vboxwebsrv en el arranque

apache2 se añade automáticamente al inicio pero `vboxwebsrv` no, es convniente añadirlo al sobretodo si se trata de un servidor que siempre tendrá máquinas corriendo.

### systemd

Añadimos el servicio `vboxweb-service.service` al inicio con `systemctl`.

{% highlight code %}
# systemctl enable vboxweb-service.service
# ls -l /etc/systemd/system/multi-user.target.wants/

lrwxrwxrwx 1 root root 13 Mar  5 22:43 vboxweb-service.service -> /lib/systemd/system/vboxweb-service.service
{% endhighlight %}

Debemos crear el fichero `/etc/default/virtualbox`, una configuración mínima puede ser la siguiente:

{% highlight code %}
# nano /etc/default/virtualbox

VBOXWEB_USER=dummyuser
VBOXWEB_HOST=localhost
{% endhighlight %}

### init

Por defecto al instalar VirtualBox se crea el servicio vboxweb-service, lo añadimos al inicio.

{% highlight code %}
# update-rc.d vboxweb-service defaults
{% endhighlight %}

Debemos crear el fichero `/etc/default/virtualbox`, una configuración mínima puede ser la siguiente:

{% highlight code %}
# nano /etc/default/virtualbox

VBOXWEB_USER=dummyuser
VBOXWEB_HOST=localhost
{% endhighlight %}

# Autenticación externa

phpVirtualBox viene con una serie de configuraciones de las que podemos hacer uso, podemos autenticarnos contra un dominio, servidor LDAP, o en este caso la autenticación básica de apache.

Creamos un nuevo usuario desde la interfaz web como en la siguiente imagen.

<center><img src="/images/phpVirtualBox-user.png"></center>

Utilizando `htpasswd` se crea el fichero `.htpasswd` que contiene los usuarios y contraseñas autorizados. Si el fichero no existe como es mi caso, al añadir el primer usuario se ha de especificar la opción `-c`.

{% highlight code %}
# htpasswd -c /etc/apache2/.htpasswd user
# htpasswd /etc/apache2/.htpasswd user1
{% endhighlight %}

Se edita el virtualhost correspondiente y se añade lo siguiente:

{% highlight code %}
# /etc/apache2/sites-enabled/000-default.conf

    <Directory "/var/www/html">
        AuthType Basic
        AuthName "Restricted Content"
        AuthUserFile /etc/apache2/.htpasswd
        Require valid-user
    </Directory>
{% endhighlight %}
Por último modificamos el fichero de configuración de phpVirtualBox y le indicamos que esta será la forma de autenticación.

{% highlight code %}
# config.php

var $authLib = 'WebAuth';
{% endhighlight %}

# De forma remota

Es posible tener el servidor web y phpVirtualBox en un servidor distinto al que corre vboxwebsrv y las máquinas virtuales, incluso es posible hacer que la aplicación interactue con múltiples servidores a la vez.

## Un solo servidor

Debemos seguir los mismos pasos anteriores añadiendo algunas modificaciones. Editamos el fichero de configuración e indicamos la IP y el nombre de usuario y contraseña de la máquina remota.

{% highlight code %}
var $username = 'usuarioremoto';
var $password = 'usuariospass';
var $location = 'http://192.168.1.240:18083';
{% endhighlight %}

En el servidor que corre VirtualBox y `vboxwebsrv` editamos el fichero creado anteriormente y cambiamos localhost por su IP local. Esta directiva le indica en que dirección y puerto debe estar escuchando.

{% highlight code %}
# nano /etc/default/virtualbox

VBOXWEB_USER=dummyuser
VBOXWEB_HOST=192.168.1.240
{% endhighlight %}

Si reiniciamos el servicio veremos los cambios.

{% highlight code %}
# service vboxweb-service restart
# service vboxweb-service status

CGroup: /system.slice/vboxweb-service.service
           ├─  762 /usr/lib/virtualbox/VBoxXPCOMIPCD
           ├─  770 /usr/lib/virtualbox/VBoxSVC --auto-shutdown
           ├─ 3877 /usr/lib/virtualbox/VBoxHeadless --comment W7sp1 --start...
           ├─23663 /usr/lib/virtualbox/VirtualBox --comment Debian --startvm ...
           └─27807 /usr/lib/virtualbox/vboxwebsrv --background -H 192.168.1.240...

{% endhighlight %}

Y esto es todo, ya podemos controlar las máquinas virtuales desde un servidor distinto.


## Múltiples servidores

Editamos el fichero de configuración `config.php`, descomentamos y adaptamos el array `$servers`.

{% highlight code %}
var $servers = array(
    array(
        'name' => 'swx',
        'username' => 'usuarioremoto',
        'password' => 'usuariospass',
        'location' => 'http://192.168.1.5:18083/'
    ),
    array(
        'name' => 'sxc',
        'username' => 'usuarioremoto1',
        'password' => 'usuariospass1',
        'location' => 'http://192.168.1.240:18083/'
    ),
);
{% endhighlight %}

<center><img src="/images/phpVirtualBox-multiple-servers.png"></center>

----

# Notas finales

Podemos utilizar fail2ban para evitar ataques de fuerza bruta. En la página oficial podemos encontrar mas información y otras configuraciones mas complejas.
