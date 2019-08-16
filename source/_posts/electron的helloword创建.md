---
title: electron的helloword创建
tags:
  - electron
categories:
  - 前端
copyright: true
permalink: electron的helloword创建
top: 0
password: 
abstract: 输入密码继续阅读.
message: 欢迎来到我的博客，输入密码继续阅读.
date: 2019-07-20 16:52:00
---
Electron 可以让你使用纯 JavaScript 调用丰富的原生(操作系统) APIs 来创造桌面应用。 专注于桌面应用而不是 Web 服务器端。
人生苦短我用python
<!--more-->



 ## electron[安装](https://electronjs.org/docs/tutorial/installation)

现在还没太搞明白安装的环境的问题，在别人博客的文章里安装的方式千奇百怪，我的npm下了cnpm用的淘宝的源

应该是安装到全局了，因为我在任何一个cmd中都能使用`electron -v`的方式展示版本信息。应该是一个exe的文本
```
npm install -g electron-prebuilt
```
在app中安装，在app文件夹中运行下面的命令。应该是和python的虚拟环境类似。一个开发包中安装一个版本的ele。

```
npm install --save-dev electron
```

## Hello World

官方给出的一个helloworld包。
```
# 克隆这仓库
$ git clone https://github.com/electron/electron-quick-start
# 进入仓库
$ cd electron-quick-start
# 安装依赖库
$ npm install
# 运行应用
$ npm start
```
在安装依赖的过程中，出现了一些小的问题。

```

E:\desktop_app\electron-quick-start\node_modules\_electron@5.0.7@electron\index.js:14
    throw new Error('Electron failed to install correctly, please delete node_modules/electron and try installing again')
    ^

Error: Electron failed to install correctly, please delete node_modules/electron and try installing again
    at getElectronPath (E:\desktop_app\electron-quick-start\node_modules\_electron@5.0.7@electron\index.js:14:11)
    at Object.<anonymous> (E:\desktop_app\electron-quick-start\node_modules\_electron@5.0.7@electron\index.js:18:18)
    at Module._compile (internal/modules/cjs/loader.js:723:30)
    at Object.Module._extensions..js (internal/modules/cjs/loader.js:734:10)
    at Module.load (internal/modules/cjs/loader.js:620:32)
    at tryModuleLoad (internal/modules/cjs/loader.js:560:12)
    at Function.Module._load (internal/modules/cjs/loader.js:552:3)
    at Module.require (internal/modules/cjs/loader.js:659:17)
    at require (internal/modules/cjs/helpers.js:22:18)
    at Object.<anonymous> (E:\desktop_app\electron-quick-start\node_modules\_electron@5.0.7@electron\cli.js:3:16)
npm ERR! code ELIFECYCLE
```
按照提示删除了文件重新install。仍然不能解决，在[github的iss](https://github.com/electron/electron/issues/8466)中小哥说道

Try：
```
npm install electron --verbose
```
回显


```

E:\desktop_app\electron-quick-start>npm install electron --verbose
npm info it worked if it ends with ok
npm verb cli [ 'D:\\Program Files\\nodejs\\node.exe',
npm verb cli   'C:\\Users\\Administrator\\AppData\\Roaming\\npm\\node_modules\\npm\\bin\\npm-cli.js',
npm verb cli   'install',
npm verb cli   'electron',
npm verb cli   '--verbose' ]
npm info using npm@6.9.0
npm info using node@v11.4.0
npm verb npm-session 785d3e0a6678bd32
npm http fetch GET 304 https://registry.npmjs.org/electron 2168ms (from cache)
npm timing stage:loadCurrentTree Completed in 3236ms
npm timing stage:loadIdealTree:cloneCurrentTree Completed in 5ms
npm timing stage:loadIdealTree:loadShrinkwrap Completed in 165ms
npm http fetch GET 304 https://registry.npmjs.org/@types%2fnode 280ms (from cache)
npm http fetch GET 304 https://registry.npmjs.org/extract-zip 740ms (from cache)
npm http fetch GET 304 https://registry.npmjs.org/electron-download 1759ms (from cache)
```

再次运行

```
nmp start
```

![](https://raw.githubusercontent.com/Hatcat123/GraphicBed/master/Img/20190719172128.png)

第一个桌面级应用产生。

TODO：这么好用的工具，希望之后能放弃难看的tk。使用ele写一个应用。





