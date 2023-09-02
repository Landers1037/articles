---
title: python进程与线程
name: python-multiprocess
date: 2020-04-21
id: 0
tags: [python]
categories: []
abstract: ""
---


Python的多进程与多线程

<!--more-->

## 进程

多线程使用库**multiprocessing**实现

```python
from multiprocessing import Process,Pool
import os

def task1():
    print('现在运行的进程pid',os.getpid())
    
p = Process(target=task1)
if __name__ == '__main__':
    p.start()
    p.join()
    p.close()
```

实例化一个Process类，并执行task1任务

> 注意：子进程由主进程生成，这里的主进程对应`__main__` 模块

传统的子进程由主进程**fork**出来，这只在类unix系统上适用，windows没有fork

```python
#unix
#传统的unix fork
pid = os.fork()
print('子进程的pid',os.getpid())
```

### 进程池

使用进程池可以一次创建多个子进程

```python
from multiprocessing import Process,Pool
import os

if __name__ == '__main__':
    # 使用进程池的方式创建多个进程
    pool = Pool(4)
    for i in range(4):
        pool.apply_async(task1)
    pool.close()
    pool.join()
```

在进程池关闭后，就不能向进程池添加新的进程

进程池的大小`Pool(int)`取决于当前系统的CPU核心数

### 进程间的通信

操作系统的进程间通信有多种方式如信号，共享内存，通道，命名空间，队列

python封装了多种方式

```python
#队列
#使用queue或者pipe的方式来实现进程之间的通信
from multiprocessing import Process,Queue,Pipe
import os,time

def read(str: Queue):
    print('读进程：pid=',os.getpid())
    #从队列里读取数据
    while True:
        s = str.get(True)
        print('从写进程传递来的数据',s)

def write(str: Queue):
    print('写进程：pid=',os.getpid())
    for s in ['你好','Python','HELLO','WOLRD']:
        str.put(s)
        time.sleep(2)

if __name__ == '__main__':
    # 注意main是主进程
    q = Queue()
    subr = Process(target=read,args=(q,))
    subw = Process(target=write,args=(q,))
    #先写入再读取
    subw.start()
    subr.start()
    subw.join()
    subr.terminate()
```

## 线程

一个任务还可以通过一个进程内的多个线程完成

多线程使用`threading`库或`_Thread`库实现

```python
from threading import Thread,Lock
import os

def task(n):
    #print now running pid
    pid = os.getpid()
    print('sub threading %s is running: '%n,pid)


if __name__ == '__main__':
    main_pid = os.getpid()
    print("now running pid is: ",main_pid)
    for i in range(4):
        p = Thread(target=task,args=(i,))
        p.start()
        p.join()
```

结果

```python
now running pid is:  12420
sub threading 0 is running:  12420
sub threading 1 is running:  12420
sub threading 2 is running:  12420
sub threading 3 is running:  12420
```

> 所有的线程共享该进程的内存空间

### 线程的同步

```python
from threading import Thread,Lock

NUM = 0

def run(n):
    global NUM
    NUM +=n
    NUM -=n



def task(n):
    for i in range(1000000):
        run(n)

if __name__ == '__main__':
    t1 = Thread(target=task,args=(1,))
    t2 = Thread(target=task,args=(4,))

    t1.start()
    t2.start()
    t1.join()
    t2.join()

    print(NUM)
```

对全局变量NUM的修改，理想结果应该是0。但是由于多线程的存在对全局变量的修改可能是无序的，所以结果可能不是0。

为了保证数据的正确性，在一个线程访问数据时应该加锁，保证其他线程无法修改此变量

```python
from threading import Thread,Lock

NUM = 0
lock = Lock()

def run(n):
    global NUM
    NUM +=n
    NUM -=n



def task(n):
    for i in range(1000000):
        lock.acquire()
        try:
            run(n)
        finally:
            lock.release()

if __name__ == '__main__':
    t1 = Thread(target=task,args=(1,))
    t2 = Thread(target=task,args=(4,))

    t1.start()
    t2.start()
    t1.join()
    t2.join()

    print(NUM)
```

使用try...finally保证锁的释放

### 全局锁

GIL是python解释器的全局锁，在python解释器解释执行代码时所有的代码会加锁。python解释器保证了多线程程序代码交替执行。所以多线程的cpu占用只能使用一个cpu内核

可以使用多进程的方式在每个进程里执行线程操作，每个进程都是独立的GIL。保证程序可以更有效地使用CPU资源

### 线程间传递变量

当多个线程内部只使用仅自己可以访问的局部变量时，多个局部变量的定义和传递会比较麻烦。

可以使用内置的ThreadLocal管理局部变量，其性质类似于全局字典，键对应于线程的唯一运行ID，值对应线程的局部变量。

```python
import threading

thread_local = threading.local()

def run():
    std = thread_local.v
    print('当前线程操作的变量：',threading.current_thread(),std)

def task(name):
    thread_local.v = name #存储变量
    run()

if __name__ == '__main__':
    t1 = threading.Thread(target=task,args=('task1',))
    t2 = threading.Thread(target=task,args=('task2',))
    t1.start()
    t2.start()
    t1.join()
    t2.join()
```

注册全局的`thread_local` 把每次线程创建时的局部变量保存到该字典中