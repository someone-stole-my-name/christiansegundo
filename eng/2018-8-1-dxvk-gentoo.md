---
layout: post
title: Building DXVK's DLLs on Gentoo
---

By far the easiest way to do it is on a Debian chroot, but that ain't as fun as building your own toolchains and do it on Gentoo.

<hr>

# Dependencies

According to the official documentation:

 - wine 3.10 or newer
 - Meson build system (at least version 0.43)
 - MinGW64 compiler and headers (requires threading support)
 - glslang front end and validator

{% highlight code %}
# emerge virtual/wine dev-util/meson dev-util/glslang
{% endhighlight %}

## MinGW64

MinGW with POSIX threads is needed, problem is by default crossdev will compile GCC with Win32 threads. 

Start by creating your toolchains, tuple for x86 is `i686-w64-mingw32` and `x86_64-w64-mingw32` for x64.

{% highlight code %}
# crossdev -t i686-w64-mingw32
# crossdev -t x86_64-w64-mingw32
{% endhighlight %}

Fix GCC by enabling POSIX threads and adding the `libraries` USE to `mingw64-runtime`.

{% highlight code %}
# mkdir /etc/portage/{env,package.env}
# echo 'EXTRA_ECONF="--enable-threads=posix"' > /etc/portage/env/mingw32_posix_threads
# echo -e 'cross-i686-w64-mingw32/gcc mingw32_posix_threads\ncross-x86_64-w64-mingw32/gcc mingw32_posix_threads' > /etc/portage/package.env/mingw32_posix_threads
{% endhighlight %}

{% highlight code %}
cross-i686-w64-mingw32/mingw64-runtime libraries
cross-x86_64-w64-mingw32/mingw64-runtime libraries
{% endhighlight %}

Rebuild `mingw64-runtime` and `gcc`. Order matters, `mingw64-runtime` with `libraries` provides `pthreads.h` and other stuff that is needed to compile GCC with POSIX thread model.

{% highlight code %}
# emerge -1 cross-i686-w64-mingw32/mingw64-runtime cross-x86_64-w64-mingw32/mingw64-runtime
# emerge -1 cross-i686-w64-mingw32/gcc cross-x86_64-w64-mingw32/gcc
{% endhighlight %}

*Depending on the runtime version, libraries may end up in the wrong place (see bug #653246), if that is the case sysmlink those and then rebuild GCC.*
*eg*
{% highlight code %}
# ln -s /usr/x86_64-w64-mingw32/usr/lib64/{libmangle.a,libpthread.a,libpthread.dll.a,libwinpthread.a,libwinpthread.dll.a,libwinpthread.la} /usr/x86_64-w64-mingw32/usr/lib/
{% endhighlight %}

Final result should be:

{% highlight code %}
# i686-w64-mingw32-gcc -v
# x86_64-w64-mingw32-gcc -v
...
Thread model: posix
...
{% endhighlight %}

# Crosscompiling DLLs

{% highlight code %}
# su user
$ cd
$ git clone https://github.com/doitsujin/dxvk.git
$ cd dxvk

### 32-bit build. For 64-bit builds, replace
### build-win32.txt with build-win64.txt
### build.w32 with build.w64

$ meson --cross-file build-win32.txt --prefix /some/install/prefix build.w32
$ cd build.w32/
$ meson configure -Dbuildtype=release
$ ninja
$ ninja install
{% endhighlight %}
{% highlight code %}
$ ls /some/install/prefix/bin
d3d11.dll  dxgi.dll  setup_dxvk.sh
{% endhighlight %}

These are the toolchains I used.

{% highlight code %}
cross-i686-w64-mingw32/binutils-2.30-r3
cross-i686-w64-mingw32/gcc-7.3.0-r3
cross-i686-w64-mingw32/mingw64-runtime-5.0.4

cross-x86_64-w64-mingw32/binutils-2.30-r3
cross-x86_64-w64-mingw32/gcc-7.3.0-r3
### symlinks from ...lib64/ -> ...lib/ required! see bug #653246
cross-x86_64-w64-mingw32/mingw64-runtime-5.0.4
{% endhighlight %}

----

#### [Original in spanish](/gnu/linux/2018/08/01/dxvk-gentoo/)
