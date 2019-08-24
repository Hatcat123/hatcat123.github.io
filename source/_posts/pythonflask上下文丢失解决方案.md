RuntimeError: Working outside of application context.


这个问题的原因是在没有激活程序上下文之前进行了一些程序上下文或请求上下文的操作 
解决办法很简单就是推送程序上下文，在获得程序上下文后再执行相应的操作 
方法 1

```
rom myapp import app#myapp是我的程序文件，里面初始了Flask对象app
from flask import current_app
with app.app_context():
    print current_app.name
```
方法二
```
from myapp import app
from flask import current_app
app_ctx = app.app_context()
app_ctx.push()

print current_app.name

app_ctx.pop()
```