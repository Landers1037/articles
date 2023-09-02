---
title: python获取系统状态信息
name: python-get-status
date: 2019-05-26
id: 0
tags: [python,linux]
categories: []
abstract: ""
---


使用python获取系统的状态信息
依赖`os`，`psutil`库

<!--more-->

```python
import psutil
#获取内存信息
mem = psutil.virtual_memory()
print(mem.total) #系统内存容量
print(mem.used) #使用的内存
print(mem.total-mem.used) #剩余可用内存

#cpu信息
cpu_status = psutil.cpu_times()
print(cpu_status.user) #用户使用情况
print(cpu_status.system)
print(cpu_status.idle)

#磁盘情况
disk_status = psutil.disk_partitions() #磁盘信息
#根据磁盘名称获取信息
for item in disk_status:
        p = item.device
        disk = psutil.disk_usage(p)
        disk.total #磁盘容量
        disk.used #使用容量
 
#当前用户信息
user_status = psutil.users()

#查看进程信息
pids = psutil.pids() #全部进程的pid
proces = []
for pid in pids: #单个进程
	p = psutil.Process(pid)
    p_info = [
            p.name(),       # 进程的名字
            p.exe(),        # 进程文件位置
            p.cwd(),        # 进程的工作目录的绝对路径
            p.status(),     # 进程的状态
            jctime,         # 进程的创建时间
            p.uids(),       # 进程的uid信息
            p.gids(),       # 进程的gid信息
            p.cpu_times(),  # cup时间信息
            p.memory_info(),# 进程内存的利用率
            p.io_counters() # 进程的io读写信息
        ]
        proces.append(p_info)
```

获取当前程序的pid

```python
p=psutil.Process(os.getpid())
```

以百分比显示资源占用情况

```python
print(str(psutil.virtual_memory().percent)+'%') #内存使用情况
print(str(psutil.cpu_percent(0))+'%') #cpu使用情况
```

