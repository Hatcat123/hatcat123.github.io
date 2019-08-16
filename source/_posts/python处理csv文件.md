---
title: python处理csv文件
tags:
 - csv
 - python

categories:
  - python
 
copyright: true
permalink: python处理csv文件
top: 0
password: 
abstract: 输入密码继续阅读.
message: 欢迎来到我的博客，输入密码继续阅读.
date: 2019-05-13 08:32:58

---
CSV(Comma-Separated Values)即逗号分隔值，可以用Excel打开查看。由于是纯文本，任何编辑器也都可打开。以至于python保存数据量小的文件大多使用csv。
<!--more-->

### 0x01 CSV文件特点

CSV文件中：

 - 值没有类型，所有值都是字符串
 - 不能指定字体颜色等样式
 - 不能指定单元格的宽高，不能合并单元格
 - 没有多个工作表
 - 不能嵌入图像图表

在CSV文件中，以,作为分隔符，分隔两个单元格。像这样a,,c表示单元格a和单元格c之间有个空白的单元格。依此类推。

不是每个逗号都表示单元格之间的分界。所以即使CSV是纯文本文件，也坚持使用专门的模块进行处理。Python内置了csv模块。

### 0x02 写入数据到csv

**writer单行写入**

```
import csv

# 使用数字和字符串的数字都可以
datas = [['name', 'age'],
         ['Bob', 14],
         ['Tom', 23],
        ['Jerry', '18']]

with open('example.csv', 'w', newline='') as f:
    writer = csv.writer(f)
    for row in datas:
        writer.writerow(row)
        

```

**writer多行写入**

```
    # 还可以写入多行
    writer.writerows(datas)
```

**DictReader单行写入**

```
import csv

headers = ['name', 'age']

datas = [{'name':'Bob', 'age':23},
        {'name':'Jerry', 'age':44},
        {'name':'Tom', 'age':15}
        ]

with open('example.csv', 'w', newline='') as f:
    # 标头在这里传入，作为第一行数据
    writer = csv.DictWriter(f, headers)
    writer.writeheader()
    for row in datas:
        writer.writerow(row)
        

```

**DictReader多行写入**

```
    # 还可以写入多行
    writer.writerows(datas)
```

### 0x03 读取csv数据

**reader读取数据**

```
import csv

filename = 'test.csv'
with open(filename) as f:
    reader = csv.reader(f)
    for row in reader:
        # 行号从1开始
        print(reader.line_num, row)
```

**DictReader读取数据**

```
import csv

filename = 'test.csv'
with open(filename) as f:
    reader = csv.DictReader(f)
    for row in reader:
        # Max TemperatureF是表第一行的某个数据，作为key
        max_temp = row['Max TemperatureF']
        print(max_temp)
```

