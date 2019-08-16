---
title: ctfd搭建与优化
tags:
  - ctfd
  - 运维
categories:
  - 运维
copyright: true
permalink: ctfd搭建优化
top: 0
password: 
abstract: 输入密码继续阅读.
message: 欢迎来到我的博客，输入密码继续阅读.
date: 2019-06-05 15:16:11
---

记录关于uwsgi搭建ctfd平台的方式，以及如何优化速度的方法。
人生苦短我用python
<!--more-->


### 0x01 准备环境

阿里云Ubuntu服务器
已经默认将源为阿里源

更新
```
apt update
apt upgrade
```
默认安装python3.5。下载pip3，更新pip3版本
```
apt install python3-pip
pip3 install --upgrade pip
```

此时pip报错`from pip import main`，更改pip3源码

```
sudo vim /usr/bin/pip3
```
将原来的
```
from pip import main
if __name__ == '__main__':
    sys.exit(main())
```
修改为，注意__为两个下划线。
```
from pip import __main__
if __name__ == '__main__':
    sys.exit(__main__._main())
```

下载ctfd源码，到home下的all路径

```
cd /home/
mkdir all
cd all
git clone https://github.com/CTFd/CTFd.git
```

### 0x02 测试搭建

进入到CTFd的路径下，安装requirements.txt包。
```
pip3 insyall -r requirements.txt
```

使用python3运行serve.py

```
python3 serve.py
```
不报错，此时为本地127网段4000端口，想要发布到公网需要修改为0.0.0.0

默认访问公网ip加端口4000，便能登录ctfd网站。

当然这样的启动方式是不友好的，首先python启动flask是单线程的，flask中gun服务性能非常低。所能承受的压力非常小，对服务器而言造成资源浪费。其次当服务宕机时，不能快速重启。

下面进行简单优化配置

### 0x03 使用uwsgi配置

uwsgi的好处xxx

下载uwsgi服务
```

```
```
```




### 0x04 nginx服务器


### 0x05 监控



### 0x06 优化





