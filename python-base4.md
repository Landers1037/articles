---
title: python基础
name: python-base4
date: 2020-03-01
id: 0
tags: [python]
categories: []
abstract: ""
---


函数基础


<!--more-->


函数基础

<!--more-->

## 内置函数map，reduce

### map()

接收两个参数，(函数,迭代对象)，把迭代对象里的元素作用在函数上，返回一个新的迭代对象

```python
In [36]: m = map(lambda x: x**3,[1,2,3])

In [37]: print(m)
<map object at 0x0000027A6221C4A8>
```

返回的是一个映射对象，如果想要得到里面的可迭代对象

```python
In [38]: list(m)
Out[38]: [1, 8, 27]
```

### reduce()

接收两个参数，(函数，序列)，函数作用于两个元素，然后reduce把结果和下一个序列元素相加

实现sum

```python
In [45]: reduce(lambda x,y:x+y,[1,2,3])
Out[45]: 6
```

实现了求和的功能

输出序列对应的整数

```python
In [46]: reduce(lambda x,y:x*10+y,[1,2,3])
Out[46]: 123
```

### filter()

过滤器，接收两个参数(函数，序列)，通过计算函数得到返回值是否为True来筛选序列内的元素

```python
In [48]: list(filter(lambda x:x%2==0,[1,2,3,4]))
Out[48]: [2, 4]
```

类似于生成式的过滤机制

```python
In [49]: [x for x in [1,2,3,4] if x%2==0]
Out[49]: [2, 4]
```

**注意：过滤器返回的是一个懒惰的生成器对象，需要使用`list`来返回全部结果**

> 例如：常用的寻找偶数和寻找素数的算法，可以使用filter来返回符合情况的结果

### sorted()

内置的排序函数，接收一个list

```python
In [50]: sorted([1,3,7,4,5])
Out[50]: [1, 3, 4, 5, 7]
```

提供一个额外的key指定排序方式

```python
#按照绝对值排序
In [51]: sorted([1,3,7,4,-5],key=abs)
Out[51]: [1, 3, 4, -5, 7]
```

除此以外，还能给字符串排序

```python
>>> sorted(['bob', 'about', 'Zoo', 'Credit'], key=str.lower)
['about', 'bob', 'Credit', 'Zoo']
```

字符串排序时比较的是字母的ASCALL码值

传入reverse=True反向排序

```python
In [52]: sorted([1,3,7,4,5],reverse=True)
Out[52]: [7, 5, 4, 3, 1]
```

## 闭包与匿名函数

### 闭包

```python
def outfunc(arr):
    def func():
        print(sum(arr))
        return sum(arr)
    return func

o = outfunc([1,2,3])

o()
```

`outfunc()`函数的一个内部函数用于计算求和，在调用`output(arr)`时o被赋值这个内部函数

此时再执行o()函数就会计算和

### 匿名函数

使用`lambda`函数定义，即不需要显式函数时

```python
list(map(lambda x: x * x, [1, 2, 3, 4, 5, 6, 7, 8, 9]))
```

### 装饰器

装饰器就是匿名函数

```python
def log(func):
    def wrapper(*args, **kw):
        print('call %s():' % func.__name__)
        return func(*args, **kw)
    return wrapper
```

可以使用@符号在别的函数前添加该装饰器

```python
@log
def func():
    ...
```

相当于把`func()`函数作为参数传入了`log()`函数里

如果本身装饰器需要传入参数如`@log("hello")`则在装饰器函数里再加一层嵌套函数，返回该嵌套函数

可以使用**functools**的`wrap`来创建更加高级的装饰器