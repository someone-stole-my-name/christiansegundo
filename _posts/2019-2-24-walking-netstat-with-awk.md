---
layout: post
title: Walking /proc/net/netstat like a pro with awk
category: Gnu-Linux
tags:
- awk
- networking
---

Little command to parse `/proc/net/netstat` and format the output: 

```bash
$ awk '{for(i=1;i<=NF;i++)title[i] = $i; getline; print title[1]; for(i=2;i<=NF;i++)printf " %s: %s\n", title[i], $i }' /proc/net/netstat

TcpExt:
 ...
IpExt:
 ...

```

----
## Source:

[alternative to "netstat -s"](https://unix.stackexchange.com/questions/258711/alternative-to-netstat-s/)