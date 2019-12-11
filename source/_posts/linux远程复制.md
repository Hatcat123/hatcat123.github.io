---
title: linux远程复制
tags:
  - linux
categories:
  - linux
copyright: true
permalink: linux远程复制
top: 0
password: 
abstract: 输入密码继续阅读.
message: 欢迎来到我的博客，输入密码继续阅读.
date: 2019-12-10 21:06:32
---
Scp 远程拷贝
<!--more-->

1. 将本地文件复制到目标服务器
```
scap /home/user/ user@ip:/home/user/ 
```
2. 将服务器文件复制到本地电脑
```
scap user@ip:/home/user/ /home/user/
```
在传输过程中出现权限不允许
```
 scp xxx :Permission denied
```
一种将目标文件夹加入可写权限、另一种将被复制的文件也加入权限

```
sudo chmod 777 文件
```