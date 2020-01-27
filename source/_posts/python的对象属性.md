---
title: python的对象属性
tags:
  - python
categories:
  - python
copyright: true
permalink: python的对象属性
top: 0
password: 
abstract: 输入密码继续阅读.
message: 欢迎来到我的博客，输入密码继续阅读.
date: 2020-01-18 22:59:12
---
本文将简单介绍四种获取对象的方法。
<!--more-->
假如有以下的类：
```
class Person(object):
    def __init__(self, name, age):
        self.name = name
        self.age = age
        
    def __str__(self):
        return 'name=%s, age=%s' % (self.name, self.age)
    
xmr = Person('lisi',12)
```
方法一：使用属性运算符
```
print(xmr.name)
```
方法二：通过属性字典__dict__
```
print(xmr.__dict__['name'])
```
方法三：通过getattr函数
```
print(getattr(xmr, 'name'))
```
方法四：operator.attrgetter
```
import operator
 
op = operator.attrgetter('name')
print(op(xmr))
```
方法四可以用于对象的排序，比如需要根据年龄age来排序Person对象：
```
import operator
 
p_list = [Person('xiemanR', 18), Person('zhangshan', 17), Person('lisi', 20), Person('wangwu', 25)]
 
r = sorted(p_list, key=operator.attrgetter('age'))
 
for i in r:
    print(i)
输出结果：

 
Person(name=zhangshan, age=17)
 
Person(name=xiemanR, age=18)
 
Person(name=lisi, age=20)
 
Person(name=wangwu, age=25)
————————————————
```
## 对象属性查看
1. type()
type函数用于基本类型的判断，比如int，str等，也可以判断该对象属于那个函数或类，但对于继承类来说不是很方便。

2. isinstance()
isinstance函数可以用于基本类型的判断，也可以用于对继承类的判断。

3. dir()
dir函数可以返回一个对象的所有属性和方法，显示内容较多。

4. getattr() setattr() hasattr()
hasattr（object，x）主要判断该对象object是否有x这个属性，如果有，则返回true。若没有则会报错，但我们可以设置默认返回值。
getattr(object，x) 主要返回对象object的x属性，也可以将对象object的x属性赋值给另外一个对象。
setattr(object，x，y) 主要设置对象object的x属性值为y。

