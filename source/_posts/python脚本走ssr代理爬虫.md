---
title: python脚本走ssr代理爬虫
tags:
  - python爬虫
  
categories:
  - python

copyright: true
permalink: python脚本走ssr代理爬虫
top: 0
password: 
abstract: 输入密码继续阅读.
message: 欢迎来到我的博客，输入密码继续阅读.
date: 2019-05-04 21:35:51

---

python脚本走ssr代理爬虫，访问外网,解除了我之前的疑问
<!--more-->

在我之前的认知里，我不太了解这个代理的真正含义，总想着用python爬虫爬取外网的话就要本地连接vps，我意识中的代理只用在国内的爬虫，显然到今天才知道这样是不对的。

`requests`库一样可以ssr的代理。

在我使用代理查看国外的文章时候，无意间f12看到网页请求的地址竟然是本地`127.0.0.1:1081`，我就尝试使用requests加上代理能不能访问，结果是可以的。

```
import requests
import time

def req_tianmao(url):
    proxies = {
        "http": "http://127.0.0.1:1080",
        "https": "http://127.0.0.1:1080",
    }
    req = requests.get(url,proxies=proxies)
    text = req.text
    return text
```
然后使用异步协程`aiohttp`,发现源码里没有使用https代理的的情况，原作者说这将是一件麻烦的事情。

``` 部分源码
        self.proxies = {
            "http": "http://127.0.0.1:1081",
            "https": "http://127.0.0.1:1081",
        }

    async def get(self, url):
        async with aiohttp.ClientSession() as session:
            async with session.get(url, proxy=self.proxies.get('http'), timeout=30) as resp:
                print('网站的状态', resp.status)
                print(resp.url)
                result = await resp.text()
        return result

```

记录开启新世界的一次探索




