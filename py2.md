---
title: Python入门 字符处理
name: py2
date: 2019-02-17
id: 0
tags: [python]
categories: []
abstract: ""
---


# 常见的编码问题

## 中文字符

中文字符属于GBK编码字符集，在爬虫保存的时候数据一般保存为utf-8的形式，这个时候文件里的中文会出现乱码，通常使用以下的几种办法解决
<!--more-->

```python
# 把utf-8保存为gbk格式
def ReadFile(filePath,encoding="utf-8"):
    with codecs.open(filePath,"r",encoding) as f:
        return f.read()
 #通过utf-8格式读取文件
def WriteFile(filePath,u,encoding="gbk"):
    with codecs.open(filePath,"w",encoding) as f:
        f.write(u)
 #文件保存为gbk
def UTF8_to_GBK(src,dst):
    content = ReadFile(src,encoding="utf-8")
    WriteFile(dst,content,encoding="gbk")
```

不想通过读取的方式转换可以直接把数据保存为gbk的格式
通常转码的时候会遇到的错误

```python
UnicodeEncodeError: 'gbk' codec can't encode character u'\xa0' in position 4813: illegal multibyte sequence
```

使用编码gbk超集解决

```python
encoding=gkb18030
error='igonre'
```

## ascii码

```python
ord('a') # 返回字母a的ascii
chr(100) # 返回100 int对应的字符
unichr(100) # 返回对应的unicode
```

