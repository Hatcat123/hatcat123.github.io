---
title: git仓库清空
tags:
  - git
categories:
  - git
copyright: true
permalink: git仓库清空
top: 0
password: 
abstract: 输入密码继续阅读.
message: 欢迎来到我的博客，输入密码继续阅读.
date: 2019-05-18 19:16:42
---

删除github上的历史信息与删除某一部分敏感数据的方法

人生苦短我用python
<!--more-->

### 删除commits

Checkout 切换分支

    git checkout --orphan latest_branch

Add all the files 添加内容

    git add -A

Commit the changes 保存第一次

    git commit -am "init commit"
  

Delete the branch 删除主分支

    git branch -D master

Rename the current branch to master 创建主分支

    git branch -m master

Finally, force update your repository 强制覆盖

    git push -f origin master
