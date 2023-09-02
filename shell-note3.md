---
title: shell学习笔记3
name: shell-note3
date: 2021-05-23 13:11:38
id: 0
tags: [shell,linux]
categories: [shell]
abstract: ""
---

`trap`命令和`watch`命令的使用

<!--more-->

# 使用背景

在使用shell编写动态程序的时候，我们可能会动态打印输出到控制台

使用`watch` 和`trap`可以使这个操作更加简单

## trap

trap是一个捕捉linux信号量的内建命令

**使用方式**

```bash
trap: usage: trap [-lp] [[arg] signal_spec ...]
```

`arg`可以使系统命令或者shell脚本

`signal_spec`为要监听的信号量，可以使多个组成数组

**支持的信号**

```bash
 $ trap -lp
 1) SIGHUP       2) SIGINT       3) SIGQUIT      4) SIGILL       5) SIGTRAP
 6) SIGABRT      7) SIGBUS       8) SIGFPE       9) SIGKILL     10) SIGUSR1
11) SIGSEGV     12) SIGUSR2     13) SIGPIPE     14) SIGALRM     15) SIGTERM
16) SIGSTKFLT   17) SIGCHLD     18) SIGCONT     19) SIGSTOP     20) SIGTSTP
21) SIGTTIN     22) SIGTTOU     23) SIGURG      24) SIGXCPU     25) SIGXFSZ
26) SIGVTALRM   27) SIGPROF     28) SIGWINCH    29) SIGIO       30) SIGPWR
31) SIGSYS      34) SIGRTMIN    35) SIGRTMIN+1  36) SIGRTMIN+2  37) SIGRTMIN+3
38) SIGRTMIN+4  39) SIGRTMIN+5  40) SIGRTMIN+6  41) SIGRTMIN+7  42) SIGRTMIN+8
43) SIGRTMIN+9  44) SIGRTMIN+10 45) SIGRTMIN+11 46) SIGRTMIN+12 47) SIGRTMIN+13
48) SIGRTMIN+14 49) SIGRTMIN+15 50) SIGRTMAX-14 51) SIGRTMAX-13 52) SIGRTMAX-12
53) SIGRTMAX-11 54) SIGRTMAX-10 55) SIGRTMAX-9  56) SIGRTMAX-8  57) SIGRTMAX-7
58) SIGRTMAX-6  59) SIGRTMAX-5  60) SIGRTMAX-4  61) SIGRTMAX-3  62) SIGRTMAX-2
63) SIGRTMAX-1  64) SIGRTMAX
```

**使用示例**

```bash
trap "echo exit" INT
```

监听INT信号，捕捉到此信号时打印`exit`

```bash
root@landers:/code# cat test.sh 
trap "echo catch sig-SIGINT" SIGINT
trap "echo catch sig-INT" INT
while [ 1 ]
do
echo "test is runnig "
sleep 1
done
```

运行结果如下

```bash
test is runnig 
test is runnig 
test is runnig 
^Ccatch sig-INT
test is runnig
```

可以看到使用ctrl + c尝试停止程序时会打印我们自定义的`catch sig-INT`

但是此时并不能退出进程，因为ctrl + c的默认行为已经被`trap`改变为我们自己定义的命令

**捕捉信号并退出**

```bash
trap "echo 'catch sig-INT';exit 0" INT
```

```bash
root@landers:/code# ./test.sh 
test is runnig 
test is runnig 
test is runnig 
test is runnig 
^Ccatch sig-INT
```

**位置**

`trap`可以在收到信号前的任意位置设置，并非需要在脚本的第一行

### 恢复信号

使用`trap - sig`或者`trap sig sig`可以恢复已经被自定义的信号量

```bash
$ trap - INT
```

### 恢复信号后退出进程

```bash
# 恢复后使用kill杀掉自身进程
trap "echo get signal;trap - INT;kill -s INT "$$" " INT
```

### 注意

子shell进程中无法继承trap自定义的信号

`trap`自定义的信号监听只能在`trap`所在的shell进程使用

## watch

watch是一个linux自带的命令作用是实时监控进程

### 使用

```bash
Usage:
 watch [options] command

Options:
  -b, --beep             beep if command has a non-zero exit
  -c, --color            interpret ANSI color and style sequences
  -d, --differences[=<permanent>]
                         highlight changes between updates
  -e, --errexit          exit if command has a non-zero exit
  -g, --chgexit          exit when output from command changes
  -n, --interval <secs>  seconds to wait between updates
  -p, --precise          attempt run command in precise intervals
  -t, --no-title         turn off header
  -x, --exec             pass command to exec instead of "sh -c"

 -h, --help     display this help and exit
 -v, --version  output version information and exit

For more details see watch(1).
```

`-n` 指定监控的时间间隔

`-c` 同时输出转义后的颜色控制符

`-t` 是否显示标题

示例

```bash
watch -n 1 "echo now: $(date)"

Every 1.0s: echo now: Sun May 23 17:10:44 CST 2021                          landers: Sun May 23 17:10:47 2021

now: Sun May 23 17:10:44 CST 2021
```

每隔1s watch会输出当前的时间

`watch`会以全屏的方式运行，并且数据实时刷新至终端上
