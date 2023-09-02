---
title: mysql的主从复制
name: mysql-slave
date: 2020-03-06
id: 0
tags: [mysql]
categories: []
abstract: ""
---


Mysql的主从复制

<!--more-->

### 测试用例

mariaDB 8.0 windows10

mysql 7.5 Ubuntu windows subsystem

## 原理

通过二进制文件记录的sql语句的执行记录，在slave服务器中再次执行一遍

## 主服务器

windows上的mariaDB，需要开启日志的记录

修改安装目录下的配置文件`my.ini`

```mysql
[mysqld]
...
port=3306
character-set-server=utf8
server-id=1
log-bin=D:/tmp/mysql
```

添加`server-id`这是每个服务器的唯一标识，`log-bin`是开启的日志所在位置，可以直接写`mysql-bin`保存在当前目录下

除此以外还可以添加其他参数

```mysql
binlog-do-db=xxx #指定同步的数据库，多个使用分号隔开
binlog-ignore-db=xxx #指定排除的数据库
```

修改完成后重新启动mysql服务

### 添加专用用户

添加一个专门用于主从复制的用户`slave`

```mysql
create user salve@localhost identified by '123456';
```

授予复制的权限

```mysql
grant replication slave on *.* to slave@localhost;
flush privileges
```

**一定要记得刷新表权限**

## 从服务器

修改`my.cnf`文件ini

```ini
[mysqld]
...
port=3307
server-id=2
```

因为两个数据库都在本机所以这里的端口不同，`server-id`一定要唯一

### 查看主服务器的log日志

使用`show master status;`

```mysql
MariaDB [(none)]> show master status;
+--------------+----------+--------------+------------------+
| File         | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+--------------+----------+--------------+------------------+
| mysql.000003 |      338 |              |                  |
+--------------+----------+--------------+------------------+
1 row in set (0.000 sec)
```

此时的`File`即是从服务器需要执行的二进制文件，`Position`即为开始执行的位置

### 从服务器执行二进制文件

```mysql
mysql> change master to master_host='localhost',
master_user='slave',
master_password='123456',
master_log_file='mysql.000003',
master_log_pos=338;

Query OK, 0 rows affected, 2 warnings (0.07 sec)
```

注意日志文件的位置和名称输入正确

然后启动从服务器

```mysql
mysql> start slave
Query OK, 0 rows affected (0.16 sec)
```

检查是否启动成功

```mysql
mysql> show slave status\G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: localhost
                  Master_User: pig
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql.000003
          Read_Master_Log_Pos: 338
               Relay_Log_File: skypig-relay-bin.000002
                Relay_Log_Pos: 449
        Relay_Master_Log_File: mysql.000003
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB:
          Replicate_Ignore_DB:
           Replicate_Do_Table:
       Replicate_Ignore_Table:
      Replicate_Wild_Do_Table:
  Replicate_Wild_Ignore_Table:
                   Last_Errno: 0
                   Last_Error:
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 338
              Relay_Log_Space: 657
              Until_Condition: None
               Until_Log_File:
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File:
           Master_SSL_CA_Path:
              Master_SSL_Cert:
            Master_SSL_Cipher:
               Master_SSL_Key:
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error:
               Last_SQL_Errno: 0
               Last_SQL_Error:
  Replicate_Ignore_Server_Ids:
             Master_Server_Id: 1
                  Master_UUID:
             Master_Info_File: /var/lib/mysql/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind:
      Last_IO_Error_Timestamp:
     Last_SQL_Error_Timestamp:
               Master_SSL_Crl:
           Master_SSL_Crlpath:
           Retrieved_Gtid_Set:
            Executed_Gtid_Set:
                Auto_Position: 0
         Replicate_Rewrite_DB:
                 Channel_Name:
           Master_TLS_Version:
1 row in set (0.00 sec)
```

找到 `Slave_IO_Running: Yes`和 `Slave_SQL_Running: Yes`看到从服务器运行成功

## 测试

在主服务器新建一个数据库`slave`

```mysql
MariaDB [(none)]>create database slave;
```

查看从服务器的数据库

```mysql
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| slave              |
| sys                |
+--------------------+
5 rows in set (0.00 sec)
```

看到`slave`已经成功同步过来

### 注意

因为从服务器只是用来同步数据，没有写入任务时为了安全应该使用读锁，禁止写

在my.cnf添加

```mysql
[mysqld]
read_only=1
```

对于在主从服务器配置前就已经存在的数据，不能通过日志方式恢复。可以使用执行sql文件恢复

使用mysqldump