---
title: bash常用指令
name: bash-useful
date: 2020-03-03
id: 0
tags: [bash,linux]
categories: []
abstract: ""
---


常用的bash指令


<!--more-->


常用的bash指令

<!--more-->

### 帮助类指令

| 指令    | 作用                         | 参数            |
| ------- | ---------------------------- | --------------- |
| help    | shell内部帮助                |                 |
| whatis  | 指令的简介                   | -w 以“”为开头的 |
| info    | 指令详细介绍                 |                 |
| whereis | 指定程序的位置和源码所在位置 |                 |
| which   | 程序的位置                   |                 |

### 目录管理

文件的属性和文件权限，下表表示linux系统内部权限组成

| d开头或者_开头       | rwx            | rwx          | rwx            |
| -------------------- | -------------- | ------------ | -------------- |
| d表示目录，_表示文件 | 当前用户的权限 | 用户组的权限 | 其他用户的权限 |

除了`d`和`_`以外，还有`l`表示链接，`c`表示串口

一个用户可以属于多个用户组

### 文件权限和操作

| 指令   | 功能                             | 参数                                                         |
| ------ | -------------------------------- | ------------------------------------------------------------ |
| cd     | 进入目录                         |                                                              |
| ls     | 列出目录下的可见文件             | -l 详情<br />-la 详情和隐藏文件<br />-lh 文件大小<br />-lt 按照时间列出 -ltr按照修改时间列出 |
| pwd    | 绝对路径                         |                                                              |
| mkdir  | 创建目录                         | -p 创建带子目录的目录<br />-m 设置目录的权限                 |
| rmdir  | 删除目录                         |                                                              |
| tree   | 查看目录的树结构                 |                                                              |
| touch  | 更新文件时间为系统时间和新建文件 |                                                              |
| ln     | 创建链接                         | -s 创建软连接(软链接只是个符号需要外部依赖)                  |
| rename | 重命名和批量重命名               |                                                              |
| stat   | 状态信息比ls更加详细             |                                                              |
| file   | 显示文件的类型                   |                                                              |

#### 权限操作

根据二进制的对应关系**RWX**分别对应**421**

| 指令  | 功能             | 参数                                                         |
| ----- | ---------------- | ------------------------------------------------------------ |
| chmod | 设置权限         | 可以分用户设置u表示用户，g表示组，o表示其他<br />分用户来设置比如u=rwx g=rx o=x<br />全部用户用a表示如a+x<br />-R 表示目录下的全部文件 |
| chown | 设置所属用户和组 | 用法chown 用户 文件                                          |

#### 文件操作

| 指令   | 功能           | 参数                                        |
| ------ | -------------- | ------------------------------------------- |
| locate | 查找文件或目录 |                                             |
| find   | 指定目录下查找 | --name 文件名<br />不定查找 --name *.后缀名 |
| cp     | 复制           |                                             |
| scp    | 远程复制       |                                             |
| mv     | 移动文件       |                                             |
| rm     | 删除文件       | -r或者-R 删除目录下全部文件                 |

#### 查看与编辑

| 指令      | 功能             | 参数                                                         |
| --------- | ---------------- | ------------------------------------------------------------ |
| cat       | 显示文件内容     | 同时操作两个文件可以实现流处理，如cat m1 m2 > file           |
| head      | 显示头部10行     |                                                              |
| tail      | 显示尾部10行     |                                                              |
| more/less | 基于vi的文件浏览 |                                                              |
| vi        | vi编辑器         | i 插入， q 退出 ，w 保存 ，wq 保存退出 ，q!强制退出          |
| grep      | 正则表达式搜索   | grep "文件名" . 当前目录下搜索<br />grep --include 包含某些名称，如*.{html.css}<br />grep -- exclude 排除某些名称 |

### 压缩与解压

| 指令 | 功能                        | 参数                                                         |
| ---- | --------------------------- | ------------------------------------------------------------ |
| tar  | 打包文件(tar为文档类型文件) | -cvf 文件名 .tar 文件 只打包不压缩<br />-zcvf 打包为tar.gz<br />-jcvf 打包为bzip2<br />-zcvf 打包为zip |
| gzip | gzip压缩                    | -l 显示信息<br />-dv 解压<br />-r *.tar 压缩为tar文件        |
| zip  | 压缩                        |                                                              |
| uzip | 解压                        |                                                              |

### 用户管理

| 指令     | 功能                         | 参数                                                         |
| -------- | ---------------------------- | ------------------------------------------------------------ |
| groupadd | 添加用户组                   |                                                              |
| groupdel | 删除用户组                   |                                                              |
| groupmod | 修改用户组                   |                                                              |
| useradd  | 添加用户                     |                                                              |
| userdel  | 删除用户                     |                                                              |
| password | 修改当前用户密码             | 指定用户名时修改该用户的密码<br />-d 用户名 删除指定用户名密码 |
| su       | 切换用户                     |                                                              |
| sudo     | 指定用户运行指令，默认是root |                                                              |

### 系统管理

| 指令         | 功能              | 参数                                |
| ------------ | ----------------- | ----------------------------------- |
| reboot       | 重启              |                                     |
| shutdown     | 关机              | -h now 立刻关机<br />+5 5分钟后关机 |
| date         | 系统时间          |                                     |
| mount/unmout | 挂载分区/取消挂载 |                                     |
| ps           | 列出运行的程序    | -aux                                |
| systemctl    | 系统服务管理程序  |                                     |

### 网络管理

| 指令       | 功能                                                         |
| ---------- | ------------------------------------------------------------ |
| curl       | 内置的下载工具                                               |
| wget       | 下载工具                                                     |
| telnet     | 连接到远程服务器                                             |
| ip         | 查看网络配置                                                 |
| ipconfig   | 网卡信息                                                     |
| hostname   | 主机名称                                                     |
| route      | 查看路由表                                                   |
| traceroute | 查看全部路径上的路由表                                       |
| nslookup   | dns查看工具                                                  |
| netcat     | 设置路由器                                                   |
| ping       | 使用icmp协议的请求远程主机                                   |
| netstat    | 打印网络信息<br />-a 全部端口<br />-au 全部udp端口<br />-at 全部tcp端口<br />-l socket端口 |

### 硬件管理

| 指令  | 功能                 | 参数                                                         |
| ----- | -------------------- | ------------------------------------------------------------ |
| df    | 查看硬盘占用         | -h 以kb以上的大小显示<br />-a 列出文件系统                   |
| du    | 查看目录所占空间大小 |                                                              |
| top   | 系统整体状态         | -i 设置间隔时间<br />-p 指定pid<br />k 终止一个进程<br />q 退出<br />m 显示内存<br />c 显示完整的命令<br />M 按照内存大小显示<br />P 按照cpu排序显示 |
| free  | 内存占用             |                                                              |
| iotop | 查看系统io情况       |                                                              |

知识点：

load average是什么？

系统的负载，是对cpu工作量的度量。进程队列的长度，值的**最大值**是cpu的核心数，所以小于该最大值时表示系统运作正常，大于时表示系统的负载超出了cpu负载会造成拥堵

### 运维

linux的host文件位置 `/etc/host`

linux的dns位置 `/etc/resolv.conf`

自启动文件的位置`/etc/rc.d/init.d`下

添加自启动文件前 需要知道系统的运行级别，使用以下命令

```bash
runlevel
```

启动界别在0-6之前，0表示关机，6表示重启

使用`upgrade-rc.d`命令添加自启动

```bash
upgrade-rc.d 命令脚本 start 98(这是一个序号) 启动级别 .
```

注意最后的`.`不要忘记
