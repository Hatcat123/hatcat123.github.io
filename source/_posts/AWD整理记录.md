---
title: AWD整理记录
tags:
  - ctf
  - 安全
categories:
  - 安全
copyright: true
permalink: AWD整理记录
top: 0
password: woaini2.
abstract: 输入密码继续阅读.
message: 欢迎来到我的博客，输入密码继续阅读.
date: 2020-11-19 21:09:05
---

<!--more-->
整理复现的内容都是几年前awd的知识，好久没有玩过awd。


## 工欲行其事必先利其器 更新工具确保工具都能使用

链接ssh ftp的工具 ：xshell、xftp、finalshell
编程工具： pycharm
查看文档工具： notepad++ vscode
数据库查看工具： navicat hedisql redismanage
听歌工具： 网易云离线歌单，被打的很惨的时候可以听歌，这样死的愉快点。
docker容器：docker 启动mysql redis
抓包工具：bps
各种扫描工具 ：框架型的扫描工具 goby、lodon、nessus


## pycharm

创建awd的脚本管理项目，包含批量打poc的，批量提交flag的。如果有幸拿到，我手快当场就可以写或修改提交flag的脚本。

## 靶机操作

拿到靶机进行的操作

查看靶机的系统版本
```
cat /proc/version
```
查看所有的用户 cut -f 1 -d : 切割简化
```
 cat /etc/passwd |cut -f 1 -d :
```

查看存在的python版本，系统低版本存在python2，高版本存在python。或者都兼备

检查是否能连上外网，更新补丁


### linux常用操作口令

```
ssh <-p 端口> 用户名@IP　　
scp 文件路径  用户名@IP:存放路径　　　　
tar -zcvf web.tar.gz /var/www/html/　　
w 　　　　当前登录的用户
pkill -kill -t <用户tty>　　 　　
ps aux | grep pid或者进程名　　　　
#查看已建立的网络连接及进程
netstat -antulp | grep EST
#查看指定端口被哪个进程占用
lsof -i:端口号 或者 netstat -tunlp|grep 端口号
#结束进程命令
kill PID
killall <进程名>　　
kill - <PID>　　
#封杀某个IP或者ip段，如：.　　
iptables -I INPUT -s . -j DROP
iptables -I INPUT -s ./ -j DROP
#禁止从某个主机ssh远程访问登陆到本机，如123..　　
iptable -t filter -A INPUT -s . -p tcp --dport  -j DROP　　
#备份mysql数据库
mysqldump -u 用户名 -p 密码 数据库名 > back.sql　　　　
mysqldump --all-databases > bak.sql　　　　　　
#还原mysql数据库
mysql -u 用户名 -p 密码 数据库名 < bak.sql　　
find / *.php -perm  　　 　　
awk -F:  /etc/passwd　　　　
crontab -l　　　　
#检测所有的tcp连接数量及状态
netstat -ant|awk  |grep |sed -e  -e |sort|uniq -c|sort -rn
#查看页面访问排名前十的IP
cat /var/log/apache2/access.log | cut -f1 -d   | sort | uniq -c | sort -k  -r | head -　　
#查看页面访问排名前十的URL
cat /var/log/apache2/access.log | cut -f4 -d   | sort | uniq -c | sort -k  -r | head -　　
```


