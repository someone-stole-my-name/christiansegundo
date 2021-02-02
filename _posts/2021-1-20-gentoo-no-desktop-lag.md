---
layout: post
title: Say goodbye to desktop lag while compiling your @world
category: Gnu-Linux
tags: 
- gentoo
- linux
- cgroups
- systemd
---

## Create a new slice

Start by creating a new systemd unit `/etc/systemd/system/portage.slice`:

{% gist 06918fe93857d32e7ca16be9551a7cd6 portage.slice %}

Enable and start the portage.slice unit you just created:

```
systemctl enable --now portage.slice
```

CPUShares option defaults to 1024, systemd will create a user slice for each user with an active session, and all processes that user run will be assigned to that slice, anything that a user may run will receive 4 times the CPU time of processes assigned to the portage slice.

```
➜  ~ cat /sys/fs/cgroup/cpu,cpuacct/user.slice/cpu.shares
1024
```

## Repurpose PORTAGE_IONICE_COMMAND

`PORTAGE_IONICE_COMMAND` is those awesome variables you can set in your `make.conf` to alter how you build stuff, it should be a command string for portage to call to modify its own priority with a `\${PID}` placeholder that will be substituted with a pid. Maybe it was created with ionice in mind, but we can abuse that placeholder to write pids to the `cgroup.procs` file in the portage slice.

`cgroup.procs` file is present in every cgroup and contains a list of processes that are members of that particular cgroup. Writing a pid to this file will move all threads in that process at once to the cgroup. And that, is awesome :D

Add the following line to your `/etc/portage/make.conf`:

{% gist 06918fe93857d32e7ca16be9551a7cd6 make.conf %}

----
