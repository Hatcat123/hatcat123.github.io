---
title: python切换宽带账号
tags:
  - python
  

date: 2019-5-2 13:14:00


categories:
 - python

---
在一次实战中客户要求切换vps的宽带账号，使用windows命令操作
<!--more-->

### windows下rasdial命令介绍

```
C:\>rasdial /?
用法:
        rasdial entryname [username [password|*]] [/DOMAI
                [/PHONE:phonenumber] [/CALLBACK:callbackn
                [/PHONEBOOK:phonebookfile] [/PREFIXSUFFIX

        rasdial [entryname] /DISCONNECT

        rasdial

        有关联机隐私信息，请参阅
        "http://go.microsoft.com/fwlink/?LinkId=104288"

```

**连接宽带**

    rasdial 宽带连接 057128280802 34795

**断开连接**

    rasdial 宽带连接 /disconnect

### 集成到脚本中

```
    def connect(self):
        name = "宽带连接"
        username ='057128280802'#  填写你的宽带名字
        password = "34795"# 填写你的宽带密码
        cmd_str = "rasdial %s %s %s" % (name, username, password)
        res = os.system(cmd_str)
        if res == 0:
            print("connect successful")
        else:
            print(res)
        time.sleep(5)

    def disconnect(self):
        name = "宽带连接"
        cmdstr = "rasdial %s /disconnect" % name
        os.system(cmdstr)
        time.sleep(5)
    def check_kuandai(self):
        self.disconnect()

```

示例：

![](https://raw.githubusercontent.com/Hatcat123/GraphicBed/master/Img/20190503210347.gif)
