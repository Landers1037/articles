---
title: 数据库实验二
name: sqltest2
date: 2019-04-14
id: 0
tags: [sql]
categories: []
abstract: ""
---


## 使用Tsql语句描述汉诺塔

汉诺塔简介：[参考](https://blog.csdn.net/qq_37873310/article/details/80461767)

使用sql server的tsql语句 利用存储过程实现
<!--more-->


## 使用Tsql语句描述汉诺塔

汉诺塔简介：[参考](https://blog.csdn.net/qq_37873310/article/details/80461767)

使用sql server的tsql语句 利用存储过程实现<!--more-->

```sql
create procedure hannuo(@num int,@a char,@b char,@c char)
as declare
    @numm int
    select @numm=@num-1
    begin
        if @num=1
            begin
            print 'move'+convert(char,@num)+' from  '+@a+'  to  '+@c
            end
        else
            begin
            exec hannuo @numm,@a,@c,@b
            print 'move'+convert(char,@numm)+' from  '+@a+'  to  '+@c
            exec hannuo @numm,@b,@a,@c
			end
    end

exec hannuo 10 ,a,b,c
```

存储过程的变量分别为num盘子数目，abc为柱子的名称

## 使用Tsql语句描述猫鼠问题

猫鼠问题简介：在一个m\*n的方格中，老鼠位于最顶端的1\*1方格内，猫位于一个特定的方格内，出口在最右下角，老鼠每次移动一格，只能向右和向下移动，不能碰到猫。求解一共有多少条路径

```sql
create procedure mouse(@m int,@n int,@i int,@j int)
as declare @mm int ,@nn int,@l int,@r int
select @mm=@m-1,@nn=@n-1
    begin
        if @m=@i and @n=@j or @m=0 or @n=0
            begin
			return 0
            end
        else if @m=1 and @n=1
        begin
		print '找到一条路径'
            return 1
        end
    exec @l= mouse @mm,@n,@i,@j
	exec @r= mouse @m,@nn,@i,@j
	
	
	return  @l+@r
	
    end

exec mouse 5,3,2,2
```

存储过程的变量分别为mn方格的行列数，ij猫所在的位置，每次走到出口打印一次