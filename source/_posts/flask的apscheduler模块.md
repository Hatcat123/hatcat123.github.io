

### 介绍flask_apscheduler

Flask-Apscheduler 是集成了 Apscheduler 的扩展, 给 Flask 程序提供了 定时任务 的支持, 使用此扩展能够很方便的添加定时任务, 删除任务。如果将作业存储在数据库中，它们也将在调度程序重新启动后继续运行并保持其状态。重新启动调度程序后，它将运行它应该在脱机时运行的所有作业。


除此之外，APScheduler还可以用作特定于平台的调度程序（如cron守护程序或Windows任务调度程序）的跨平台，特定于应用程序的替代程序。但请注意，APScheduler本身不是守护程序或服务，也不附带任何命令行工具。它主要用于在现有应用程序中运行。也就是说，APScheduler确实为您提供了一些构建块来构建调度程序服务或运行专用的调度程序进程。

APScheduler是一个python的第三方库，用来提供python的后台程序。包含四个组件，分别是：

- triggers： 任务触发器组件，提供任务触发方式，支持三种任务触发方式：date、interval、corn
- job stores： 任务商店组件，提供任务保存方式，支持四种任务存储方式：memory、mongdb、sqlachemy、redis
- executors： 任务调度组件，提供任务调度方式
- schedulers： 任务调度组件，提供任务工作方式，调度器主要分三种，一种独立运行的，一种是后台运行的，最后一种是配合其它程序使用


### apscheduler的几种写法

在GitHub源码中提供了几种写法：https://github.com/viniciuschiele/flask-apscheduler/tree/master/examples

封装任务到配置中
```advanced.py
from apscheduler.jobstores.sqlalchemy import SQLAlchemyJobStore
from flask import Flask
from flask_apscheduler import APScheduler


class Config(object):
    JOBS = [
        {
            'id': 'job1',
            'func': 'advanced:job1',
            'args': (1, 2),
            'trigger': 'interval',
            'seconds': 10
        }
    ]

    SCHEDULER_JOBSTORES = {
        'default': SQLAlchemyJobStore(url='sqlite://')
    }

    SCHEDULER_EXECUTORS = {
        'default': {'type': 'threadpool', 'max_workers': 20}
    }

    SCHEDULER_JOB_DEFAULTS = {
        'coalesce': False,
        'max_instances': 3
    }

    SCHEDULER_API_ENABLED = True


def job1(a, b):
    print(str(a) + ' ' + str(b))


if __name__ == '__main__':
    app = Flask(__name__)
    app.config.from_object(Config())

    scheduler = APScheduler()
    scheduler.init_app(app)
    scheduler.start()

    app.run()

```
直接写
```decorated.py
from flask import Flask
from flask_apscheduler import APScheduler


class Config(object):
    SCHEDULER_API_ENABLED = True


scheduler = APScheduler()


# interval examples
@scheduler.task('interval', id='do_job_1', seconds=30, misfire_grace_time=900)
def job1():
    print('Job 1 executed')


# cron examples
@scheduler.task('cron', id='do_job_2', minute='*')
def job2():
    print('Job 2 executed')


@scheduler.task('cron', id='do_job_3', week='*', day_of_week='sun')
def job3():
    print('Job 3 executed')


if __name__ == '__main__':
    app = Flask(__name__)
    app.config.from_object(Config())

    # it is also possible to enable the API directly
    # scheduler.api_enabled = True
    scheduler.init_app(app)
    scheduler.start()

    app.run()
```
数据库上下文
```flask_context.py

from apscheduler.jobstores.sqlalchemy import SQLAlchemyJobStore
from flask import Flask
from flask_apscheduler import APScheduler
from flask_sqlalchemy import SQLAlchemy


db = SQLAlchemy()


class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True)
    email = db.Column(db.String(120), unique=True)


def show_users():
    with db.app.app_context():
        print(User.query.all())


class Config(object):
    JOBS = [
        {
            'id': 'job1',
            'func': show_users,
            'trigger': 'interval',
            'seconds': 2
        }
    ]

    SCHEDULER_JOBSTORES = {
        'default': SQLAlchemyJobStore(url='sqlite:///flask_context.db')
    }

    SCHEDULER_API_ENABLED = True


if __name__ == '__main__':
    app = Flask(__name__)
    app.config.from_object(Config())

    db.app = app
    db.init_app(app)

    scheduler = APScheduler()
    scheduler.init_app(app)
    scheduler.start()

    app.run()
```

