---
title: 分析if__name__==__main__
date: 2017-07-14 17:47:37

tags:


- python
- main

categories: python

thumbnail: http://ww4.sinaimg.cn/large/006iLK2Bly1fiaxrn1y23j30m809u0st.jpg

---
# 分析if __ name __ =='__ main __'

----------

刚开始学习python的时候看到其中的if __ name __==' __main __'这个代码，（当时模糊理解为类似于c的主函数）不知道它真正是干嘛的，什么作用？
#### **“Make a script both importable and executable”**      
<!--more-->
最经典的就是这就话意思是 “**让你写的脚本模块不仅可以导入到别人的模块中使用，也能自己去执行**”。不明白也没关系，下面就来举几个最简单的例子

首先，创建一个a.py的文件

    print('this is my      a         .py************************')
    print('a.py__name__: %s' %__name__)
这里是先确定在什么都没有的情况下，输出__ name __ 是什么？输出后发现是缺省的__ main __ 

    this is my      a         .py************************
    a.py__name__: __main__
这就是最基础的:'这句话为什么会执行它下面的内容',因为'name==main'啊！验证一下：

    print('this is my      a         .py************************')
    print('a.py__name__: %s' %__name__)
    if __name__=='__main__':
        print('this is a.py--------------')
同时if条件执行，输出'this is a.py---------'，文件a.py的name为main

    this is my      a         .py************************
    a.py__name__: __main__
    this is a.py--------------

这时创建另一个b.py文件（先类似于a.py），来弄清两者之间的关系

    print('this is my b.py************************')
    print('b.py__name__:%s' %__name__)
    if __name__=='__main__':
    	print('this is b.py------------')
毫无疑问他的输出完全和a.py类似，此时在它的头加上import a，再分析其中的变化

    import a
    print('this is my     b.    py************************')
    print('b.py__name__: %s' %__name__)
    if __name__=='__main__':
    	print('this is b.py------------')

输出后：

    this is my      a         .py************************
    a.py__name__: a
    this is my     b.    py************************
    b.py__name__: __main__
    this is b.py------------
调用a.py后会生成一个名为“__ pycache __”的文件夹，里面pyc文件“a.cpython-35.pyc”，输出同时也发生了变化。

只打印了a.py文件if前面的语句，说明if语句并没有执行，也就是a.py中的name！==main，找下原因，原来b.py的输出-->a.py__ name __ : a，在之前a.py中输出的a.py__ name __:main 。就可以认定为此时被通过import导入文佳模块 __ name __ 的值就是我们这个文件名字而不是 __ main __；所以它不能执行a.py的if语句，就只能输出a中if前面的语句，b.py的name此时是main，执行了if语句，就打印了“this is b.py”如此类推我们我们再去新建一个c.py，去把带有a模块的模块b.py加入到c.py中应该也是同样的原理。

验证c.py

    import b
    print('this is my      c.       py************************')
    print('c.py__name__:  %s' %__name__)
    if __name__=='__main__':
	    print('this is c.py------------')
输出c.py

    this is my      a         .py************************
    a.py__name__: a
    this is my     b.    py************************
    b.py__name__: b
    this is my      c.       py************************
    c.py__name__:  __main__
    this is c.py------------

--------------

异想天开的想法：如果我把a.py文件改名为 __ main __ .py会不会就能在b.py调用模块__ main __.py（改名a.py）的时候就直接能能是if成立呢？（思路：b.py中返回的name就是被调用模块文件的名字）


 **往往现实是这样的提示“invalid syntax”哈哈！mdzz想想人家怎么会给你留这样的bug呢？失败告终，所以创建名字的时候老老实实的，避开"__ main__.py”的奇葩名**


----------

## 总结：

**每个python的模块都包含一个默认的变量__ main__，模块中的name的值取决于如何运用模块。1、当模块a（上面自己创建的py文件）被自身运行的时候，理解为main就是main，if的返回结果为true,执行其下面的条件。2、如果a被调用import到其他的模块b中去，此时a的mian的值就变成了文件名，a模块中if条件返回flase，便不能执行被调用函数a中的if条件下的函数，只执行模块a中if前面的语句和调用它的函数b中的语句，理解为：a模块变残缺，b模块正常运行标准程序**

**此函数理解为：可以保留一个模块脚本能独立的运行，又能使得该模块脚本的函数能够成为其他模块脚本的拓展，如果给一个模块做测试，不想让它运行的函数将不会被执行**
