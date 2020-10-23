---
title: supervisor安装卸载路
tags:
    - 运维
categories:
  - 运维
copyright: true
permalink: supervisor安装卸载路
top: 0
password: 
abstract: 输入密码继续阅读.
message: 欢迎来到我的博客，输入密码继续阅读.
date: 2019-10-23 23:56:33
---

Ubuntu下apt装的superviosr突然失效。由于安装的是老版本的supervisor，重启supervisor自身服务的时候告诉我python找不到supervisor.supervisord的库。
<!--more-->
大概长这样
```
Starting supervisord: Traceback (most recent call last):
  File "/usr/bin/supervisord", line 2, in <module>
    from supervisor.supervisord import main
ImportError: No module named supervisor.supervisord
```

在早期supervisor使用python2.*开发的，只使用python2的环境，由于我装的比较早。所以讲python3作为默认环境的时候就不能启动原来的项目

我们先卸载原来的supervisor

## Uninstall supervisor
To remove just supervisor package itself from Ubuntu 16.04 (Xenial Xerus) execute on terminal:

sudo apt-get remove supervisor
Uninstall supervisor and it's dependent packages
To remove the supervisor package and any other dependant package which are no longer needed from Ubuntu Xenial.

sudo apt-get autoremove supervisor
Purging supervisor
If you also want to delete configuration and/or data files of supervisor from Ubuntu Xenial then this will work:

sudo apt-get purge supervisor
To delete configuration and/or data files of supervisor and it's dependencies from Ubuntu Xenial then execute:

sudo apt-get autoremove --purge supervisor
More information about apt-get remove
Advanced Package Tool, or APT, is a free software user interface that works with core libraries to handle the installation and removal of software on Debian, Ubuntu and other Linux distributions. APT simplifies the process of managing software on Unix-like computer systems by automating the retrieval, configuration and installation of software packages, either from precompiled files or by compiling source code.

apt-get is the command-line tool for handling packages, and may be considered the user's "back-end" to other tools using the APT library.

apt-get remove is identical to install except that packages are removed instead of installed. Note that removing a package leaves its configuration files on the system. If a plus sign is appended to the package name (with no intervening space), the identified package will be installed instead of removed.

上面介绍了几种卸载的方法。

## 安装supervisor
然后查阅资料显示supervisor也支持python3的版本。使用pip下载一个最新版本的。
在官网中介绍了如是使用pip下载并运行supervisor，http://supervisord.org/installing.html
```
pip install supervisor

```
创建配置文件

Creating a Configuration File
Once the Supervisor installation has completed, run echo_supervisord_conf. This will print a “sample” Supervisor configuration file to your terminal’s stdout.

Once you see the file echoed to your terminal, reinvoke the command as echo_supervisord_conf > /etc/supervisord.conf. This won’t work if you do not have root access.

If you don’t have root access, or you’d rather not put the supervisord.conf file in /etc/supervisord.conf, you can place it in the current directory (echo_supervisord_conf > supervisord.conf) and start supervisord with the -c flag in order to specify the configuration file location.

For example, supervisord -c supervisord.conf. Using the -c flag actually is redundant in this case, because supervisord searches the current directory for a supervisord.conf before it searches any other locations for the file, but it will work. See Running Supervisor for more information about the -c flag.

我默认放到的路径在`/etc/supervisor`于原来保持一致。


## 配置supervisor

wechat.conf
```
[program:wechat_doonsec_test]
directory=/home/all/WechatTogether
command=python app.py
autostart=false
autorestart=false
startretries=10
redirect_stderr=true
stderr_logfile=/etc/supervisor/logs/wechat_err.log
stdout_logfile=/etc/supervisor/logs/wechat.log

[program:wechat_doonsec_bak]
directory=/home/all/WechatTogether
command=gunicorn -b 127.0.0.1:7010 -w 1 --threads 20 -k gevent app:app
autostart=true
autorestart=true
startretries=10
redirect_stderr=true
stderr_logfile=/etc/supervisor/logs/wechat_err.log
stdout_logfile=/etc/supervisor/logs/wechat.log



[program:wechat_doonsec]
directory=/home/all/WechatTogether
command=gunicorn -b 127.0.0.1:7001 -w 1 --threads 20 -k gevent app:app
autostart=true
autorestart=true
startretries=10
redirect_stderr=true
stderr_logfile=/etc/supervisor/logs/wechat_err.log
stdout_logfile=/etc/supervisor/logs/wechat.log

```
开启网页版服务的端口
supervisord.conf
```
找到端口部分开启
```