### 高级配置
高级的内容我们可以查看文档https://apscheduler.readthedocs.io/en/latest/genindex.html
开启api接口

获取任务
```
scheduler.get_job()
```
获取全部任务
```
scheduler.get_jobs()
```
删除任务
```
scheduler.remove_job(str(job_id))
```
```
scheduler.start() # 开始调度
scheduler.shutdown() # 停止调度
scheduler.pause_job(job_id='my_job_id') # 暂停指定JOB调度
scheduler.resume_job(job_id='my_job_id') # 恢复指定JOB调度
scheduler.pause() # 暂停调度
scheduler.resume() # 恢复调度
```

### job stores搭配数据库

**redis** 、**sql**
```
    SCHEDULER_JOBSTORES = {
        'default': SQLAlchemyJobStore(url='sqlite:///flask_context.db'
        "redis":RedisJobStore(host='172.16.120.120', port='6379'))
    }

```
不过我配置上带有密码的redis好像没有加入到数据库中。

### triggers三种模式的定时任务

举例

**date**：固定日期触发器，任务只运行一次，运行完毕自动清除；若错过指定运行时间，任务不会被创建

| 参数 | 说明 |
| :-: |:-: |
| run_date (datetime 或 str) | 作业的运行日期或时间 |
| timezone (datetime.tzinfo 或 str) | 指定时区 |
```
例如# 在 2019-4-24 00:00:01 时刻运行一次 start_system 方法
scheduler .add_job(start_system, 'date', run_date='2019-4-24 00:00:01', args=['text'])
```
**interval**：时间间隔触发器，每个一定时间间隔执行一次。

| 参数 | 说明 |
|:-: |:-: |
| weeks (int) | 间隔几周 |
| days (int) | 间隔几天 |
| hours (int) | 间隔几小时 |
| minutes (int) | 间隔几分钟 |
| seconds (int) | 间隔多少秒 |
| start_date (datetime 或 str) | 开始日期 |
| end_date (datetime 或 str) | 结束日期 |
```
# 在 2019-4-24 00:00:00 - 2019-4-24 08:00:00 之间, 每隔两小时执行一次 alarm_job 方法
scheduler .add_job(alarm_job, 'interval', hours=2, start_date='2019-4-24 00:00:00' , end_date='2019-4-24 08:00:00')
scheduler.add_job(func=print_args, args=['a', 'b', 'c'], trigger='interval', days=3, hours=17, minutes=19,seconds=7) # 表示每隔3天17时19分07秒执行一次任务
scheduler.add_job(func=print_args, args=['a', 'b', 'c'], trigger='interval', hours=17, minutes=19,seconds=7) # 表示每隔17时19分07秒执行一次任务
scheduler.add_job(func=print_args, args=['a', 'b', 'c'], trigger='interval', minutes=19, seconds=7) # 表示每隔19分07秒执行一次任务
scheduler.add_job(func=print_args, args=['a', 'b', 'c'], trigger='interval', minutes=19) # 表示每隔19分执行一次任务
scheduler.add_job(func=print_args, args=['a', 'b', 'c'], trigger='interval', seconds=1) # 表示每秒执行一次任务
scheduler.add_job(func=print_args, args=['a', 'b', 'c'], trigger='interval', seconds=0.5) # 表示每500毫秒执行一次任务

```
**cron**：cron风格的任务触发

