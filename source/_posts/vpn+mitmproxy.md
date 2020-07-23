
## 问题

在使用微信公众号自动化采集的过程中，常常存在下面的情况：1. 采集的数据不够及时。2. 采集过程可能会因为网络问题中断，如果不在电脑身边，无法及时恢复。

## 解决思路

（本片文章是对上述问题解决的一个研究过程，可能最终无结果。所以写到哪，想到哪些方法都记录下来)
目前的解决思路是从手机端出发。原来的方式是将微信的流量通过本地搭建的mitmproxy代理获取数据，我们为了让数据可以随时随地被采集，想要从手机端采集入手。
首先手机上不能再搭建mitmproxy代理服务，所以我选择使用云服务器搭建一个公开代理，让手机的流量走到公开的代理上，在使用手机阅读公众号的时候就能不受限制的将数据采集到数据库中。


### mitmproxy搭建

在ubuntu18中搭建mitmproxy的方式多种，安装的方式也比较多。之后为了要配合py脚本。我们选择使用pip的形式安装，在虚拟环境中连同py所需要的库都一同安装上。

为了方便查看数据，我们先用mitmweb运行在web端查看到数据，`--set block_global=false`的作用是允许互联网的设备访问代理，这种弊端是会有许多扫描器来扫你开放的端口，造成不必要的连接与访问流量。
```
mitmweb --web-iface 0.0.0.0 --web-port 8003 --set block_global=false --ssl-insecure --mode transparent
```
验证代理正确性，首先在浏览器上配置一个代理。服务器ip+代理端口。没有问题。
然后再手机上使用wifi的时候验证代理。在wifi配置中加上代理地址，也没有问题。

想用4g的时候也能用代理，毕竟这才是解决问题的最有效的办法。

打开手机4g，没有找到任何可以直接设置代理的地方，然后求救群友，群友扔出vpn的解决方式。

手机自带vpn设置配置，需要我在服务器上搭建一个vpn，然后手机访问网站，流量走到vpn上，在将vpn上的流量转发到mitmproxy中，实现mitmproxy也能代理到4g访问的流量。


在mitmproxy的论坛中找到了一个和自己想法一样老哥的描述`https://discourse.mitmproxy.org/t/using-mitmproxy-for-all-http-https-traffic-coming-from-an-ipsec-vpn-server/792`

这是他想要表达的想法
```

       +----------------------+
       |                      |
       |       iPhone         |
       |                      |
       +-----------+----------+
                   |
                   |
+--------------------------------------+
|      +-----------v----------+        |
|      |                      |        |
|      |        VPN server    +-----+  |
|      |                      |     |  |
|      +----------------------+     |  |
|                 |HTTP/HTTPS       |  | My server
|                 |                 |  |
|      +----------v-----------+     |  |
|      |                      |     |  |
|      |      mitmproxy       |     |  |
|      |                      |     |  |
|      +----------------------+     |  |
+--------------------------------------+
                  |                 |
                  |                 |non-HTTP/HTTPS traffic
                  |                 |
        +---------v-----------+     |
        |                     |     |
        |      Internet       +^----+
        |                     |
        +---------------------+
```

然后有老哥给出建议`https://github.com/abcnews/data-life/tree/master/server`

中间会用到iptables端口转化

```
https://gist.github.com/Tyilo/03889ddc651fcf96e1208b65bfc7aa7f#file-docker-compose-yml-L21
```

### vps搭建

选择docker一键搭建vps
```
docker pull hwdsl2/ipsec-vpn-server
```
到仓库中`https://github.com/hwdsl2/docker-ipsec-vpn-server/blob/master/README-zh.md`找到env文件，运行
```
docker run \
    --name ipsec-vpn-server \
    --env-file ./vpn.env \
    --restart=always \
    -p 500:500/udp \
    -p 4500:4500/udp \
    --privileged \
    hwdsl2/ipsec-vpn-server
```

服务器必须要开启500与4500的udp端口。

运行成功后能看到这些信息，所有的配置信息都在env文件中

