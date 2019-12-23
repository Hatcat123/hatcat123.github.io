---
title: 安装pymouse错误处理
tags:
  - python
categories:
  - python
copyright: true
permalink: 安装pymouse错误处理
top: 0
password: 
abstract: 输入密码继续阅读.
message: 欢迎来到我的博客，输入密码继续阅读.
date: 2019-12-19 11:46:14
---
安装pymouse出现报错。
<!--more-->


```
pip install pymouse
````

提示如下：

```
Traceback (most recent call last):
  File "4opeenrij.py", line 1, in <module>
    from pymouse import PyMouseEvent
  File "C:\Users\lcdew\AppData\Local\Programs\Python\Python37-32\lib\site-packages\pymouse\__init__.py", line 92, in <module>
    from windows import PyMouse, PyMouseEvent
ModuleNotFoundError: No module named 'windows'
```

修改源码__init__.py

```
from pymouse.windows import PyMouse, PyMouseEvent
```
之后出现

```
Traceback (most recent call last):
  File "",line 604,in 
  File "",line 314,in install
  File "",line 152,in LoadSystemModule
ImportError: DLL load failed: 找不到指定的模块。
```

看了下，自己使用的是一个虚拟环境。不知道是不是因为虚拟环境的问题。导致找不到底层的包。

然后将在python的大环境中运行程序，安装pymouse。

提示我找不到pyhook

去[https://www.lfd.uci.edu/~gohlke/pythonlibs/#pyhook](https://www.lfd.uci.edu/~gohlke/pythonlibs/#pyhook)

下载一个对应的内容安装。