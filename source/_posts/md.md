
---
title: screen使用
tags:
  - linux
categories:
  - 安全
copyright: true
permalink: screen使用
top: 0
password: 
abstract: 输入密码继续阅读.
message: 欢迎来到我的博客，输入密码继续阅读.
date: 2020-04-03 21:09:05
---

今天再帮别人部署项目的时候，见识了几个东西。开发人员给的是root用户，但是权限控制的很低。装个应用都要找运维的审核，所有操作都要问运维的。他们用的窗screen。一直没有理解是怎么玩的。

<!--more-->
## linux-使用screen，防止断网导致异常退出

在远程管理服务的时候，偶尔会出现断网导致，脚本或者命令没执行完就异常退出了。为了异常断开导致脚本出现异常，一般都会使用到screen这个工具。

### 特点
1、会话恢复。只要screen本身没有终止，会话就一直存在。如果出现断网的情况，可以使用screen -ls查看之前已经开启的会话，使用screen -r进行恢复即可继续使用，之前终端。


2、多窗口。在Screen环境下，所有的会话都独立的运行，并拥有各自的编号、输入、输出和窗口缓存。用户可以通过快捷键在不同的窗口下切换，并可以自由的重定向各个窗口的输入和输出。


3、窗口共享。在同一台机器上，可以实现两个终端限制同样的界面，screen -x实现共享。


### 使用

screen [-AmRvx -ls -wipe][-d <作业名称>][-h <行数>][-r <作业名称>][-s ][-S <作业名称>]


-A 　将所有的视窗都调整为目前终端机的大小。

-d <作业名称> 　将指定的screen作业离线。

-h <行数> 　指定视窗的缓冲区行数。

-m 　即使目前已在作业中的screen作业，仍强制建立新的screen作业。

-r <作业名称> 　恢复离线的screen作业。

-R 　先试图恢复离线的作业。若找不到离线的作业，即建立新的screen作业。

-s 　指定建立新视窗时，所要执行的shell。

-S <作业名称> 　指定screen作业的名称。

-v 　显示版本信息。

-x 　恢复之前离线的screen作业。

-ls或--list 　显示目前所有的screen作业。

-wipe 　检查目前所有的screen作业，并删除已经无法使用的screen作业。

###常用例子



screen -S yourname # 新建一个叫yourname的session


screen -ls #列出当前所有的session


screen -r yourname #回到yourname这个session


screen -d yourname #远程detach某个session


screen -d -r yourname #结束当前session并回到yourname这个session


screen -wipe #清理已经Dead的会话




