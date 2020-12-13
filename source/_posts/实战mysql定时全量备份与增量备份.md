---
title: 实战mysql定时全量备份与增量备份
tags:
  - mysql
categories:
  - 运维
copyright: true
permalink: 实战mysql定时全量备份与增量备份
top: 0
password: woaini2.
abstract: 输入密码继续阅读.
message: 欢迎来到我的博客，输入密码继续阅读.
date: 2019-10-14 19:29:52
---

产品上线后，数据库中的数据是非常重要的，容不得有半点闪失。线上的数据库需要每天做一次增量备份。每个月做一个全备份。保存三个月内的数据与日志
<!--more-->

## 全备份

1、Mysql语句备份一个数据库:
备份的语句mysqldump的基本语法: 
```
mysqldump -u username -p dbname table1 table2...->BackupName.sql;
```
参数解析:
```
dbname：要备份数据库的名称；

table1和table2参数表示的是需要备份的数据库表的名称，假如为空则表示需要备份整个数据库；

BackupName.sql表示的是将数据库备份到指定的这个以后缀米国.sql的文件中，这个文件的前面可以执行一个详细的绝对路径下；
```

2、Mysql备份多个数据库：
数据库备份其实都是差不多的语句，他们最基本的差异就是添加一些命令用于区别数据库备份的深度和广度；

备份语法:
```

mysqldump -u username -p --databases dbname2 dbname2 > Backup.sql
```

3、备份所有的数据库操作:
mysqldump命令备份所有数据库的语法如下：
```
mysqldump -u username -p --all-databases > BackupName.sql
```


4、备份数据时候进行压缩

```
mysqldump < mysqldump options> | gzip > outputfile.sql.gz
```
例子：
```
mysqldump -uroot -p your_database_name | gzip > outputfile.sql.gz
```
5、 高级命令
加上下面的命令
```
--single-transaction -
--lock-tables=false 导出数据的时候不锁表
```
```
mysqldump --single-transaction -uusername -ppassword dbname > dbname.sql  
```

### 备份脚本
```
# vi zabbixdbbak.sh 
FILENAME=`date +%Y%m%d%H`
cd /data/dbbak
mysqldump -h192.168.0.3 -P3306 -uroot -ppwd123 --single-transaction --default-character-set=utf8 -R -E zabbix --log-error='zabbix'$FILENAME.log |gzip > 'zabbix'$FILENAME.sql.gz
find /data/dbbak/zabbix*.gz -mtime +7 -exec rm -f {} \;
find /data/dbbak/zabbix*.log -mtime +7 -exec rm -f {} \;

# crontab -e
30 0 * * * sh /data/dbbak/zabbixdbbak.sh 1>/data/dbbak/zabbixdbbakcron.log 2>>/data/dbbak/zabbixdbbakcron.bad

 
```
```
#!/bin/bash
backdir=/home/shaowei/dbbak
dbuser=‘dbusername‘
dbpass=‘dbpasswd‘
dblist=$(ls -p /var/lib/mysql | grep / | tr -d /)
today=$(date +%Y%m%d)
mkdir $backdir
mkdir $backdir/$today
for dbname in $dblist
do mysqldump -u$dbuser -p$dbpass $dbname | gzip -v > $backdir/$today/$dbname-$today.sql.gz
echo $dbname ‘OK‘
done
```

## 增量备份


## mysql转化sqlite

使用神器 navicat种的数据转化功能，使用hedisql的过程发现有些字段无法转换。

navicat激活过程参考龙哥推荐：https://www.cnblogs.com/Kathrine/p/12844846.html

如微信数据备份的流程：

先从服务器压缩备份一份数据，导出需要4个表的数据
```
 mysqldump -h127.0.0.1 -uroot -pboqAl7nAVo  --force --hex-blob --opt wechat wechat_article_list wechat_account t_wechat_tag t_announcement t_wechat_account |gzip >wechat.sql.gz

```
然后下载数据到本地电脑

导入数据到本地mysql中。

使用navicat进行数据转化。保存sqlite上传云盘。



第二种方法。bavucat直接连接数据库，进行工具-数据传输-数据同步。在线同步数据，稍微有点慢但是无碍，傻子操作。大概10分钟就能备份完。数据同步估计是增量数据的备份。还没有验证。

导出的过程还能保存成配置文件。还能定时自动运行。敲击棒棒。