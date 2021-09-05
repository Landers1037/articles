---
title: python基础
name: python-base3
date: 2020-03-01
id: 0
tags: [python]
categories: []
abstract: ""
---


生成器，迭代器


<!--more-->


生成器，迭代器

<!--more-->

## 迭代

可迭代的对象即可以使用for循环遍历获取内部的值，`Iterator`对象

常见的列表，元组，字典都可以进行迭代，但是它们不是迭代器对象

```python
In [13]: from collections import Iterable,Iterator
D:\Python3.7\Scripts\ipython:1: DeprecationWarning: Using or importing the ABCs from 'collections' instead of from 'collections.abc' is deprecated, and in 3.8 it will stop working

In [14]: isinstance([],Iterable)
Out[14]: True

In [15]: isinstance([],Iterator)
Out[15]: False
```

看到它们都是**可迭代**的，但是不是**迭代器**对象

### 列表生成式

可以使用生成式生成一个可迭代的列表

```python
In [1]: [x**2 for x in range(1,10)]
Out[1]: [1, 4, 9, 16, 25, 36, 49, 64, 81]
```

在`for`前面的是表达式

可以在`for`循环后面添加`if`过滤器

```python
In [5]: [x for x in ["acv","hj",123] if isinstance(x,str)]
Out[5]: ['acv', 'hj']
```

这里过滤掉了非字符串对象

在`for`前面的`if-else`语句属于表达式

```python
In [17]: [x+2 if x<=10 else 10 for x in range(12)]
Out[17]: [2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 10]
```

在表达式里的`if-else`语句判断此时遍历的`x`值，根据不同的情况计算得到`x+2`的值

## 生成器

比生成器更加节省内存，因为列表生成时是把所有的内容保存在内存里。使用生成器可以按顺序推算出一个list里的元素，而不需要建立整个list。

因为是推算所以使用`next()`函数访问生成器里的下一个对象。它会一直查找存在的下一个对象直到生成器最终没有元素

### 生成式

```python
In [7]: (x*2 for x in [1,2,3])
Out[7]: <generator object <genexpr> at 0x0000027A61AB9750>
```

使用next遍历

```python
In [7]: (x*2 for x in [1,2,3])
Out[7]: <generator object <genexpr> at 0x0000027A61AB9750>

In [8]: g = (x*2 for x in [1,2,3])

In [9]: next(g)
Out[9]: 2

In [10]: next(g)
Out[10]: 4

In [11]: next(g)
Out[11]: 6

In [12]: next(g)
---------------------------------------------------------------------------
StopIteration                             Traceback (most recent call last)
<ipython-input-12-e734f8aca5ac> in <module>
----> 1 next(g)

StopIteration:
```

可以看到在访问最后一个元素以后，生成器已经没有元素，此时返回`StopIteration`错误

除了使用`next`还可以使用`for`遍历一个生成器，因为使用`next()`遍历会多次调用`next`

```python
In [21]: for i in (i for i in range(6)):
    ...:     print(i)
    ...:
0
1
2
3
4
5
```

使用for循环遍历生成器的实现其实还是在for内部调用了`next()`函数

### yield

同时如果一个函数内含有关键字`yield`，那么它是一个特殊的生成器函数

```python
def gen(max):
    n = 0
    while n < max:
        yield n+1
        n = n + 1
    return 'done'
```

调用该函数

```python
In [23]: def gen(max):
    ...:     n = 0
    ...:     while n < max:
    ...:         yield n+1
    ...:         n = n + 1
    ...:     return 'done'
    ...:

In [24]: f= gen(6)

In [25]: f
Out[25]: <generator object gen at 0x0000027A62630228>
```

可以看到此时的f是一个生成器对象，我们可以使用`next()`输出它

```python
In [26]: next(f)
Out[26]: 1

In [27]: next(f)
Out[27]: 2

In [28]: next(f)
Out[28]: 3

In [29]: next(f)
Out[29]: 4

In [30]: next(f)
Out[30]: 5

In [31]: next(f)
Out[31]: 6

In [32]: next(f)
---------------------------------------------------------------------------
StopIteration                             Traceback (most recent call last)
<ipython-input-32-aff1dd02a623> in <module>
----> 1 next(f)

StopIteration: done
```

## 总结

生成器和列表，元组，字典，set都是可以迭代的对象(即可以使用for遍历的对象)

生成器属于迭代器(itertor)，还能使用next()遍历

对于列表，元组，字典，set虽然不是迭代器但是可以迭代。还能够使用`iter()`将其转变为`Itertor`对象

```python
In [33]: iter([1,2,3])
Out[33]: <list_iterator at 0x27a629ccb70>
```

