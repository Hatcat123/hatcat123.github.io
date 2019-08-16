---
title: python3中base64编码与解码
tags:
  - python
  - base64
categories:
  - python
copyright: true
permalink: python3中base64编码与解码
top: 0
password: 
abstract: 输入密码继续阅读.
message: 欢迎来到我的博客，输入密码继续阅读.
date: 2019-05-13 02:53:52
---
在项目中我常常使用base64对接口报文进行加密处理，这种方式虽然没有太高深的加密方式，但是仍然能提高接口的逼格。

<!--more-->

### 0x01 base64优缺点

**优点**：速度快，ascil编码，属于一次加密
**缺点**：容易破解，只能使用在非关键信息的场合

### 0x02 python2与python3对比

**python2**

```
python2中进行Base64编码和解码
>>> import base64
>>> s = '我是字符串'
>>> a = base64.b64encode(s)
>>> print a
ztLKx9fWt/u0rg==
>>> print base64.b64decode(a)
我是字符串
```

**python3**

python3不太一样：因为3.x中字符都为unicode编码，而b64encode函数的参数为byte类型，所以必须先转码。

```
import base64

encodestr = base64.b64encode('abcr34r344r'.encode('utf-8'))
print(encodestr)
打印结果为
b'YWJjcjM0cjM0NHI='
```


结果和我们预想的有点区别，我们只想要获得YWJjcjM0cjM0NHI=，而字符串被b''包围了。
这时肯定有人说了，用正则取出来就好了。。。别急。。。
b 表示 byte的意思，我们只要再将byte转换回去就好了。。。源码如下

直接进行转译

```
import base64

encodestr = base64.b64encode('abcr34r344r'.encode('utf-8'))
print(str(encodestr,'utf-8'))
打印结果为
YWJjcjM0cjM0NHI=

```