---
title: 钉钉
date: 2019-04-26 16:21:50
tags:
 - python工具
 
categories:
 - python
 
---

记录使用钉钉开发者接口传输信息，比较好用。接口很丰富。
<!--more-->


[钉钉开发文档-群机器人](https://open-doc.dingtalk.com/microapp/serverapi2/qf2nxq)

钉钉的开发者文档看着比较舒服就仔细看看了

### 自定义机器人如何创建

创建自己的组织->创建群->创建添加自定义机器人（最多6个）->获得机器人的webhook->编程使用

## 机器人发送消息有6种

- text类型
- link类型
- markdow类型
- 整体跳转ActionCard类型
- 独立跳转ActionCard类型
- FeedCard类型

## 每种类型的格式

### text类型

```
{
    "msgtype": "text", 
    "text": {
        "content": "我就是我, 是不一样的烟火@156xxxx8827"
    }, 
    "at": {
        "atMobiles": [
            "156xxxx8827", 
            "189xxxx8325"
        ], 
        "isAtAll": false
    }
}
```

text 简单的收发消息，能@成员，钉钉机器人接口有限制，一个机器人一分钟之内只能发送20条信息。对于信息内容阿里做了监控，屏蔽一些不健康的信息。

这里可以尝试买安恒密盾

### link类型
```

    "msgtype": "link", 
    "link": {
        "text": "这个即将发布的新版本，创始人陈航（花名“无招”）称它为“红树林”。
而在此之前，每当面临重大升级，产品经理们都会取一个应景的代号，这一次，为什么是“红树林”？", 
        "title": "时代的火车向前开", 
        "picUrl": "", 
        "messageUrl": "https://www.dingtalk.com/s?__biz=MzA4NjMwMTA2Ng==&mid=2650316842&idx=1&sn=60da3ea2b29f1dcc43a7c8e4a7c97a16&scene=2&srcid=09189AnRJEdIiWVaKltFzNTw&from=timeline&isappinstalled=0&key=&ascene=2&uin=&devicetype=android-23&version=26031933&nettype=WIFI"
    }
}
```

link不能跳转到app内应用

### markdow类型
```
{
     "msgtype": "markdown",
     "markdown": {
         "title":"杭州天气",
         "text": "#### 杭州天气 @156xxxx8827\n" +
                 "> 9度，西北风1级，空气良89，相对温度73%\n\n" +
                 "> ![screenshot](https://gw.alipayobjects.com/zos/skylark-tools/public/files/84111bbeba74743d2771ed4f062d1f25.png)\n"  +
                 "> ###### 10点20分发布 [天气](http://www.thinkpage.cn/) \n"
     },
    "at": {
        "atMobiles": [
            "156xxxx8827", 
            "189xxxx8325"
        ], 
        "isAtAll": false
    }
 }
```
对于分散性的数据，可以集中的推送。减少接口掉的频率，同时增加客观性

### 整体跳转ActionCard类型

```
{
    "actionCard": {
        "title": "乔布斯 20 年前想打造一间苹果咖啡厅，而它正是 Apple Store 的前身", 
        "text": "![screenshot](serverapi2/@lADOpwk3K80C0M0FoA) 
 ### 乔布斯 20 年前想打造的苹果咖啡厅 
 Apple Store 的设计正从原来满满的科技感走向生活化，而其生活化的走向其实可以追溯到 20 年前苹果一个建立咖啡馆的计划", 
        "hideAvatar": "0", 
        "btnOrientation": "0", 
        "singleTitle" : "阅读全文",
        "singleURL" : "https://www.dingtalk.com/"
    }, 
    "msgtype": "actionCard"
}
```
### 独立跳转ActionCard类型

```
{
    "actionCard": {
        "title": "乔布斯 20 年前想打造一间苹果咖啡厅，而它正是 Apple Store 的前身", 
        "text": "![screenshot](serverapi2/@lADOpwk3K80C0M0FoA) 
 ### 乔布斯 20 年前想打造的苹果咖啡厅 
 Apple Store 的设计正从原来满满的科技感走向生活化，而其生活化的走向其实可以追溯到 20 年前苹果一个建立咖啡馆的计划", 
        "hideAvatar": "0", 
        "btnOrientation": "0", 
        "btns": [
            {
                "title": "内容不错", 
                "actionURL": "https://www.dingtalk.com/"
            }, 
            {
                "title": "不感兴趣", 
                "actionURL": "https://www.dingtalk.com/"
            }
        ]
    }, 
    "msgtype": "actionCard"
}
```
### FeedCard类型

```
{
    "feedCard": {
        "links": [
            {
                "title": "时代的火车向前开", 
                "messageURL": "https://www.dingtalk.com/s?__biz=MzA4NjMwMTA2Ng==&mid=2650316842&idx=1&sn=60da3ea2b29f1dcc43a7c8e4a7c97a16&scene=2&srcid=09189AnRJEdIiWVaKltFzNTw&from=timeline&isappinstalled=0&key=&ascene=2&uin=&devicetype=android-23&version=26031933&nettype=WIFI", 
                "picURL": "https://www.dingtalk.com/"
            },
            {
                "title": "时代的火车向前开2", 
                "messageURL": "https://www.dingtalk.com/s?__biz=MzA4NjMwMTA2Ng==&mid=2650316842&idx=1&sn=60da3ea2b29f1dcc43a7c8e4a7c97a16&scene=2&srcid=09189AnRJEdIiWVaKltFzNTw&from=timeline&isappinstalled=0&key=&ascene=2&uin=&devicetype=android-23&version=26031933&nettype=WIFI", 
                "picURL": "https://www.dingtalk.com/"
            }
        ]
    }, 
    "msgtype": "feedCard"
}
```

![](https://raw.githubusercontent.com/Hatcat123/GraphicBed/master/Img/20190426164106.png)

## 代码示例python

```
import json
from datetime import datetime

import requests


class DingMsg():
    def __init__(self):
        self.HEADERS = {
            "Content-Type": "application/json ;charset=utf-8 "
        }

    def parse_data(self, data):
        self.keyword = data.get('keyword')
        self.img_src = data.get('img_src')
        self.title = data.get('title')
        self.xianyu_url = data.get('pic_href')
        self.pub_time = data.get('pub_time')
        self.price = data.get('price')
        self.location = data.get('location')
        self.desc = data.get('desc')

    def text_msg_content(self):
        self.parse_data(self.data)
        text = '{title}\n{xianyu_url}\n关键字：{keyword}\n价格：{price}  \n时间:{pub_time}'.format(title=self.title,
                                                                                          xianyu_url=self.xianyu_url,
                                                                                          keyword=self.keyword,
                                                                                          price=self.price,
                                                                                          pub_time=self.pub_time)

        return {
            "msgtype": "text",
            "text": {
                "content": text
            },
            "at": {
                # "atMobiles": [
                # ],
                "isAtAll": True
            }
        }

    def link_msg_content(self):
        self.parse_data(self.data)
        # 链接的话，手机端没法直接点进去展示，电脑端可以展示
        link_text = '{title}\n时间：{pub_time}'.format(title=self.title, pub_time=self.pub_time)
        return {
            "msgtype": "link",
            "link": {
                "text": link_text,
                "title": '闲鱼"{}"最新商品消息'.format(self.keyword),
                "picUrl": self.img_src,
                "messageUrl": self.xianyu_url
            }
        }

    def markd_msg_content(self):
        markdown_content = ""
        for item in self.data:
            self.parse_data(item)
            markdown_content += "---------------------------------\n \n" \
                                "#### **关键字:** {keyword}\n" \
                                "###### 价格:{price} 时间:{pub_time}\n" \
                                "#### 标题:{title}\n" \
                                "#### 链接:[{xianyu_url}]({xianyu_url})\n".format(keyword=self.keyword, price=self.price,
                                                                                pub_time=self.pub_time,
                                                                                title=self.title,
                                                                                xianyu_url=self.xianyu_url)
        return {
            "msgtype": "markdown",
            "markdown": {
                "title": "新增闲鱼商品",
                "text": "## **新增闲鱼商品** \n" +
                        " ###### {} 发布\n".format(datetime.now().strftime('%Y-%m-%d %H:%M:%S')) + markdown_content
            },
            "at": {
                # "atMobiles": [
                #     "156xxxx8827",
                #     "189xxxx8325"
                # ],
                "isAtAll": True
            }
        }

    def check_msg_content(self):
        return {
            "msgtype": "text",
            "text": {
                "content": self.data
            },
            "at": {
                # "atMobiles": [
                # ],
                "isAtAll": True
            }
        }

    def send_msg(self, webhook_url, data, type):
        # 这里的message是你想要推送的文字消息三种格式 ：text_msg_content  link_msg_content  markd_msg_content
        self.data = data
        if type == 1:
            mesBody = self.text_msg_content()
        elif type == 2:
            mesBody = self.link_msg_content()
        elif type == 3:
            mesBody = self.markd_msg_content()
        else:
            mesBody = self.check_msg_content()
        MessageBody = json.dumps(mesBody)
        result = requests.post(url=webhook_url, data=MessageBody, headers=self.HEADERS)
        print(result.text)
        try:
            if result.json().get('errmsg') == 'ok':
                return True
            elif result.json().get('errmsg') == '消息中包含不合适的内容':
                return True
            else:
                return False
        except Exception as e:
            return False

```