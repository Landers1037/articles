---
title: subprocess使用
name: python-subprocess
date: 2020-02-23
id: 0
tags: [python]
categories: []
abstract: ""
---


使用`subprocess`执行命令

<!--more-->

linux的bash命令和windows的cmd命令执行方式差不多

假设有一个命令`ps ax`

```python
import subprocess
subprocess.run("ps ax")
```

使用`run`直接执行一条命令

在**windows**上要通过cmd运行指令，需要加上`shell`参数

```python
subprocess.run("dir",shell=True)
```

### Popen

如果需要执行多条指令，可以使用`Popen`创建子通道

```python
subprocess.Popen('pwd', shell=True)
subprocess.Popen('ls', shell=True)
```

在执行命令时，执行过程的输出也会被打印到python的终端上，如果需要隐藏过程输出

```python
subprocess.Popen('pwd', shell=True, stdout=subprocess.PIPE, stderr=subprocess.PIPE)
```

### 返回值

我们有时需要在执行命令后获得返回的结果

方法一：

```python
subprocess.getoutput(cmd)
```

方法二：

```python
p = subprocess.Popen('pwd', shell=True)
p.wait()
print(p.returncode)
print(p.poll())
```

使用`returncode`得到指令完成后的返回值，使用`poll()`可以判断指令是否执行完毕