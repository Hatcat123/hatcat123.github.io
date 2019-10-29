---
title: selenium过美团验证笔记
tags:
  - selenium
  - 爬虫
  
categories:
  - python
 
copyright: true
permalink: selenium过美团验证笔记
top: 0
password: woaini2.
abstract: 输入密码继续阅读.
message: 欢迎来到我的博客，输入密码继续阅读.
date: 2019-10-28 10:56:32
---

## selenium过美团验证，去除模拟浏览器属性
<!--more-->

## 0x01 序言

在使用那种程序直接简单调用浏览器内核的操作都会在模拟浏览器中标有特定的属性。一些电商的网站反爬的墙比较高。往往都有一些数据加密，普通的爬取不是很方便。
当爬取的复杂度过大时，便会选择模拟浏览的方式爬取。虽然这种方式的效率比较低，但比爬取不到数据要好的多，不必再费心网站数据的加解密。所需要的数据绝大多数都会在浏览器的页面中展示出来。

电商的网站当然也了解用户使用模拟浏览器获取数据，这些流量都是无用的。所以出现了网站使用js来判断浏览器的属性。真实的浏览器与模拟浏览器中的参数会有所不同。反爬虫就抓住这点设置针对模拟浏览器的反爬。

## 0x02 爬虫工具selenium

使用selenium模拟浏览器进行数据抓取无疑是当下最通用的数据采集方案，它通吃各种数据加载方式，能够绕过客户JS加密，绕过爬虫检测，绕过签名机制。它的应用，使得许多网站的反采集策略形同虚设。由于selenium不会在HTTP请求数据中留下指纹，因此无法被网站直接识别和拦截。

这是不是就意味着selenium真的就无法被网站屏蔽了呢？非也。selenium在运行的时候会暴露出一些预定义的Javascript变量（特征字符串），例如"window.navigator.webdriver"，在非selenium环境下其值为undefined，而在selenium环境下，其值为true（如下图所示为selenium驱动下Chrome控制台打印出的值）

### 不仅仅这些属性，还有一些其他属性（不同浏览器内核的属性不同）

```
webdriver  
__driver_evaluate  
__webdriver_evaluate  
__selenium_evaluate  
__fxdriver_evaluate  
__driver_unwrapped  
__webdriver_unwrapped  
__selenium_unwrapped  
__fxdriver_unwrapped  
_Selenium_IDE_Recorder  
_selenium  
calledSelenium  
_WEBDRIVER_ELEM_CACHE  
ChromeDriverw  
driver-evaluate  
webdriver-evaluate  
selenium-evaluate  
webdriverCommand  
webdriver-evaluate-response  
__webdriverFunc  
__webdriver_script_fn  
__$webdriverAsyncExecutor  
__lastWatirAlert  
__lastWatirConfirm  
__lastWatirPrompt  
$chrome_asyncScriptInfo  
$cdc_asdjflasutopfhvcZLmcfl_  
```


## 0x03 美团案例

>案例遇到的也有淘宝的滑动验证，如果是模拟浏览器操作无论怎么滑动都不能过验证码

一个美团的案例：

当用户使用模拟浏览器去访问美团的网站具体商家列表时候，会返回一个`请求异常、操作拒绝`的页面。

使用正常的浏览器打开是正常的数据展示

分析页面源码中js，`https://static.meituan.net/bs/yoda-static/file:file/d/js/yoda.xxxx.js`。将这个js文件格式化后发现检测了`webdriver`, `__driver_evaluate`, `__webdriver_evaluate`等等这些selenium的特征串。提交验证码的时候抓包可以看到一个_token参数（很长），selenium检测结果应该就包含在该参数里，服务端借以判断“请求异常,拒绝操作”

我们已经知道了屏蔽的原理，只要我们能够隐藏这些特征串就可以了。但是还不能直接删除这些属性，因为这样可能会导致selenium不能正常工作了。我们采用曲线救国的方法，使用中间人代理，比如fidder, proxy2.py或者mitmproxy，将JS文件（本例是yoda.*.js这个文件）中的特征字符串给过滤掉（或者替换掉，比如替换成根本不存在的特征串），让它无法正常工作，从而达到让客户端脚本检测不到selenium的效果。

只使用一点点的webdriver的方式去除属性都太弱了。你能想到、源码能做到、当然反爬肯定也知道。

## 0x04 mitmproxy

我已经多次使用mitmproxy这个代理，尤其对应电商网站监测模拟浏览器属性的特别有效。当然理论上任何的代理工具都可以，只是mimtproxy支持python，对于python的爬虫更顺手。

下载安装使用看文档

当请求的连接是包含`/js/yoda.`（也就是这个检测的js），在响应的数据包中将这些特定属性的参数去掉。

windows运行方式
> mitmproxy在linux运行
```
mitmdump.exe -S proxy.py   -p 8888
```
这样就会设置一个8888代理的代理

```proxy.py
# coding: utf-8  
# modify_response.py  
  
import re  
from mitmproxy import ctx  
    
def response(flow):  
  """修改应答数据 
  """  
  if '/js/yoda.' in flow.request.url:  
      # 屏蔽selenium检测  
      for webdriver_key in ['webdriver', '__driver_evaluate', '__webdriver_evaluate', '__selenium_evaluate', '__fxdriver_evaluate', '__driver_unwrapped', '__webdriver_unwrapped', '__selenium_unwrapped', '__fxdriver_unwrapped', '_Selenium_IDE_Recorder', '_selenium', 'calledSelenium', '_WEBDRIVER_ELEM_CACHE', 'ChromeDriverw', 'driver-evaluate', 'webdriver-evaluate', 'selenium-evaluate', 'webdriverCommand', 'webdriver-evaluate-response', '__webdriverFunc', '__webdriver_script_fn', '__$webdriverAsyncExecutor', '__lastWatirAlert', '__lastWatirConfirm', '__lastWatirPrompt', '$chrome_asyncScriptInfo', '$cdc_asdjflasutopfhvcZLmcfl_']:  
          ctx.log.info('Remove "{}" from {}.'.format(webdriver_key, flow.request.url))  
          flow.response.text = flow.response.text.replace('"{}"'.format(webdriver_key), '"NO-SUCH-ATTR"')  
      flow.response.text = flow.response.text.replace('t.webdriver', 'false')  
      flow.response.text = flow.response.text.replace('ChromeDriver', '') 
```

## 0x05 selenium

在selenium操作方面使用的是chrome的内核，比较快。

只要给他设置一个代理就可。

```
from selenium import webdriver#模拟浏览器，爬取异步加载的网页
from selenium.webdriver.chrome.options import Options

chrome_options = Options()
chrome_options.add_experimental_option('excludeSwitches', ['enable-automation'])  # 此步骤很重要，设置为开发者模式，防止被各大网站识别出来使用了Selenium

chrome_options.add_argument('--proxy-server=http://127.0.0.1:8080')  # IP为你的代理服务器地址:如‘127.0.0.0’，字符串类型


driver =  webdriver.Chrome( chrome_options=chrome_options)
driver.get(url)#driver.get返回该页面所有东西，包括异步加载；requests.get返回当前网页源代码
driver.implicitly_wait(10)#等待10秒，滑块操作
#content = driver.find_element_by_xpath('//*[@id="region-nav"]').text
print(BeautifulSoup(driver.page_source, "lxml"))
```

如果不能连上代理，需要个浏览器加上mitm的证书


## 0x06 总结

碰到的都使用这种方法，十分好用。mitm做一个简单被动扫描器也是很ok的。另外一个用途代理可以改变爬虫的方向也是十分好用。

















