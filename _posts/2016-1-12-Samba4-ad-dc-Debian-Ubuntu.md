---
layout: post
title: Samba4 como AD DC en Debian/Ubuntu
category: Gnu-Linux
tags:
- samba
- active directory
- domain controller
- linux
---

Hace no mucho tiempo el proceso de instalación era muy engorroso, principalmente porque había que compilar manualmente Samba. Este tutorial es válido para Debian 8 y Ubuntu 14. El resultado final será un controlador de dominio de AD al que se podrán unir clientes Windows, Linux...etc

### Contenido

* [Configuración de red](#configuración-de-red)
* [Opciones de montaje](#opciones-de-montaje)
* [Samba/Kerberos](#autenticación-externa)
  * [Instalación](#instalación)
  * [Configuración](#configuración)
  * [Test de funcionamiento](#test-de-funcionamiento)

# Configuración de red

Lo primero que necesitamos es un IP estática y especificar el servidor de nombres (en este caso nosotros).

```
# nano /etc/network/interfaces

auto eth0
iface eth0 inet static
address 192.168.1.56
netmask 255.255.255.0
gateway 192.168.1.1
dns-nameservers 192.168.1.56
dns-search christian.com
```

```
# nano /etc/resolv.conf
domain christian.com
search christian.com
nameserver 192.168.1.56
nameserver 192.168.1.1
```

También necesitamos un FQDN. Modificamos y añadimos una línea al hosts.

```
# nano /etc/hosts

127.0.1.1 server1.christian.com server1
192.168.1.56 server1.christian.com server1
```

# Opciones de montaje

Hay que modificar /etc/fstab y añadir nuevas opciones de montaje (user_xattr, acl, barrier=1) que son necesarias para que Samba funcione.

```
# nano /etc/fstab

UUID=XXXXX  / ext4 user_xattr,acl,barrier=1,errors=remount-ro,relatime 0 1
```

Conviene comprobar si podemos montar el disco y asegurarnos de que el fstab está bien configurado.

```
# mount -a
```

# Samba/Kerberos
## Instalación

Instalamos samba y kerberos. Durante la instalación nos hará varias preguntas; la primera de ellas el default realm que será nuestro dominio y las otras dos relacionadas con los servidores en mi caso server1 (Es importante introducirlo todo en mayúsculas).

```
# aptitude install samba krb5-user smbclient
```

## Configuración

Editamos el nuevo smb.conf para indicar un servidor DNS (el router en mi caso) y permitir las actualizaciones no seguras desde el router.

```
# nano /etc/samba/smb.conf

dns forwarder = 192.168.1.1
allow dns updates = nonsecure and secure
```

Por último hacemos un `reboot` para que todos los cambios surjan efecto.

### Test de funcionamiento

Podemos comprobar si el servidor está funcionando correctamente con los siguientes comandos (cambiando el dominio y el nombre del servidor por supuesto).

```
$ host -t SRV _ldap._tcp.christian.com.
$ host -t SRV _kerberos._tcp.christian.com.
$ host -t A server1.christian.com
```

Si todo ha ido bien podemos proceder a intentar iniciar sesión como administrator (con la contraseña que usamos al generar el smb.conf)

```
kinit administrator
```

Por último hacemos una conexión y nos autenticamos para comprobar que todo realmente funciona...

```
# smbclient //localhost/netlogon -U 'administrator'
```

Si todo ha ido bien ya podemos añadir equipos al dominio y definir nuestras políticas.
