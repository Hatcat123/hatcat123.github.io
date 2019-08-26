---
title: Gitpython操作git库
tags:
  - python
  - git
categories:
  - git
copyright: true
permalink: Gitpython操作git库
top: 0
password: 
abstract: 输入密码继续阅读.
message: 欢迎来到我的博客，输入密码继续阅读.
date: 2019-08-19 11:47:21
---
GitPython 是一个用于操作 Git 版本库的 python 包，


它提供了一系列的对象模型(库 - Repo、树 - Tree、提交 - Commit等)
用于操作版本库中的相应对象。

<!--more-->

## 安装

```
pip install GitPython
```
##版本库对象 - Repo

### 0x01 初始化仓库

```
from git import Repo
Repo.init('/data/test2') # 创建一个git文件夹
```
### 0x02 创建版本库对象
```
repo = git.Repo(r'E:\Notes')
```

### 0x03 操作

```
# 版本库是否为空版本库
repo.bare

# 当前工作区是否干净
repo.is_dirty()

# 版本库中未跟踪的文件列表
repo.untracked_files

# 克隆版本库
repo.clone('clone_path')

# 压缩版本库到 tar 文件
with open('repo.tar', 'wb') as fp:
    repo.archive(fp)

# 新建分支
repo.create_head('branchname')

# 查看当前分支
repo.active_branch
```
















