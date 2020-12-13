---
title: linux常用命令总结
tags:
  - linux
categories:
  - linux
copyright: true
permalink: linux常用命令总结
top: 0
password: woaini2.
abstract: 输入密码继续阅读.
message: 欢迎来到我的博客，输入密码继续阅读.
date: 2019-06-26 15:16:50
---

人生苦短我用python
<!--more-->

常用命令

## 查看Linux系统版本的命令（3种方法）

```
lsb_release -a

root@iZuf69dbrbjskszhgtnvctZ:~# lsb_release -a
LSB Version:	core-9.20160110ubuntu0.2-amd64:core-9.20160110ubuntu0.2-noarch:security-9.20160110ubuntu0.2-amd64:security-9.20160110ubuntu0.2-noarch
Distributor ID:	Ubuntu
Description:	Ubuntu 16.04.6 LTS
Release:	16.04
Codename:	xenial

```
这个命令适用于所有的Linux发行版，包括RedHat、SUSE、Debian…等发行版
```
uname -a
```
```
Linux user 4.15.0-124-generic #127-Ubuntu SMP Fri Nov 6 10:54:43 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux

```
## linux解压
```
linux解压
tar –xvf file.tar //解压 tar包
tar -xzvf file.tar.gz //解压tar.gz
tar -xjvf file.tar.bz2 //解压 tar.bz2
tar –xZvf file.tar.Z //解压tar.Z
unrar e file.rar //解压rar
unzip file.zip //解压zip
1、.tar 用 tar –xvf 解压
2、.gz 用 gzip -d或者gunzip 解压
3、.tar.gz和.tgz 用 tar –xzf 解压
4、.bz2 用 bzip2 -d或者用bunzip2 解压
5、.tar.bz2用tar –xjf 解压
6、.Z 用 uncompress 解压
7、.tar.Z 用tar –xZf 解压
8、.rar 用 unrar e解压
9、.zip 用 unzip 解压

```

## 切换用户

su switch user

su 后面加- 后会从新启动一个全新环境

切换root
```
su root
```
切换普通用户
```
su 用户名
```


## 挂载u盘

查看磁盘的内容

```
cat /proc/partitions 
```
```
major minor  #blocks  name

   7        0     100084 loop0
   7        1      88964 loop1
   2        0       1440 fd0
   8        0   20971520 sda
   8        1       1024 sda1
   8        2   20968448 sda2
  11        0      63900 sr0
  11        1     831488 sr1

```

挂在磁盘

```
sudo mount -t vfat /dev/U盘位置  /挂在位置
```
## 查看端口占用
```
lsof -i:21
```

```
netstat -anop|grep 22
```

## 查看进程
结果比较详细
```
ps -aux|grep xx
```

```
root      63701  0.1  0.0      0     0 ?        I    06:23   0:01 [kworker/0:2]
root      65559  0.0  0.3 105704  6916 ?        Ss   03:24   0:00 sshd: user [priv]
root      65578  0.0  0.3 105704  6916 ?        Ss   03:24   0:00 sshd: user [priv]
user      65789  0.0  0.3 108000  6208 ?        S    03:24   0:00 sshd: user@notty
user      65790  1.1  0.3 108352  6444 ?        R    03:24   2:22 sshd: user@pts/0,pts/1
user      65791  0.0  0.1  13068  2172 ?        Ss   03:24   0:00 /usr/lib/openssh/sftp-server
user      65793  0.0  0.2  21588  5464 pts/0    Ss   03:24   0:00 -bash

```