```
Connect to your new VPN with these details:

Server IP: 你的VPN服务器IP
IPsec PSK: 你的IPsec预共享密钥
Username: 你的VPN用户名
Password: 你的VPN密码
```
如：
```
# Define your own values for these variables
# - DO NOT put "" or '' around values, or add space around =
# - DO NOT use these special characters within values: \ " '
VPN_IPSEC_PSK=foobar
VPN_USER=foo
VPN_PASSWORD=bar
```
尝试使用苹果手机连接vpn，在设置中找到vpn，添加ip、`L2TP `，账号、密码等参数。连接成功。

windows连接vpn需要更改一些配置，详情查看原github文档介绍。



### iptables

此时流量只会走到vpn中，如何让流量转发到mitm代理？我们选择使用iptable，将端口转发到另一个端口，下面是具体步骤。

**他说他的有效`https://github.com/abcnews/data-life/tree/master/server`**

添加策略
```
iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 8080
iptables -t nat -A PREROUTING -p tcp --dport 443 -j REDIRECT --to-port 8080
ip6tables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 8080
ip6tables -t nat -A PREROUTING -p tcp --dport 443 -j REDIRECT --to-port 8080
iptables -A INPUT -p tcp --dport 8080 -j ACCEPT
ip6tables -A INPUT -p tcp --dport 8080 -j ACCEPT
```
意思是将连接vpn中的http与https数据转发到8080端口。只要我在8080开启代理流量就会进入代理中。

删除策略
```
iptables -t nat -D PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 8080
iptables -t nat -D PREROUTING -p tcp --dport 443 -j REDIRECT --to-port 8080
ip6tables -t nat -D PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 8080
ip6tables -t nat -D PREROUTING -p tcp --dport 443 -j REDIRECT --to-port 8080
iptables -D INPUT -p tcp --dport 8080 -j ACCEPT
ip6tables -D INPUT -p tcp --dport 8080 -j ACCEPT
```

备份测试(验证后无效)
```
iptables -t nat -A PREROUTING -i eth+ -p tcp --destination-port 80 -j DNAT --to-destination 47.100.199.103:8080 
iptables -t nat -A PREROUTING -i eth+ -p tcp --destination-port 443 -j DNAT --to-destination 47.100.199.103:8080 
```

```
iptables -t nat -D PREROUTING -i eth+ -p tcp --destination-port 80 -j DNAT --to-destination 47.47.100.199.103:8080 
iptables -t nat -D PREROUTING -i eth+ -p tcp --destination-port 443 -j DNAT --to-destination 47.100.199.103:8080 
```

iptables 基础

```
iptables  -nL -t nat
```


### 报错


```
HTTP protocol error in client request: Invalid HTTP request form (expected: authority or absolute, got: relative)
```
目前无法解决了，是无效的http请求


### 成功案例

`https://github.com/abcnews/data-life`


查看文件与我们搭建的有何不同。

下载源码，进入到server目录，构建并运行镜像容器

```docker
docker-compose build
docker-compose up 
```
运行结果
```
Digest: sha256:4b594f1d297314e2ff20082c257db971bda36a13d2df6e863573a29f12cf595b
Status: Downloaded newer image for hwdsl2/ipsec-vpn-server:latest
Creating ipsec-vpn-server ... 
Creating mitmproxy ... 
Creating ipsec-vpn-server
Creating mitmproxy ... done
Attaching to ipsec-vpn-server, mitmproxy
ipsec-vpn-server | 
ipsec-vpn-server | VPN credentials not set by user. Generating random PSK and password...
ipsec-vpn-server | 
ipsec-vpn-server | Trying to auto discover IP of this server...
ipsec-vpn-server | 
ipsec-vpn-server | ================================================
ipsec-vpn-server | 
ipsec-vpn-server | IPsec VPN server is now ready for use!
ipsec-vpn-server | 
ipsec-vpn-server | Connect to your new VPN with these details:
ipsec-vpn-server | 
ipsec-vpn-server | Server IP: 49.233.171.51
ipsec-vpn-server | IPsec PSK: DypyiWAnwRztowJKCA2c
ipsec-vpn-server | Username: vpnuser
ipsec-vpn-server | Password: E5ZGJvcX7xWpuVnp

ipsec-vpn-server | Redirecting to: /etc/init.d/ipsec start
ipsec-vpn-server | Starting pluto IKE daemon for IPsec: Initializing NSS database
ipsec-vpn-server | 
mitmproxy    | Loading script /home/mitmproxy/.mitmproxy/jsondump.py
mitmproxy    | Writing all data frames to /proxydata/dump.json
mitmproxy    | Loading script /home/mitmproxy/.mitmproxy/tls_passthrough.py
mitmproxy    | Writing all data frames to /proxydata/dump.json
mitmproxy    | Proxy server listening at http://*:8080

```

