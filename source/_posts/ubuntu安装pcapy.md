---
title: 在Ubuntu上python2.7安装pcapy
tags:
  - python
categories:
  - python
copyright: true
permalink: 在Ubuntu上python2.7安装pcapy
top: 0
password: 
abstract: 输入密码继续阅读.
message: 欢迎来到我的博客，输入密码继续阅读.
date: 2020-04-28 17:48:46
---


How To Install "python-pcapy" Package on Ubuntu

<!--more-->


在Ubuntu上python2.7安装pcapy
```
 pip install pcapy
```

报错

```
  pcapdumper.cc:11:10: fatal error: pcap.h: No such file or directory
   #include <pcap.h>
            ^~~~~~~~
  compilation terminated.
  error: command 'x86_64-linux-gnu-gcc' failed with exit status 1
  ----------------------------------------
  ERROR: Failed building wheel for pcapy
```


Quick Install Instructions of python-pcapy on Ubuntu Server. It’s Super Easy! simply click on Copy button to copy the command and paste into your command line terminal using built-in APT package manager.

See below for quick step by step instructions of SSH commands, Copy/Paste to avoid miss-spelling or accidently installing a different packag

Step 1
```
sudo apt-get update -y
```
Step 2
```
sudo apt-get install -y python-pcapy
```













