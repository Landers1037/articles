---
title: python基础
name: python-base5
date: 2020-03-01
id: 0
tags: [python]
categories: []
abstract: ""
---


python多线程与进程


<!--more-->


python多线程与进程

<!--more-->

实现的三种模式：

多进程，开启多个进程一个进程一个线程执行任务

多线程，开启一个进程里面多个线程

多线程与多进程，开启多个进程，一个进程里允许多个线程

### 多线程

```python
from threading import Thread,local,Lock
import threading

def run(name):
    print("now run %s",threading.currentThread(),name)
    
print('thread %s is running...' % threading.current_thread().name)    
t1 = Thread(target=run,args=("1",))
t2 = Thread(target=run,args=("2",))

t1.start()
t1.join()
t2.start()
t2.join()
```

使用Python的threading模块创建新的线程实例

使用`threading.current_thread()`可以得到当前执行的线程的名称

python运行程序时会自动创建一个主线程，然后在主线程里fork出子线程

```python
thread MainThread is running...
now run %s <Thread(Thread-1, started 3100)> 1
now run %s <Thread(Thread-2, started 11508)> 2
```

### 锁🔒

默认的在每个子线程里定义的局部变量只有对应的子线程可以访问，但是当有一个全局变量需要多个线程一同访问修改时就会出现问题

比如

```python
g = 10
def run(num):
    # print("now run %s",threading.currentThread(),num)
    global g
    g +=num
    g -=num

def runlong(num):
    for i in range(1000000):
        run(num)

t1 = Thread(target=runlong,args=(14,))
t2 = Thread(target=runlong,args=(4,))

print('thread %s is running...' % threading.current_thread().name)
t1.start()
t2.start()
t1.join()
t2.join()

print(g)
```

因为线程的运算是无序的，所以最后的结果可能不是期望的结果

引入threading的Lock🔒可以保证线程访问的时候不会有其他线程干扰

```python
g = 10
lock = Lock()
def run(num):
    global g
    g +=num
    g -=num

def runlong(num):
    for i in range(1000000):
        try:
            lock.acquire()
            run(num)
        finally:
            lock.release()
            

t1 = Thread(target=runlong,args=(14,))
t2 = Thread(target=runlong,args=(4,))

print('thread %s is running...' % threading.current_thread().name)
t1.start()
t2.start()
t1.join()
t2.join()
```

在每次访问修改前都给线程加上锁，为了保证锁能够最后释放使用`finally`关键字，避免出现死线程