连接上vpn，日志中显示连接记录并且访问网页mitm也有记录。

分析下这个server中每个文件的作用

```docker-compose.yml
version: "3"

services:
  vpn:
    image: hwdsl2/ipsec-vpn-server
    environment:
      - VPN_IPSEC_PSK=${VPN_IPSEC_PSK}
      - VPN_USER=${VPN_USER}
      - VPN_PASSWORD=${VPN_PASSWORD}
    ports:
      - "500:500/udp"
      - "4500:4500/udp"
    hostname: ipsec-vpn-server
    privileged: true
    container_name: ipsec-vpn-server
    volumes:
      - /lib/modules:/lib/modules:ro
    command: /opt/src/run.sh
    restart: unless-stopped
    networks:
      static-network:
        ipv4_address: 172.20.128.3
#    network_mode: "host"
  mitmproxy:
    build: .
    ports:
      - "8080:8080/tcp"
    volumes: 
      - proxydata:/proxydata
    hostname: mitmproxy
    container_name: mitmproxy
    # command: mitmdump --mode transparent --showhost
    command: mitmdump --ssl-insecure # --set proxyauth=${PROXY_USER}:${PROXY_PASS}
    restart: unless-stopped
    networks:
      static-network:
        ipv4_address: 172.20.128.2
#    network_mode: "host"
volumes:
  proxydata:
networks:
  static-network:
    ipam:
      config:
        - subnet: 172.20.0.0/16
```

可以看出：用了一个ipsec的镜像，然后自己构造了一个镜像mitm

看下dockerfile中的内容

```dockerfile
FROM mitmproxy/mitmproxy:4.0.4

COPY --chown=mitmproxy .mitmproxy/* /home/mitmproxy/.mitmproxy/

COPY requirements.txt /tmp/requirements.txt
RUN pip3 install -r /tmp/requirements.txt && rm /tmp/requirements.txt

RUN mkdir /proxydata && chown mitmproxy:mitmproxy /proxydata
```

这个dockerfile是原始镜像`mitmproxy/mitmproxy:4.0.4`的构造。加了一个`.mitmproxy/*`目录与python库。

另外还有一个有价值的文件，是vpn的配置信息，应该是通过docker-compose配置的。
```.envrc-template
export VPN_IPSEC_PSK=<YourSharedKey>
export VPN_USER=<YourUserName>
export VPN_PASSWORD=<YourPassword>
export VPN_IP=<IPAddrewssOfDockerHost>
```
这个我们可以设置.env后缀的文件
```.env

```
2020年6月26日中午
经过测试vpn是连接上了，就是不知道mitmdump有没有获取到流量，进入到mitm的docker容器中，mitm容器没有bash，通过sh执行命令。查看文件发现是空的。似乎没有捕获到流量，然后我想者将mitmdump换成mitmweb，找到server文件夹下的docker-compose.yml文件更改成为mitmweb，并且加上web-iface等信息。并映射出docker的8003端口。
```
mitmweb --web-iface 0.0.0.0 --web-port 8003 --ssl-insecure
```
*很神奇的地方，mitmweb对用的host有的是web-host有的是web-iface应该与版本有关系*

测试过程终还是失败？github原作者说他这是个成功的例子，可流量还是没有经过mitm。再一次爆出下面的问题。
```
HTTP protocol error in client request: Invalid HTTP request form (expected: authority or absolute, got: relative)
```

