---
layout: post
title: Various ways to increase the size of a LVM partition
category: Gnu-Linux
tags:
- lvm
- linux
- cheatsheet
---

## Content

  * [By adding a new disk](#by-adding-a-new-disk)
  * [By increasing a PV size](#by-increasing-a-pv-size)

## By adding a new disk

### Identify the new disk
  
```
lsblk
```

### Initialize the disk

```
pvcreate /dev/vdh
```

### Add the PV to the existing VG
  
```
vgextend influxdb /dev/vdh
```
  
### Resize partition and filesystem

```
lvextend -l +100%FREE /dev/influxdb/influxdb
resize2fs /dev/influxdb/influxdb
```

----

## By increasing a PV size

### Scan disks for physical volumes
  
```
pvscan
```

### Resize the PV to the maximum size

```
pvresize /dev/sdh
```
  
### Resize partition and filesystem

```
lvextend -l +100%FREE /dev/influxdb/influxdb
resize2fs /dev/influxdb/influxdb
```