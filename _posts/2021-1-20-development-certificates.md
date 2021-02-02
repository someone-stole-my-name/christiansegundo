---
layout: post
title: How to create a Certificate Authority and development certificates
category: Gnu-Linux
tags: 
- openssl
---

Setting up a private Certificate Authority for development purposes can be tricky business. Whether you are doing integration tests with services that require certificates or just developing something, getting your own certificates kind of sucks unless you know how to create a CA and sign certificates.

<h3>Table of contents</h3>

<div id="inline_toc" markdown="1">

* TOC
{:toc}

</div>

----

## Generate a CA

### Private key

```
openssl genrsa -des3 -out myCA.key 2048
```

You will be prompted for a passphrase, the passphrase prevents anyone from generating a root certificate of their own.

```
Generating RSA private key, 2048 bit long modulus (2 primes)
................................................................................................+++++
..........+++++
e is 65537 (0x010001)
Enter pass phrase for myCA.key: 1234
Verifying - Enter pass phrase for myCA.key: 1234
```

### Certificate

```
openssl req -x509 -new -nodes -key myCA.key -sha256 -days 1825 -out myCA.pem
```

You will be prompted for the previous passphrase and the usual information when generating a certificate.

```
Enter pass phrase for myCA.key: 1234

-----
Country Name (2 letter code) [AU]:ES
State or Province Name (full name) [Some-State]:Madrid
Locality Name (eg, city) []:Madrid
Organization Name (eg, company) [Internet Widgits Pty Ltd]:Development
Organizational Unit Name (eg, section) []:
Common Name (e.g. server FQDN or YOUR name) []:Christian Segundo
Email Address []:
```

## Generate self signed certificates

### Private key

```
openssl genrsa -out dev.environment.com.key 2048
```

### CSR

```
openssl req -new -key dev.environment.com.key -out dev.environment.com.csr
```

### Certificate

#### extfile

Create an extfile with the following configuration:

```
authorityKeyIdentifier = keyid,issuer
basicConstraints = CA:FALSE
extendedKeyUsage = serverAuth, clientAuth
nsComment = "OpenSSL Generated Development Certificate"
subjectAltName = @alt_names
[alt_names]
DNS.1 = dev.environment.com
```

#### CSR signing

```
openssl x509 -req -in dev.environment.com.csr -CA myCA.pem -CAkey myCA.key -CAcreateserial \
-out dev.environment.com.crt -days 825 -sha256 -extfile dev.environment.com.ext
```

## Easy mode

Once you get the hang of how everything works, use a wrapper script to easy your life. See [development-certificates](https://github.com/someone-stole-my-name/development-certificates) repository or something like https://devcerts.netlify.app

----
