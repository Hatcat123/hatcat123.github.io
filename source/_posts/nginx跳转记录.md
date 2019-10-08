---
title: nginx重定向端口丢失解决方案
tags:
    - flask
    - 运维
categories:
  - 运维
copyright: true
permalink: nginx重定向端口丢失解决方案
top: 0
password: 
abstract: 输入密码继续阅读.
message: 欢迎来到我的博客，输入密码继续阅读.
date: 2019-09-08 23:56:33
---

场景：在架设nginx反向代理，后端python应用启动在7000端口。python程序中有个302跳转操作。前端ngxin使用8800作为映射端口。

访问xx.xx.xx.xx:8800 出现302重定向 紧接着400 地址为xx.xx.xx.xx丢失了8800端口（就是转向默认的80端口）

<!--more-->

如果手动加上8800端口就可以正常访问

附配置
server {

    listen       8800;
    server_name  localhost xxxxx;

    #charset koi8-r;

    #access_log  logs/host.access.log  main;

    location / {
        proxy_pass xxxxx;
        proxy_set_header Host $host:8800;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header REMOTE-HOST $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}



其中`        proxy_set_header Host $host:8800;`，存在较大的争议。有人说去掉8800，有人说加上8800。我刚开始没加，不能正常跳转，后来加上也不能正常跳转，然后去掉，还是不能，然后加上……可以了。神奇的操纵？



## nginx只允许域名访问，禁止ip访问

### 为什么要禁止ip访问页面呢?

这样做是为了避免其他人把未备案的域名解析到自己的服务器IP，而导致服务器被断网，我们可以通过禁止使用ip访问的方法，防止此类事情的发生。

解决方法：
这里介绍修改配置文件nginx.conf两种方法：
1）在server段里插入如下正则：
listen       80;
server_name  www.yuyangblog.net;
if ($host != 'www.yuyangblog.net'){
   return 403;
}


2)添加一个server
新加的server（注意是新增，并不是在原有的server基础上修改）
server {
  listen 80 default;
  server_name _;
  return 403;
}
原来server里面插入：
listen       80;

server_name  www.yuyangblog.net;

 