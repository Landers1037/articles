---
title: Python文件读取
name: py5
date: 2019-02-28
id: 0
tags: [python]
categories: []
abstract: ""
---


# Python的文件处理1

使用with open()方法打开csv文件时出现多余空行
<!--more-->


# Python的文件处理1

使用with open()方法打开csv文件时出现多余空行<!--more-->

```python
# 使用附加参数newline
with open('1.csv',newline='')
```

因为在unix系统中的换行符和windows系统下的换行符号不同

python默认处理的时候使用LF的方式打开，windows下的换行符为CRLF
所以会出现多余一行的现象

