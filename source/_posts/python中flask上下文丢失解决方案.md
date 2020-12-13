---
title: python中flask上下文丢失解决方案
tags:
    - flask
categories:
  - flask
copyright: true
permalink: python中flask上下文丢失解决方案
top: 0
password: 
abstract: 输入密码继续阅读.
message: 欢迎来到我的博客，输入密码继续阅读.
date: 2019-09-08 23:56:33
---
RuntimeError: Working outside of application context.


这个问题的原因是在没有激活程序上下文之前进行了一些程序上下文或请求上下文的操作 
<!--more-->



解决办法很简单就是推送程序上下文，在获得程序上下文后再执行相应的操作 
方法 1

```
rom myapp import app#myapp是我的程序文件，里面初始了Flask对象app
from flask import current_app
with app.app_context():
    print current_app.name
```
方法二
```
from myapp import app
from flask import current_app
app_ctx = app.app_context()
app_ctx.push()

print current_app.name

app_ctx.pop()
```


## 循环导入

碰到的问题：循环导入问题，初始化某个插件，需要导入flask实例化的配置信息。但是还没有实例化完，就导致了初始化失败，提示循环导入没有上下文。

之前对着一知半解，今天又专门看了一些文章，才缕清一直以来的疑惑，为什么解决循环导入可以从上下文中找到实例化的对象？


在初始化flask的时候，如果加入了三方的库，会用到`xx.init_app(app)`的方法，还有一个方式`xx.app=app`。在此之前我一直认为他们的作用是一样的，看源码的时候大致看了下都包含在一个类里面，就觉得是一致的东西。

现在发现他们是两个不一样的方法。而且不同的三方库还有可能更不一样。所以用三方库的时候，可以大致的看一眼源码的内容。

`init_app(app)`初始化app。将app里面的配置信息传给三方库调用

`xx.app=app`传入一个app的实例。

为啥说他门不一样。都有三方库写的时候将`xx.app=app`的方法放到`init_app`的方法中。而有的源码是将两者分开的。

这也就是我一开始没有注意到。有的时候能在第三方库中调用app。有的时候提示我上下文中没有初始得app。

如看下面的源码：
```
class SQLite3(object):

    def __init__(self, app=None):
        if app is not None:
            self.app = app
            self.init_app(self.app)
        else:
            self.app = None

    def init_app(self, app):
        app.config.setdefault('SQLITE3_DATABASE', ':memory:')
```
只有`xx.app=app`的方法才会将app的实例对象传入到插件中，另外也会执行初始化`init_app`加载配置文件
而单单的`xx.init_app(app)`只会加载配置，并不会传入对象，所以这下明白了为啥后面再上下文中缺少`app`的实例化对象。

更稳妥的方式是看一眼源码再决定是否使用`xx.app=app`。绝大多数`xx.app=app`包含了`init_app()`的方法，但不能保证那个晕子开发者的奇葩写法。


### 实际问题

这也是我遗留下得老问题了。之前初始化app得时候使用得`flask_sqlalchemy`。以至于我写redis初始化加载配置得时候，找不到app.config。就算引入得上下文得函数，还是提示我找不到上下文。

现在看来一切都是明朗的。没有使用`db.app=app`确实不会再上下文中找到`app`的配置。


## 动态加载配置

### 动态加载邮件服务求配置

写好邮件服务器功能后，想要在后台动态配置邮件服务器的内容，又笨的方法加载和修改一个本地文件来重启邮件服务器。这样相对麻烦且不友好。

当修改邮件配置后，我们重新的将appconfig的配置信息更新一次，更新配置文件可以使用`current_app.config.update`的方式。但是这中更新只是更新了配置信息。email加载的实例还是没有更新，实例的配置仍然是上次配置的信息。

我们还有重新的实例化一次email插件成对象。







