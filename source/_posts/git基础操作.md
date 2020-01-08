---
title: git基础操作
tags:
  - git
categories:
  - git
copyright: true
permalink: git基础操作
top: 0
password: 
abstract: 输入密码继续阅读.
message: 欢迎来到我的博客，输入密码继续阅读.
date: 2019-05-18 21:00:14
---

在开发过程中经常用到git与github，对于一个稳定的项目常常就用到极少的命令操作，但是仍然会遇到一些奇奇怪怪的现象，就需要对git进行深入的了解。
<!--more-->

### 0x01 git常用操作



### 0x02 git基础操作

#### 初始化仓库

```
git init 初始化仓库,新建一个Git仓库(新建了一个隐藏目录.git)
　　
把远程仓库克隆到本地
git clone git@github.com:lioilwin/lioilwin.github.io.git
git clone git@git.coding.net:lifec/lioilwin.git

把本地仓库关联到远程仓库
git remote add github git@github.com:lioilwin/lioilwin.github.io.git
git remote add coding git@git.coding.net:lifec/lioilwin.git

```


### 0x03 git进阶操作

#### 远程仓库别名

```
如果git clone一个远程仓库, Git会自动添加url,别名为origin
git remote      列出远程仓库别名    
git remote -v   远程仓库别名对应的实际url
git remote add [alias] [url]   添加一个新远程仓库
git remote rm [alias]          删除远程仓库别名
git remote rename [old-alias] [new-alias]   重命名
git remote set-url [alias] [url]   更改url,可以加上—push和fetch参数,为同一个别名set不同地址

```

#### 日志回滚

```
git log --name-only --oneline fileName
git log --oneline --number  每条log只显示一行,显示number条
git log --oneline --graph   图形化显示分支合并历史
git log branchname          显示特定分支
git log --decorate
git log --author=[author name] 指定作者的提交历史.
git log --since --before --until --after  根据提交时间筛选
git log --grep 根据commit信息过滤
git log --stat 改动信息     
    
git reflog
    reflog记录分支变化或者HEAD引用变化, 当git reflog不指定引用时, 默认列出HEAD的reflog,
    HEAD@{0}代表HEAD当前的值, HEAD@{3}代表HEAD在3次变化之前的值,
    git会将变化记录到HEAD对应的reflog文件中, 其路径为.git/logs/HEAD, 分支reflog文件都放在.git/logs/refs的子目录

git show commitID
git diff
    不加参数: show diff of unstaged changes.

    git diff --cached 命令
        已经暂存的文件和上次提交之间的差异
        
    git diff HEAD
        show diff of all staged or unstated changes.
        
git checkout commitID fileName
git revert
git reset --hard

git分为三个区域: 
    1.工作区(working directry)
    2.暂缓区(stage index)   
    3.历史记录区(history)
    
git reset --mixed id  history变了(提交记录变了),但staged 和 working没变  (默认方式)
git reset --soft id   history变了(提交记录变了)和staged区都变了,但working没变
git reset --hard id   全都变了

变化范围:
soft (history) < mixed (history + stage) < hard (history + stage + working)


```

#### 分支

```
git branch -v  每一个分支的最后一次提交.
git branch     列出本地所有分支,当前分支会被星号标示出 
git branch mybranch        创建分支
git branch -D mybranch     删除分支 

git checkout mybranch      切换分支
git checkout -b mybranch   创建并切换分支
git rebase master          把master分支更新到当前分支
git merge mybranch         分支合并

git push [remote-name] :branch-name 删除远程分支
```

### 0x04 事件处理


#### 合并冲突

#### xxx

寻找大文件的id
```
git verify-pack -v .git/objects/pack/pack-*.idx | sort -k 3 -g | tail -5

```

查看文件


```
git rev-list --objects --all |grep bc8880e2a3d3a4cb0f6a1661e8ce5ad65d3a95a3 


```

删除文件

```
git filter-branch --index-filter 'git rm --cached --ignore-unmatch    DianpingCrawler/dianping.docx'

```

垃圾回收

```

rm -rf .git/refs/original/
git reflog expire --expire=now --all
git gc --prune=now
git gc --aggressive --prune=now
git push origin master --force


```

```

git remote prune origin
```

首先，里面最重要的两条命令是 git filter-branch 和 gc, filter-branch 真正在清理，但是只运行它也是没用的，需要再删除备份的文件，重新打包之类的，最后的gc命令，

用来收集产生的垃圾，最终清除大文件。

#### 清空仓库



#### 删除敏感文件

不小心上传了关于密码的代码，然后在几次commits中都有操作记录。

或者上传了一个很大的文件，并在几次commit中都有操作记录。

虽然删除了，但是在.git中保存了记录。

