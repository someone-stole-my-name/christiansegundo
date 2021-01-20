---
layout: post
title: Convert PKCS12 and PEM back and forth
category: Gnu-Linux
tags: 
- openssl
---

### Convert a PKCS#12 file to Key and Certificate information

```
P12=my.p12
KEY=my.key
CRT=my.crt

openssl pkcs12 -in $P12 -out $CRT -nokeys -clcerts
openssl pkcs12 -in $P12 -nodes -out $KEY -nocerts
```

### Convert a Key and Certificate information to PKCS#12 file

```
KEY=mykey
CRT=mycrt
OUT=my.p12

openssl pkcs12 -export -in $CRT -inkey $KEY -out $OUT
```

## Bonus

### Add a password to an existing Private Key

```
IN=insecure.key
OUT=my.key

openssl rsa -aes256 -in $IN -out $OUT
```

----
