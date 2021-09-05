---
title: python的进度条
name: python-progress-bar
date: 2019-06-17
id: 0
tags: [python]
categories: []
abstract: ""
---


python实现在终端显示的进度条

**python3**


<!--more-->


python实现在终端显示的进度条

**python3**

<!--more-->

### 进度条

```python
import sys,time
for i in range(100):
    sys.stdout.write('>')
    sys.stdout.flush()
    time.sleep(0.1)
```

### 百分比

```python
import sys,time
for i in range(100):
    sys.stdout.write("\r%s%%"%(i+1))
    sys.stdout.flush()
    time.sleep(0.1)
```



### 带百分比进度条

```python
import sys,time
for i in range(100):
    k = i + 1
    str = '>'*i+' '*(100-k)
    sys.stdout.write('\r'+str+'[%s%%]'%(i+1))
    sys.stdout.flush()
    time.sleep(0.1)
```

```python
import time
import sys
for i in range(101):
    sys.stdout.write('\r')
    sys.stdout.write("%s%% |%s" %(int(i%101), int(i%101)*'#'))
    sys.stdout.flush()
    time.sleep(0.5)
```

