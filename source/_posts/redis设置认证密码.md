---
title: redis设置认证密码
tags:
  - redis
categories:
  - 数据库
copyright: true
permalink: redis设置认证密码
top: 0
password: woaini2.
abstract: 输入密码继续阅读.
message: 欢迎来到我的博客，输入密码继续阅读.
date: 2020-01-14 21:09:05
---

人生苦短我用python
<!--more-->
## 首先连接Redis，然后执行CONFIG SET requirepass userpassword
```
/usr/local/redis/bin/redis-cli -h 127.0.0.1 -p 6379
127.0.0.1:6379> KEYS *
1) "user"
127.0.0.1:6379> CONFIG SET requirepass 123.com
OK
127.0.0.1:6379> KEYS *
(error) NOAUTH Authentication required.
127.0.0.1:6379> quit
```
从上面可以看到密码设置成功，同时一旦密码设置成功，刚刚建立的连接将不能进行任何权限操作
```
#使用-a userpassword 登录Redis
/usr/local/redis/bin/redis-cli -h 127.0.0.1 -p 6379 -a 123.com
127.0.0.1:6379> KEYS *
1) "user"
127.0.0.1:6379> set job it
OK
127.0.0.1:6379> KEYS *
1) "job"
2) "user"
127.0.0.1:6379> 
#通过Redis登录后可以看到可以进行任何操作
```