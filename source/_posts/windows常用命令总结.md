---
title: windows常用命令总结
tags:
  - windows
categories:
  - windows
copyright: true
permalink: windows常用命令总结
top: 0
password: woaini2.
abstract: 输入密码继续阅读.
message: 欢迎来到我的博客，输入密码继续阅读.
date: 2019-06-26 15:17:03
---

人生苦短我用python
<!--more-->

cls 清空命令行

## 文件操作
md 创建目录
```
md 1234
```
rd 删除目录
```
rd 1234
```

创建一个文件空

```
type nul>1.txt
```
写入内容到txt中

```
echo 123 > 1.txt
```


## Ping

```
ping -t -l 65500 ip
```
-l 65500 表示数据长度限制
-t 表示不停的ping目标

## 查看系统属性

### ip

```
ipconfig 查看ip
ipconfig /all 查看想起ip信息
ipconfig /release 释放当前ip
ipconfig /renew 重新获取ip
```
查看windows版本

```
ver
```


systeminfo 查看系统信息

```
systeminfo
```
```
arp -a
```
可以显示网关的IP和MAC ，用于查看高速缓存中的所有项目，可以用来查看地址解析表的,还有可以查看所有连接了我的计算机，显示对方IP和MAC地址。


```
net view   查看局域网内其他计算机名
shutdown -s -t 180 -c “提示信息”     180秒后关机，并弹出提示信息。
shutdown -a  取消180秒的关机
```


```
netstat -a  查看开启哪些端口
netstat -n   查看端口的网络连接情况
netstat -v 查看正在进行的情况
```

查看用户账户信息
```
net user 
```
查看登录用户
```
query user
```

 获取所有进程：get-process
```
get-process
```

## netstat

获取路由信息
```
netstat -r
```
## 远程桌面


查看系统是否允许远程连接
```
REG QUERY "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections
```
回显，1表示关闭 0表示开启
```
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server
    fDenyTSConnections    REG_DWORD    0x0
```
查看远程连接端口

```
REG QUERY "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v PortNumber
```
回显 0xd3d转为十进制为3389

开启3389远程方法
1.cmd
```
REG ADD "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 00000000 /f
REG ADD "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v PortNumber /t REG_DWORD /d 0x00000d3d /f
```
2.reg
```
Windows Registry Editor Version 5.00
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server]
"fDenyTSConnections"=dword:00000000
[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp]
"PortNumber"=dword:00000d3d
```
之后导入注册表
```
regedit /s a.reg
```

修改防火墙配置，允许3389端口的命令如下:
```
netsh advfirewall firewall add rule name="Remote Desktop" protocol=TCP dir=in localport=3389 action=allow
```

## 域控

```
根据Windows版本不同，命令有所不同。
查本机用户表 net user
查本机管理员 net localhroup administrators
查域管理用户 net group "domain dadmins" /domain
查域用户 net user /domain
查域工作组 net group /domain
查域名称 net view /domain
查域内计算机 net view /domain：XX
查域控制器 net time /domain
查域管理员用户组 net localgroup administrators /domain
域用户添加到本机 net localgroup administrators workgroup\user001 /add
查看域控制器，多台 net group "Domain controllers"
查本机IP段所在域 ipconfig /all
查同一域内机器列表 net view
需要安装了域管理工具才能使用命令 dsquery
查所有域控制器并显示DNS信息 dsquery server -domain super.com | dsget server -dnsname -site 
查域内所有计算机 dsquery computer
查域内名称以admin开头的前10台机器 dsquery computer domainroot -name admin -limit 10
查域内联系人 dsquery contact
查域内子网 dsquery subnet
查域内用户组 dsquery group
查域内组织单位 dsquery ou
查域内站点 dsquery site -o rdn
查域内所有计算机 net group "domain computer" /domain
查超过4周未登录的计算机 dsquery computer -inactive 4
查超过4周未登录的用户 dsquery user -inactive 4
通过组织单位查询计算机 dsquery computer "ou=xx,dc=xx,dc=com"
列出本机所有驱动器 fsutil.exe fsinfo drives
显示路由表 route print
显示IP路由表中以10开头的路由条目 route print 10
备份导出服务器网络配置 netsh dump>d:/netbak.txt
查看邮件服务器记录 nslookup -qt=mx google.com
查看子域名服务器记录 nslookup -qt=ns google.com
列出所有记录 》ls-d domain d:\xxx.txt

```






