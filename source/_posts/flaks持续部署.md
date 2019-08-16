---
title: flask持续自动化部署私有项目
tags:
  - python
  - 运维
categories:
  - 运维
copyright: true
permalink: flask持续部署
top: 0
password: 
abstract: 输入密码继续阅读.
message: 欢迎来到我的博客，输入密码继续阅读.
date: 2019-06-01 09:03:39
---

最近在维护一个企业官网的一个项目，当项目进行更新，便需要先上传github，然后进入到服务器进行pull项目，这个过程非常麻烦。介绍一个自己写的简单的项目持续部署。
人生苦短我用python
<!--more-->

### 产生背景

对于一个持续集成的项目。需要我们为了需求持久的跟进。一次部署过程如下：

开发人员更新版本->推送到github仓库->登录到对应服务器->clone代码->测试部署效果

简单的部署一次两次并不会觉得麻烦。每次更新一点点代码都要去重复一遍部署过程。实在麻烦，之前就看到github上的webhook选项，一直没有用过。个人理解webhook就是一个能在github上发生事件的时候触发的钩子函数，这个flask中的hook差不多。这次是一个很好的实验过程。

**为什么自己造轮子？**

在github上找到一个关于[flask自动部署的项目](https://github.com/NetEaseGame/git-webhook)，已有1K人start。看了他的思路，1觉得部署自己的项目没必要这么重，需要安装的三方应用太多。2.还是想着能有自己的思考的过程。

![图1](https://raw.githubusercontent.com/Hatcat123/GraphicBed/master/Img/20190601092528.png)


### 需求分析

我们期望每次更新代码后都能够进行自动部署，只使用上图中的1、2步骤。减少3、4步骤的重复工作。

使用github的webhook需要一个公网url，它会自动测试这个公网的url能不能put访问。

项目仓库为私有仓库，所以在部署的过程中要稍微的麻烦一点。需要用到秘钥。

### 设计

使用flask作为服务器应用，获取github上webhook的请求，然后本地执行请求下载仓库。部署到nginx服务器上。


对github进行配置，配置hook的域名与公钥。

服务器部署Git仓库能下载私有仓库。

### 实现

####开发flask应用

核心代码
```
#!/usr/bin/env python
# encoding: utf-8
from flask import Flask, request
import sh
app = Flask(__name__)
@app.route('/api/git_hook/', methods=['POST'])
def git_hook():
    sh.cd('/var/www/html/Doon') # 进入web项目的目录
    # 执行pull操作
    sh.git.pull(['origin', 'master'])
    r = sh.service("nginx","stop",_ok_code=[1,2,3])
    if r.exit_code == 0:
        return 'sucess'
    else:

        return '<pre>pull fali</pre>'
  
if __name__ == '__main__':
    app.run(port=5000)

```

#### 部署nginx

需要部署的项目是一个静态网站。使用nginx指定项目的路径，配置到服务器的80端口。

持续化部署应用服务部署到网站的非80端口。

```
  location /api {
            proxy_set_header X-REAL-IP $remote_addr;
            proxy_set_header Host  $http_host;
            proxy_pass       http://127.0.0.1:5000;

        }

```

使用supervisor做flask应用管理。

#### 服务器配置git仓库

生成公钥

```
ssh-keygen
```
保存到默认路径，输入空的pass

查看服务器上生成的.pub
添加到github上

然后使用sh下载私有项目，不能再使用https传输。
#### github配置webhooks

配置webhooks

![](https://raw.githubusercontent.com/Hatcat123/GraphicBed/master/Img/20190605200416.png)


### 演示效果

本地上传github，查看服务器git仓库日志，发现已经自动更新。

![](https://raw.githubusercontent.com/Hatcat123/GraphicBed/master/Img/20190604224757.png)


之后nginx服务器重启。完美！