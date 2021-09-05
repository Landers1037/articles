---
title: pymysql基础操作
name: pymysql1
date: 2019-02-18
id: 0
tags: [pymysql]
categories: []
abstract: ""
---


# pymysql操作

## 创建新的数据库

```python
sql = 'CREATE DATABASE spiders DEFAULT CHARACTER SET utf-8'
cursor.execute(sql)
```


<!--more-->


# pymysql操作

## 创建新的数据库

```python
sql = 'CREATE DATABASE spiders DEFAULT CHARACTER SET utf-8'
cursor.execute(sql)
```

<!--more-->

## 创建新的表

```python
# pymysql
import pymysql
# 前提 我已经建立好了一个名为spiders的数据库，里面有一个表为student
# connect database
db = pymysql.connect(host='localhost',user='root',password='123456',port=3306,db='spiders')
cursor = db.cursor() # 操作的游标
cursor.execute('select version()') # 执行sql语句的方法
data = cursor.fetchone() # 获取第一行的数据 就是版本号

print('版本号', data)
# 将爬虫爬取的租房信息存储到数据库中
sql = 'CREATE TABLE IF NOT EXISTS house (location VARCHAR(255) NOT NULL,detail VARCHAR (255) NOT NULL,price INT NOT NULL\
        ,PRIMARY KEY (location))'
cursor.execute(sql)
# 添加多条数据在数据库中
db.close()

```

建立后就会发现在spider数据库下多出了一张表
[![k6bQzT.png](https://s2.ax1x.com/2019/02/18/k6bQzT.png)](https://imgchr.com/i/k6bQzT)

```python
pymysql.err.InternalError: (1046, 'No database selected')
```

出现这种错误，表明本地没有这个数据库

