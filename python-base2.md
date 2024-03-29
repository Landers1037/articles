---
title: python基础
name: python-base2
date: 2020-02-29
id: 0
tags: [python]
categories: []
abstract: ""
---


Python的深拷贝和浅拷贝

<!--more-->

### 普通的对象传值

```python
a = 123
b = a
print(id(a),id(b))
```

会发现a的值等于b，并且a的内存地址等于b的内存地址

### 浅拷贝

使用copy函数可以为变量重开一个内存空间存储copy下的值

```python
import copy
a = [1,2,3]
b = copy.copy(a)
b.append(4)
print(a,b)
```

输出

```python
[1,2,3]
[1,2,3,4]
```

可以看到此时a和b的值不同，因为它们的地址不同

### 深拷贝

```python
import copy
a = [1,2,3,[4,5]]
b = copy.deepcopy(b)
b.append(4)
print(a,b)
```

输出

```
[1,2,3,[4,5]]
[1,2,3,[4,5],4]
```

可以看到深拷贝的数据内存地址也是和原数据不同的

### 深浅拷贝区别

<font color=red>浅拷贝只拷贝原数据的第一层</font>

```python
import copy

def softcopy():
    a = [1,2,3,[4,5]]
    b= a
    c = copy.copy(b)
    bef = "原来的值为a {},b {},c {}: ".format(a,b,c)
    befadd = "原来的值的内存地址a {},b {},c {}: ".format(id(a),id(b),id(c))
    print(bef)
    print(befadd)
    b.append(6)
    print(a,b,c,id(a),id(b),id(c))
    c.append(7)
    print(a, b, c, id(a), id(b), id(c))
    b[3].append("浅拷贝")
    print(a, b, c, id(a), id(b), id(c))

softcopy()

def deepcopy():
    a = [1,2,3,[4,5]]
    b =a
    c = copy.deepcopy(b)
    bef = "原来的值为a {},b {},c {}: ".format(a,b,c)
    befadd = "原来的值的内存地址a {},b {},c {}: ".format(id(a),id(b),id(c))
    print(bef)
    print(befadd)
    b.append(6)
    print(a,b,c,id(a),id(b),id(c))
    c.append(7)
    print(a, b, c, id(a), id(b), id(c))
    b[3].append("深拷贝")
    print(a, b, c, id(a), id(b), id(c))

deepcopy()
```

输出

```python
原来的值为a [1, 2, 3, [4, 5]],b [1, 2, 3, [4, 5]],c [1, 2, 3, [4, 5]]: 
原来的值的内存地址a 2182143042120,b 2182143042120,c 2182175069832: 
[1, 2, 3, [4, 5], 6] [1, 2, 3, [4, 5], 6] [1, 2, 3, [4, 5]] 2182143042120 2182143042120 2182175069832
[1, 2, 3, [4, 5], 6] [1, 2, 3, [4, 5], 6] [1, 2, 3, [4, 5], 7] 2182143042120 2182143042120 2182175069832
[1, 2, 3, [4, 5, '浅拷贝'], 6] [1, 2, 3, [4, 5, '浅拷贝'], 6] [1, 2, 3, [4, 5, '浅拷贝'], 7] 2182143042120 2182143042120 2182175069832

#deep
原来的值为a [1, 2, 3, [4, 5]],b [1, 2, 3, [4, 5]],c [1, 2, 3, [4, 5]]: 
原来的值的内存地址a 2182143042056,b 2182143042056,c 2182143042120: 
[1, 2, 3, [4, 5], 6] [1, 2, 3, [4, 5], 6] [1, 2, 3, [4, 5]] 2182143042056 2182143042056 2182143042120
[1, 2, 3, [4, 5], 6] [1, 2, 3, [4, 5], 6] [1, 2, 3, [4, 5], 7] 2182143042056 2182143042056 2182143042120
[1, 2, 3, [4, 5, '深拷贝'], 6] [1, 2, 3, [4, 5, '深拷贝'], 6] [1, 2, 3, [4, 5], 7] 2182143042056 2182143042056 2182143042120
```

我们从最后一次执行的结果可以看到，当数据复杂，比如`[1,2,3,[4,5]]`是一个两层结构的列表时。浅拷贝仅拷贝了列表的第一层结构，所以对b直接的操作仍是在新开辟的内存上。

而对`b[3]`即`[3,4]`的操作是在原数据的内存上，所以a和b都发生了变化。

深拷贝则是把数据的全部层次完整的拷贝到新的内存上，即一个完全新的数据。