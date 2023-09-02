---
title: mysql最全的配置教程
name: mysql1
date: 2019-02-14
id: 0
tags: [mysql]
categories: []
abstract: ""
---


# 在安装mysql中遇到各种问题

## cmd输入密码无法连接数据库

```cmd
Enter password: ********
ERROR 1045 (28000): Access denied for user
```

<!--more-->
设置数据库的配置文件my.ini

```mysql
[mysql]
# 设置mysql客户端默认字符集
default-character-set=utf8 
[mysqld]
#设置3306端口
port = 3306 
# 设置mysql的安装目录
basedir=C:/Program Files/MySQL/MySQL Server 8.0
# 设置mysql数据库的数据的存放目录
datadir=C:/Program Files/MySQL/MySQL Server 8.0/data
# 允许最大连接数
max_connections=200
# 服务端使用的字符集默认为8比特编码的latin1字符集
character-set-server=utf8
# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB
#skip_grant_tables
```

结尾添加`skip_grant_tables`
重新打开cmd输入

```mysql
# 先关闭mysql服务
net stop mysql
mysql -u root -p
```

```mysql
# 提示输入密码直接按回车
# 使用sql语句
use mysql;
# 第一种重置密码的方法
UPDATE user SET password=PASSWORD(‘123456’)WHERE user=’root’; 
flush privileges;
# 第二种重置密码的方法
update user set authentication_string=password("123456") where user="root";
# 第三种重置密码的方法
ALTER USER 'root'@'localhost' IDENTIFIED BY '123456';
# 第四种重置密码的方法
SET PASSWORD = '123456';
```

设置成功后打开mysql服务

```mysql
net start mysql
```

## Navicat无法连接数据库

mysql5.8以上的版本因为密码的保存机制发生了变化所以不能使用传统的验证方式登录数据库

```mysql
mysql -u root -p
# 输入你的密码
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'password' PASSWORD EXPIRE NEVER;
Query OK, 0 rows affected (0.10 sec)
# 重置密码为123
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '123';
FLUSH PRIVILEGES; # 刷新权限
```

