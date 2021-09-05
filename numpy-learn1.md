---
title: Numpy打包出现的问题
name: numpy-learn1
date: 2019-04-07
id: 0
tags: [numpy]
categories: []
abstract: ""
---


# 版本：Numpy1.6.01

在对新版本的numpy库进行打包的时候，经常会出现导入包错误，最终运行时提醒

`no module name numpy.core.***`

<!--more-->


# 版本：Numpy1.6.01

在对新版本的numpy库进行打包的时候，经常会出现导入包错误，最终运行时提醒

`no module name numpy.core.***`
<!--more-->

往往是由于numpy的版本出现问题，对numpy降级或更新，对python自带的setuptools进行更新

最简单的解决办法，在需要打包的py脚本中导入缺失的包

```python
import numpy.core._dtype_ctypes
```

