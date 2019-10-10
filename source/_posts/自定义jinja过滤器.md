---
title: 自定义jinja过滤器
tags:
  - flask
categories:
  - flask
copyright: true
permalink: 自定义jinja过滤器
top: 0
password: 
abstract: 输入密码继续阅读.
message: 欢迎来到我的博客，输入密码继续阅读.
date: 2019-10-10 11:09:51
---

使用jinja2自定义拓展过滤器，从简单的例子到源码的分析
<!--more-->
## 一个最小列子

将过滤器写成一个函数，然后再filter过滤函数中定义一个tag对于应到定义的过滤函数
```
import markdown

def markdown2html(text):
    return markdown.markdown(text, extensions=['extra'])

app=Flask(__name__)
env =app.jinja_env
env.filters['markdown1']=markdown2html
```
```
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Title</title>
</head>
<body>
{{post|markdown1|safe}}
</body>
</html>
```

上面是一个比较简单的例子，很方便实用，看看标准写法怎么集成的

## 标准库怎么做

在使用flask-markdown的仓库的时候，想要在前端使用`markdown`这个过滤器，看到文档中说这个仓库已经集成了`markdown`方法的过滤器，这个过滤器的作用是将md文档转换成html文档。

看了一下flask-markdown的库

```
# -*- coding: utf-8 -*-
"""
    flask_markdown
    ~~~~~~~~~~~~~~

    A flask extension to add markdown support.

    :copyright: (c) 2013 by Daniel Chatfield
"""

from jinja2_markdown import MarkdownExtension


def markdown(app):
    app.jinja_env.add_extension(MarkdownExtension)

```

这里引入了一个初始化app，然后将app中的jinja环境添加以一个拓展`MarkdownExtension`。看下拓展源码里面是什么

```
# -*- coding: utf-8 -*-
"""
    jinja2_markdown
    ~~~~~~~~~~~~~~~~~~~~~~~~~

    A jinja2 extension that adds a `{% markdown %}` tag.

    :copyright: (c) 2014 by Daniel Chatfield
"""

import markdown
from jinja2.nodes import CallBlock
from jinja2.ext import Extension


class MarkdownExtension(Extension):
    tags = set(['markdown'])

    def __init__(self, environment):
        super(MarkdownExtension, self).__init__(environment)
        environment.extend(
            markdowner=markdown.Markdown(extensions=['extra'])
        )

    def parse(self, parser):
        lineno = next(parser.stream).lineno
        body = parser.parse_statements(
            ['name:endmarkdown'],
            drop_needle=True
        )
        return CallBlock(
            self.call_method('_markdown_support'),
            [],
            [],
            body
        ).set_lineno(lineno)

    def _markdown_support(self, caller):
        block = caller()
        block = self._strip_whitespace(block)
        return self._render_markdown(block)

    def _strip_whitespace(self, block):
        lines = block.split('\n')
        whitespace = ''
        output = ''

        if (len(lines) > 1):
            for char in lines[1]:
                if (char == ' ' or char == '\t'):
                    whitespace += char
                else:
                    break

        for line in lines:
            output += line.replace(whitespace, '', 1) + '\r\n'

        return output.strip()

    def _render_markdown(self, block):
        block = self.environment.markdowner.convert(block)
        return block

```
在`MarkdownExtension`拓展类中初始化的方法实际上和我最开始写的自定义模板是一样的功能`markdowner=markdown.Markdown(extensions=['extra'])`，但是下面为啥要写这么多东西。
这是拓展的另一种写法，带着注释分析下

