---
title: python-async异步
name: python-asyncio
date: 2020-05-31
id: 0
tags: [python]
categories: []
abstract: ""
---


Python的异步协程处理，`asyncio`，`async/await`


<!--more-->


Python的异步协程处理，`asyncio`，`async/await`

<!--more-->

## asyncio

内置协程处理库，方便创建原生协程任务

常规的同步操作

```python
def sync_test():
    print('sync test')
    time.sleep(2)

if __name__ == '__main__':
    sync_test()
    sync_test()
```

函数执行是阻塞的，只有在第一个操作结束后系统才会调用第二个操作，密集IO中这将浪费大量等待时间

```python
import asyncio
import time

@asyncio.coroutine
def hello():
    print('hello world1')
    yield from asyncio.sleep(5)
    print('hello world2')

loop = asyncio.get_event_loop()
loop.run_until_complete(hello())
loop.close()
```

使用asyncio库的装饰器，装饰需要进行异步执行的操作函数，使用`yield from`关键字修饰**子生成器函数**，在这里修饰 的是内置的`asyncio.sleep(5)`。所有需要进行异步IO中断的操作都在这里修饰。

最后创建事件循环`asyncio.get_event_loop()`，运行创建的异步任务组，这里传入的是异步函数。也可以传入一个任务列表

```python
@asyncio.coroutine
def main():
    tasks = [task1(), task2()]  # 任务列表
    done,pending = yield from asyncio.wait(tasks) # 子生成器
    for r in done:
        print(r.result())
```

使用`asyncio.wait`接受需要执行的任务列表，并在整个事件循环里执行

最后执行完毕后，关闭事件循环`loop.close`，可以看到协程实际是单线程的任务

## async/await

- 把`@asyncio.coroutine`替换为`async`
- 把`yield from`替换为`await`

即可使用新的异步表示`async`和`await`

```python
async def func():
    await func2()
    return 'func1'

async def func2():
    await asyncio.sleep(3)
    return 'func2'

asyncio.run(func())
```

在真实环境下func2内部的异步等待函数应该换成编写的异步函数，异步函数的编写参考异步类的实现使用`__await__`属性编写。

未完...