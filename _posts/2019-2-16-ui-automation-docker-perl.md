---
layout: post
title: Docker image ready to use for perl-Selenium Firefox automation
category: Gnu-Linux
tags:
- perl
- docker
- selenium
- automation
- firefox
---

<center><img src="/images/selenium-docker-perl.png"></center>

## Dockerfile

```dockerfile
FROM ubuntu:18.04

RUN apt-get update && apt-get install -y \
    wget \
    xvfb \
    firefox \
    perl \
    lsof \
    build-essential \
    libxml-parser-perl \
    libjson-perl \
    libfile-which-perl \
    libxml-simple-perl \
    libtest-lwp-useragent-perl \
    libarchive-zip-perl \
    libclone-perl \
    libio-string-perl \
    libtest-longstring-perl \
    libtest-deep-perl \
    libtest-fatal-perl \
    cpanminus

RUN cpanm install Selenium::Firefox \
&& rm -fr root/.cpanm

# Gecko Driver
ENV GECKODRIVER_VERSION 0.24.0
RUN wget --no-verbose -O /tmp/geckodriver.tar.gz https://github.com/mozilla/geckodriver/releases/download/v$GECKODRIVER_VERSION/geckodriver-v$GECKODRIVER_VERSION-linux64.tar.gz \
  && rm -rf /opt/geckodriver \
  && tar -C /opt -zxf /tmp/geckodriver.tar.gz \
  && rm /tmp/geckodriver.tar.gz \
  && mv /opt/geckodriver /opt/geckodriver-$GECKODRIVER_VERSION \
  && chmod 755 /opt/geckodriver-$GECKODRIVER_VERSION \
  && ln -fs /opt/geckodriver-$GECKODRIVER_VERSION /usr/bin/geckodriver
```

----