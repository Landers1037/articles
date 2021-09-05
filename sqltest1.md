---
title: 数据库实验1
name: sqltest1
date: 2019-03-27
id: 0
tags: [sql]
categories: []
abstract: ""
---


# 环境：

SQL server	Postgresql
<!--more-->


# 环境：

SQL server	Postgresql<!--more-->

## 实验一：（postgre）

根据供应商表完成数据库操作
建表

```python
import psycopg2
import sys
import json

class sql():
    def connect(self):
    #连接数据库
        self.db = psycopg2.connect(host='localhost',user='postgres',password='123456',database='db1',port='5432')
        self.cursor = self.db.cursor()

    def roll(self):
        self.db.rollback()

    def creat_table(self):
        self.connect()
        sql = 'CREATE TABLE IF NOT EXISTS \
            S(SNO VARCHAR (100) NOT NULL ,\
            SNAME VARCHAR (100) NOT NULL,\
            STATUS INT NOT NULL ,\
            CITY VARCHAR (100) NOT NULL,\
            PRIMARY KEY (SNO))'
        self.cursor.execute(sql)

        sql = 'create table if not exists \
            p(pno varchar (100) not null ,\
            pname varchar (100) not null ,\
            color varchar(100) not null ,\
            weight varchar (100) not null ,\
            primary key(pno))'
        self.cursor.execute(sql)

        sql = 'create table if not exists \
            j(jno varchar (100) not null ,\
            jname varchar (100) not null ,\
            city varchar (100) not null ,\
            primary key(jno))'
        self.cursor.execute(sql)

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

        self.db.commit()
```

输入数据

找出所有供应商的姓名和所在城市

```sql
select sname,city from s
```

所有零件的名称，颜色和重量

```sql
select pname,color,weight from p
```

S1提供的零件的工程号码

```sql
select jno from spj where sno='s1'
```

J2项目使用的各种零件的名称和重量

```sql
select pname,qty from spj,p where spj.pno=p.pno and spj.jno='j2'
```

上海厂商提供的所有零件号码

```sql
select pno from spj,s where spj.sno=s.sno and s.city='上海'
```

使用上海零件的工程名称

```sql
select jname from j,spj,s where spj.sno=s.sno and spj.jno=j.jno and s.sno='s5'
```

没有使用天津零件的工程号码

```sql
select jno from spj,s where spj.sno=s.sno and not s.city='天津'
```

把红色零件全部换为蓝色

```sql
update p set color='蓝' where color='红'
```

把由s5供给s4零件p6改为由s3提供

```sql
update spj set sno='s3' where sno='s5'and pno=’p6' and jno='j4'
```

从供应商关系中删除s2，并从供应情况关系中删除相应记录

```sql
delete from spj where sno='s2'
delete from s where sno='s2'
#涉及外键
```

插入数据

```sql
insert into spj (sno,jno,pno,qty)values ('s2','j4','p6','200')
```



## 实验二（sql server2012）

统计数学的成绩分布，按照分段统计

```sql
create procedure count
@hcname char(20)
as
select count(sno) as 优秀人数
from course,sc
where course.cno=sc.cno and cname=@hcname and grade between 90 and 100
select count(sno) as 良好人数
from course,sc
where course.cno=sc.cno and cname=@hcname and grade between 80 and 90
select count(sno) as 一般人数
from course,sc
where course.cno=sc.cno and cname=@hcname and grade between 70 and 80
select count(sno) as 及格人数
from course,sc
where course.cno=sc.cno and cname=@hcname and grade between 60 and 70
select count(sno) as 不及格人数
from course,sc
where course.cno=sc.cno and cname=@hcname and grade between 0 and 60
exec count "数学"
```

统计任意一门课平均成绩

```sql
create procedure avg
@hcourse char(20)
as declare @grade_avg float
begin
select cname,avg(grade*1.0)from sc,course where sc.cno=course.cno and @hcourse=course.cname group by cname
end
exec avg "数学"
```

将成绩从百分制变为ABCDE

```sql
create procedure abcde
as 
declare @sno char(10),@cno char(4),@grade smallint,@grade_char char(1)
declare cur_grade cursor for select sno,cno,grade
from sc
open cur_grade
fetch next from cur_grade into @sno,@cno,@grade
while @@FETCH_STATUS=0
begin
if @grade>90 and @grade<100 set @grade_char='A'
else if @grade>80 and @grade<=90 set @grade_char='B'
else if @grade>70 and @grade<=80 set @grade_char='C'
else if @grade>60 and @grade<=70 set @grade_char='D'
else if @grade<60 set @grade_char='E'
update sc
set newgrade=@grade_char
where sno=@sno and cno=@cno
fetch next from cur_grade into @sno,@cno,@grade
end
close cur_grade
deallocate cur_grade

exec abcde
```

