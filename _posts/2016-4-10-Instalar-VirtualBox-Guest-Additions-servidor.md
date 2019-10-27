---
layout: post
title: VirtualBox Guest Additions sin X11
category: Gnu-Linux
tags:
- linux
- virtualbox
- vm
---

{% highlight code %}
# aptitude install gcc make linux-headers-*.*.*-*
# wget http://download.virtualbox.org/virtualbox/*.*.*6/VBoxGuestAdditions_*.*.*.iso
# mkdir /mnt/iso
# mount -o loop VBoxGuestAdditions_*.*.*.iso /mnt/iso
# bash /mnt/iso/VBoxLinuxAdditions.run --nox11
# umount /mnt/iso
{% endhighlight %}
