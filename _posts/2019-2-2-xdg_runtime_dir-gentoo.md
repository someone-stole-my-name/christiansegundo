---
layout: post
title: XDG_RUNTIME_DIR and Gentoo
category: Gnu-Linux
tags:
- systemd
- xdg
- gentoo
- consolekit
---

On systemd/ConsoleKit systems this variable is set when the user logs in, if neither is used and you need `XDG_RUNTIME_DIR` for some reason (eg Flatpak), it can be easily set adding some lines to your `.bash_profile`:

```bash
if test -z "${XDG_RUNTIME_DIR}"; then
    export XDG_RUNTIME_DIR=/tmp/${UID}-runtime-dir
    if ! test -d "${XDG_RUNTIME_DIR}"; then
        mkdir "${XDG_RUNTIME_DIR}"
        chmod 0700 "${XDG_RUNTIME_DIR}"
    fi
fi
```

----
## Fuentes:

[Weston](https://wiki.gentoo.org/wiki/Weston)