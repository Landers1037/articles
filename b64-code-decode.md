---
title: base64编解码
name: b64-code-decode
date: 2019-08-13
id: 0
tags: [python]
categories: []
abstract: ""
---


基于Base64的字符串编解码

Base64广泛用于非安全传输加密数据
解析SS订阅链接的时候遇到Base64解码的问题，所以总结一下

<!--more-->

## Base64

Base64，就是选出64个字符小写字母a-z、大写字母A-Z、数字0-9、符号"+"、"/"以及用作缺省补充的"="，一共65个字符作为一个基本字符集，将需要编码的数据转化为这个字符集里的字符

### 加密编码

```python
import base64
encode = base64.b64encode(b'hello world')
print(encode)
```

常用64位编码所以使用`b64encode`，需要加密的字符串前加`'b'`为了兼容python2的字符串格式

### 解码

```python
str = 'SSBsb3ZlIHlvdQ=='
decode = base64.b64decode(str)
print(decode)
```

### 特殊解码

当解码的字符串实际是url链接(即含有:,//)时，使用一般的`b64decode`可能出现错误
此时使用url解码函数，`urlsafe_b64decode`

### 实际应用的问题

对一串SS订阅链接进行解密时默认返回的是兼容的bytes类型，里面的'\n'等特殊字符不能正常打印
此时针对python3使用bytes转str方法把解码后的数据转为字符串

```python
decode = base64.b64decode(str)
final = decode.decode('utf-8')
print(final)
```

