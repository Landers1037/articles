---
title: python读写方式的区别
name: python-rw
date: 2020-02-23
id: 0
tags: [python]
categories: []
abstract: ""
---


python不同读写方式的区别
`r,w,r+,w+,rb,wb,a,a+`


<!--more-->


python不同读写方式的区别
`r,w,r+,w+,rb,wb,a,a+`

<!--more-->

### r 只读

文件必须存在，读取文件的内容

```python
with open("test.txt",'r',encoding=xxx)as f:
    f.readline()
```

### r+ 读写

可读可写，文件必须存在。按照文件光标读写，如果写入时光标在首行就会覆盖全部内容

```python
    f = open("test.txt","r+",encoding='utf-8')
    f.write("直接覆盖内容")
    print(f.seek(0))
    print(f.read())
```

### rb 二进制读

以二进制方式读，如果读取文本返回其二进制文本格式

```python
    f = open("test.txt","rb",encoding='utf-8')
    print(f.read())
    # b'测试内容'
```

### w 写模式

存在则打开，不存在则创建新文件，直接覆盖原内容写入新内容

```python
    f = open("test.txt","w",encoding='utf-8')
    f.write("直接覆盖内容")
    print(f.seek(0))
    print(f.read())
```

### w+ 写读模式

存在则打开，不存在则创建新文件，直接覆盖原内容写入新内容。可以读根据seek换行

```python
    f = open("test.txt","w+",encoding='utf-8')
    f.write("直接覆盖内容")
    print(f.seek(0))
    print(f.read())
```

### wb 二进制写

用作写入二进制文件时使用（图片，视频流）

```python
    f = open("test.txt","wb")
    f.write("直接覆盖内容".encode("utf-8"))
```

### a 追加

不存在则创建，在文件末尾追加内容，不能读

```python
    f = open("test.txt","a",encoding='utf-8')
    f.write("追加内容")
```

### a+ 追加读写

不存在则创建，在文件末尾追加内容。可以读使用seek选择文件指针位置

```python
    f = open("test.txt","a",encoding='utf-8')
    f.write("追加内容")
    f.read() #不会返回内容因为此时文件指针在末尾
    f.seek(0)
    f.read() #从头读取
```

### 读写函数

#### seek 文件指针

`f.seek(offset)` 0表示文件开头，1表示当前位置，2表示文件末尾

#### tell

`f.tell()` 返回当前光标位置

#### truncate

`f.truncate(offset)` 从指定光标处截断删除后续内容

#### flush

`f.flush()` 刷新内存缓冲区域，把数据写入硬盘