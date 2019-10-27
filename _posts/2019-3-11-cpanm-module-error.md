---
layout: post
title: "Cpanm module error: No such file or directory opening compressed index"
category: Gnu-Linux
tags:
- perl
- cpanm
- docker
---

I got this error while installing a module inside a Docker container using `cpanm`.

```bash
! Finding install on cpanmetadb failed.
! cannot open file '/root/.cpanm/sources/http%www.cpan.org/02packages.details.txt.gz': No such file or directory opening compressed index
! Couldn't find module or a distribution install
```

The problem is caused by `LWP::Protocol::https` module. Run cpanm with `--no-lwp` option.

----