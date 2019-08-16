---
title: 华为交换机ACL使用
tags:
  - huawei
  - 交换机
  - acl

date: 2018-12-07 20:40:32

categories:
 - 运维
---
ACL是什么、有什么作用、分哪几类、规则是如何定义的、ACL的匹配机制、规则的匹配顺序、规则的匹配。
<!--more-->

本文介绍

ACL是什么、有什么作用、分哪几类、规则是如何定义的、ACL的匹配机制、规则的匹配顺序、规则的匹配。

### 0x01 ACL是什么
ACL是Access Control List的简称，中文名称叫“访问控制列表”，它由一系列规则（即描述报文匹配条件的判断语句）组成。这些条件，可以是报文的源地址、目的地址、端口号等。

### 0x02 ACL的类别

acl类别 | 定义与描述|编号范围
---|---|---
标准acl |仅使用报文的源IP地址、分片标记和时间段信息来定义规则|2000~2999
扩展acl | 既可使用报文的源IP地址，也可使用目的地址、IP优先级、ToS、DSCP、IP协议类型、ICMP类型、TCP源端口/目的端口、UDP源端口/目的端口号等来定义规则|3000~3999
二层ACL|可根据报文的以太网帧头信息来定义规则，如根据源MAC地址、目的MAC地址、以太帧协议类型等|4000～4999
用户自定义ACL|可根据报文偏移位置和偏移量来定义规则。|5000～5999
用户ACL|既可使用IPv4报文的源IP地址或源UCL（User Control List）组，也可使用目的地址或目的UCL组、IP协议类型、ICMP类型、TCP源端口/目的端口、UDP源端口/目的端口号等来定义规则|6000～9999



### 0x03 ACL定义规则

**acl应用原则**


标准acl，尽量使用在靠近目标的点。扩展acl，尽量使用在靠近源对的地方，节约保护带宽资源

**举例：基于时间的acl配置**

配置时间端>创建高级acl>配置流分类>配置流行为>配置流策略>应用流策略

**1.配置时间段**

```
time-range name-deny 6:00 to 6:30 daily
```

**2.创建高级acl**

```
acl number 3001
    rule deny ip source 192.168.0.1 0.0.0.255 time-range name-deny
```

**3.配置流分类**

```
traffic classifier athorname
    if-match acl 3001
```
**4.配置流行为**

```
traffic behavior athorname
    deny
```
**5.配置流策略**

```
traffic policy all_deny
    ckassifier athorname behavior name-deny
```
**6.应用流策略**

```
interface vlan xxx
    ip address 192.168.2.1 255.255.255.0
    traffic-policy all_deny 
```
### 0x04 ACL匹配机制

![image](http://image.jiantuku.com/18-12-11/21372384.jpg?e=1544617210&token=el7kgPgYzpJoB23jrChWJ2gV3HpRl0VCzFn8rKKv:hPTcAi3UNWI-5lg3aR0hVU5_Pic=)




ACL在匹配报文时遵循“一旦命中即停止匹配”的原则


从整个ACL匹配流程可以看出，报文与ACL规则匹配后，会产生两种匹配结果：“匹配”和“不匹配”。

 - 匹配（命中规则）：指存在ACL，且在ACL中查找到了符合匹配条件的规则。不论匹配的动作是“permit”还是“deny”，都称为“匹配”，而不是只是匹配上permit规则才算“匹配”。

 - 不匹配（未命中规则）：指不存在ACL，或ACL中无规则，再或者在ACL中遍历了所有规则都没有找到符合匹配条件的规则。切记以上三种情况，都叫做“不匹配”。


无论报文匹配ACL的结果是“不匹配”、“允许”还是“拒绝”，该报文最终是被允许通过还是拒绝通过，实际是由应用ACL的各个业务模块来决定的。不同的业务模块，对命中和未命中规则报文的处理方式也各不相同。例如，在Telnet模块中应用ACL，只要报文命中了permit规则，就允许通过；而在流策略中应用ACL，如果报文命中了permit规则，但流行为动作配置的是deny，该报文会被拒绝通过。

#### 0x05 ACL匹配顺序

只要报文未命中规则且仍剩余规则，系统会一直从剩余规则中选择下一条与报文进行匹配。permit与deny规则相互矛盾的时候。就会按照一定顺序进行匹配。如果先执行permit就会被允许。如果限制性deny就会被先阻止。

**acl定义的匹配顺序**

配置顺序（config）和自动排序（auto）

**配置顺序**

即系统按照ACL规则编号从小到大的顺序进行报文匹配，规则编号越小越容易被匹配。后插入的规则，如果你指定的规则编号更小，那么这条规则可能会被先匹配上。

**自动排序**

自动排序，是指系统使用“深度优先”的原则，将规则按照精确度从高到底进行排序，系统按照精确度从高到低的顺序进行报文匹配。规则中定义的匹配项限制越严格，规则的精确度就越高，即优先级越高，那么该规则的编号就越小，系统越先匹配。

例子

```
acl number 3501
rule 5 permit ip source 10.52.1.0 0.0.0.255 destination 10.52.0.0 0.0.255.255
acl number 3001                           
rule 0 permit ip source 10.52.1.0 0.0.0.255 destination 10.52.1.0 0.0.0.255
rule 5 permit ip source 10.52.1.0 0.0.0.255 destination 10.50.0.0 0.0.0.255
rule 10 permit ip source 10.52.1.0 0.0.0.255 destination 192.168.12.0 0.0.0.255
quit
//流分类
traffic classifier 3001 operator and
if-match acl 3001
quit
traffic classifier 3501 operator and
if-match acl 3501
quit
//流行为
traffic behavior 3001
permit
quit
traffic behavior 3501
deny
quit
//流策略
traffic policy p101
classifier 3001 behavior 3001
classifier 3501 behavior 3501
quit
//流应用
vlan 501
traffic-policy p101 inbound
quit

```

在对acl了解不清楚的时候。就只是依照这葫芦画瓢的配值，配通之后不通也不知到为什么。

看到上面的acl的流程图之后，才明白rule后面的数字的含义。acl number后面的数字也是排序作用。将先执行的语句的数字应用小的。通过查看华为的文档知识和原来的串起来了。使得基础知识更加的牢固。


