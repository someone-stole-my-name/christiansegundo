---
layout: post
title: Dynamic DNS for CloudFlare with cfdns
category: Gnu-Linux
tags:
- cloudflare
- ddns
- dns
---

## Why?

cfdns is simple and clean solution to update your CloudFlare dynamic A records.

## Installation

### Download and install

Create the cfdns directory under ´/opt´

```
mkdir /opt/cfdns && cd /opt/cfdns
```

Get the latest version here: https://github.com/someone-stole-my-name/cfdns/releases

```
wget https://github.com/someone-stole-my-name/cfdns/releases/download/v1.0/cfdns_linx64
```

### Configuration

cfdns uses a simple and self explanatory JSON file that looks like this:

```
{
   "IPEndpoint": "https://ipinfo.io/ip",
   "Sleep": 60,
   "Records":
   [
       {
           "Username": "myaccount@someone.com",
           "API-Key": "88b2b8e3d2b68b9cc4b945d81516v91d77k6g",
           "Zone": "myzone.xyz",
           "Entry": "myentry.xyz"
       },
       {
           "Username": "myaccount_1@someone.com",
           "API-Key": "55b2b8e3d2b68b9cc4b945d81516v91d77k6g",
           "Zone": "anotherzone.xyz",
           "Entry": "anotherentry.xyz"
       }
   ]
}
```

Add as many `Records` as you need and write it to `/opt/cfdns/config.json`.


### Systemd

Download the systemd Unit file and move it to `/etc/systemd/system/`

```
wget https://raw.githubusercontent.com/someone-stole-my-name/cfdns/master/cfdns.service
mv cfdns.service /etc/systemd/system/
```

Enable and start the new service

```
sudo systemctl daemon-reload
sudo systemctl enable cfdns
sudo systemctl start cfdns
```

*Note: By default the Unit is configured to read the config file from `/opt/cfdns/config.json`, if you created the file somewhere else you have to modify the Unit accordingly.*