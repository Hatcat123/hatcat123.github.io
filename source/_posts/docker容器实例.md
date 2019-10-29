---
title: docker容器实例
tags:
  - docker
categories:
  - docker
copyright: true
permalink: docker容器实例
top: 0
password: 
abstract: 输入密码继续阅读.
message: 欢迎来到我的博客，输入密码继续阅读.
date: 2019-06-18 08:35:45
---
docker 常用命令
人生苦短我用python
<!--more-->



** docker 查看容器网络ip**


```
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' container_name_or_id

可直接获得容器的ip地址如：172.18.0.4

显示所有容器IP地址：
docker inspect --format='{{.Name}} - {{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' $(docker ps -aq)

```
## 判断是否工作在docker环境中
**方法一：判断根目录下是否有.dockerenv**

docker环境下有，非docker环境下没有
>定制化的容器也可能没有这个文件
```
ls -alh /.dockerenv 
-rwxr-xr-x 1 root root 0 Sep  6 07:09 /.dockerenv
```

**方法二：查询系统进程的cgroup信息**

docker 环境下：`cat /proc/1/cgroup`

```
cat /proc/1/cgroup 
10:devices:/docker/4cb54de415d470461a636d52a9a4f731eddbbcfdf80b4d0b46466ec1cf27f730
9:perf_event:/docker/4cb54de415d470461a636d52a9a4f731eddbbcfdf80b4d0b46466ec1cf27f730
8:net_cls,net_prio:/docker/4cb54de415d470461a636d52a9a4f731eddbbcfdf80b4d0b46466ec1cf27f730
7:cpu,cpuacct:/docker/4cb54de415d470461a636d52a9a4f731eddbbcfdf80b4d0b46466ec1cf27f730
6:freezer:/docker/4cb54de415d470461a636d52a9a4f731eddbbcfdf80b4d0b46466ec1cf27f730
5:memory:/docker/4cb54de415d470461a636d52a9a4f731eddbbcfdf80b4d0b46466ec1cf27f730
4:cpuset:/docker/4cb54de415d470461a636d52a9a4f731eddbbcfdf80b4d0b46466ec1cf27f730
3:blkio:/docker/4cb54de415d470461a636d52a9a4f731eddbbcfdf80b4d0b46466ec1cf27f730
2:pids:/docker/4cb54de415d470461a636d52a9a4f731eddbbcfdf80b4d0b46466ec1cf27f730
1:name=systemd:/docker/4cb54de415d470461a636d52a9a4f731eddbbcfdf80b4d0b46466ec1cf27f730
```

物理机`cat /proc/1/cgroup `
```
cat /proc/1/cgroup 
10:cpuset:/
9:freezer:/
8:memory:/init.scope
7:perf_event:/
6:blkio:/init.scope
5:net_cls,net_prio:/
4:cpu,cpuacct:/init.scope
3:pids:/init.scope
2:devices:/init.scope
1:name=systemd:/init.scope
```