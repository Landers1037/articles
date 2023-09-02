---
title: python面向对象编程
name: python-advace-object
date: 2020-05-30
id: 0
tags: [python]
categories: []
abstract: ""
---


Python面向对象编程

<!--more-->

## 对象属性限制

使用`__slots__`可以定义允许添加的属性，非法属性将会抛出异常

```python
In [9]: class Father:
   ...:     __slots__ = ('name')
   ...:

In [10]: f = Father()

In [11]: f.name = 'baba'

In [12]: f.name
Out[12]: 'baba'

In [13]: f.age = 10
---------------------------------------------------------------------------
AttributeError                            Traceback (most recent call last)
<ipython-input-13-b25e57e12b4d> in <module>
----> 1 f.age = 10

AttributeError: 'Father' object has no attribute 'age'
```

`__slots__`的值是一个元组类型，使用`f.age`添加age属性时因为age不在定义的元组内所以不能添加成功

## 属性判断

如果该属性有实际的意义，例如`age`，必须是整型**int**且不能超过100。

在定义该属性变量时就不能随便赋值，需要提前进行检查

```python
class Test(object):
    def __init__(self,name):
        self.name = name
    
    def set_age(self,age):
        if isinstance(age,int) and age <= 100:
            self.age = age
        else:
            raise  ValueError('age must between 0-100')
```

使用`set_age`方式对传入的age进行判断并赋值

### 使用@property装饰器

```python
    @property
    def age(self):
        return self.__age
    
    @age.setter
    def age(self,value):
        if isinstance(value,int) and value <= 100:
            self.__age = value
        else:
            raise ValueError('age must between 0-100')

...
    t.age = 10
    print(t.age)
    t.age = 1000
    print(t.age)
```

赋值为10时返回正常的数值，赋值为1000时不满足条件会返回错误

```bash
10
  File "E:/pyproject/simplepy/类的继承/class4.py", line 34, in <module>
    t.age = 1000
  File "E:/pyproject/simplepy/类的继承/class4.py", line 24, in age
    raise ValueError('age must between 0-100')
ValueError: age must between 0-100
```

使用`@property`装饰后的函数可以作为类的属性进行调用，如果想要设置一个属性为只读不可写只需要不设置setter函数

```python
@property
def song_list(self):
    return self.__song_list
```

此时可以通过`t.song_list`访问到该对象的属性，但是进行赋值操作`t.song_lis = xxx`时会操作失败

## 类的多重继承

当类的分类更加细致，每个子类需求的功能多变时，可以通过多个继承实现

```python
class Walk:
    def walk(self):
        print('i can walk')

class Run:
    def run(self):
        print('i can run')

class GrandPa(Walk):
    pass

class Father(GrandPa,Walk):
    pass

class Son(Father,Walk,Run):
    pass

if __name__ == '__main__':
    g = GrandPa()
    f = Father()
    s = Son()
    g.walk()
    f.walk()
    s.walk()
    s.run()
```

首先定义两个类`Walk`和`Run`，分别实现`walk`和`run`的方法**。GrandPa**作为**Father**和**Son**的初始类继承了Walk拥有了walk方法，而Son还继承了Run拥有了run方法

> 一个子类可以继承多个父类，实现其全部功能，这个父类称为`Mixin`
>
> Mixin的作用就是为继承它的子类提供更多功能

## 类的内置函数

诸如`__slots__`这类的内置函数可以针对当前类设置

### `__len__`

返回一个长度值，在外部可以使用`len(Class)`的方式得到类实例的长度

`__str__`

定义类实例的默认返回名称

```python
In [1]: class Test:
   ...:     pass
   ...:

In [2]: t = Test()

In [3]: t
Out[3]: <__main__.Test at 0x1c8d0d090f0>

In [4]: class Test2:
   ...:     def __str__(self):
   ...:         return 'Test2'
   ...:

In [5]: t2 = Test2()

In [6]: t2
Out[6]: <__main__.Test2 at 0x1c8d0cf70b8>

In [7]: print(t2)
Test2
```

定义`__str__`后类实例就会返回实现定义好的语句，使用print输出

### `__repr__`

使用`__str__`只是定义默认的输出，直接调用类实例返回的依旧是类在Python内存的地址。使用`__repr__`函数可以自定义打印输出的语句，无需调用print语句

```python
In [8]: class Test3:
   ...:     def __repr__(self):
   ...:         return 'Test3'
   ...:

In [9]: t3 = Test3()

In [10]: t3
Out[10]: Test3
```

### `__iter__`

返回一个迭代器的类型对象，可以通过for循环不断调用`__next__()`方法来得到值，直到`StopIteration`错误抛出停止

### `__getitem__`

使类可以像列表一样进行下标操作，例如`t = Test() t[0]`

`__getattr__`

自定义获取类属性的方法，当调用`t.name`时应该返回name属性，当name属性不存在就会抛出None错误

```python
   def __getattr__(self, attr):
        if attr=='name':
            return 'please enter your name'
```

使用`getattr`定义当前类不存在属性时应该返回的数据，默认时返回值为`None`

`__call__`

为类定义调用函数，可以直接调用实例化的对象

```python
In [11]: class Test4:
    ...:     def __call__(self):
    ...:         print('hello world')
    ...:

In [12]: t4 = Test4()

In [13]: t4()
hello world
```

还可以向实例传参，如

```python
In [14]: class Test5:
    ...:     def __call__(self,name):
    ...:         print('hello ' + name)
    ...:

In [15]: t5 = Test5()

In [16]: t5('mike')
hello mike
```

## super

对父类或者超类的方法调用

用于解决多重继承时的方法顺序问题

```python
class Father:
    def __init__(self):
        print('father is speaking')

    def name(self,name):
        self.name = name
        print('father is speaking',self.name)

class Son(Father):
    def __init__(self):
        super().__init__()
        print('this is child')

    def name(self,name):
        super().name(name)
        print('son is speaking',self.name)

if __name__ == '__main__':
    s = Son()
    s.name('erzi')
```

在`Son`类的初始化函数里调用了`Father`的初始化函数，在`Son`的name方法里调用了`Father`的name方法

在Python2中super的使用应该为`super(class,self).function()`需要传入当前类和self对象

## 静态方法

使用`@staticmethod`装饰器可以装饰类的方法，返回一个静态方法

原本类的实例调用方法需要先将类实例化如`class1().func()`，在使用装饰器装饰后可以直接`class1.func()`的方式调用方法

```python
class Test:
    @staticmethod
    def print():
        print('this is a Test func')


if __name__ == '__main__':
    t = Test()
    t.print()
    Test.print()
>>>
this is a Test func
this is a Test func
```

可以看到使用`@staticmethod`装饰的函数无需传入**self**对象自身