---
title: 记一次线上mysql时区错乱
tags:
  - 运维
categories:
  - 运维
copyright: true
permalink: 记一次线上mysql时区错乱
top: 0
password: 
abstract: 输入密码继续阅读.
message: 欢迎来到我的博客，输入密码继续阅读.
date: 2019-08-28 23:33:22
---
Mysql查询时间和页面显示时间相差八个小时。

<!--more-->

在一次线上程序调用mysql内部函数转化时间戳的时候的bug记录。在本地开发与测试环境都没得问题。但是上线后，程序总是不再`状态`。

遂开启审阅代码的过程，修bug轻轻松松耗费几个小时。最后在数据库中执行使用的原生sql语句，大概长这样：

```
  (UNIX_TIMESTAMP(CURRENT_TIMESTAMP) - UNIX_TIMESTAMP(last_spider_time)
                        ) > 600
```
一直回显不出数据。

猜想应该是时间戳出现了问题。验证一下执行时间戳的内容。

```
SELECT

                    UNIX_TIMESTAMP(CURRENT_TIMESTAMP) ,
                    UNIX_TIMESTAMP('2019-08-28 18:30:35');
```
当前的时间已经大于18点，但是返回的18点的时间戳确实2点的时间戳。快了8个小时。
```
1567006237 

1567017035
```
在本地开发环境执行的结果是

![](https://raw.githubusercontent.com/Hatcat123/GraphicBed/master/Img/20190828234411.png)

这是一个正常的结果。

输出当前的时区

```
show variables like "%time_zone%";
```


开发测试环境结果为显示系统时间

```

mysql> show variables like "%time_zone%";
+------------------+--------+
| Variable_name    | Value  |
+------------------+--------+
| system_time_zone |        |
| time_zone        | SYSTEM |
+------------------+--------+
2 rows in set, 1 warning (0.00 sec)

```

服务器显示结果也为系统时间

```
mysql>  show variables like "%time_zone%";
+------------------+--------+
| Variable_name    | Value  |
+------------------+--------+
| system_time_zone | UTC    |
| time_zone        | SYSTEM |
+------------------+--------+
2 rows in set (0.00 sec)

```


我们看下服务器系统的时区

    date
发现是utc的时区。我们需要增加8个小时的时区了。
## 方案
更改系统时区，傻瓜式更改
```
tzselect
```
选择 4911

```
cp /usr/share/zoneinfo/Asia/Shanghai  /etc/localtime

ntpdate time.windows.com
```
重启


## 或者 

设置时区更改为东八区
```
set global time_zone = ‘+8:00’;

```
 刷新权限
```
flush privileges;
```



## 结果

时区正常

```
mysql> SELECT
    -> 
    ->                     UNIX_TIMESTAMP(CURRENT_TIMESTAMP) ,
    ->                     UNIX_TIMESTAMP('2019-08-28 18:30:35');
+-----------------------------------+---------------------------------------+
| UNIX_TIMESTAMP(CURRENT_TIMESTAMP) | UNIX_TIMESTAMP('2019-08-28 18:30:35') |
+-----------------------------------+---------------------------------------+
|                        1567009347 |                            1566988235 |
+-----------------------------------+---------------------------------------+
```