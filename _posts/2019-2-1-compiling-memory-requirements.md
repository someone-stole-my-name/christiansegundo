---
layout: post
title: There is NOT at least X GiB RAM 
category: Gnu-Linux
tags:
- rust
- ebuild
- gentoo
---

Recently while compiling latest Rust version I came across a problem on my very old Macbook, where `emerge` complained about not enough RAM to compile Rust. 

```bash
...
pre_build_checks() {
    CHECKREQS_DISK_BUILD="7G"
    CHECKREQS_MEMORY="4G"
...
    check-reqs_pkg_setup
}
```

`check-reqs_pkg_setup` from `check-reqs.eclass` takes care of the check and looks if `${I_KNOW_WHAT_I_AM_DOING}` is set, if it is then the checks will succeed even if the space constraints are not met.

<hr>

## tl;dr Solution

Some packages require X ammount of disk space and RAM to build and Rust is one of them. Add enough Swap space and run `emerge` again with `${I_KNOW_WHAT_I_AM_DOING}` set.

```
# fallocate -l 8G /mnt/swap.swap
# mkswap /mnt/swap.swap
# swapon /mnt/swap.swap
# chmod 600 /mnt/swap.swap
# I_KNOW_WHAT_I_AM_DOING=1 emerge -1 rust
```

----
