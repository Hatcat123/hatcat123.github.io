docker 搭建dvwa

```
docker pull citizenstig/dvwa

docker run -d -p 8000:80 -p 33060:3306 -e MYSQL_PASS="root" citizenstig/dvwa
```

默认用户名与密码

admin/password

![20200620232516](https://raw.githubusercontent.com/Hatcat123/GraphicBed/master/Img2/20200620232516.png)