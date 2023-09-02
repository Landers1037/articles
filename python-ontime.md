---
title: python定时脚本
name: python-ontime
date: 2019-05-24
id: 0
tags: [python]
categories: []
abstract: ""
---


在服务器上运行python定时脚本
基于`python3.7  centos7.3`

<!--more-->

#### 定时脚本的写法

我是用`time`库和`schedule`库

```python
import schedule
import time
 
def job():
    print("hello world")
 
schedule.every(10).minutes.do(job)
schedule.every().hour.do(job)
schedule.every().day.at("10:30").do(job)
schedule.every(5).to(10).days.do(job)
schedule.every().monday.do(job)
schedule.every().wednesday.at("13:15").do(job)
 
while True:
    schedule.run_pending()
    time.sleep(1)
```

每隔十分钟执行一次任务

每隔一小时执行一次任务

每天的10:30执行一次任务

每隔5到10天执行一次任务 

每周一的这个时候执行一次任务

每周三13:15执行一次任务

run_pending：运行所有可以运行的任务

#### 让脚本运行在后台

```sh
python3 test.py & #运行在后台，退出终端即停止
nohup python3 test.py & #忽略远程连接终端的退出请求，一直在后台运行
```

#### 查看

`ps -aux`  查看全部后台

`ps -ef |grep python3` 查看python3的后台任务

#### 关闭脚本

`kill PID`  按照运行的PID关闭
`kill -9 PID`  强制关闭

参考原文：https://blog.csdn.net/liao392781/article/details/80521194 

