---
layout: post
title: Migrate Droplet from DigitalOcean to local KVM
category: Gnu-Linux
tags:
- kvm
- digitalocean
- qcow2
- lvm
- cloud
---

In this tutorial, I’ll show you how to use the recovery mode to migrate a Droplet to a local KVM instance (or any other cloud provider). 

These steps will also work on any cloud provider as long as they offer a recovery mode or allows you to boot to an ISO.

# Overview

The overall process will look like this: 

* We’ll boot the Droplet to recovery mode.
* Clone the disk and send it over using SSH to a temporary place.
* Create a VM and attach a disk that is as big as the cloned disk.
* Write the clone to our new disk.

# Boot into recovery mode

Select your Droplet, head to Recovery, and select Boot from Recovery ISO. Shut it off and then boot it back up from the panel.

Open up the VPS console, and type 6 to start an interactive shell in the recovery ISO. Change the root password (we'll use it on the next step).

```
# passwd
```

# Clone

From another machine we'll connect using SSH to the Droplet and literally clone the disk over the internet to a raw file using dd.

```
# ssh root@my.drop.let.ip "dd if=/dev/vda " | dd of=/mnt/mydropletdisk.raw bs=64k
```

# Create a LV

Get the size of the raw file and create a LV of the same size.

```
# ls -l /mnt/mydropletdisk.raw
-rw-------  1 root root  85899345920 Oct 21 21:54 mydropletdisk.raw

# lvcreate -L 85899345920b -n cloned-disk
```

# Write the clone to your LV

```
dd if=/mnt/mydropletdisk.raw of=/dev/vg-raidpool-10/cloned-disk bs=4M
```

### Voila

Attach the LV to your VM and enjoy.

If you are using Qcow2 instead of LVs, convert the raw file to Qcow2 using `qemu-img`.

```
qemu-img convert -p -f raw -O qcow2 [input filename] [output filename]
```