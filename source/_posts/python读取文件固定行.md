---
title: python读取文件固定行
tags:
  - python
categories:
  - python
copyright: true
permalink: python读取文件固定行
top: 0
password:
abstract: 输入密码继续阅读.
message: 欢迎来到我的博客，输入密码继续阅读.
date: 2019-08-26 17:00:27
---

内置库 `linecache`
<!--more-->


```

>>> import linecache
>>> file_path = r'D:\work\python\test.txt'
>>> line_number = 5
>>> def get_line_context(file_path, line_number):
...     return linecache.getline(file_path, line_number).strip()
...
>>> get_line_context(file_path, line_number)
'This is line 5.'


```