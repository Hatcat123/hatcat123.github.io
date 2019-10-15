---
title: 线上docker中sql服务错误配置修改
tags:
  - mysql
  - docker
categories:
  - docker
copyright: true
permalink: 线上docker中sql服务错误配置修改
top: 0
password: 
abstract: 输入密码继续阅读.
message: 欢迎来到我的博客，输入密码继续阅读.
date: 2019-10-14 19:29:18
---

修改了docker容器中的配置信息，由于配置信息错误，导致docker无法启动

<!--more-->

在做增量备份的时候，修改了数据库配置，其中的一个配置没改，然后就restart整个mysql服务配置，导致docker服务直接down掉，无法重启。

如何挽救配置错误的docker

## 方法一

找到并修改宿主机内对应docker服务中的文件配置

docker容器的配置信息一般都在`var\lib\docker\ovverlay2`下面，只需要在这里找到对应的配置，修改成功后重启对应的docker容器即可


### 1.查询docker日志

```
docker logs 容器id
```
找到对应容器中出错的原因

```
ERROR: mysqld failed while attempting to check config
command was: "mysqld --verbose --help"

2019-10-14T06:20:31.783220Z 0 [ERROR] You have enabled the binary log, but you haven't provided the mandatory server-id. Please refer to the proper server start-up parameters documentation
2019-10-14T06:20:31.785755Z 0 [ERROR] Aborting

ERROR: mysqld failed while attempting to check config
command was: "mysqld --verbose --help"

```
这个错误的原因是因为少配置了`server-id`
这个参数是在我配置mysqld.cnf的时候忘了加上了，需要找到宿主机对应docker中mysql容器的配置文件

### 2.查找文件

```
find / -name mysqld.cnf
```
如果之前启动了多个mysql容器，应该会有多个结果

```
root@iZuf69dbrbjskszhgtnvctZ:/var/lib/docker/overlay2# find / -name mysqld.cnf
/var/lib/docker/overlay2/3d11edc1c751fdc3f23921a791b42f90995c5c4574f2815aa9bc3b6c3e2aac33/diff/etc/mysql/mysql.conf.d/mysqld.cnf
/var/lib/docker/overlay2/4a1bee2a6a1736d211d4f7a317e76762c017bcbad9e74effec9bf2bd9c9f3572/diff/etc/mysql/mysql.conf.d/mysqld.cnf
/var/lib/docker/overlay2/d10125ec4c340e3ba85431f3b00e10891128186522414bc4ba92e91cb79ddca3/diff/etc/mysql/mysql.conf.d/mysqld.cnf
/var/lib/docker/overlay2/b9784268d5c723a6957b3071152a87c11455813e7d8f1e433df7dadefadb7197/diff/etc/mysql/mysql.conf.d/mysqld.cnf

```
找到当前的mysql的配置文件，再次修改

### 3.修改文件

找到文件修改、重启

## 方法二

将docker容器中的配置文件复制到主机中，在主机中修改然后再次复制到docker容器中。

### 1.复制docker容器中文件到主机

```
docker cp 容器id:docker容器中的配置 主机路径
```

### 2.修改配置文件

### 3.将配置文件复制到docker容器中

```
docker cp 修改后的文件 容器id:容器中对应的路径
```

### 方法三


如果在创建容器的时候，将容器的配置命令映射到宿主机的目录下，直接修改目录中的文件也可以















