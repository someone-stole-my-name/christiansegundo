---
layout: post
title: Compilando las DLLs de DXVK en Gentoo
category: Gnu-Linux
tags:
- wine
- dxvk
- gentoo
---

[*English version*](/eng/2018-8-1-dxvk-gentoo/)

La forma mas sencilla de compilar DXVK en Gentoo es con una chroot de Debian, pero... Donde está la diversión en eso? La primera vez que lo intenté me topé con varios problemas y no encontré documentado el proceso en ningún sitio. Esta entrada es un intento de resolver ese problema.

<hr>

# Dependencias

Según la documentación oficial necesitamos:

 - wine 3.10 or newer
 - Meson build system (at least version 0.43)
 - MinGW64 compiler and headers (requires threading support)
 - glslang front end and validator

{% highlight code %}
# emerge virtual/wine dev-util/meson dev-util/glslang
{% endhighlight %}

## MinGW64

Se necesita MinGW con threads de POSIX. Por suerte existe `crossdev`, el problema es que por defecto compila GCC con threads de Win32. 

Se crea primero la toolchain, la tupla para x86 es `i686-w64-mingw32` y para x64 `x86_64-w64-mingw32`.

{% highlight code %}
# crossdev -t i686-w64-mingw32
# crossdev -t x86_64-w64-mingw32
{% endhighlight %}

Para arreglar GCC se necesitan dos cosas, indicar que lo queremos con posix threads y las librerías que hacen falta para compilarlo.

{% highlight code %}
# mkdir /etc/portage/{env,package.env}
# echo 'EXTRA_ECONF="--enable-threads=posix"' > /etc/portage/env/mingw32_posix_threads
# echo -e 'cross-i686-w64-mingw32/gcc mingw32_posix_threads\ncross-x86_64-w64-mingw32/gcc mingw32_posix_threads' > /etc/portage/package.env/mingw32_posix_threads
{% endhighlight %}

Se añade la `USE` `libraries` al runtime.

{% highlight code %}
cross-i686-w64-mingw32/mingw64-runtime libraries
cross-x86_64-w64-mingw32/mingw64-runtime libraries
{% endhighlight %}

Por último recompila `mingw64-runtime` y `gcc` por separado, en ese orden.

{% highlight code %}
# emerge -1 cross-i686-w64-mingw32/mingw64-runtime cross-x86_64-w64-mingw32/mingw64-runtime
# emerge -1 cross-i686-w64-mingw32/gcc cross-x86_64-w64-mingw32/gcc
{% endhighlight %}

*Dependiendo de la versión del runtime puede que se instalen algunas librerias en el lugar equivocado y esto impide compilar GCC, si es el caso basta con hacer enlaces. Bug #653246*  
*Ejemplo*:

{% highlight code %}
# ln -s /usr/x86_64-w64-mingw32/usr/lib64/{libmangle.a,libpthread.a,libpthread.dll.a,libwinpthread.a,libwinpthread.dll.a,libwinpthread.la} /usr/x86_64-w64-mingw32/usr/lib/
{% endhighlight %}

El resultado final debe ser este:

{% highlight code %}
# i686-w64-mingw32-gcc -v
# x86_64-w64-mingw32-gcc -v
...
Thread model: posix
...
{% endhighlight %}

# Compilar las DLLs

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

Estas son las toolchains que usé y que producen binarios que funcionan.

{% highlight code %}
cross-i686-w64-mingw32/binutils-2.30-r3
cross-i686-w64-mingw32/gcc-7.3.0-r3
cross-i686-w64-mingw32/mingw64-runtime-5.0.4

cross-x86_64-w64-mingw32/binutils-2.30-r3
cross-x86_64-w64-mingw32/gcc-7.3.0-r3
### Requiere symlinkear de ...lib64/ -> ...lib/
cross-x86_64-w64-mingw32/mingw64-runtime-5.0.4
{% endhighlight %}

----

## Fuentes:

[DXVK Github](https://github.com/doitsujin/dxvk)  
[Bug #631460](https://bugs.gentoo.org/631460)  
[Bug #653246](https://bugs.gentoo.org/653246)  
