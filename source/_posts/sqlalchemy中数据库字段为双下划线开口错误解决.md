---
title: sqlalchemy中数据库字段为双下划线开头错误解决
tags:
  - flask
categories:
  - flask
copyright: true
permalink: sqlalchemy中数据库字段为双下划线开头错误解决
top: 0
password: 
abstract: 输入密码继续阅读.
message: 欢迎来到我的博客，输入密码继续阅读.
date: 2019-08-17 14:54:47
---
sqlalchemy中数据库字段为双下划线开头错误解决
人生苦短我用python
<!--more-->


遇到头疼的问题：有一个数据库中其中一个表中的字段为'__test'，（两个下划线）。我现在使用flask sqlalchemy 连接数据库。他显示我字段不对。


例如：
```
class TEST(db.Model):

    __tablename__ = 't_test'

    id = db.Column(db.Integer, primary_key=True, autoincrement=True)
    title = db.Column(db.String(50), nullable=True)
    context = db.Column(db.Text, nullable=True)
    __test =  db.Column(db.String(50), nullable=True)
    _test =  db.Column(db.String(50), nullable=True)

```

```
@manager.option('-t', '--test', dest='test')
def add_account(test):
    # 读取公众号的列表
    test = TEST(__test='1',_test='2')
    db.session.add(test)
    db.commit()
    print('添加成功')

错误：TypeError: <flask_script.commands.Command object at 0x000000000DBDB160>: '__test' is an invalid keyword argument for TEST
```

然后 我使用sqlalchemy 映射一个测试的数据库。发现这个字段改变

![](https://raw.githubusercontent.com/Hatcat123/GraphicBed/master/Img/20190817145936.png)


_TEST__test字段前面加了 _TEST



现在主要问题。我的另一个数据库中的字段就是`__zid`双下划线开头。


要解决这个问题，需要看下Python中关于隐私变量的说明


 >Any identifier of the form__spam (at least two leading underscores, at most one trailing underscore)is textually replaced with _classname__spam, where classname is thecurrent class name with leading underscore(s) stripped


是说类的定义中任何以双下划线开头的变量都将被重命名为 _类名__变量名 的形式

所以回到sqlalchemy的问题，就会发现字段被重命名加上类名，而数据表中并没有带有类名的字段，导致查询出错。

 

在不能够改变数据库的前提下如何修复这个错误呢？其实很简单，只要将变量放在类定义外面就好。


```
class WechatAccount(db.Model):
    '''
        爬取的微信公众号信息
    '''
    __tablename__ = 'wechat_account'
    id = db.Column(db.Integer, primary_key=True, autoincrement=True)

WechatAccount.__biz = db.Column(db.String(50), nullable=True, comment='公众号id')
```

运行sqlalchemy不再报错！

nice
