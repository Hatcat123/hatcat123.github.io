---
title: sqlalchemy速写笔记
tags:
  - sql
  - python
categories:
  - python
copyright: true
permalink: sqlalchemy速写笔记
top: 0
password: 
abstract: 输入密码继续阅读.
message: 欢迎来到我的博客，输入密码继续阅读.
date: 2019-11-18 21:57:12
---

记录常用的sqlalchemy编写的方式，方便我ctrl+V、C

<!--more-->

最近写
https://www.jianshu.com/p/637ede0939d1 
https://www.cnblogs.com/mengqingjian/articles/8521512.html
https://www.cnblogs.com/qingqing-919/p/8337865.html


mysql：insert ignore、insert和replace区别

|指令|已存在|不存在|举例|
|---|---|---|---|
|insert|报错|插入|insert into names(name, age) values(“小明”, 23);
|insert ignore|忽略|插入|insert ignore into names(name, age) values(“小明”, 24);
|replace|替换|插入|replace into names(name, age) values(“小明”, 25);

insert
插入已存在, id会自增，但是插入不成功，会报错

replace
已存在替换，删除原来的记录，添加新的记录

insert ignore
插入已存在，忽略新插入的记录，id会自增，不会报错