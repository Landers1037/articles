---
title: Go使用linux信号量
name: go-linux-signal
date: 2020-10-31
id: 0
tags: [linux,go]
categories: []
abstract: ""
---


Go实现通过linux信号量通信

<!--more-->

## linux 信号

在linux上`kill`可以通过新开协程的方式发送信号量给进程

通过信号可以控制程序的运行

```bash
$ kill -l
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

使用`kill -l`可以查看当前系统的内置信号量，一般为0-64数字定义

常用的信号有如下：

| 信号编号 | 信号名称 | 信号含义               |
| -------- | -------- | ---------------------- |
| 1        | SIGHUP   | 挂起信号               |
| 2        | SIGINT   | 中断信号（同Ctrl + C） |
| 3        | SIGQUIT  | 退出信号（同Ctrl + \） |
| 9        | SIGKILL  | 杀死信号               |
| 11       | SIGSEGV  | 段错误信号             |
| 15       | SIGTERM  | 终止信号（默认）       |
| 18       | SIGCONT  | 继续运行信号           |
| 19       | SIGSTOP  | 暂停信号（同Ctrl + Z） |

`SIGUSR1`和`SIGUSR2`是用户定义的信号，在win平台没有这两个信号

### 发送信号

```bash
kill -2 2333
```

向PID 2333发送信号`SIGINT`

`kill` 后面的参数应该是进程PID或者job ID

使用`jobs -l`查看当前的任务ID

## Go使用信号

在Go里原生支持使用信号量

调用`syscall` 和 `os/signal`库

### channel

通道是Go用于解决和goroutine通信的数据结构

用于异步发送接收数据

```go
func main()  {
	c := make(chan int)
	defer close(c)
	go func() {
		var testInt = 10
		c <- testInt
	}()
	
	res := <- c
	fmt.Println(res)
}
```

创建一个int类型的通道，异步操作中向通道发送数字10

使用`<-`通道操作符从通道c获取内容，赋值给res

通道的操作符是`<-`，根据代码从左到右的习惯顺序

`<-c`是读取操作

`c<-`是写入操作

> 只读通道定义 `make(<-chan type)`
>
> 只写通道定义 `make(chan<- type)`

### 信号库

`syscall`库里定义了各个平台的信号量

`signal`库定义了信号的监听函数

```go
	c := make(chan os.Signal)
	defer close(c)

	signal.Notify(c, syscall.SIGINT)

	res := <- c
	fmt.Println(res)
```

使用`signal.Notify`注册监听信号，第一个参数是要监听的信号通道，后面是要监听的信号

运行程序，使用`CTRL+C`停止程序，可以看到接收到**interrupt**信号

### 使用select

go的`select`语句专用于从通道选择条件

类似以`switch`语句的通道专用版

```go
func test()  {
	c := make(chan os.Signal)
	defer close(c)

	signal.Notify(c, syscall.SIGINT)
	go func() {
		for {
			select {
			case <-c:
				fmt.Println(<-c)
				os.Exit(1)
			}
		}
	}()
	res := <- c
	fmt.Println(res)
}
```

使用select在协程中对信号监听，仅对c通道中注册的信号响应

### 使用switch

通过`switch`可以实现对不同信号区分

```go
func test()  {
	c := make(chan os.Signal)
	defer close(c)

	signal.Notify(c, syscall.SIGINT)
	go func() {
		for sig := range c {
			switch sig {
			case syscall.SIGINT:
				...
             case syscall.SIGHUP:
                 ...   
			}
		}
	}()
	res := <- c
	fmt.Println(res)
}
```

