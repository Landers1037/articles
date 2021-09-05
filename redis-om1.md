---
title: redis使用备忘录
name: redis-om1
date: 2020-03-12
id: 0
tags: [linux]
categories: []
abstract: ""
---


Redis的使用备份记录


<!--more-->


Redis的使用备份记录

<!--more-->

Redis是一个高效的键值对非关系型数据库

### 数据类型

支持常见的数据类型，`string`字符串，`list`数组，`set`无序集合，`zset`有序集合，`hash`散列

这里很特殊，string是字符串安全的，可以存储byte类型的字符串。理解为可以存储二进制图片，文件的序列化数据

### 启动

```shell
D:\Landers pro\Redis-x64-3.2.100>redis-server.exe
[10248] 12 Mar 17:02:20.222 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server.exe /path/to/redis.conf
                _._
           _.-``__ ''-._
      _.-``    `.  `_.  ''-._           Redis 3.2.100 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 10248
  `-._    `-._  `-./  _.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |           http://redis.io
  `-._    `-._`-.__.-'_.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |
  `-._    `-._`-.__.-'_.-'    _.-'
      `-._    `-.__.-'    _.-'
          `-._        _.-'
              `-.__.-'

[10248] 12 Mar 17:02:20.227 # Server started, Redis version 3.2.100
[10248] 12 Mar 17:02:20.227 * DB loaded from disk: 0.000 seconds
[10248] 12 Mar 17:02:20.228 * The server is now ready to accept connections on port 6379
```



### 配置

默认的配置文件在安装目录下的`redis.conf`，可以通过`./redis-server -c xxx.conf`的方式指定配置

**redis**有很多配置参数，只记录正常的

| 参数                          | 内容                                           |
| ----------------------------- | ---------------------------------------------- |
| daemonize                     | 默认no，可填yes，是否使用守护进程              |
| port                          | 默认6379，运行的端口                           |
| bind                          | 默认127.0.0.1，设置为0.0.0.0以运行外部访问     |
| timeout                       | 客户端闲置时间，超过后关闭连接                 |
| databases                     | 数据库数量，默认0                              |
| save [seconds] [change]       | 在seconds内数据变化change次就写入到文件        |
| dbfilename [dump.rdb]         | 指定本地数据库名称                             |
| requirepass                   | 设置密码，默认为空                             |
| maxclients [0]                | 最大允许的客户端连接数                         |
| maxmemory [bytes]             | 允许使用的内存，超过后会清除过期键或只允许读   |
| appendonly no                 | 默认no，是否每次更新记录就持久化数据到本地文件 |
| appendfilename appendonly.aof | 本地数据的名称                                 |
| appendfsync everysec          | 同步方式no,always,everysec                     |
| include /path/to/local.conf   | 同时包含其他的配置文件                         |

### 主从配置

传统的主从配置，分别配置主数据库和从数据库

**主数据库** master.conf

```ini
bind 127.0.0.1
port 6379
```

**从服务器** slave.conf

```ini
bind 127.0.0.1
port 6380
slaveof 127.0.0.1 6379
slave-read-only yes
```

启动实例

```bash
./redis-server master.conf
###
./redis-server slave.conf
```

查看状态

```bash
 redis-cli -h 127.0.0.1 -p 6379 info Replication
```

### 集群配置

由多个redis实例组成的redis集群

创建多个实例的配置文件，在redis目录下新建一个`cluster`目录。假设集群有三台实例为`7000`,`7001`,`7002`。在cluster目录下新建对应的实例目录如`/cluster/7000`，该目录下创建配置文件`redis.conf`

```ini
#redis.conf
port 7000
bind 127.0.0.1
#作为守护进程运行
daemonize yes
pidfile /var/run/redis/redis-7000.pid
cluster-enabled yes
cluster-config-file /usr/local/redis/cluster/7000/7000.conf
#连接集群的超时时间
cluster-node-timeout 600
appendonly yes
logfile /usr/local/redis/cluster/7000/7000.log
```

其他的实例配置仅需修改端口和名称

可以使用redis自带的脚本`create-cluster`启动多个实例，需要修改`./util/create-cluster`里的port端口为自己设置的端口

或者使用`redis-cli --cluster create`命令启动集群

```bash
./redis-cli --cluster create 127.0.0.1:7000 127.0.0.1:7001 127.0.0.2:7002 --cluster-replicas 1
```

使用`./create-cluster stop`来停止集群

### 常用操作

**string**

APPEND,GET,SET,MGET,MSET,STRLEN

**list**

LPUSH,RPUSH,LPUSHX,RPUSHX,LSET,LREM,,LPOP,RPOP,LLEN,LINSERT

`LPUSH`从左端插入数据

`LPUSHX`从左端插入数据，如果该数组存在

`LREM`删除数据

`LSET`根据index索引设置数据

`RPOP`从尾部执行pop操作

`LLEN`得到数据长度

`LINSERT`插入到某个索引数据后

**set**

没用过

**database**

`DEL`删除某条数据

`EXISTS`判断是否存在键

`EXPIRES`设置键的过期时间

`KEYS`得到匹配的所有键

`PERSIST`永久存储没有过期

`RENAME`重命名键

`SCAN`遍历输出键

`TYPE`输出键类型

[官网命令](https://redis.io/commands)

### 压力测试

使用自带的benchmark工具

```bash
redis-benchmark -q -n 100000
```

静默请求100000次

指定操作

```bash
redis-benchmark -t set,lpush -n 100000 -q
```