| 参数 | 说明 |
|:-: |:-: |
|year (int 或 str)|	表示四位数的年份 （2019）|
|month（int\	str）|	月 (范围1-12)
|day（int\	str）	|日 (范围1-31）
|week（int\	str）	|周 (范围1-53)
|day_of_week (int\	str)|	表示一周中的第几天，既可以用0-6表示也可以用其英语缩写表示|
|hour (int\	str)|	表示取值范围为0-23时
|minute (int\	str)|	表示取值范围为0-59分
|second (int\	str)|	表示取值范围为0-59秒
|start_date |(datetime\	str)|	表示开始时间
|end_date (datetime\	str)|	表示结束时间
|timezone (datetime.tzinfo\	str)|	表示时区取值

>(int|str) 表示参数既可以是int类型，也可以是str类型
>(datetime | str) 表示参数既可以是datetime类型，也可以是str类型

```
# [参数取值格式]
# * any Fire on every value 如: day='*' 即,每天执行
# */a any Fire every a values, starting from the minimum 如: hour='*/2' 即,每两小时执行
# a-b any Fire on any value within the a-b range (a must be smaller than b) 如: minute='6-8,21-23' 即,6-8点和21-23点执行
# a-b/c any Fire every c values within the a-b range 如: minute='9-21/2' 即,9点-21点之间每两小时执行
# xth y day Fire on the x -th occurrence of weekday y within the month
# last x day Fire on the last occurrence of weekday x within the month
# last day Fire on the last day within the month
# x,y,z any Fire on any matching expression; can combine any number of any of the above expressions # 表达式之间用逗号进行分隔

例如：表示每5秒执行该程序一次，相当于interval 间隔调度中seconds = 5

sched.add_job(my_job, 'cron',second = '*/5')

scheduler.add_job(func=print_args, args=['a', 'b', 'c'], trigger='cron', year=2018, month=3, day=21, hour=21, minute=19,
second=8) # 表示2018年3月21日21时19分08秒执行
scheduler.add_job(func=print_args, args=['a', 'b', 'c'], trigger='cron', day_of_week='mon-fri', hour=5, minute=30,start_date='2016-06-06',end_date='2018-08-08') # 表示周一到周五的5点半执行,有效日期是2016-06-06 00:00:00 至 2018-08-08 00:00:00
scheduler.add_job(func=print_args, args=['a', 'b', 'c'], trigger='cron', second='*/10') # 表示每10秒执行该程序一次，相当于interval 间隔调度中seconds = 10
```


## 部署的问题


### 多次执行任务问题


添加文件锁、线程锁、端口锁、redis数据库锁

多进程部署，定时任务重复启动解决方法

文件锁
```
def create_app():
    app =Flask(__name__)
    # 启动定时任务
    scheduler_init(app)
    return app

def scheduler_init(app):
    """
    保证系统只启动一次定时任务
    :param app:
    :return:
    """
    if platform.system() != 'Windows':
        fcntl = __import__("fcntl")
        f = open('scheduler.lock', 'wb')
        try:
            fcntl.flock(f, fcntl.LOCK_EX | fcntl.LOCK_NB)
            scheduler.init_app(app)
            scheduler.start()
            app.logger.debug('Scheduler Started,---------------')
        except:
            pass

        def unlock():
            fcntl.flock(f, fcntl.LOCK_UN)
            f.close()

        atexit.register(unlock)
    else:
        msvcrt = __import__('msvcrt')
        f = open('scheduler.lock', 'wb')
        try:
            msvcrt.locking(f.fileno(), msvcrt.LK_NBLCK, 1)
            scheduler.init_app(app)
            scheduler.start()
            app.logger.debug('Scheduler Started,----------------')
        except:
            pass

        def _unlock_file():
            try:
                f.seek(0)
                msvcrt.locking(f.fileno(), msvcrt.LK_UNLCK, 1)
            except:
                pass

        atexit.register(_unlock_file)
```

**端口锁**
当一个端口被占用时候，就不能创建新的任务
```
    # Fix to ensure only one Gunicorn worker grabs the scheduled task
    try:
        sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        sock.bind(("127.0.0.1", 47200))
    except socket.error:
        pass
    else:
        scheduler.init_app(app)
        scheduler.start()
```

### gun的问题

**gun下改模式**

用–preload启动gunicorn，确保scheduler只在loader的时候创建一次

**服务器时区**

部署生产环境时，一上去就给我丢了一个大大的异常：“TimeZone offset does not match system offset”，大致意思是说我的运行的时区和系统时区不匹配。


读了下 flask-apscheduler 的源码发现，他会读取 SCHEDULER_TIMEZONE 这个值，作为当前运行的时区。
```
timezone = self.app.config.get('SCHEDULER_TIMEZONE')
if timezone:
    options['timezone'] = timezone
```
心想这简单，我在配置文件里把这个时区配置上应该就完事了。
```
# config.py
SCHEDULER_TIMEZONE = "Asia/Shanghai"
```
配置了时区为“Asia/Shanghai”后，发现依然报这个错，既然运行环境的时区正确了，那会不会我生产环境的时区也有问题？

生产环境是 docker 容器，进入容器用 date 查看了他的当前时间，发现时间是和系统时间一致的，再查看 cat /etc/timezone 的时区，发现这里出了问题，显示的是 Etc/UTC，解决的思路是修改 Dockerfile，配置正确的时区，在 Dockerfile 中加入此行。
```
RUN echo "Asia/Shanghai" > /etc/timezone
```
修改之后重新运行，已经没有出现上述报错了，这个坑算是到这就排完了。
