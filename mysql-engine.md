---
title: mysql的engine
name: mysql-engine
date: 2020-03-11
id: 0
tags: [mysql]
categories: []
abstract: ""
---


MySQL的引擎，**InnoDB**和**MyIsam**的区别

<!--more-->

数据库引擎针对的是表，即每张表所使用的引擎

```mysql
MariaDB [blog]> show create table blog_tag;
+----------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Table    | Create Table
                                                                  |
+----------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| blog_tag | CREATE TABLE `blog_tag` (
  `id` int(11) NOT NULL,
  `name` varchar(255) NOT NULL,
  `time` varchar(255) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 |
+----------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.021 sec)
```

在`ENGINE`中的属性就是当前表使用的存储引擎

## InnoDB和Myisam

**支持事务**，每一句执行的sql语句都会封装成事务。如：

执行create table xxx;会封装为

```mysql
begin
create table xxx
commit
```

区别：**myisam**不支持事务

**支持外键**

区别：myisam不支持外键

**支持行锁**

使用聚集索引方式存放数据，数据和索引存储在一起

**myisam**使用非聚集索引，数据文件分开存放使用文件指针的方式索引

**myisam**支持全文索引而**innodb**不支持索引在查询大量数据时，**myisam**效率更高

### 使用场景

需要设计外键的时候，只能使用**innodb**，把原本为**innodb**引擎的表转为**myisam**时里面的外键会导致转换失败

使用事务时，只能使用**innodb**

由大量的读要求时使用**myisam**效率更高，同时读写时使用**innodb**

### 修改引擎

创建时

```mysql
create table xxx engine=xxx
```

修改时

```mysql
alter table xxx engine=xxx
```

