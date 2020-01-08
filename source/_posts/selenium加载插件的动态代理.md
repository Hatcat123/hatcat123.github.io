---
title: selenium加载插件的动态代理
tags:
  - selenium
categories:
  - 爬虫
copyright: true
permalink: selenium加载插件的动态代理
top: 0
password: 
abstract: 输入密码继续阅读.
message: 欢迎来到我的博客，输入密码继续阅读.
date: 2019-12-31 23:49:28
---

人生苦短我用python
<!--more-->

selenium使用代理的方式（chrome）：


### 固定ip的代理，连接无用户名密码认证的代理


```
from selenium import webdriver
chromeOptions = webdriver.ChromeOptions()
 
# 设置代理
chromeOptions.add_argument("--proxy-server=http://host:port")
# 一定要注意，=两边不能有空格，不能是这样--proxy-server = http://202.20.16.82:10152
browser = webdriver.Chrome(chrome_options = chromeOptions)
 
# 查看本机ip，查看代理是否起作用
browser.get("http://httpbin.org/ip")
print(browser.page_source)
 
# 退出，清除浏览器缓存
browser.quit()
```

### 有用户名和密码的连接

```
from selenium import webdriver
def create_proxyauth_extension(proxy_host, proxy_port,
                               proxy_username, proxy_password,
                               scheme='http', plugin_path=None):
    """Proxy Auth Extension

    args:
        proxy_host (str): domain or ip address, ie proxy.domain.com
        proxy_port (int): port
        proxy_username (str): auth username
        proxy_password (str): auth password
    kwargs:
        scheme (str): proxy scheme, default http
        plugin_path (str): absolute path of the extension       

    return str -> plugin_path
    """
    import string
    import zipfile

    if plugin_path is None:
        plugin_path = 'd:/webdriver/vimm_chrome_proxyauth_plugin.zip'

    manifest_json = """
    {
        "version": "1.0.0",
        "manifest_version": 2,
        "name": "Chrome Proxy",
        "permissions": [
            "proxy",
            "tabs",
            "unlimitedStorage",
            "storage",
            "<all_urls>",
            "webRequest",
            "webRequestBlocking"
        ],
        "background": {
            "scripts": ["background.js"]
        },
        "minimum_chrome_version":"22.0.0"
    }
    """

    background_js = string.Template(
    """
    var config = {
            mode: "fixed_servers",
            rules: {
              singleProxy: {
                scheme: "${scheme}",
                host: "${host}",
                port: parseInt(${port})
              },
              bypassList: ["foobar.com"]
            }
          };

    chrome.proxy.settings.set({value: config, scope: "regular"}, function() {});

    function callbackFn(details) {
        return {
            authCredentials: {
                username: "${username}",
                password: "${password}"
            }
        };
    }

    chrome.webRequest.onAuthRequired.addListener(
                callbackFn,
                {urls: ["<all_urls>"]},
                ['blocking']
    );
    """
    ).substitute(
        host=proxy_host,
        port=proxy_port,
        username=proxy_username,
        password=proxy_password,
        scheme=scheme,
    )
    with zipfile.ZipFile(plugin_path, 'w') as zp:
        zp.writestr("manifest.json", manifest_json)
        zp.writestr("background.js", background_js)

    return plugin_path

proxyauth_plugin_path = create_proxyauth_extension(
    proxy_host="proxy.crawlera.com",
    proxy_port=8010,
    proxy_username="fea687a8b2d448d5a5925ef1dca2ebe9",
    proxy_password=""
)


co = webdriver.ChromeOptions()
co.add_argument("--start-maximized")
co.add_extension(proxyauth_plugin_path)


driver = webdriver.Chrome(chrome_options=co)
driver.get("http://www.amazon.com/")
```
以上直接通过python代码生成chrome所需的zip插件文件，IP端口用户名密码写上自己的，原文出处：

https://vimmaniac.com/blog/bangal/selenium-chrome-driver-proxy-with-authentication/

 

插件源代码 https://github.com/RobinDev/Selenium-Chrome-HTTP-Private-Proxy