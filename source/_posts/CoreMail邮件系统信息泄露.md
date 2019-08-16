---
title: CoreMail邮件系统信息泄露
tags:
  - 安全
categories:
  - 安全
copyright: true
permalink: CoreMail邮件系统信息泄露
top: 0
password: 
abstract: 输入密码继续阅读.
message: 欢迎来到我的博客，输入密码继续阅读.
date: 2019-06-14 19:21:19
---

今天主流的邮件系统Coremail爆出信息泄露漏洞。
人生苦短我用python
<!--more-->

前两天参与到了红队信息扫描，分到一个国企的目标。目标的站关了很多，剩下主站和一个邮件系统-coremail。大佬让找coremail的漏洞信息。找了一圈翻到了以往爆出漏洞的信息，具体细节也没给。万万没想到在最无助的时候，直接爆出0day。


若不是HW估计也不会爆出这个0day漏洞。

### Coremail
Coremail大规模电子邮件系统，是拥有自主知识产权的专业邮件软件产品，在中国大陆地区拥有10亿终端用户，为163、139等超大型运营商提供邮件底层完整支持，同时也为国务院新闻办公室、中国科学院、清华大学、等政府、科教、500强企业客户等提供邮件系统整体解决方案，是中国用户实际使用最广泛、最频繁的邮件系统。


### 漏洞原理

该漏洞可通过固定的url地址查看coremail的配置文件信息，使邮件系统关键配置通过http协议进行读取，且无需身份验证。

使用域名后加上
```
/mailsms/s?func=ADMIN:appState&dumpConfig=/
```

回显信息

![](https://raw.githubusercontent.com/Hatcat123/GraphicBed/master/Img/20190614213615.png)


### 漏洞危害

通过漏洞可以看到配置的文件信息。sql数据库的ip、端口、密码、默认邮箱、密码。以及敏感文件的路径，这些都有可能对邮件系统造成危害。

### 修复建议

目前官方暂时没有推出修复方案。企业还是建议通过vpn连接，降低信息泄露的风险。

该漏洞仅对Coremail XT3有效，对最新版XT5无效，尽快升级至最新版

### 后记


通过fofa找到大概20多个站，几乎80%都能用。通过泄露的信息，能够登陆到邮件系统内部。看了虎牙、YY的邮件系统，到下午2点左右他们就采取了措施，已经poc已经不能回显信息了。

```

<result>
<code>FS_IP_NOT_PERMITTED</code>
</result>
```

