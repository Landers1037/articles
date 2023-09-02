---
title: python入门指南
name: py1
date: 2019-02-02
id: 0
tags: [python,学习笔记]
categories: []
abstract: ""
---


# 环境：Python 3.7

## 库的安装和使用

```shell
pip install ***
```

使用这种方法安装库的前提是，python的路径已经安装在了环境变量里。python的所有第三方库都在安装目录的Lib文件夹内
<!--more-->
有的时候使用pip安装会出现各种环境错误，这个时候使用离线库文件.whl安装
第三方python库：[点击进入](https://www.lfd.uci.edu/~gohlke/pythonlibs/)
下载对应的库后，安装方法

```shell
cd /存放whl文件的目录
pip install ***.whl
```

导入安装好的库

```python
import PyQt5 #导入这个类
from PyQt5.Qwidget import QApp #从库中导入一个类
from PyQt5 import* #从库中导入全部初始类
# 第三句仅仅是导入在PyQt5下面init的所有类，不包括其他的类
```



## 模块的相互调用

自定义一个模块后，怎么在其他的类中使用

例如：定义了一个son.py

```python
from son import son
```



## 习惯操作

我们定义一个main（）函数作为程序的入口，在有调用关系的时候，如果只是想测试当前的模块，常常使用惯例写法

```python
if __name__=="__main__"
```



## 命名

python模块解释器默认从当前文件夹开始寻找是否有在模块中调用的库，如果你的package命名和内置的库名称一致，就会优先调用用户定义的库，这个时候就会出错



## 定义根目录

自带的根目录是src，如果你在下面还建立了多个package，在引入第三方的库时会有错误提示但是运行不会出问题，使用pycharm在当前的package下mark directory as source即可


## 文件相对路径

如果在运行代码的目录下放置图片资源1.jpg，调用的时候使用

```python
（‘1.jpg')
```

如果在src下的例如img目录下

```python
（’src/img/1.png')
```



## 中文乱码

Pycharm跟据操作系统默认的编码是GBK，进入settings，找到code encoding，设置为UTF-8