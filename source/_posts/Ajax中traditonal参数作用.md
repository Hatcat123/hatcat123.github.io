---
title: Ajax中traditonal参数作用
tags: 
- Ajax
- Flask
categories:
- 前端
copyright: true
permalink: Ajax中traditonal参数作用
top: 0
password: 
abstract: 输入密码继续阅读.
message: 欢迎来到我的博客，输入密码继续阅读.
date: 2019-12-04 20:25:56
---

在写Flask的时候遇到这样的问题，前端使用`Ajax`传递数据的时候，要不是数字类型，要不就是字符串，没有传递过数组(在`js`中的叫法，python中交列表，但是使用效果基本相同。)等复杂的数据类型。

<!--more-->
这次场景中就遇到了这个问题，比如批量删除文章时，我需要传递多个文章的ID，需要一个数据类型包裹这些ID。前端知识不足的我，之前都是先存成一个数组，将这些ID `push`到数组中，然后再用某个特殊符号（我通常使用`,`）来`join`成字符串，传递给后端。这样就出现了好多麻烦的地方：

- 前端代码过于冗杂
- 前后端需要商量拼接字符串,如我常用的`,`字符,还好前后端我都写，这个麻烦可能在我这不算问题
- 后端需要的过滤条件增多

直到找到`Ajax`中的`traditional `参数，完全可以解决上面麻烦。

在`Ajax`的参数中`traditional `参数默认`false`,在这个状态下，data中的value数据类型只支持数字、字符串，如果设置为`true`就可以支持数字以及字典了，其中的关系到序列化的问题。

```javascript
$.ajax{
url:"xxxx",
traditional: true,
data:{
	p: values 
	}
}
```

同时Flask也有函数专门接受数组数据

```python
p = request.form.getlist('p')  #p就是传递的列表
```

