---
title: dvwa
tags:
  - dvwa
categories:
  - 安全
copyright: true
permalink: dvwa
top: 0
password: 
abstract: 输入密码继续阅读.
message: 欢迎来到我的博客，输入密码继续阅读.
date: 2020-06-20 17:48:46
---
docker 搭建dvwa

<!--more-->
```
docker pull citizenstig/dvwa

docker run -d -p 8000:80 -p 33060:3306 -e MYSQL_PASS="root" citizenstig/dvwa
```

默认用户名与密码

admin/password

![20200620232516](https://raw.githubusercontent.com/Hatcat123/GraphicBed/master/Img2/20200620232516.png)