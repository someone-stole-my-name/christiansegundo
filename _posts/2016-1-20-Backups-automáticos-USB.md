---
layout: post
title: Backups automáticos con Memorias/Discos USB
category: Gnu-Linux
tags:
- backup
- rsync
- linux
- script
---

La idea es que al conectar el disco, el sistema lo reconozca y realice automáticamente un backup. Dependiendo de las necesidades será un backup del disco externo o de un directorio en concreto del ordenador hacia el disco externo. Utilizaremos un script, para el backup rsync y para reconocer el disco udev. 

  * Instalamos el paquete rsync. Si se trata de un servidor podemos instalar también el paquete beep, cuando comience el backup sonará un beep y cuando termine sonaran dos.
  
   {% highlight bash %}
# aptitude install rsync beep
{% endhighlight %}
   
   * Para escribir la regla udev necesitamos algunos datos, empecemos por averiguar el nombre del disco.
   
   {% highlight bash %}
# fdisk -l
Disco /dev/sdb: 1000.2 GB, 1000204885504 bytes
255 cabezas, 63 sectores/pista, 121601 cilindros, 1953525167 sectores en total
Unidades = sectores de 1 * 512 = 512 bytes
Tamaño de sector (lógico / físico): 512 bytes / 4096 bytes
Tamaño E/S (mínimo/óptimo): 4096 bytes / 33553920 bytes
Identificador del disco: 0x01a8375a

Dispositivo Inicio    Comienzo      Fin      Bloques  Id  Sistema
/dev/sdb1            2048  1953521663   976759808    7  HPFS/NTFS/exFAT
{% endhighlight %}

 En mi caso el disco está en /dev/sdb, el siguiente paso es obtener mas información con udevadm.
 
    {% highlight bash %}
 # udevadm info -a -p $(udevadm info -q path -n /dev/sdb)
 looking at parent device '/devices/pci0000:00/0000:00:1d.0/usb2/2-1/2-1.3/2-1.3:1.0/host4/target4:0:0/4:0:0:0':
    KERNELS=="4:0:0:0"
    SUBSYSTEMS=="scsi"
    DRIVERS=="sd"
    ATTRS{rev}=="0302"
    ATTRS{type}=="0"
    ATTRS{scsi_level}=="7"
    ATTRS{model}=="BUP Slim RD     "
    ATTRS{state}=="running"
    ATTRS{queue_type}=="ordered"
    ATTRS{iodone_cnt}=="0x18ad3"
    ATTRS{iorequest_cnt}=="0x18ad3"
    ATTRS{device_busy}=="0"
    ATTRS{evt_capacity_change_reported}=="0"
    ATTRS{timeout}=="30"
    ATTRS{evt_media_change}=="0"
    ATTRS{ioerr_cnt}=="0x2"
    ATTRS{queue_depth}=="30"
    ATTRS{vendor}=="Seagate "
    ATTRS{evt_soft_threshold_reached}=="0"
    ATTRS{device_blocked}=="0"
    ATTRS{evt_mode_parameter_change_reported}=="0"
    ATTRS{evt_lun_change_reported}=="0"
    ATTRS{evt_inquiry_change_reported}=="0"
    ATTRS{iocounterbits}=="32"
    ATTRS{vpd_pg80}==""
    ATTRS{vpd_pg83}==""
    ATTRS{eh_timeout}=="10"
{% endhighlight %}


   * Creamos la regla udev con la información que identificará el disco o la memoria.
   
   {% highlight bash %}
# vi /etc/udev/rules.d/50-auBackup.rules

# Backup automático
KERNEL=="sd?1", ACTION=="add", SUBSYSTEMS=="scsi", ATTRS{vendor}=="Seagate ", ATTRS{model}=="BUP Slim RD     ", RUN+="/bin/systemctl --no-block start backup@%k.service", ENV{UDISKS_IGNORE}=1
{% endhighlight %}

   * Con la regla anterior iniciamos un servicio que a su vez lanzará el script que hace el backup pasando como parámetro la primera partición del disco. Crear el servicio es muy simple:
   
   {% highlight bash %}
# nano /etc/systemd/system/backup@.service

[Unit]
Description=auBackup
BindsTo=dev-%i.device
[Service]
Type=simple
ExecStart=/root/backup.sh %i
{% endhighlight %}

   * Ya solo queda crear el script, en mi caso el path es /root/backup.sh. Para que el script funcione descomentamos el sistema de ficheros y cambiamos el directorio de origen/destino. Podemos quitar el beep, usar avisos gráficos, ajustar el comando rsync, mandar notificaciones por email...etc
   
    {% highlight bash %}
# vim /root/backup.sh

#!/bin/bash

## BEEP
#/usr/bin/beep
## Aviso GNOME 
#zenity --info --text 'Backup iniciado'

## Entrada en el log
#/usr/bin/logger Backup - `date`
if [ ! -d /mnt/Backup ] ; then mkdir /mnt/Backup ; fi

## EXT
#/bin/mount -t auto /dev/$1 /mnt/Backup

## FAT32
#/bin/mount -t vfat -o shortname=mixed,iocharset=utf8 /dev/$1 /mnt/Backup

## NTFS
#/bin/mount -t ntfs-3g /dev/$1 /mnt/Backup

sleep 5
/usr/bin/logger Backup - NOMBRE
/usr/bin/rsync -rtvz --del --modify-window=2 /var/www/html/ /mnt/Backup/www
/bin/sync
/bin/umount /mnt/backup
## Entrada en el log
#/usr/bin/logger Backup - End at `date`
for i in {1..2}; do /usr/bin/beep; done
{% endhighlight %}    

   * Después de añadir un nuevo servicio a systemd o una regla a udev debemos reiniciar el equipo o los demonios antes conectar el disco. 
   
   {% highlight bash %}
# systemctl daemon-reload
# udevadm control --reload
{% endhighlight %}

Se puede aplicar en cualquier distro, simplemente hay que asegurarse de que coinciden las rutas (/bin/mount, /bin/systemctl...etc).