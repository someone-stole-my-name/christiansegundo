---
layout: post
title: Certificado de Cloudflare en nginx
category: Gnu-Linux
tags:
- nginx
- ssl
- cloudflare
- https
---

### Contenido

  * [Generar el certificado TLS](#generar-el-certificado-tls)
  * [Instalar el certificado](#instalar-el-certificado)
  * [Authenticated Origin Pulls](#authenticated-origin-pulls)
<hr>

*Los certificados de Cloudflare son trusted por Cloudflare, si el servidor no está conectado a Cloudflare los usuarios no tienen forma de validarlo y será igual que haber generado uno propio.*

# Generar el certificado TLS

Desde la Dashboard->Crypto.

<center><img src="/images/cloudflare-1.png"></center>

<hr>

Seleccionamos el tipo de certificado que queremos generar, en este caso RSA y la lista de hosts para los que son válidos. eg "\*.example.com" o "blog.example.com" ... etc

<center><img src="/images/cloudflare-2.png"></center>

<hr>

En el siguiente dialogo se nos presenta el certificado de origen y la clave privada con posibilidad de cambiar el formato del mismo, el mas comun para nginx o apache es PEM (seleccionado por defecto).
Se copia el contenido de ambos recuadros a dos ficheros de texto en el servidor, en este caso `cert.pem` y `privkey.pem`.

<center><img src="/images/cloudflare-3.png"></center>
<center><img src="/images/cloudflare-4.png"></center>
<center><img src="/images/cloudflare-5.png"></center>

<hr>

# Instalar el certificado

Se añade al bloque `server`:

{% highlight code %}
server {
    listen              443 ssl;
    server_name         example.com;
    ssl_certificate     /ruta/al/cert.pem;
    ssl_certificate_key /ruta/a/privkey.pem;
    ...
}
{% endhighlight %}

# Authenticated Origin Pulls

La configuración anterior asegura a Cloudflare que está 'hablando' con nuestro servidor, pero el servidor no sabe si las requests vienen de Cloudflare o no.
Usando Authenticated Origin Pulls podemos asegurarnos de que el servidor solamente responda a Cloudflare y cualquier petición que venga sin un certificado válido de Cloudflare será descartada.

Descargamos el certificado de Cloudflare en el servidor.

{% highlight code %}
# wget https://support.cloudflare.com/hc/en-us/article_attachments/201243967/origin-pull-ca.pem
{% endhighlight %}

Se añade al bloque `server`:

{% highlight code %}
server {
    ...
    ssl_client_certificate /ruta/al/origin-pull-ca.pem;
    ssl_verify_client on;
    ...
}
{% endhighlight %}

Por último activamos la opción en Cloudflare (Dashboard->Crypto).

<center><img src="/images/cloudflare-6.png"></center>

----

# Fuentes:

[Configuring HTTPS servers](https://nginx.org/en/docs/http/configuring_https_servers.html)


[Protecting web origins with Authenticated Origin Pulls](https://blog.cloudflare.com/protecting-the-origin-with-tls-authenticated-origin-pulls/)


[Setting up NGINX to use TLS Authenticated Origin Pulls](https://support.cloudflare.com/hc/en-us/articles/204494148-Setting-up-NGINX-to-use-TLS-Authenticated-Origin-Pulls)
