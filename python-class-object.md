---
title: python面向对象基础
name: python-class-object
date: 2020-05-30
id: 0
tags: [python]
categories: []
abstract: ""
---


Python面向对象


<!--more-->


Python面向对象

<!--more-->

## 类的初始化

```python
class Test:

    def __init__(self,name):
        self.name = name

if __name__ == '__main__':
    t = Test('ben')
    print(t.name)
    t.age = 10
    print(t.age)
```

使用`__init__`函数可以定义在初始化类时必须加入的参数，这里必须加入`name`参数

在实例化一个t对象以后，可以向对象添加任意的对象属性，这里添加了`age`属性

### 访问内部属性

在类的内部定义函数可以自由输出其属性

```python
class Test:

    def __init__(self,name):
        self.name = name

    def getname(self):
        return self.name

if __name__ == '__main__':
    t = Test('ben')
    print(t.getname())
```

定义`getname`函数在调用时返回对象的`name`属性

### 类的私有属性

所有公共属性都可以通过实例化后进行修改如`t.name = xx`。对于禁止修改的属性应该设置其为**私有变量**

```python
class Test:

    def __init__(self,name,private_name):
        self.name = name
        self.__private_name = private_name

    def getname(self):
        return self.name,self.__private_name


if __name__ == '__main__':
    t = Test('ben','hanhan')
    print(t.getname())
```

使用前缀`__`的方式定义私有变量，此时如果想要修改`private_name`

```python
if __name__ == '__main__':
    t = Test('ben','hanhan')
    print(t.getname())
    t.__private_name = 'tiehanhan'
    print(t.getname())
```

结果打印出来仍为`'hanhan'`

```python
print(t.__private_name)
```

直接打印内部变量会抛出`attribute`错误

```bash
    print(t.__private_name)
AttributeError: 'Test' object has no attribute '__private_name'
```

**内部变量的修改**

此时对于内部变量的修改就可以交给类自身处理，在类里定义修改函数

```python
class Test:

    def __init__(self,name,private_name):
        self.name = name
        self.__private_name = private_name

    def getname(self):
        return self.name,self.__private_name

    def set_name(self,new):
        self.__private_name = new
        
 if __name__ == '__main__':
    t = Test('ben','hanhan')
    t.set_name('tiehanhan')
    print(t.getname())
```

此时修改内部变量`__private_name`为`'tiehanhan'`

```python
('ben', 'tiehanhan')
```

## 类的继承与多态

```python
class Father(object):
    def jump(self):
        print('i can jump')

class Son(Father):
    pass

if __name__ == '__main__':
    f = Father()
    s = Son()
    f.jump()
    s.jump()
```

一个子类继承父类时就自动继承了其所有的方法函数

### 方法重写

```python
class Father(object):
    def jump(self):
        print('i can jump')

class Son(Father):
    def jump(self):
        print('son can jump too')

if __name__ == '__main__':
    f = Father()
    s = Son()
    f.jump()
    s.jump()
    
>>> 
i can jump
son can jump too
```

在Son里对`jump`函数重写就能和父类的`jump`区分开，执行同一个jump函数但是返回不同的结果

**这就是多态**

对于初始化后的对象，不需要知道它具体属于什么类型(如：Father Son)，因为他们都有`jump`方法，所以都可以执行`jump`方法，执行出来的结果根据类的不同而不同

```python
def choose_to_jump(class):
    class.jump()

choose_to_jump(Father())
choose_to_jump(Son())
```

定义好`choose_to_jump`函数后，只需要传入初始化的对象，就会自动调用对象的jump方法，无需考虑传入的对象具体的类，只需保证继承的类都具有jump方法

> 特殊： 传入的对象可以不是继承自父类，可以是一个全新的类并且具有`jump()`方法
>
> 这是典型’鸭子类型‘，只要看上去像鸭子，就是鸭子
>
> 类似于Go里的接口实现
>
> 对于类似Java的静态语言，在进行此类操作时传入的类需要是父类或者继承的子类

## 对象判断

判断类的方式可以使用`type`或者`isinstance`判断

```python
class Test(object):
    def __init__(self,name):
        self.name = name

if __name__ == '__main__':
    t = Test('test')
    print(isinstance(t,Test))
    print(getattr(t,'name',None))
    print(getattr(t,'age',None))
```

结果

```bash
True
test
None
```

当前实例化的对象t属于Test类，所以类型检查为true

使用`getattr`函数可以判断对象内部是否存在某个属性，当属性不存在时会抛出异常，定义默认属性为`None`，在属性不存在时返回

## 类与实例属性

例如

```python
class Test:
    name = 'hello'
```

并非通过init函数初始化时，Test类就具有属性name。此时访问`Test.name`可以得到值`hello`

此时如果实例化Test类

```python
In [4]: t = Test()

In [5]: t.name
Out[5]: 'hello'

In [6]: t.name = 'world'

In [7]: t.name
Out[7]: 'world'
    
In [8]: Test.name
Out[8]: 'hello'    
```

通过`t.name`访问时首先会访问`self.name`即当前实例下的name。如果无法找到就会寻找类的属性name

在修改`t.name`后其值发生变化，变化的值实际为`t`对象实例的值而非类的属性

通过`Test.name`访问时仍未最初定义的值