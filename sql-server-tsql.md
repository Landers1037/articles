---
title: sql server的tsql语句学习
name: sql-server-tsql
date: 2019-04-14
id: 0
tags: [sql]
categories: []
abstract: ""
---


# Tsql语句属于微软为sql server设计的高级sql语句

## 新建查询<!--more-->

## 变量赋值

```sql
declare @n int
set @n=5
print @n
```

声明变量n，用set方法赋值为5，打印

```sql
declare @name char,@age int
select @name='pig',@age=20
```

使用select语句可以对多个变量同时赋值

## 循环

```sql
declare @a int
declare @sum int
set @a=1 
set @sum=0 
while @a<=100 
begin
    set @sum+=@a 
    set @a+=1 
end
print @sum
```

计算1到100循环

## 条件语句

```sql
if(1+1=2) 
begin
    print '对'
end
else
begin
    print '错'
end
```

## 游标

```sql
declare @ID int
declare @Oid int
declare @Login varchar(50) 
  
--定义一个游标 
declare user_cur cursor for select ID,Oid,[Login] from ST_User 
--打开游标 
open user_cur 
while @@fetch_status=0 
begin
--读取游标 
    fetch next from user_cur into @ID,@Oid,@Login 
    print @ID 
    --print @Login 
end
close user_cur 
--摧毁游标 
deallocate user_cur
```

## 触发器

```sql
--创建触发器 
Create trigger User_OnUpdate  
    On ST_User  
    for Update 
As 
    declare @msg nvarchar(50) 
    --@msg记录修改情况 
    select @msg = N'姓名从“' + Deleted.Name + N'”修改为“' + Inserted.Name + '”' from Inserted,Deleted 
    --插入日志表 
    insert into [LOG](MSG)values(@msg) 
      
--删除触发器 
drop trigger User_OnUpdate
```

## 存储过程

```sql
CREATE PROCEDURE PR_Sum 
    @a int, 
    @b int, 
    @sum int output
AS
BEGIN
    set @sum=@a+@b 
END  
```

带有output参数

```sql
--创建Return返回值存储过程 
CREATE PROCEDURE PR_Sum2 
    @a int, 
    @b int
AS
BEGIN
    Return @a+@b 
END
```

```sql
--执行存储过程获取Return型返回值 
declare @mysum2 int
exec @mysum2= PR_Sum2 1,2 
print @mysum2
```

## 函数

```sql
--新建标量值函数 
create function FUNC_Sum1 
( 
    @a int, 
    @b int
) 
returns int
as
begin
    return @a+@b 
end
```

```sql
--新建内联表值函数 
create function FUNC_UserTab_1 
( 
    @myId int
) 
returns table
as
return (select * from ST_User where ID<@myId) 
```

```sql
--新建多语句表值函数 
create function FUNC_UserTab_2 
( 
    @myId int
) 
returns @t table
( 
    [ID] [int] NOT NULL, 
    [Oid] [int] NOT NULL, 
    [Login] [nvarchar](50) NOT NULL, 
    [Rtx] [nvarchar](4) NOT NULL, 
    [Name] [nvarchar](5) NOT NULL, 
    [Password] [nvarchar](max) NULL, 
    [State] [nvarchar](8) NOT NULL
) 
as
begin
    insert into @t select * from ST_User where ID<@myId 
    return
end
  
--调用表值函数 
select * from dbo.FUNC_UserTab_1(15) 
--调用标量值函数 
declare @s int
set @s=dbo.FUNC_Sum1(100,50) 
print @s 
  
--删除标量值函数 
drop function FUNC_Sum1
```

参考文章：[参考博客](https://www.cnblogs.com/chaoa/articles/3894311.html)

