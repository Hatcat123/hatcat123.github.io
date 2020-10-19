---
title: 记sql语言in查询优化案例
tags:
  - 运维
categories:
  - 运维
copyright: true
permalink: 记sql语言in查询优化案例
top: 0
password: 
abstract: 输入密码继续阅读.
message: 欢迎来到我的博客，输入密码继续阅读.
date: 2019-08-26 17:50:24
---

环境描述：使用了一部分别人的数据库，导致和自己的数据库中的表没有关系映射。在写程序的时候依照先跑为主。

<!--more-->

上线之后发现问题，查询速度极度慢。定位代码，找到原因，大概是这样的一个描述吧，关键字替换了！

### 问题描述：

有一个学校  有一个班级的表 有一个学生的表（学生表的学生id唯一）  学生有外键是班级id 。 和一个学生的分数的表 ， 分数表中存着学生的id（只是id）。但是没有存班级的id。  我想用sqlalchemy中最简单的快速的方法查询到某个班级的分数的集合。之后计算平均分

![](https://raw.githubusercontent.com/Hatcat123/GraphicBed/master/Img/20190826180248.png)

### 解决方案

**方案一：**

第一个想法 就是 传入班级的id ，然后根据班级的id获取到班级内的所人的id 再用for循环查询分数表中学生的id 得到结果。

显然这个方式会随着班级人数的增多而变得越来越慢的。
**方案二：**

优化查询方法是使用sql的`in`语句。查询到班级所有学会说呢过然后再查看分数中存在这个学生的分值。只要两次查询语句。

举个例子:

如果你的表比较小的话，可以对每个id单独查询，然后组成一个list：

[Shoe.query.filter_by(id=id).one() for id in my_list_of_ids]
1
如果你的表很大，上述查询方法会很慢。此时建议使用in条件的查询

shoes = Shoe.query.filter(Shoe.id.in_(my_list_of_ids)).all()


**最终我们的语句**

```

article = WechatArticle.query.filter(WechatArticle.__biz.in_(set([i.account_id for i in cat.accounts]))).filter(
        rule).order_by(WechatArticle.publish_time.desc())
```



## sql笔记

一个数据库比较大。当我插入一个新的字段的时候，提示

```
error1114 table is full
```
查看数据库，发现竟然网上竟然说的是这样做是压力测试的时候报错。原因有二：1

1 得知是由于内存表的大小超过了规定的范围。经过查看二者值默认均是16M，

2.需要设置tmp_table_size 大于等于max_heap_table_size

首先查看内存表得大小：

在mysql里面查询
```
show  VARIABLES like '%%table_size%'
```
显示16xxxxxxx

修改 my.cnf 配置文件，并重启 mysql。

- Linux 下 MySQL 的配置文件是 my.cnf，一般会放在 /etc/my.cnf，/etc/mysql/my.cnf。

```
/etc/mysql/mysql.conf.d# vim mysqld.cnf
```

添加

```

[mysqld]
max_heap_table_size = 32M
tmp_table_size = 64M
```


### 更新sql某个字段的部分字

update 表名 set 字段=replace(字段，‘替换的部分’，‘替换后的字符串’)；

update 表名 set A=replace( A, '海淀', '朝阳') where A like '海淀';  （将A字段中的“海淀”替换成“朝阳”）；
update yb_user_img set image=replace( image, 'http://192.168.0.126', 'http://yuebei.web66.cn');将yb_user_img表中image字段所有的“http://192.168.0.126”替换成“http://yuebei.web66.cn”；