## 运行supervisor
```
supervisord -c /etc/supervisor/supervisord.conf
```

## 查看任务的方式

### 命令行版本

```
supervisorctl  status
```

### 启动任务、暂停任务等
```
reread ;重新加载配置文件
update ;将配置文件里新增的子进程加入进程组，如果设置了autostart=true则会启动新新增的子进程
status ;查看所有进程状态
status <name> ;查看指定进程状态
start all; 启动所有子进程
start <name>; 启动指定子进程
restart all; 重启所有子进程
restart <name>; 重启指定子进程
stop all; 停止所有子进程
stop <name>; 停止指定子进程
reload ;重启supervisord
add <name>; 添加子进程到进程组
reomve <name>; 从进程组移除子进程，需要先stop。注意：移除后，需要使用reread和update才能重新运行该进程
```
### 常用命令
```
sudo service supervisor stop 停止supervisor服务
 
sudo service supervisor start 启动supervisor服务
 
supervisorctl shutdown #关闭所有任务 
 
supervisorctl stop|start program_name #启动或停止服务 
 
supervisorctl status #查看所有任务状态
```
### 网页版本

输入ip+开启的端口+账密。就能看到所有的任务，傻瓜式操作。



### 开机启动

**a.在指定目录下创建文件supervisord.service**
```
vim /usr/lib/systemd/system/supervisord.service
```
**b.输入以下内容：**
```
[Unit]
Description=Supervisor daemon 

[Service]
Type=forking
ExecStart=/usr/bin/supervisord -c /etc/supervisor/supervisord.conf
ExecStop=/usr/bin/supervisorctl shutdown
ExecReload=/usr/bin/supervisorctl reload
KillMode=process
Restart=on-failure
RestartSec=42s 

[Install]
WantedBy=multi-user.target 
```
保存并退出 

执行以下命令：
```
systemctl enable supervisord
```
提示：
```
Created symlink from /etc/systemd/system/multi-user.target.wants/supervisord.service to /usr/lib/systemd/system/supervisord.service.
```
验证是否为开机启动：
```
systemctl is-enabled supervisord
```
提示：
```
enabled
```
表示设置成功！

至此，创建supervisor守护进程完毕。


## 附件

### 配置说明

```
;*为必须填写项
;*[program:应用名称]
[program:cat]
 
;*命令路径,如果使用python启动的程序应该为 python /home/test.py, 
;不建议放入/home/user/, 对于非user用户一般情况下是不能访问
command=/bin/cat
 
;当numprocs为1时,process_name=%(program_name)s;
当numprocs>=2时,%(program_name)s_%(process_num)02d
process_name=%(program_name)s
 
;进程数量
numprocs=1
 
;执行目录,若有/home/supervisor_test/test1.py
;将directory设置成/home/supervisor_test
;则command只需设置成python test1.py
;否则command必须设置成绝对执行目录
directory=/tmp
 
;掩码:--- -w- -w-, 转换后rwx r-x w-x
umask=022
 
;优先级,值越高,最后启动,最先被关闭,默认值999
priority=999
 
;如果是true,当supervisor启动时,程序将会自动启动
autostart=true
 
;*自动重启
autorestart=true
 
;启动延时执行,默认1秒
startsecs=10
 
;启动尝试次数,默认3次
startretries=3
 
;当退出码是0,2时,执行重启,默认值0,2
exitcodes=0,2
 
;停止信号,默认TERM
;中断:INT(类似于Ctrl+C)(kill -INT pid),退出后会将写文件或日志(推荐)
;终止:TERM(kill -TERM pid)
;挂起:HUP(kill -HUP pid),注意与Ctrl+Z/kill -stop pid不同
;从容停止:QUIT(kill -QUIT pid)
;KILL, USR1, USR2其他见命令(kill -l),说明1
stopsignal=TERM
 
stopwaitsecs=10
 
;*以root用户执行
user=root
 
;重定向
redirect_stderr=false
 
stdout_logfile=/a/path
stdout_logfile_maxbytes=1MB
stdout_logfile_backups=10
stdout_capture_maxbytes=1MB
stderr_logfile=/a/path
stderr_logfile_maxbytes=1MB
stderr_logfile_backups=10
stderr_capture_maxbytes=1MB
 
;环境变量设置
environment=A="1",B="2"
 
serverurl=AUTO
```