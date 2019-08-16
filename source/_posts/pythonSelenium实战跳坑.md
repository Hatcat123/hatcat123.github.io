---
title: pythonSelenium实战跳坑
tags:
  - python
  

date: 2019-5-3 13:14:00


categories:
 - python

---
pythonSelenium实战跳坑
关于启动selenium开启`DevTools listening `
<!--more-->

当我使用selenium操作google浏览器的时候，用pycharm运行没问题，可是用终端运行  就会卡在监听
```
    def do_work(self,urls):
        chrome_options = webdriver.ChromeOptions()
        chrome_options.add_argument('--disk-cache-dir=./cache')
        self.browser = webdriver.Chrome(chrome_options=chrome_options)
```

终端显示
```
DevTools listening on ws://127.0.0.1:21466/devtools/browser/a7850440-0dfa-4837-9366-109ca4bbd523


```


解决方式

关闭已打开的Chrome后台实例，初始化Chrome实例时指定driver的path。