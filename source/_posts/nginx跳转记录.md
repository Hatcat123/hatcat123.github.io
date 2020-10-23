---
title: nginx配置
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

## nginx配置支持下划线。

在请求头中传递了一个带有下划线的参数，结果web服务器得不到这个参数一致报错。 

如果想要支持下划线的话，需要增加如下配置：
underscores_in_headers on;

可以加到http或者server中

语法：underscores_in_headers on|off
默认值：off
使用字段：http, server
是否允许在header的字段中带下划线


## Nginx一个server配置多个location



公司测试环境使用nginx部署多个前端项目。网上查到了两个办法：

在配置文件中增加多个location，每个location对应一个项目
比如使用80端口，location / 访问官网； location /train 访问培训管理系统
配置多个站点
我选择了配置多个location。
```
   location / {
         root   /data/html/;
         index  index.html index.html;
    }
    location /train {
         root   /data/trainning/;
         index  index.html index.html;
    }
```

配置完以后访问。http://xxxx/train 提示404
找了好久才搞明白， location如果一个特定的url 要使用别名，不能用root，alias指定的目录是准确的，root是指定目录的上级目录，改动后即可以使用了
```
location /train {
     alias  /data/trainning/;
     index  index.html index.html;
}
```

==========================================
补充
==========================================
留言中有小伙伴问及alias和root区别，个人理解：
root与alias主要区别在于nginx如何解释location后面的uri，这会使两者分别以不同的方式将请求映射到服务器文件上。 root的处理结果是：root路径＋location路径 alias的处理结果是：使用alias路径替换location路径 alias是一个目录别名的定义，root则是最上层目录的定义。 还有一个重要的区别是alias后面必须要用“/”结束，否则会找不到文件的。。。而root则可有可无~~