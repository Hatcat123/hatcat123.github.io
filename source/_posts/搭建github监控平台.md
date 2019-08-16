---
title: 搭建github敏感信息监控平台
tags:
  - 研发
  - github
categories:
  - 研发
copyright: true
permalink: 搭建github敏感信息监控平台
top: 0
password: 
abstract: 输入密码继续阅读.
message: 欢迎来到我的博客，输入密码继续阅读.
date: 2019-06-18 08:38:45
---

Github敏感信息泄露监控系统搭建与使用
人生苦短我用python
<!--more-->

## Haweye

[Haweye](https://github.com/0xbug/Hawkeye),监控github代码库，及时发现员工托管公司代码到GitHub行为并预警，降低代码泄露风险。根据搜索的语法一样可以搜集到github的公开的敏感信息。

### 项目特点

开发依赖采用Python3.x、Flask、Mongodb、Redis、Vue

- Docker部署
- 前后端分离。前端使用Vue，后端使用Python API
- 爬虫使用任务队列
- 及时警告预警

## 部署方式

### Docker部署

```
git clone https://github.com/0xbug/Hawkeye.git 
cd Hawkeye
docker build -t hawkeye .
docker run -ti -p 80:80 -e MONGODB_URI=mongodb://username:password@ip:27017/hawkeye -e MONGODB_USER= -e MONGODB_PASSWORD= -d hawkeye ## mongodb 需认证
docker run -ti -p 80:80 -e MONGODB_URI=mongodb://ip:27017 -d hawkeye ## mongodb 无认证
```
部署docker mongodb

```
docker pull mongo
docker run -p 27017:27017 -v $PWD:/data/db --name docker_mongodb -d mongo
```



### 手动部署


```

```

## 部署错误

在实际操作过程中，网站页面能打开，但是一直报500错误。

然后猜想应该是容器内部的api服务出错，查看源码的Dockerfile文件，发现服务运行依靠supervisor进行管控。

打算进入到容器内查看supervisor.conf的任务。


```

```
查看容器映射端口状态

```
docker ps -a

6db7632ef6a9        hawkeye             "/usr/bin/supervisor…"   2 hours ago         Up 3 minutes                0.0.0.0:8002->80/tcp         vibrant_mahavira
890903c7d1ba        mongo               "docker-entrypoint.s…"   2 hours ago         Up 4 minutes                127.0.0.1:27017->27017/tcp   dock

```

测试服务容器连接到数据库容器

```
docker exec 容器id curl 127.0.0.1:27017
```

查看容器的ip地址

```
ifconfig
```

查看端口占用情况

```
lsof -i:8888
``








