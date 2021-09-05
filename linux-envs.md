---
title: linux下的环境变量
name: linux-envs
date: 2021-07-22 22:59:37
tags: [linux]
categories: [linux]
abstract: 
---
linux下的`/etc/profile`,  `bashrc`, `bash_profile`

在linux环境下配置环境变量文件的作用和差异

<!--more-->

## /etc/profile

此文件为系统的每个用户设置环境信息，当用户第一次登录时，该文件被执行

修改此文件后，需要`source /etc/profile`以生效

```bash
root@ubuntu:/home/code# echo $NAME

# 修改/etc/profile后
cat /etc/profile

...
NAME=Landers

root@ubuntu:/home/code# source /etc/profile
root@ubuntu:/home/code# echo $NAME
Landers
```

## /etc/bashrc (/etc/bash.bashrc)

为每一个运行 bash shell 的用户执行此文件。当 bash shell 被打开时，该文件被读取

每次新开shell终端时会读取会生效

```bash
root@ubuntu:/home/code# echo $NAME

# 修改/etc/bash.bashrc
root@ubuntu:/home/code# echo 'export NAME=Landers' >> /etc/bash.bashrc
# 重开shell终端
ctrl + D
root@ubuntu:~# echo $NAME
Landers
```

## ~/.bash_profile (~/.profile)

每个用户都可使用该文件输入专用于自己使用的 shell 信息，当用户登录时，该文件仅仅执行一次

使用source 执行此文件才会生效

```bash
root@ubuntu:/home/code# echo $NAME

# 修改文件
root@ubuntu:~# echo 'export NAME=Landers' >> ~/.profile
root@ubuntu:~# source ~/.profile 
root@ubuntu:~# echo $NAME
Landers
```

## ~/.bashrc 

专用于当前shell的信息， 每个用户～目录都会存在此文件

在每次登录或者重新打开新shell时生效

```bash
root@ubuntu:~# echo $NAME

root@ubuntu:~# echo 'export NAME=Landers' >> .bashrc
# 退出
ctrl + D
root@ubuntu:~# echo $NAME
Landers
```

## ~/.bash_logout

每次退出系统（shell）时执行此文件

## 执行顺序

1. `/etc/profile`

2. `~/.bash_profile`或`~/.bash_login`或`~/.profile`

3. 一般会在`~/.bash_profile`中执行`~/.bashrc`

   >   在~/.bashrc中会先执行/etc/bashrc
   >
   > ```bash
   > # .bashrc
   > # User specific aliases and functions
   > alias rm='rm -i'
   > alias cp='cp -i'
   > alias mv='mv -i'
   > # Source global definitions
   > if [ -f /etc/bashrc ]; then
   >         . /etc/bashrc
   > fi
   > ```

4. `~/.bash_logout`

## sudo的局限

使用`sudo`执行脚本时往往会遇到变量不存在问题

```bash
root@ubuntu:~# cat test.sh 
echo $NAME
echo $PATH

root@ubuntu:~# bash test.sh

/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

### 尝试增加环境变量

```bash
root@ubuntu:~# bash test.sh 
Landers
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/home
```

可以看到此时的环境变量可以被直接获取

### 尝试使用sudo执行

```bash
root@ubuntu:~# sudo bash test.sh 

/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

可以看到刚才新增的环境变量全部为空

sudo运行时，会默认重置环境变量为安全的环境变量，当前环境设置的变量都会失效，只有少数配置文件中指定的环境变量能保存下来。

sudo的配置文件是 `/etc/sudoers` 需要root权限才能读取:

### 通过`sudo -l`查看sudo的限制

```bash
root@ubuntu:~# sudo -l
Matching Defaults entries for root on ubuntu:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User root may run the following commands on ubuntu:
    (ALL : ALL) ALL
```

可以看到`env_reset`默认执行会将环境变量重置

> 特殊的地方echo
>
> ```bash
> root@ubuntu:~# sudo echo $NAME
> Landers
> ```
>
> 使用echo环境变量却是可以读取的，因为shell命令行的替换&重组功能
>
> shell会先依据分隔符将命令行切割成字段，对每个字段查找有没有变量或命令替换，再替换完成后，重组成新的命令，再去执行
>
> 所以重组后的脚本就是`echo $NAME`

### 使用`-E`参数保留环境变量

```bash
root@ubuntu:~# sudo -E bash test.sh 
Landers
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/home
```

可以看到现在环境变量正常显示

### 修改sudo配置

可以使用sudo visudo的方式修改默认的sudo配置

```bash
root@ubuntu:~# sudo visudo
# Defaults env_reset # 注释此句不再每次重置
# Defaults env_keep="" # 加入需要保留的环境变量  
```

使用`Defaults !env_reset` 同样可以取消重置环境变量