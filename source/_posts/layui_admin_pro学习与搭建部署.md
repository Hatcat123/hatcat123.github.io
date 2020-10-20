---
title: layui_admin_pro学习与搭建部署
tags:
  - layui
categories:
  - 前端
copyright: true
permalink: layui_admin_pro学习与搭建部署
top: 0
password: 
abstract: 输入密码继续阅读.
message: 欢迎来到我的博客，输入密码继续阅读.
date: 2020-09-04 21:09:05
---
具体内容仍然要看官网的文档。

记录下今天的收获


## layuiadmin pro单页版是前后端分离的框架

之前尝试搭建源码的过程发现不能像std常规版一样进行查看，因为需求不大没有深入找原因。今天仔细看了看文档，

std模式更像是前后端不分离的
```
将 src 目录中的 views 文件夹整个复制到你 服务端工程 的 view 层中，通过本地 web 服务器访问（Tomcat / IIS / Apache / Nginx 等）
确保可以访问后，修改好 HTML 文件中的 JS/CSS 路径，即可正常运行页面。
iframe 常规版 相比于 单页面模式的专业版 ，无论是在目录结构还是开发模式上都要简单很多。因为单页版是接管了服务端 MVC 的视图层，而 iframe 版则将视图交给了服务端来控制和输出，可以避免鉴权的复杂程度，直接可衔接好新老项目（因为你们的大部分老项目都是采用 iframe 模式）。
```
而pro模式是前后端分离的
```
由于 layuiAdmin 可采用前后端分离开发模式，因此你无需将其放置在你的服务端 MVC 框架中，你只需要给 layuiAdmin 主入口页面（我们也称之为：宿主页面）进行访问解析，它即可全权完成自身路由的跳转和视图的呈现，而数据层则完全通过服务端提供的异步接口来完成。
```

主要是view视图的区分


## 部署
```
解压文件后，将 layuiAdmin 完整放置在任意目录
通过本地 web 服务器去访问 ./start/index.html 即可运行 Demo
```
采用nginx部署，创建一个虚拟的主机配置，

```
server {
      listen       80;
       server_name  www.xx.com;	
     
    #    server_name  somename  alias  another.alias;
       location / {
           root   C:\layuiAdmin.pro-v1.2.1;
           index  .\start\index.html index.html index.htm;
       }
    }
```
其他通读了一遍文档，包括最后如何开发，打包

用到的登录，jwt，localstorage，sessionStorage。等等都经过封装，添加页面只需要些一些html。整理源码中解耦强。看起来比较舒服。

感受很大一定的架构程度和vue很像。

网上都说layui垃圾 我的水平还没有到骂layui的程度

## 参考文档
https://www.cnblogs.com/niuben/p/11038897.html

