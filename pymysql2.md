---
title: pymysql操作表插入数据
name: pymysql2
date: 2019-02-21
id: 0
tags: [pymysql,python]
categories: []
abstract: ""
---


# 字符串操作基础

## join()方法

```python
str.join(data) # 常用来拼接，指定data数据用什么str连接
data = {"1","2","3"}
str = ','
print(str.join(data))
# 输出结果应为1,2,3
```

## 
<!--more-->


# 字符串操作基础

## join()方法

```python
str.join(data) # 常用来拼接，指定data数据用什么str连接
data = {"1","2","3"}
str = ','
print(str.join(data))
# 输出结果应为1,2,3
```

## <!--more-->format()方法

```python
# 用来格式化字符串
{0}{1}{0}.format('hello','world')
# 结果为hello world hello
```

```python
# 用来传入参数
str = '名字:{name},年龄:{age}'format(name='landers',age='2')
```

## tuple()方法

```python
# 操作列表转化为元组
list = [1,2,3,4]
m = tuple(data)
print(m) # (1,2,3,4)

dict = {1:2,3:4}
m = tuple(dict)
# (1,3) 返回key值
```



# Mysql插入数据

## 使用传统插入方法

```python
import pymsql
code = [{
        'name':'github',
        'count':'landers1037',
        'pass':'2333'
    }]
# sql = 'CREATE TABLE IF NOT EXISTS code(名称 VARCHAR(100) NOT NULL,账户 VARCHAR(100) NOT NULL,密码 VARCHAR(100) NOT NULL\
#     ,PRIMARY KEY (名称))'
# cursor.execute(sql)
print('database created')
sql = 'insert into code(名称, 账户, 密码) values (%s,%s,%s)'
try:
    for i in range(0,len(code)):
        data = code[i]
        # print(data)
        cursor.execute(sql,(data['name'],data['count'],data['pass']))
        db.commit() # 这句话别忘了
except Exception as e:
    print(e.args)
    db.rollback()
db.close()
```

以上方法是把一个元组传入到数据库中这样适合数据格式固定的数据存储

## 使用字典动态传入数据

```python
data  = {
    'id':'201901',
    'name':'landers',
    'age':'2'
}
table = 'student'
keys = ','.join(data.keys())
values = ','.join([%s]*len(data))

sql = 'insert into {table}({keys}) values({values})'.format(table=table,keys=keys,values=values)
cursor.execute(sql,tuple(data.values()))
db.commit()
db.close()
```

### 针对mysql的禁止数据重复的报错

```mysql
insert into # 传统插入 不能插入重复的数据
insert ignore into # 忽略重复插入
insert into ***表 键 值***on duplicate key update # 如果主键存在就更新里面的数据
replace into # 如果数据之前存在就替换
```

