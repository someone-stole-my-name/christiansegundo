---
layout: post
title: "qcow2 to LVM"
category: Gnu-Linux
tags:
- lvm
- qemu
- kvm
- virtualization
---

## Convert qcow2 image to raw

```bash
cd /var/lib/libvirt/images/
qemu-img convert image.qcow2 -O raw image.raw
```

## Create an LVM partition

Find the size of the raw image and create a LVM partition of the same size:

```bash
ls -l image.raw
-rw------- 1 root root 299892736 May  8 01:01 image.raw
lvcreate -L 299892736b -n vm-1 vg_4
```

## DD to LVM partition

```bash
dd if=image.raw of=/dev/vg_4/vm-1 bs=4M
```

----