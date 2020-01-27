---
title: editormd动态加载文本
tags:
  - 研发
categories:
  - 研发
copyright: true
permalink: editormd动态加载文本
top: 0
password: woaini2.
abstract: 输入密码继续阅读.
message: 欢迎来到我的博客，输入密码继续阅读.
date: 2020-01-09 16:10:47
---
使用editor.md编辑器解决动态替换数据的需求
<!--more-->

editor.md是一个很好的md编辑器。已经不在维护。

[http://editor.md.ipandao.com/examples/set-get-replace-selection.html](http://editor.md.ipandao.com/examples/set-get-replace-selection.html)

使用过程中渲染数据是靠第一次加载的数据。当我想通过js传入某些值重新渲染的时候，发现不能直接赋值与textarea。

通过官方文档，没有找到类似的接口。

但是有`插入字符 / 设置和获取光标位置 / 设置、获取和替换选中的文本`的功能。
其中替换文本与选取光标可以组合成一个`伪`赋值。

选中当前所有内容，然后替换成新的内容。再将光标移动至顶端。

选中内容
```
$("#btn3").click(function(){
    testEditormd.setSelection({line:1, ch:0}, {line:5, ch:100});
});

```

替换内容

```

$("#btn5").click(function(){
    testEditormd.replaceSelection("$$$$$$$$$");
});
```
设置光标位置

```
$("#btn1").click(function(){
    testEditormd.setCursor({line:1, ch:2});
});
```
