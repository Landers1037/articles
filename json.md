---
title: Python中的json数据
name: json
date: 2019-02-20
id: 0
tags: [json,python]
categories: []
abstract: ""
---


# json数据

## 什么是 JSON ？

- JSON 指的是 JavaScript 对象表示法（*J*ava*S*cript *O*bject *N*otation）
- JSON 是轻量级的文本数据交换格式
- JSON 独立于语言 *
- JSON 具有自我描述性，更易理解
  
<!--more-->


# json数据

## 什么是 JSON ？

- JSON 指的是 JavaScript 对象表示法（*J*ava*S*cript *O*bject *N*otation）
- JSON 是轻量级的文本数据交换格式
- JSON 独立于语言 *
- JSON 具有自我描述性，更易理解
  <!--more-->

\* JSON 使用 JavaScript 语法来描述数据对象，但是 JSON 仍然独立于语言和平台。JSON 解析器和 JSON 库支持许多不同的编程语言。

## python中对json的读取

```python
import json
file = open('test.json','r',encoding='utf-8')
json.load(file)
```

## python中对json的写入

```python
import json
with open('list.json','w',encoding='utf-8')as file:
        file.write(json.dumps(list,indent=2,ensure_ascii=False))
```

## dump和dumps区别

## load和loads的区别

```python
import json
a = {"num":"12"} #这是一个字典
b = json.dumps(a) #把字典转化为str
c = b #这是一个字符串
d = json.loads(c) #把字符串转为字典

fp = open('1.json','r')
e = json.load(fp) #dump,load属于文件操作还需要传入参数,不是简单的只有json对象
```

## 注意

使用dump函数转为json的时候，json里有中文的时候，需要传入参数`ensure_ascii=False`，否则会报错。
因为dump序列化时，会将中文转码