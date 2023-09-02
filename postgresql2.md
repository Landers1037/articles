---
title: postgresql使用
name: postgresql2
date: 2019-03-12
id: 0
tags: [postgresql]
categories: []
abstract: ""
---


#### Python使用psycopg2库对postgresql数据库进行连接

连接数据库

```python
def connect(self):
    #连接数据库
	self.db = psycopg2.connect(host='localhost',user='postgres',password='123456',database='db1',port='5432')
    self.cursor = self.db.cursor()
```

## <!--more-->对数据库进行操作

### 建表

```python
sql = 'CREATE TABLE IF NOT EXISTS \
            S(SNO VARCHAR (100) NOT NULL ,\
            SNAME VARCHAR (100) NOT NULL,\
            STATUS INT NOT NULL ,\
            CITY VARCHAR (100) NOT NULL,\
            PRIMARY KEY (SNO))'
        self.cursor.execute(sql)
```

```python
sql = 'create table if not exists \
            spj(sno varchar (100)not null ,\
            pno varchar (100)not null ,\
            jno varchar (100) not null ,\
            qty varchar (100)not null ,\
            primary key(sno,pno,jno) ,\
            foreign key(sno)references S(sno),\
            foreign key(pno)references p(pno),\
            foreign key(jno)references j(jno))'
        self.cursor.execute(sql)
    # 同时指定外键 用于和其他的表建立参照完整性
```



### 查询

```python
sql = 'select sname,city from s'
self.cursor.execute(sql)
# 从表s中查询sname和city字段信息
```

```python
 sql = 'select jno from spj where sno=\'s1\''
 self.cursor.execute(sql)
# 查询符合sno=s1条件的字段信息
```

```python
sql = 'select pname,qty from spj,p where spj.pno=p.pno and spj.jno=\'j2\''
# 连接两个表，查询满足关系的字段，在where语句内要注意参照完整性的表达
```

```python
 sql = 'select jno from spj,s where spj.sno=s.sno and not s.city=\'天津\''
# 否定形式的语句使用not判断
```

### 提取

```python
result = self.cursor.fetchall()
        print(result)
    # 将执行后的sql返回值全部提取出来
```

### 更新

```python
sql = 'update s set color=\'蓝\' where color=\'红\''
self.cursor.execute(sql)
# 把s表内的color字段为红的全部修改为蓝
```

### 插入

```python
sql = 'insert into spj (sno,jno,pno,qty) values (\'s2\',\'j4\',\'p6\',\'200\')'
self.cursor.execute(sql)
# 插入语句注意，前面的字段和后面的数据是一一对应的，如果只是插入部分字段数据，一定要把字段和数据对应填写
```

### 删除

```python
sql = 'delete from s,spj where sno=\'s2\''
self.cursor.execute(sql)
# 删除s表中sno为s2的数据
```

所有的修改后都要执行 `db.commit()`数据库才会更新
当有错误发生时，使用 `db.rollback()`数据库回滚