```
# -*- coding: utf-8 -*-
"""
    jinja2_markdown
    ~~~~~~~~~~~~~~~~~~~~~~~~~

    A jinja2 extension that adds a `{% markdown %}` tag.

    :copyright: (c) 2014 by Daniel Chatfield
"""

import markdown
from jinja2.nodes import CallBlock
from jinja2.ext import Extension

# 首先引入jinja2的拓展包与一个回调函数

class MarkdownExtension(Extension):
# 自定义一个拓展类，集成Extension这个拓展包
    tags = set(['markdown'])
# 定义该扩展的语句关键字，这里表示模板中的{% code %}语句会该扩展处理

    def __init__(self, environment):
# 初始化一个父类，模板是这样写的
        super(MarkdownExtension, self).__init__(environment)
# 在环境中引入一个拓展
        environment.extend(
            markdowner=markdown.Markdown(extensions=['extra'])
        )

# 重写Extension中的parse函数
# 在Extension中parse函数的注释
``
		If any of the :attr:`tags` matched this method is called with the
        parser as first argument.  The token the parser stream is pointing at
        is the name token that matched.  This method has to return one or a
        list of multiple nodes.
``
    def parse(self, parser):

		# 进入此函数时，即表示{% code %}标签被找到了
        # 下面的代码会获取当前{% code %}语句在模板文件中的行号
        lineno = next(parser.stream).lineno

        # 获取{% code %}语句中的参数，比如我们调用{% code 'python' %}，
        # 这里就会返回一个jinja2.nodes.Const类型的对象，值为'python'
        # 解析从{% code %}标志开始，到{% endcode %}为止中间的所有语句
        # 将解析完后的内容存在body里，并将当前流位置移到{% endcode %}之后
        body = parser.parse_statements(
            ['name:endmarkdown'],
            drop_needle=True
        )

        # 返回一个CallBlock类型的节点，并将其之前取得的行号设置在该节点中
        # 初始化CallBlock节点时，传入我们自定义的"_markdown_support"方法的调用，
        # 两个空列表，还有刚才解析后的语句内容body
        return CallBlock(
            self.call_method('_markdown_support'),
            [],
            [],
            body
        ).set_lineno(lineno)

    # 这个自定义的内部函数，包含了本扩展的主要逻辑。
    # 其实上面parse()函数内容，大部分扩展都可以重用
    def _markdown_support(self, caller):
        block = caller()
        block = self._strip_whitespace(block)
        return self._render_markdown(block)

    def _strip_whitespace(self, block):
        lines = block.split('\n')
        whitespace = ''
        output = ''

        if (len(lines) > 1):
            for char in lines[1]:
                if (char == ' ' or char == '\t'):
                    whitespace += char
                else:
                    break

        for line in lines:
            output += line.replace(whitespace, '', 1) + '\r\n'

        return output.strip()

    def _render_markdown(self, block):
        block = self.environment.markdowner.convert(block)
        return block

```
总的来说，扩展中核心部分就在parse()函数里，而最关键的就是这个parser对象，它是一个jinja2.parser.Parser的对象。

* parser.stream 获取当前的文档处理流，它可以基于文档中的行迭代，所以可以使用next()方法向下一行前进，并返回当前行
* parser.parse_expression() 解析下一个表达式，并将结果返回
* parser.parse_statements() 解析下一段语句，并将结果返回。可以连续解析多行。它有两个参数 
	1. 第一个是结束位置end_tokens，上例中是`{ % endcode % }`标签，它是个列表，可是设置多个结束标志，遇到其中任意一个即结束
	2. 第二个是布尔值drop_needle，默认为False，即解析完后流的当前位置指向结束语句`{ % endcode % }`之前。设为True时，即将流的当前位置设在结束语句之后

在parse()函数最后，我们创建了一个nodes.CallBlock的块节点对象，并将其返回。初始化时，我们先传入了_markdown_support()方法的调用；然后两个空列表分别对应了字段和属性，本例中用不到，所以设空；再传入解析后的语句块body。CallBlock节点初始化完后，还要记得将当前行号设置进去。接下来，我们对于语句块的所有操作，都可以写在_markdown_support()方法里了。

_markdown_support()里的内容我就不多介绍了，只需要记得声明这个方法时，最后一定要接收一个参数caller，它是个回调函数，可以获取之前创建CallBlock节点时传入的语句块内容。

## 总结

代码看一遍发现flask-markdown就是一个自定义过滤器，也就这一个功能。
然而在我前端调用的时候发现还不能进行使用`markdown`的过滤器。就暂时使用最简单的方法定义过滤器，回头想起来再看下库怎么回事。