[github官网帮助](https://help.github.com/en/articles/removing-sensitive-data-from-a-repository)

```
参考GitHub官网: https://help.github.com/articles/removing-sensitive-data-from-a-repository

例如, 代码库提交了大量mp3文件, 使用下面命令清除后代码库由233M缩小为1.3M, 提交到GitHub部署博客轻快飞速
# 清除垃圾文件(大量无用的mp3文件)
git filter-branch --force --index-filter 'git rm --cached --ignore-unmatch *.mp3' --prune-empty --tag-name-filter cat -- --all

# 提交到远程仓库(如GitHub, 我再次从git clone GitHub代码库会变小为1.3M)
git push origin --force --all

# 必须回收垃圾,本地仓库才变小
git for-each-ref --format='delete %(refname)' refs/original | git update-ref --stdin    
git reflog expire --expire=now --all
git gc --prune=now

rm -rf .git/refs/original
git reflog expire --expire=now --all
git gc --prune=now
git gc --aggressive --prune=now

```

### git rebase 提交干净的主分支

在开发的过程中，我们使用master主分支进行开发，在进行一次bug修改多次commit后，便产生了很多的无用的commit日志。这对我们来说不容易进行回滚或者快速定位到版本。

我们可以使用创建dev分支，当在分支中开发后，也许在dev提交了很多次的commit，这个时候我们如果在master直接mergen，那么在master主分支上也将产生许多的commit记录。这与我们的想法产生了分歧。我们希望在主分支上只进行版本发布推送，一个很`干净`的master。


我们先拉取master分支

```
git pull origin master

```

在本地切换到dev分支，没有分支创建

```
git checkout dev
```

查看dev分支的提交日志
```
git log --oneline
```

修改任意文件 做3次提交

```
git commit -m"xxx"
git commit -m"xxx"
git commit -m"xxx"
```

再次查看提交日志

```
git log --oneline
```

使用rebase合并commit

>不太明白为啥是master

```
git rebase -i master

```
这会打开编辑器

```
pick 32618c4 Start developing a feature
pick 62eed47 Fix something from the previous commit
 
# Commands:
#  p, pick = use commit
#  r, reword = use commit, but edit the commit message
#  e, edit = use commit, but stop for amending
#  s, squash = use commit, but meld into previous commit
#  f, fixup = like "squash", but discard this commit's log message
#  x, exec = run command (the rest of the line) using shell
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#
# Note that empty commits are commented out
```

pick就是使用commit

squash 将这次的commit合并到上一个commit中

fixup 和squash一样，但是不显示commit的log

使用fixup修改

```
pick 32618c4 Start developing a feature
fixup 62eed47 Fix something from the previous commit
 
# Commands:
#  p, pick = use commit
#  r, reword = use commit, but edit the commit message
#  e, edit = use commit, but stop for amending
#  s, squash = use commit, but meld into previous commit
#  f, fixup = like "squash", but discard this commit's log message
#  x, exec = run command (the rest of the line) using shell
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#
# Note that empty commits are commented out
```
然后wq保存

查看git 提交记录，只看到合并之后的commit了

```
git log --oneline
```

切换到主分支

```
git checkout master
```
将dev分支与master分支合并


```
git merge dev

```

查看master的日志

```
git log --oneline
```

```
$ git log --oneline
3fac5ad (HEAD -> master, dev) 修改主readme增加mitmproxy
77625f3 (origin/master, origin/dev, origin/HEAD) Update readme.md
def9d0e Update README.md
f9cd80b Update README.md
290cffd Update README.md
0c8b770 Delete test.py
91e8d1e Update README.md
2cafbde add autohomeCrawler
bab4c67 fix sohu_news select and upadte progress
2fdd921 Delete code_dict.txt
1fdb1fe Rename Readme.md to readme.md
ab88072 Update readme.md
ffe1cae init commit

```

推送到github

```
git push
```

### git commit 一个标题加一个描述

快速commit -m

方法一：
```
git commit -m '' -m ''
```

方法二：
```
git commit  -v
首行标题 次行换行 接着 描述
```

方法三： 
```
git commit
直接回车
```

### commit -m 的内容写错

git commit 提供了一个参数  --amend

可以修改最后一次提交的信息


### 写好一个commit的方法

- 格式
  ```
  类型 标题
  // 空一行
  描述详情
  ```
- 类型
  * init：初始化项目
  * feat：新功能
  * fix：修复bug
  * docs：文档
  * opt：优化与完善
  * style：格式变动
  * refactor：重构
  * test：测试用例
  * save：单纯保存
  * other：难以分类

  ```
  e.g：
  feat(#Student):增加学生数据库 --增加学生管理功能
  ```
- emoji
  这个网站是一个很好的类型标识标签图。可以帮助我快速区分每次commit的变动。
  [https://gitmoji.carloscuesta.me/](https://gitmoji.carloscuesta.me/)
  常用的几个
  * :bug: `:bug:` 修复错误。
  * :fire: `:fire:` 删除代码或文件。
  * :zap: `:zap:` 提高性能。
  *  :art:`:art:` 改进代码的结构/格式。
  * :sparkles: `:sparkles:` 引入新功能。
  * :pencil: `:pencil:` 编写文档。
  * :white_check_mark: `:white_check_mark:` 更新测试。
  * :lock: `:lock:` 解决安全问题。
  * :penguin: `:penguin:` 在Linux上修复问题。
  * :checkered_flag: `:checkered_flag:` 在Windows上修复问题。
  * :whale: `:whale:` 关于Docker的工作。
  * :wrench: `:wrench:` 更改配置文件。
  * :pencil2: `:pencil2:` 修正错别字。
  * :see_no_evil: `:see_no_evil:` 添加或更新.gitignore文件
  * :triangular_flag_on_post: `:triangular_flag_on_post:` 添加，更新或删除功能标志
  * :beers: `:beers:` 干杯