这句话到底是什么意思，然后晚上我又去google一波，在仔细看了一遍所有的网页，找到之前漏掉的内容。

在mitmproxy的github issue中发现也有人提出的问题与我的类似，作者回答的内容都是指向了**mitmproxy的官方文档中代理这一章**。

在文档中介绍了mitmproxy这几种代理模式：
1. Regular (the default)常规代理
2. Transparent 透明代理
3. Reverse Proxy 反向代理
4. Upstream Proxy 上游代理
5. SOCKS Proxy socks代理

![1593185108(1)](https://raw.githubusercontent.com/Hatcat123/GraphicBed/master/Img2/1593185108(1).jpg)

![](https://docs.mitmproxy.org/stable/schematics/proxy-modes-flowchart.png)

**1. 常规代理**

Mitmproxy的常规模式是最简单，最容易设置的。

在一个网段路由下启动mitm，默认会在8080端口开启代理服务，客户端配置代理模式为mitm，https需安装证书。

![常规代理](https://docs.mitmproxy.org/stable/schematics/proxy-modes-regular.png)

**2. 透明代理**

使用透明代理时，流量将重定向到网络层的代理中，而无需任何客户端配置。这使得透明代理非常适合那些您无法更改客户端行为的情况-代理不兼容的移动应用程序
重定向机制，可以透明地将发往Internet上服务器的TCP连接重新路由到侦听代理服务器。这通常采用与代理服务器位于同一主机上的防火墙的形式 -Linux 上的iptables或OSX 上的 pf。

**3. 反向代理**

使用反向代理模式，可以使用mitmproxy充当普通的HTTP服务器。
```
mitmproxy --mode reverse:https://example.com
```
![](https://docs.mitmproxy.org/stable/schematics/proxy-modes-reverse.png)

**4. 上游代理**

使用上游代理模式，可以在其他代理服务器前加上mitm代理，官网说理论上可以无限加。

![](https://docs.mitmproxy.org/stable/schematics/proxy-modes-upstream.png)

**5. SOCKS5代理**

可以充当socks5代理

### 最终解决方案

在了解了mitm的几种代理与测试后，发现问题就出现在透明代理上，以上出错的脚本或者配置都是只是用了普通代理，导致流量转发到mitm中后不知道下一跳指向哪里。

现在使用透明代理配置一次

mitmproxy与iptables重定向机制集成在一起以实现透明模式

1. 开启端口转发
```shell
sysctl -w net.ipv4.ip_forward=1
sysctl -w net.ipv6.conf.all.forwarding=1
```
2. 禁用ICMP重定向
```
sysctl -w net.ipv4.conf.all.send_redirects=0
```
3. iptables重定向到8080端口
```
iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 80 -j REDIRECT --to-port 8080
iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 443 -j REDIRECT --to-port 8080
ip6tables -t nat -A PREROUTING -i eth0 -p tcp --dport 80 -j REDIRECT --to-port 8080
ip6tables -t nat -A PREROUTING -i eth0 -p tcp --dport 443 -j REDIRECT --to-port 8080
```
4. mitm开启透明代理模式
```
mitmproxy --mode transparent --showhost
```
5. 启动vpn
```

```
再次测试，连接vpn，重新安装证书，信任证书，vpn流量经过mitm。

### 配置采集微信公众号

下载源码
```
git clone https://github.com/Hatcat123/WechatSpiderDocker.git
```
开启透明代理

```
cd server/spider
mitmweb --web-iface 0.0.0.0 --web-port 8004 -p 8080 -s capture_packet.py  --set block_global=false --ssl-insecure --mode transparent 
```
或者

```
mitmdump -s capture_packet.py  --set block_global=false --ssl-insecure --mode tranarent 
```

开启vpn

在`.env`同级目录
```
docker run \
    --name ipsec-vpn-server \
    --env-file ./.env \
    --restart=always \
    -p 500:500/udp \
    -p 4500:4500/udp \
    --privileged \
    hwdsl2/ipsec-vpn-server
```
重定向参照上面配置
```
见上
```

docker安装

```
略
```

