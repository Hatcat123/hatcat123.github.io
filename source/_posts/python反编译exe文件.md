---
title: python反编译exe文件
tags:
  - python
categories:
  - python
copyright: true
permalink: python反编译exe文件
top: 0
password: 
abstract: 输入密码继续阅读.
message: 欢迎来到我的博客，输入密码继续阅读.
date: 2019-08-26 16:47:22
---
之前记录了一个打包python为exe的文章。现在根据实例将打包后的exe还原为python文件

<!--more-->

>前提是打包的文件没有加密，还没有碰到过加密的exe的例子。


### 0x01 解开exe


使用`pyinstxtractor.py`从网上找下文件

放到与exe同级目录下，方便使用。

```
python pyinstxtractor.py xxx.exe
```
得到一个依照`exe_extracted`结尾的文件夹

这里面会吧当时打包exe的python环境全部都显示出来。打包exe也是个技术活，若打包前python不是一个新的环境，可能会吧用不到的库也打包进去，打包一小文件几十兆那就不足为奇了。相对应的解压出来可能会有神马mysql numpy flask django等等用不到的内容。

文件夹中存在`PYZ-00.pyz_extracted`文件夹，这个文件夹为真正python程序加载时候需要到python包，但是python程序与内置包一起编程了`pyc`格式的文件，同时存在上千个这这样的文件。

### 0x02 解`pyc`文件

一款[在线查看`pyc`工具](https://tool.lu/pyc/)
将pyc文件上传到网站就能出啦结果。但这是一件麻烦的事情。

可以使用python`uncompyle`进行批量解


```
pip install uncompyle
```

安装好后，注意执行的命令为uncompyle6，而不是uncompyle。

查看帮助命令：uncompyle6 --help、uncompyle6 -h

反编译单个文件 ：uncompyle6 foo.pyc > foo.py

反编译多个文件：uncompyle6 -o . *.pyc

像一般的人开发的文件，一个文件很少有20Kb以上的文件，所以文件夹中像600Kb的文件基本上可以断定为内置文件包了。不必解。

或者写一个例子 。在我没看到文档之前写的。垃圾！

```
import os
import sys


def main(path):
    for root ,dirs,files in os.walk(path):
        print(root)
        for file in files:
            fileName,oldflag = os.path.splitext(file)
            if oldflag==".pyc":
                old_file = os.path.join(root,file)
                new_file =os.path.join(root,'new',fileName+'.py')
                command = "D://virenv//35python35//Scripts//uncompyle6.exe "+ old_file + " > " + new_file
                os.system(command)
                print(old_file)
                print(new_file)


if __name__ == '__main__':
    path =r'E:\PYZ-00.pyz_extracted'
    main(path)
```
然后将`pyc`转成`py`文件之后还有成千上百的文件该怎么办？就要进行过滤

### 0x03 过滤


分析解出的py文件的格式，发现内置文件为site-page第四行 ，程序员写的文件不存在这个标志。写个脚本过滤

```
import os

import linecache

root = "E:\\new"

for dirpath, dirnames, filenames in os.walk(root):
    for filepath in filenames:
        file_name = os.path.join(dirpath, filepath)

        try:
            content=linecache.getline(file_name,4,module_globals='gbk').strip()
  
            if 'site-page'in content:
                os.remove(file_name)
        except Exception as e:
            print('出错',file_name)

```

过滤的过程中会提示我说有些文档不能gbk？后来想了想大概是程序员在程序中写了中文注释才不会编码错误吧。


第一遍过滤之后文件数减少了不少。但是还没有得到源文件。我们再过滤一遍。查看剩下的文件的内容。这里有个技巧。

一般程序员会在使用pycharm的过程加上自己的标志如

```
"""
Created on 201xxx10:59 PM
---------
@summary:
---------
@author:
"""
```
这种方式又提供了更好的过滤标志

继续改上面的过滤脚本。找到对应行中对应的关键字清洗。

这样的话文件差不多了。

### 0x04 还原


如果程序员写的不是单个文件，而是写成自己的包。如

```
定义一个db文件中的mysqldb的py文件

from db import mysqldb

```

这种情况解出来的文件格式便为`db.mysqldb.py`

现在所有的文件都在同一级目录下，不能将程序正常运行。

所以继续写脚本。分割文件名，得到目录。还原文件的路径格式。


```
E:.
    config.py
    core.capture_packet.py
    core.data_pipeline.py
    core.deal_data.py
    core.task_manager.py
    create_tables.py
    db.mysqldb.py
    db.redisdb.py
    utils.log.py
    utils.prpcrypt.py
    utils.selector.py
    utils.tools.py
    verify.py


```

还原程序为

```
E:.
│  config.py
│  config.yaml
│  create_tables.py
│  verify.py
│
├─core
│      capture_packet.py
│      data_pipeline.py
│      deal_data.py
│      task_manager.py
│      __init__.py
│
├─db
│  │  mysqldb.py
│  │  redisdb.py
│  │  __init__.py
│
├─utils
│  │  log.py
│  │  prpcrypt.py
│  │  selector.py
│  │  tools.py
│  │  __init__.py
│  


```
### 0x05 后续

一般一些python被打包的exe程序，会有三种情况，

* 一、程序的质量比较好，不想被广泛传播，带有少量性质的商业化。同时里面包含一些加密、认证的函数或与服务器有交互。但拿到源码验证的部分可以做一些事情，如：域名服务器的就修改本地host，自己写个接口。加解密的就看能不能反向。一般会与用户的mac地址有关。
* 二、程序中有见不得人的东西。向后台服务器发送一些奇奇怪怪的东西，如窃取运行电脑本地某些程序的文档发送给服务器。
* 三、菜鸟练手，我就想打包个程序玩玩，里面没东西。


打完收工