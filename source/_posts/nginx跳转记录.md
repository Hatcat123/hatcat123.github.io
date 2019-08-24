
nginx 重定向 端口丢失解决方案

场景：在架设nginx反向代理，后端python应用启动在7000端口。python程序中有个302跳转操作。前端ngxin使用8800作为映射端口。

访问xx.xx.xx.xx:8800 出现302重定向 紧接着400 地址为xx.xx.xx.xx丢失了8800端口（就是转向默认的80端口）

如果手动加上8800端口就可以正常访问

附配置
server {

    listen       8800;
    server_name  localhost xxxxx;

    #charset koi8-r;

    #access_log  logs/host.access.log  main;

    location / {
        proxy_pass xxxxx;
        proxy_set_header Host $host:8800;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header REMOTE-HOST $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}



其中`        proxy_set_header Host $host:8800;`，存在较大的争议。有人说去掉8800，有人说加上8800。我刚开始没加，不能正常跳转，后来加上也不能正常跳转，然后去掉，还是不能，然后加上……可以了。神奇的操纵？