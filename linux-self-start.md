---
title: linux编写自启动脚本
name: linux-self-start
date: 2019-06-10
id: 0
tags: [linux]
categories: []
abstract: ""
---


了解linux的init.d的作用，如何控制自启动脚本

<!--more-->

#### rc.local的作用

它是一个链接，作用是写在这个文件里的脚本内容，都会在开机的时候执行

我们进入`rc.d`（按照系统的启动级别）会看到系统定义的自启动链接，其中会有一个S9*链接指向`rc.local`文件，这个文件放置于init.d下，系统启动时会自动运行init.d下的可执行文件，于是会自启动`rc.local`里的全部脚本

#### 编写启动脚本

所有的自启动脚本都放在`init.d`文件夹下

```sh
cd /etc/init.d
nano test
```

```sh
#! /bin/sh
echo "test start successfully"
```

```sh
chmod 755 test
```

##### 启动级别

`sudo runlevel`获取系统的启动级别，假设为2

##### 创建链接

```sh
cd .etc/rc2.d
update-rc.d test start 98 2 . 
```

成功后，会在当前文件夹下看见`S98test`字样

##### 注意

多用户系统注意，使用`ll`先查看系统自启动的用户归属，然后`su user`切换到对应用户进行**创建链接**操作

##### 移除启动链接

```sh
update-rc.d -f test remove
```



#### init原理

最先加载内核
执行init程序
`/etc/rc.d/rc.sysinit` # 由init执行的第一个脚本
`/etc/rc.d/rc $RUNLEVEL` # $RUNLEVEL为缺省的运行模式
`/etc/rc.d/rc.local`     #相应级别服务启动之后、在执行该文件（其实也可以把需要执行的命令写到该文件中）
`/sbin/mingetty` # 等待用户登录

