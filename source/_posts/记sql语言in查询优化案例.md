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