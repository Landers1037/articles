---
title: Python的时间模块和数学取整
name: py7
date: 2019-04-11
id: 0
tags: [学习笔记]
categories: []
abstract: ""
---


## datatime模块

使用datatime模块获取系统的时间

```python
import datatime
time = datatime.datatime.now()
```

将获取的时间格式化为字符串
<!--more-->


## datatime模块

使用datatime模块获取系统的时间

```python
import datatime
time = datatime.datatime.now()
```

将获取的时间格式化为字符串<!--more-->

```python
time.strftime('%Y-%m-%d-%H-%M-%S')
#格式化为年-月-日 -小时 -分钟- 秒
```

字符串转化为时间

```python
 time = datetime.datetime.strptime(string,'%Y-%m-%d %H:%M:%S')
```

格式

```python
%a 星期几的简写
%A 星期几的全称
%b 月分的简写
%B 月份的全称
%c 标准的日期的时间串
%C 年份的后两位数字
%d 十进制表示的每月的第几天
%D 月/天/年
%e 在两字符域中，十进制表示的每月的第几天
%F 年-月-日
%g 年份的后两位数字，使用基于周的年
%G 年分，使用基于周的年
%h 简写的月份名
%H 24小时制的小时
%I 12小时制的小时
%j 十进制表示的每年的第几天
%m 十进制表示的月份
%M 十时制表示的分钟数
%n 新行符
%p 本地的AM或PM的等价显示
%r 12小时的时间
%R 显示小时和分钟：hh:mm
%S 十进制的秒数
%t 水平制表符
%T 显示时分秒：hh:mm:ss
%u 每周的第几天，星期一为第一天 （值从0到6，星期一为0）
%U 第年的第几周，把星期日做为第一天（值从0到53）
%V 每年的第几周，使用基于周的年
%w 十进制表示的星期几（值从0到6，星期天为0）
%W 每年的第几周，把星期一做为第一天（值从0到53）
%x 标准的日期串
%X 标准的时间串
%y 不带世纪的十进制年份（值从0到99）
%Y 带世纪部分的十制年份
%z，%Z 时区名称，如果不能得到时区名称则返回空字符。
%% 百分号
```



## math模块

python的数值计算取整import math库

```python
import math
#取整
n = 3.25
print(int(n))
>>> 3 
#向上取整
print(math.ceil(n))
>>> 4 
#向下取整
print(math.floor(n))
>>> 3 
#四舍五入
a = 3.25
b = 3.75
print(math.round(a))
print(math.round(b))
#取整数和小数部分
n = 3.75
print(math.modf(n))
>>> (0.75, 3.0)
```

保留一位小数

```python
print(format((10/3),'.1f'))
print('%.1f'%(10/3))
print(round(10/3,1))
```

