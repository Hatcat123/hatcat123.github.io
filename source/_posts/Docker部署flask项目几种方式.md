


Docker flask部署项目的几种方式

无非是运行的flask项目的方式
一、 python直接运行flask项目
二、 gun部署项目
其他、 uwsgi部署方式



## 一、Docker部署flask基础篇

创建`Dockerfile`文件，使用封装好的python镜像，复制内容到镜像内，python直接运行。没有难度。如：
```
FROM python:3.6

COPY ./src /app/src

WORKDIR /app/src

RUN pip install -r requirements.txt -i https://pypi.douban.com/simple

CMD ["python", "app.py"]
```
生成镜像

```
 docker build -t 'wechattogether' .
```
运行容器

```
 docker run  -name doonsec_wechat -p 9000:8000 wechattogether -d
```

## 二、 Dockers部署flask项目gun

增加了gun就要在pip中增加gun与gevent的库

```
FROM python:3.6

COPY ./src /app/src

WORKDIR /app/src

RUN pip install -r requirements.txt -i https://pypi.douban.com/simple
RUN pip install gunicorn==19.5.0 gevent==1.4.0 -i https://pypi.douban.com/simple

CMD ["gunicorn", "-k", "gevent", "-b", "0.0.0.0:8000", "app:app"]
```
生成镜像的方式与上一致




## 附docker-compose
编写docker-compose.yml
```
version: '3'
services:
 web:
   build: .
   ports:
    - "8000:8000"
```
运行
```
docker-compose up -d
```