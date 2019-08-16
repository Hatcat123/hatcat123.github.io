---
title: frp进行内网穿透
tags:
  - 安全
categories:
  - 安全
copyright: true
permalink: frp进行内网穿透
top: 0
password: 
abstract: 输入密码继续阅读.
message: 欢迎来到我的博客，输入密码继续阅读.
date: 2019-05-26 15:11:35
---


frp 是一个可用于内网穿透的高性能的反向代理应用，支持 tcp, udp 协议，为 http 和 https 应用协议提供了额外的能力，且尝试性支持了点对点穿透。

人生苦短我用python
<!--more-->

## FRP
[项目地址](https://github.com/fatedier/frp)或许官网的[中文文档](https://github.com/fatedier/frp/blob/master/README_zh.md)更加详细。

网上的教程博客都是把文档贴一份，很少从实战的几种方式记录。

frp就是一个反向代理软件，它体积轻量但功能很强大，可以使处于内网或防火墙后的设备对外界提供服务，它支持HTTP、TCP、UDP等众多协议。我们今天仅讨论TCP和UDP相关的内容。一些简单易操作的工具如：teamviwer、蒲公英、花生壳……要么限制并发要么收费且在功能上不够丰富。
搭建frp服务器进行内网穿透，可以达到不错的速度，且理论上可以开放任何想要的端口，可以实现的功能远不止远程桌面或者文件共享。

## 环境搭建
###0x01 准备物理环境
 - 内网环境windows主机(或内网环境Linux服务器)，部署web服务

>`主要是测试windows下的服务`，双linux机器的差别不大

 - 公网阿里云Ubuntu16x86_64服务器

###0x02 解析域名

将自己的域名添加解析记录，解析到公网服务器的ip上

###0x03 搭建linux版服务端
1.查看服务器版本
```
arch
```
2.到rfp项目中的`releases`下载对应的版本。

在公网服务器下载`linux`版本`arm64`
```
wget  https://github.com/fatedier/frp/releases/download/v0.27.0/frp_0.27.0_linux_arm64.tar.gz
```
3.然后解压gz格式

```
tar -zxvf frp_0.27.0_linux_arm64.tar.gz
```
4.复制到新的文件名下

```
cp -r frp_0.27.0_linux_arm64 frp
```
5.查看文件

```
ls -a
```
frps为服务端启动文件
frpc为客户端启动文件

此时公网服务器作为服务端，只需要用到frps程序，与基础配置的`frps.ini`，`frps_full.ini`是高级配置，在测试中未使用。

6.编辑frps.ini文件
```
vim frps.ini
```
```
[common]
# frps服务器绑定的端口与之后客户端一致
bind_port = 8007 
# 访问网站的端口
vhost_http_port = 8006
[common]
# dashboard可视化界面的端口
dashboard_port = 8005
# dashboard 用户名密码，默认都为 admin
dashboard_user = admin
dashboard_pwd = admin
```
保存退出
7.运行frps服务
```
.frps -c frps.ini
#或nohup、supervisor后台运行
nohup .frps -c frps.ini &
```
服务器启动成功
```
# ./frps -c ./frps.ini
 [I] [service.go:139] frps tcp listen on 0.0.0.0:8007
 [I] [service.go:181] http service listen on 0.0.0.0:8006
 [I] [service.go:232] Dashboard listen on 0.0.0.0:8005
 [I] [root.go:204] Start frps success

```
8.访问ip+端口8005即可看到仪表盘

![](https://raw.githubusercontent.com/Hatcat123/GraphicBed/master/Img/20190626160150.png)



###0x04 搭建windows版客户端

1.搭建web服务

在windows选用flask创建一个简单的本地服务端口5000
```
from  flask import Flask

app = Flask(__name__)

@app.route('/')
def index():
    return 'helloword\n'*20000

if __name__ == '__main__':
    app.run(port=5000)
```
2.测试本地web服务
浏览器访问`http://1270.0.1:5000`
浏览器返回helloword
```
helloword helloword
helloword helloword
```
3.下载frp的windows amd64版本

```
https://github.com/fatedier/frp/releases/download/v0.27.0/frp_0.27.0_windows_amd64.zip
```
4.解压更改frpc.ini配置
在客户端中主要用到frpc与frpc.ini基础配置
```
[common]
server_addr = 47.100.xxx.xxx # 公网ip地址
server_port = 8007           # frpc服务绑定的端口

[web]
type = http
local_port = 5000  # 本地的web服务端口
custom_domains = www.changlxxx.com # 解析的域名
```

5.运行frpc客户端

```
frpc.exe -c frpc.ini
```
6.运行成功结果

```
2019/06/26 10:18:25 [I] [service.go:221] login to server success, get run id [3c2aad0a15d8bcdf], server udp port [0]
2019/06/26 10:18:25 [I] [proxy_manager.go:137] [3c2aad0a15d8bcdf] proxy added: [web]
2019/06/26 10:18:26 [I] [control.go:144] [web] start proxy success
```

###0x05 验证效果

访问域名+web端口


![](https://raw.githubusercontent.com/Hatcat123/GraphicBed/master/Img/20190626164820.png)


## 服务
**在windows上做成服务，开机运行**

编写一个bat脚本

```
编辑一个内容如下的文件，frp.bat
@echo off  
start  "C:\Windows\System32\cmd.exe"   
cd C:\Users\Administrator\Desktop\frp_0.20.0_windows_amd64\frp_0.20.0_windows_amd64 
frpc -c frpc.ini
exit  
```
```
* start  "C:\Windows\System32\cmd.exe"   表示打开一个cmd命令行
* 命令段
* exit  退出打开的命令行
```
验证bat脚本能使用后
将其添加到服务
```
sc create frp binPath=  C:\Users\Administrator\Desktop\fr
p.bat start= auto
```
另一个脚本
rpc运行时始终有一个命令行窗口运行在前台，影响美观，我们可以使用一个批处理文件来将其运行在后台，而且可以双击执行，每次打开frpc不用再自己输命令了。
在任何一个目录下新建一个文本文件并将其重命名为“frpc.bat”，编辑，粘贴如下内容并保存。

```

@echo off
if "%1" == "h" goto begin
mshta vbscript:createobject("wscript.shell").run("""%~nx0"" h",0)(window.close)&&exit
:begin
REM
cd C:\frp
frpc -c frpc.ini
exit
```

**linux一键安装脚本**

https://github.com/clangcn/onekey-install-shell


















