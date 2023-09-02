---
title: python基础
name: python-base1
date: 2020-02-28
id: 0
tags: [python]
categories: []
abstract: ""
---


字符串,**list**,**tulpe**,**dict**的处理

<!--more-->

## 字符串

同列表类似，字符串是顺序存储的也可以直接使用切片的方式读取内容

例如：`str[1:]`

均支持的操作

`str + "test" `	拼接

`str *2` 	重复两次

### 内置函数

```python
str =  "I'm a test String "
li = []

li.append(str.strip())
li.append(str.split())
li.append(str.replace("a","A"))
li.append(str.index("m"))
li.append(str.find("m"))
li.append(str.encode(encoding="utf-16"))
li.append(str.capitalize())
li.append(str.lower())
li.append(str.upper())
li.append(str[2:])
li.append(str*2)
li.append(str + "what is ADDED")
li.append(",".join((str,"ADDED string")))
li.append(u"unicode string")
li.append(r"what is origin\n")

for i in li: print(i);
```

结果

```python
I'm a test String
["I'm", 'a', 'test', 'String']
I'm A test String 
2
2
b"\xff\xfeI\x00'\x00m\x00 \x00a\x00 \x00t\x00e\x00s\x00t\x00 \x00S\x00t\x00r\x00i\x00n\x00g\x00 \x00"
I'm a test string 
i'm a test string 
I'M A TEST STRING 
m a test String 
I'm a test String I'm a test String 
I'm a test String what is ADDED
I'm a test String ,ADDED string
unicode string
what is origin\n
```

### 格式化符

除了使用`str.format()`格式化输出还可以使用格式化符

| 符号 | 含义               |
| ---- | ------------------ |
| c    | 格式化字符和ascall |
| s    | 字符串             |
| d    | 整数               |
| f    | 浮点数             |
| o    | 八进制数           |
| x    | 十六进制           |
| p    | 对象的内存地址     |

## 列表LIST

同样支持切片操作，`+`连接操作，`*`重复

**列表操作的函数**

`len()`	得到长度

`del()`	删除列表或其中的元素

`max()`,`min()`	返回列表的最大，最小值

`list()`	把元组转换为列表

**列表内置函数**

```python
li = [1,2,3]

def func(p):
    if p:
        print(p)
    else:
        print(li)


func(li[1:])
func(li.append(4))
func(li.insert(2,6))
func(li.index(2))
func(li.remove(2))
func(li.extend([7,8]))
func(li.pop(0))
func(li.sort())
```

结果

```python
[2, 3]
[1, 2, 3, 4]
[1, 2, 6, 3, 4]
1
[1, 6, 3, 4]
[1, 6, 3, 4, 7, 8]
1
[3, 4, 6, 7, 8]
```

## 元组tuple

不可改变的数据结构

支持+和*操作，支持切片

**内置函数**

`len(tuple)`	返回长度

`tuple(list)`	列表转换为元组

## 字典dict

以键值对的形式定义

`{key:value}`键必须是不可变的数字，字符串或元组

**内置方法**

`dict.clear()`	清除内部全部元素

`dict.copy()`	返回一个浅拷贝

`dict.get(key)`	对应键的值，没有返回none

`dict.haskey(key)`	是否存在key，返回布尔值

`dict.items()`	返回可迭代的元组

`dict.keys()`	打印全部key

`dict.values()`	打印全部value

`dict.pop(key)`	pop掉key对应的值并返回该值，否则返回默认值

`dict.update(dict2)`	把dict2的键值对更新到dict里

**修改值**

字典是无序的只能通过访问键的方式修改值，键唯一存在，重复则会更新内部的值

`d = {"name": "hanhan"}`

`d["name"] = "benben"`	更新键的值

`d["age"] = 1000`	添加新的键

`d.get("name")`	得到name对应的值