---
title: pandas的loc，iloc函数
name: py6
date: 2019-03-01
id: 0
tags: [python]
categories: []
abstract: ""
---


# pandas基础
<!--more-->


# pandas基础<!--more-->

## pandas的loc函数

```python
# loc函数用于使用标题索引行列
# 当为连续的行列时
.loc['行','列']
.loc[:,'A':'B'] # 索引AB列
.loc['a':'b',:] # 索引ab行
.loc['a':'b','A':'B'] # 索引ab行和AB列
#当不连续的行列时
.loc[['c':'f'],['C':'F']] # 取c到f行 C到F列
.loc[['c','f'],['C','F']] # 取c和f行 C和F列
# 注意区分：和，的区别
```

## pandas的iloc函数

```python
# 使用行号列号进行索引注意起始位为0
.iloc[:,0:2] # 取前两列
.iloc[0:2,:] # 取前两行
# 不连续时
.iloc[[0,2],[0,3] # 取1,3行 1,4列
```

