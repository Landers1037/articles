---
title: python的IO操作
name: python-io
date: 2020-05-05
id: 0
tags: [python]
categories: []
abstract: ""
---


Python的文件读写IO操作

<!--more-->

## IO读写操作

**使用`open`和`read`完成文件流的打开和读取**

```python
In [2]: a = open('test.txt')

In [3]: a.read()
Out[3]: 'test just for test'
In [4]: a.close()    
```

当本地文件不存在时会抛出IO异常，使用try...finally可以保证文件流的正确关闭

**使用`with`语句可以自动完成文件流的关闭操作**

```python
In [5]: with open('test.txt','r')as f:
   ...:     print(f.read())
   ...:
test just for test
```

使用`readlines`读取每一行，在文件大小未知时使用`read(size)`读取特定大小字节的内容

## StringIO

非读取本地文件，而是从内存里读写str流数据时

```python
In [7]: from io import StringIO
   ...:
   ...: f = StringIO()

In [8]: f.write('hello world')
Out[8]: 11

In [9]: f.getvalue
Out[9]: <function StringIO.getvalue()>

In [10]: f.getvalue()
Out[10]: 'hello world'
```

读写内存需要实例化一个StringIO对象，使用`write`写入，使用`getvalue`读取内容

```python
In [15]: f = StringIO('hello world')

In [16]: print(f.readline())
hello world
```

## BytesIO

可以在内存里读写byte类型的数据

```python
In [17]: from io import BytesIO

In [18]: b = BytesIO()

In [19]: b.write('你好世界'.encode('utf-8'))
Out[19]: 12

In [20]: print(b.getvalue())
b'\xe4\xbd\xa0\xe5\xa5\xbd\xe4\xb8\x96\xe7\x95\x8c'
```

## 序列化映射

使用类作为一个参数对象时，需要对其进行序列化保存到本地

```python
import json

class TestClass(object):
    def __init__(self,name):
        self.name = name

    def output(self):
        print(self.name)

if __name__ == '__main__':
    t = TestClass('hello world')
    dict = json.dumps(t,default=lambda t :t.__dict__)
    print(dict)
```

使用类的属性`__dict__`可以将类以内置字典形式输出

或者定义一个序列化函数

```python
import json

class TestClass(object):
    def __init__(self,name):
        self.name = name

    def output(self):
        print(self.name)

    def todict(self):
        return {"name": self.name}

if __name__ == '__main__':
    t = TestClass('hello world')
    dict = json.dumps(t,default=lambda t:t.todict())
    print(dict)
```

使用`todict()`函数序列化后传入到dumps里

### 反序列化

定义一个反序列化函数，读取本地数据后映射为类

```python
import json

class TestClass(object):
    def __init__(self,name):
        self.name = name

    def output(self):
        print(self.name)

    def todict(self):
        return {"name": self.name}

def toclass(d):
    return TestClass(d["name"])


if __name__ == '__main__':
    t = TestClass('hello world')
    dict = json.dumps(t,default=lambda t:t.todict())
    print(dict)
    json_str = '{"name":"hello world"}'
    print(json.loads(json_str,object_hook=toclass))
```

使用`object_hook`传入反序列化的函数处理传入的json字符串，返回一个实例化的类对象

```bash
{"name": "hello world"}
<__main__.TestClass object at 0x0000020EADFC4B00>

Process finished with exit code 0
```

