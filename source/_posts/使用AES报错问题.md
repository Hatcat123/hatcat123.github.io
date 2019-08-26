---
title: 使用AES报错问题
tags:
  -AES
categories:
  - python
copyright: true
permalink: 使用AES报错问题
top: 0
password: 
abstract: 输入密码继续阅读.
message: 欢迎来到我的博客，输入密码继续阅读.
date: 2019-08-25 17:13:00
---
from Crypto.Cipher import AES报错问题

<!--more-->

安装pycrypto一直报错，无法安装，导致from Crypto.Cipher import AES也无法使用

有两个解决方案
### 0x01 修改site-page中的`crypto`为`Crypto`

在python的包中没有找到大写的`Crypto`，但是有`crypto`，而且看了下包的内容包含`Cipher`

### 0x02 下载另外的库`pycryptodome`

```
pip install pycryptodome
```

然后从这个库中导入.Cipher import AES