---
title: go内存泄漏分析
name: go-memory-leak
date: 2022-07-13 14:29:51
tags: [go]
categories: [go]
abstract: 
---
Go程序的内存泄漏分析

排查工具：`pprof`

场景：

- 临时内存泄漏
- 永久内存泄漏（无法gc）

**临时内存泄漏**

- 切片类操作

**永久内存泄漏**

- goroutine泄漏
- Ticker泄漏
- defer泄漏
- cgo泄漏

<!--more-->

## 切片操作

### 新切片引用原切片的部分

```go
var s []int
// 从数组中获取一段slice 因为共用底层内存导致array无法释放
func exampleSlice(sl []int) {
	s = sl[len(sl)-10:]
	fmt.Println(s)
}

func ExampleSlice() {
	var testArray = [10000000]int{}
	for i, _ := range testArray {
		testArray[i] = 10086
	}
	time.Sleep(5 * time.Second)
	exampleSlice(testArray[:])
}
```

s为外部定义的全局切片，`exampleSlice`传入一个切片后，将s1的部分数据赋值给s。

因为s被全局使用，容量未增加时切片赋值的底层是共用的同一切片指针，导致gc不会回收超大切片`testArray[:]`

测试函数调用前

![img](/images/go-leak-example-1.jpg)

测试函数调用后

![img](/images/go-leak-example-2.jpg)

解决方法

在函数内部对切片数据进行复制

```go
func exampleSlice(sl []int) {
	s = append([]int(nil), sl[len(sl)-10:]...)
	fmt.Println(s)
}
```

将指定长度的数据从切片`sl`中复制到`s`中，保证在`sl`在函数被调用后会被gc回收

测试函数调用前

![img](/images/go-leak-example-3.jpg)

测试函数调用后

![img](/images/go-leak-example-4.jpg)

`sl`切片的内存被正确gc回收

### string导致的泄漏

`string`可以看作只读的没有`cap`属性`slice`切片

所以如果对string切片的部分取值并赋值给新的string也会导致泄漏

```go
var ss string
func exampleString(str string) {
	ss = str[len(str)-5:]
}

func ExampleString() {
	var testStr = "qqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqq"
	time.Sleep(5 * time.Second)
	exampleString(testStr)
}
```

测试函数调用前

![img](/images/go-leak-example-5.jpg)

测试函数调用后

![img](/images/go-leak-example-6.jpg)

解决方案

同切片的内存共用处理相似

```go
func exampleString(str string) {
	ss = string([]byte(str[len(str)-5:]))
}
```

## 永久泄漏

### goroutine泄漏

创建一个channel的时候，如果不能保证channel正确接收或发送数据，可能导致内存泄漏

生产者消费者模型下，如果channel一直生产未被消费，就会导致生产者阻塞

```go
// 生产者
func produce(ch chan int) {
	go func() {
		ch <- 1
		fmt.Println("produce")
		time.Sleep(100)
	}()
}

func consume(ch chan int) {
	go func() {
		res := <-ch
		fmt.Println("consume", res)
	}()
}

func ExampleGoroutine() {
	time.Sleep(5 * time.Second)
	var ch = make(chan int)
	go func() {
		for {
			produce(ch)
		}
	}()
	
	consume(ch)
}
```

作为生产者，通道一直在发送数据，但是消费者只接受一次通道数据后就不再接收，导致`ch`通道发送阻塞

函数调用内存占用

![img](/images/go-leak-example-7.jpg)

可以看到因为通道的阻塞，内存显著增长

解决方案

增加上下文，长时间未响应时关闭通道

```go
func produce(ch chan int) {
	ctx, cancel := context.WithTimeout(context.Background(), time.Second*2)
	go func() {
		ch <- 1
		fmt.Println("produce")
		time.Sleep(100)
	}()

	select {
	case <-ctx.Done():
		fmt.Println("timeout")
		cancel()
	}
}
```



### defer导致的内存泄漏

`defer`使用中的陷阱，`defer close()`

defer保证在函数调用最后出栈，最后被调用，一般用来进行一些IO关闭的操作，保证句柄被关闭

但是在循环中调用defer时 因为for循环没有返回，所以不会触发defer被调用

```go
func deferClose() {
	for {
		f, _ := os.Open("README.md")
		defer f.Close()
		time.Sleep(10)
	}
}
func ExampleDefer() {
	time.Sleep(time.Second * 5)
	deferClose()
}
```

![](/images/go-leak-example-8.jpg)

随着循环次数增加，文件句柄被不断创建，但是`defer`不会被执行导致内存不断增长

解决方案

尽量不要再for循环内写defer

```go
func deferClose() {
	for {
		f, _ := os.Open("README.md")
		_ = f.Close()
		time.Sleep(10)
	}
}
func ExampleDefer() {
	time.Sleep(time.Second * 5)
	deferClose()
}
```



### http body的泄漏

go的http库被设计时的一个很特殊的地方就是响应body体的读取

body体被读取后，必须显式的关闭，否则会导致泄漏

```go
func readBody() {
	for _ = range [1000]int{} {
		res, err := http.Get("http://api.renj.io")
		if err != nil {
			fmt.Println(err)
			return
		}
		data, _ := ioutil.ReadAll(res.Body)
        // res.body.close()
		fmt.Println(len(data))
		time.Sleep(10 * time.Millisecond)
	}
}

func ExampleHTTP() {
	readBody()
}
```

在读取body后需要调用`res.body.close()`以关闭流

未关闭流时，内存不断增长

![img](/images/go-leak-example-9.jpg)

关闭流后，在请求完毕，内存回收

![img](/images/go-leak-example-10.jpg)

### ticker泄漏

创建大量毫秒级定时器时会占用大量cpu内存，需要做到随用随关闭

```go
func tickerRun() {
	for _ = range [100000]int{} {
		ticker := time.NewTicker(1 * time.Millisecond)
		//defer ticker.Stop()
		fmt.Println(ticker)
	}
}

func ExampleTicker() {
	tickerRun()
}
```

![img](/images/go-leak-example-11.jpg)

保证定时器关闭后

![img](/images/go-leak-example-12.jpg)

## 使用pprof分析

引入go的pprof分析由go运行时管理的资源

```go
import _ "net/http/pprof"

func main() {
	http.ListenAndServe(":8800", nil)
}
```

访问[http://localhost:8800/debug/pprof](http://localhost:8800/debug/pprof)即可进入pprof分析页面

![img](/images/go-leak-example-13.jpg)

对heap进行分析

```bash
➜  ~ go tool pprof http://192.168.100.10:8899/debug/pprof/heap 
Fetching profile over HTTP from http://192.168.100.10:8899/debug/pprof/heap
Saved profile in /root/pprof/pprof.go-leak-example.alloc_objects.alloc_space.inuse_objects.inuse_space.001.pb.gz
File: go-leak-example
Type: inuse_space
Time: Jul 13, 2022 at 7:05pm (UTC)
Entering interactive mode (type "help" for commands, "o" for options)
(pprof) top
Showing nodes accounting for 1024.24kB, 100% of 1024.24kB total
      flat  flat%   sum%        cum   cum%
  512.20kB 50.01% 50.01%   512.20kB 50.01%  runtime.malg
  512.04kB 49.99%   100%   512.04kB 49.99%  runtime.bgscavenge
         0     0%   100%   512.20kB 50.01%  runtime.newproc.func1
         0     0%   100%   512.20kB 50.01%  runtime.newproc1
         0     0%   100%   512.20kB 50.01%  runtime.systemstack
(pprof) 
```

使用`top`查看当前内存堆栈中占用最高的函数调用

以slice内存泄漏为例分析

源码

```go
var s []int

// 从数组中获取一段slice 因为共用底层内存导致array无法释放
func exampleSlice(sl []int) {
	//s = sl[len(sl)-10:]
	s = append([]int(nil), sl[len(sl)-10:]...)
	fmt.Println(s)
}

func ExampleSlice() {
	var testArray = [10000000]int{}
	for i, _ := range testArray {
		testArray[i] = 10086
	}
	time.Sleep(5 * time.Second)
	exampleSlice(testArray[:])
}
```



```bash
➜  ~ go tool pprof http://192.168.100.10:8899/debug/pprof/heap
Fetching profile over HTTP from http://192.168.100.10:8899/debug/pprof/heap
Saved profile in /root/pprof/pprof.go-leak-example.alloc_objects.alloc_space.inuse_objects.inuse_space.002.pb.gz
File: go-leak-example
Type: inuse_space
Time: Jul 13, 2022 at 7:07pm (UTC)
Entering interactive mode (type "help" for commands, "o" for options)
(pprof) top
Showing nodes accounting for 76.80MB, 100% of 76.80MB total
      flat  flat%   sum%        cum   cum%
   76.30MB 99.35% 99.35%    76.30MB 99.35%  main.ExampleSlice
    0.50MB  0.65%   100%     0.50MB  0.65%  runtime.malg
         0     0%   100%    76.30MB 99.35%  main.runExample
         0     0%   100%     0.50MB  0.65%  runtime.newproc.func1
         0     0%   100%     0.50MB  0.65%  runtime.newproc1
         0     0%   100%     0.50MB  0.65%  runtime.systemstack
(pprof) 
```

使用`tree`查看调用的关系树

```bash
(pprof) tree
Showing nodes accounting for 76.80MB, 100% of 76.80MB total
----------------------------------------------------------+-------------
      flat  flat%   sum%        cum   cum%   calls calls% + context              
----------------------------------------------------------+-------------
                                           76.30MB   100% |   main.runExample
   76.30MB 99.35% 99.35%    76.30MB 99.35%                | main.ExampleSlice
----------------------------------------------------------+-------------
                                            0.50MB   100% |   runtime.newproc1
    0.50MB  0.65%   100%     0.50MB  0.65%                | runtime.malg
----------------------------------------------------------+-------------
         0     0%   100%    76.30MB 99.35%                | main.runExample
                                           76.30MB   100% |   main.ExampleSlice
----------------------------------------------------------+-------------
                                            0.50MB   100% |   runtime.systemstack
         0     0%   100%     0.50MB  0.65%                | runtime.newproc.func1
                                            0.50MB   100% |   runtime.newproc1
----------------------------------------------------------+-------------
                                            0.50MB   100% |   runtime.newproc.func1
         0     0%   100%     0.50MB  0.65%                | runtime.newproc1
                                            0.50MB   100% |   runtime.malg
----------------------------------------------------------+-------------
         0     0%   100%     0.50MB  0.65%                | runtime.systemstack
                                            0.50MB   100% |   runtime.newproc.func1
----------------------------------------------------------+-------------
(pprof) 
```

可以看到占用最高的函数就是测试泄漏的函数`ExampleSlice`

可以对函数进行更加细致的分析

```bash
(pprof) list main.ExampleSlice
Total: 76.80MB
ROUTINE ======================== main.ExampleSlice in /code/go-leak-example/slice_leak.go
   76.30MB    76.30MB (flat, cum) 99.35% of Total
         .          .     23:   s = append([]int(nil), sl[len(sl)-10:]...)
         .          .     24:   fmt.Println(s)
         .          .     25:}
         .          .     26:
         .          .     27:func ExampleSlice() {
   76.30MB    76.30MB     28:   var testArray = [10000000]int{}
         .          .     29:   for i, _ := range testArray {
         .          .     30:           testArray[i] = 10086
         .          .     31:   }
         .          .     32:   time.Sleep(5 * time.Second)
         .          .     33:   exampleSlice(testArray[:])
(pprof) 
```

pprof会标识出函数所在行占用的内存， 可以看到占用最高的行`var testArray = [10000000]int{}`就是我们定义的超大切片

## cgo泄漏

go的内存状态可以通过`runtime.ReadMemStats()`获取

它返回了一个内存结构体

```go
type MemStats struct {
    Alloc         uint64
    TotalAlloc    uint64
    Sys           uint64
    Lookups       uint64
    Mallocs       uint64
    Frees         uint64
    HeapAlloc     uint64
    HeapSys       uint64
    HeapIdle      uint64
    HeapInuse     uint64
    HeapReleased  uint64
    HeapObjects   uint64
    StackInuse    uint64
    StackSys      uint64
    MSpanInuse    uint64
    MSpanSys      uint64
    MCacheInuse   uint64
    MCacheSys     uint64
    BuckHashSys   uint64
    GCSys         uint64
    OtherSys      uint64
    NextGC        uint64
    LastGC        uint64
    PauseTotalNs  uint64
    PauseNs       [256]uint64
    PauseEnd      [256]uint64
    NumGC         uint32
    NumForcedGC   uint32
    GCCPUFraction float64
    EnableGC      bool
    DebugGC       bool
    BySize        [61]struct {
        Size    uint32
        Mallocs uint64
        Frees   uint64
    }
}
```

内存数据结构只记录go运行时维护的gc数据，对于cgo应用，c代码中的数据如何回收由c处理，go运行时中不会记录这部分数据的状态

所以使用`pprof`只能分析go应用的内存

一旦一个变量在c代码中初始化后未回收内存，就会导致内存泄漏

## cgo内存泄漏分析

以简单的`cgo`为例

```go
/*
#include <stdio.h>
void hello(const char *s) {
	puts(s);
}
*/
import "C"
import (
	"strconv"
	"time"
)

func ExampleCgo() {
	for i := range [100000]int{} {
		str := C.CString("index:" + strconv.Itoa(i))
		C.hello(str)
		time.Sleep(1 * time.Millisecond)
	}
}
```

使用`C.CString`创建一个`C`对象的string，这个对象的内存分配在c上完成

连续创建此对象，使它一直在C上分配内存，此时go的gc不会去回收c层面的内存

```bash
index:99947
index:99948
index:99949
index:99950
index:99951
index:99952
index:99953
index:99954
index:99955
```

![img](/images/go-leak-example-14.jpg)

程序运行结束后，内存增加并没有回收

调用`free`手动释放内存

```go
/*
#include <stdio.h>
#include <stdlib.h>
void hello(const char *s) {
	puts(s);
}
*/
import "C"
import (
	"strconv"
	"time"
	"unsafe"
)

func ExampleCgo() {
	for i := range [100000]int{} {
		str := C.CString("index:" + strconv.Itoa(i))
		C.hello(str)
		C.free(unsafe.Pointer(str))
		time.Sleep(1 * time.Millisecond)
	}
}
```

程序运行结束后，内存回收

![img](/images/go-leak-example-15.jpg)

### 尝试使用pprof

```bash
➜  ~ go tool pprof http://192.168.100.10:8899/debug/pprof/heap
Fetching profile over HTTP from http://192.168.100.10:8899/debug/pprof/heap
Saved profile in /root/pprof/pprof.go-leak-example.alloc_objects.alloc_space.inuse_objects.inuse_space.003.pb.gz
File: go-leak-example
Build ID: 7fcd93e8e325230e3f63175fc1af812a10f3b99c
Type: inuse_space
Time: Jul 13, 2022 at 7:34pm (UTC)
Entering interactive mode (type "help" for commands, "o" for options)
(pprof) top
Showing nodes accounting for 512.20kB, 100% of 512.20kB total
      flat  flat%   sum%        cum   cum%
  512.20kB   100%   100%   512.20kB   100%  runtime.malg
         0     0%   100%   512.20kB   100%  runtime.newproc.func1
         0     0%   100%   512.20kB   100%  runtime.newproc1
         0     0%   100%   512.20kB   100%  runtime.systemstack
(pprof) 
```

只能看到go运行时调用部分

## valgrind工具

linux上的内存分析工具

[下载](https://valgrind.org/downloads/current.html#current)

```bash
cd valgrind-3.19.0
./autogen.sh

running: aclocal
running: autoheader
running: automake -a
running: autoconf

./configure
make && make install
```

编译安装完毕

```bash

                    Version: 3.19.0
         Maximum build arch: amd64
         Primary build arch: amd64
       Secondary build arch: 
                   Build OS: linux
     Link Time Optimisation: no
       Primary build target: AMD64_LINUX
     Secondary build target: 
           Platform variant: vanilla
      Primary -DVGPV string: -DVGPV_amd64_linux_vanilla=1
         Default supp files: ./xfree-3.supp ./xfree-4.supp glibc-2.X-drd.supp glibc-2.X-helgrind.supp glibc-2.X.supp
         
➜  valgrind-3.19.0 valgrind       
valgrind: no program specified
valgrind: Use --help for more information.         
```

检查可能发生内存泄漏的程序

```bash
➜  valgrind-3.19.0 valgrind --tool=memcheck --leak-check=full --show-reachable=yes /code/go-leak-example/go-leak-example -r cgo
==18063== Memcheck, a memory error detector
==18063== Copyright (C) 2002-2022, and GNU GPL'd, by Julian Seward et al.
==18063== Using Valgrind-3.19.0 and LibVEX; rerun with -h for copyright info
==18063== Command: /code/go-leak-example/go-leak-example -r cgo
==18063== 
==18063== Warning: set address range perms: large range [0xa054000, 0x2a054000) (noaccess)
==18063== Warning: ignored attempt to set SIGRT32 handler in sigaction();
==18063==          the SIGRT32 signal is used internally by Valgrind
==18063== Warning: ignored attempt to set SIGRT32 handler in sigaction();
==18063==          the SIGRT32 signal is used internally by Valgrind
==18063== Warning: client switching stacks?  SP change: 0x1fff0001c8 --> 0xc0000347d8
==18063==          to suppress, use: --max-stackframe=687211759120 or greater
==18063== Warning: client switching stacks?  SP change: 0xc000034778 --> 0x1fff000268
==18063==          to suppress, use: --max-stackframe=687211758864 or greater
==18063== Warning: client switching stacks?  SP change: 0x1fff000268 --> 0xc000034778
==18063==          to suppress, use: --max-stackframe=687211758864 or greater
==18063==          further instances of this message will not be shown.
==18063== Conditional jump or move depends on uninitialised value(s)
==18063==    at 0x44F7B6: runtime.adjustframe (stack.go:562)
==18063==    by 0x45B962: runtime.gentraceback (traceback.go:350)
==18063==    by 0x44FD52: runtime.copystack (stack.go:918)
==18063==    by 0x4502BA: runtime.newstack (stack.go:1097)
==18063==    by 0x466BAA: runtime.morestack.abi0 (asm_amd64.s:461)
==18063==    by 0x46B1C8: runtime.newproc.abi0 (<autogenerated>:1)
==18063==    by 0x67C90F: ??? (in /mnt/hgfs/code/go-leak-example/go-leak-example)
==18063== 
==18063== Conditional jump or move depends on uninitialised value(s)
==18063==    at 0x44F7BC: runtime.adjustframe (stack.go:562)
==18063==    by 0x45B962: runtime.gentraceback (traceback.go:350)
==18063==    by 0x44FD52: runtime.copystack (stack.go:918)
==18063==    by 0x4502BA: runtime.newstack (stack.go:1097)
==18063==    by 0x466BAA: runtime.morestack.abi0 (asm_amd64.s:461)
==18063==    by 0x46B1C8: runtime.newproc.abi0 (<autogenerated>:1)
==18063==    by 0x67C90F: ??? (in /mnt/hgfs/code/go-leak-example/go-leak-example)
==18063== Process terminating with default action of signal 2 (SIGINT)
==18063==    at 0x46A521: runtime.raise.abi0 (sys_linux_amd64.s:165)
==18063== 
==18063== HEAP SUMMARY:
==18063==     in use at exit: 2,176 bytes in 5 blocks
==18063==   total heap usage: 2,118 allocs, 2,113 frees, 24,406 bytes allocated
==18063== 
==18063== 288 bytes in 1 blocks are possibly lost in loss record 1 of 5
==18063==    at 0x4C37BFD: calloc (vg_replace_malloc.c:1328)
==18063==    by 0x4013646: allocate_dtv (dl-tls.c:286)
==18063==    by 0x4013646: _dl_allocate_tls (dl-tls.c:530)
==18063==    by 0x4E4C227: allocate_stack (allocatestack.c:627)
==18063==    by 0x4E4C227: pthread_create@@GLIBC_2.2.5 (pthread_create.c:644)
==18063==    by 0x67C050: _cgo_try_pthread_create (gcc_libinit.c:100)
==18063==    by 0x67C24E: _cgo_sys_thread_start (gcc_linux_amd64.c:73)
==18063==    by 0x4689AC: runtime.asmcgocall.abi0 (asm_amd64.s:795)
==18063==    by 0x1FFF0001A7: ???
==18063==    by 0x43D0E6: runtime.newm (proc.go:2230)
==18063==    by 0x4626E8: runtime.main.func1 (proc.go:175)
==18063==    by 0x466AC8: runtime.systemstack.abi0 (asm_amd64.s:383)
==18063==    by 0x46B1C8: runtime.newproc.abi0 (<autogenerated>:1)
==18063==    by 0x67C90F: ??? (in /mnt/hgfs/code/go-leak-example/go-leak-example)
==18063== 
==18063== 288 bytes in 1 blocks are possibly lost in loss record 2 of 5
==18063==    at 0x4C37BFD: calloc (vg_replace_malloc.c:1328)
==18063==    by 0x4013646: allocate_dtv (dl-tls.c:286)
==18063==    by 0x4013646: _dl_allocate_tls (dl-tls.c:530)
==18063==    by 0x4E4C227: allocate_stack (allocatestack.c:627)
==18063==    by 0x4E4C227: pthread_create@@GLIBC_2.2.5 (pthread_create.c:644)
==18063==    by 0x67C050: _cgo_try_pthread_create (gcc_libinit.c:100)
==18063==    by 0x67C24E: _cgo_sys_thread_start (gcc_linux_amd64.c:73)
==18063==    by 0x4689AC: runtime.asmcgocall.abi0 (asm_amd64.s:795)
==18063==    by 0x40ECC6: runtime.newobject (malloc.go:1228)
==18063==    by 0x1FFF00014F: ???
==18063==    by 0x43D0E6: runtime.newm (proc.go:2230)
==18063==    by 0x43D7CE: runtime.startm (proc.go:2485)
==18063==    by 0x43DCF9: runtime.wakep (proc.go:2584)
==18063==    by 0x441A57: runtime.newproc.func1 (proc.go:4261)
==18063== 
==18063== 288 bytes in 1 blocks are possibly lost in loss record 3 of 5
==18063==    at 0x4C37BFD: calloc (vg_replace_malloc.c:1328)
==18063==    by 0x4013646: allocate_dtv (dl-tls.c:286)
==18063==    by 0x4013646: _dl_allocate_tls (dl-tls.c:530)
==18063==    by 0x4E4C227: allocate_stack (allocatestack.c:627)
==18063==    by 0x4E4C227: pthread_create@@GLIBC_2.2.5 (pthread_create.c:644)
==18063==    by 0x67C050: _cgo_try_pthread_create (gcc_libinit.c:100)
==18063==    by 0x67C24E: _cgo_sys_thread_start (gcc_linux_amd64.c:73)
==18063==    by 0x4689AC: runtime.asmcgocall.abi0 (asm_amd64.s:795)
==18063==    by 0x40ECC6: runtime.newobject (malloc.go:1228)
==18063==    by 0x1FFF0000AF: ???
==18063==    by 0x43D0E6: runtime.newm (proc.go:2230)
==18063==    by 0x43D7CE: runtime.startm (proc.go:2485)
==18063==    by 0x43DC6B: runtime.handoffp (proc.go:2519)
==18063==    by 0x43DD74: runtime.stoplockedm (proc.go:2598)
==18063== 
==18063== 288 bytes in 1 blocks are possibly lost in loss record 4 of 5
==18063==    at 0x4C37BFD: calloc (vg_replace_malloc.c:1328)
==18063==    by 0x4013646: allocate_dtv (dl-tls.c:286)
==18063==    by 0x4013646: _dl_allocate_tls (dl-tls.c:530)
==18063==    by 0x4E4C227: allocate_stack (allocatestack.c:627)
==18063==    by 0x4E4C227: pthread_create@@GLIBC_2.2.5 (pthread_create.c:644)
==18063==    by 0x67C050: _cgo_try_pthread_create (gcc_libinit.c:100)
==18063==    by 0x67C24E: _cgo_sys_thread_start (gcc_linux_amd64.c:73)
==18063==    by 0x46896F: runtime.asmcgocall.abi0 (asm_amd64.s:765)
==18063==    by 0xB4D5DF: ???
==18063== 
==18063== 1,024 bytes in 1 blocks are still reachable in loss record 5 of 5
==18063==    at 0x4C33045: malloc (vg_replace_malloc.c:381)
==18063==    by 0x50E113B: _IO_file_doallocate (filedoalloc.c:101)
==18063==    by 0x50F1328: _IO_doallocbuf (genops.c:365)
==18063==    by 0x50F0447: _IO_file_overflow@@GLIBC_2.2.5 (fileops.c:759)
==18063==    by 0x50EE98C: _IO_file_xsputn@@GLIBC_2.2.5 (fileops.c:1266)
==18063==    by 0x50E3A3E: puts (ioputs.c:40)
==18063==    by 0x46896F: runtime.asmcgocall.abi0 (asm_amd64.s:765)
==18063==    by 0x1FFF000257: ???
==18063==    by 0x441A57: runtime.newproc.func1 (proc.go:4261)
==18063==    by 0x466BAA: runtime.morestack.abi0 (asm_amd64.s:461)
==18063==    by 0x46B1C8: runtime.newproc.abi0 (<autogenerated>:1)
==18063==    by 0x67C90F: ??? (in /mnt/hgfs/code/go-leak-example/go-leak-example)
==18063== 
==18063== LEAK SUMMARY:
==18063==    definitely lost: 0 bytes in 0 blocks
==18063==    indirectly lost: 0 bytes in 0 blocks
==18063==      possibly lost: 1,152 bytes in 4 blocks
==18063==    still reachable: 1,024 bytes in 1 blocks
==18063==         suppressed: 0 bytes in 0 blocks
==18063== 
==18063== Use --track-origins=yes to see where uninitialised values come from
==18063== For lists of detected and suppressed errors, rerun with: -s
==18063== ERROR SUMMARY: 86 errors from 6 contexts (suppressed: 0 from 0)
```

根据官方解释 

```bash
"definitely lost" means your program is leaking memory -- fix those leaks!

"indirectly lost" means your program is leaking memory in a pointer-based structure. (E.g. if the root node of a binary tree is "definitely lost", all the children will be "indirectly lost".) If you fix the "definitely lost" leaks, the "indirectly lost" leaks should go away.

"possibly lost" means your program is leaking memory, unless you're doing unusual things with pointers that could cause them to point into the middle of an allocated block; see the user manual for some possible causes. Use --show-possibly-lost=no if you don't want to see these reports.

"still reachable" means your program is probably ok -- it didn't free some memory it could have. This is quite common and often reasonable. Don't use --show-reachable=yes if you don't want to see these reports.

"suppressed" means that a leak error has been suppressed. There are some suppressions in the default suppression files. You can ignore suppressed errors.
```



```bash
==18063== LEAK SUMMARY:
==18063==    definitely lost: 0 bytes in 0 blocks
==18063==    indirectly lost: 0 bytes in 0 blocks
==18063==      possibly lost: 1,152 bytes in 4 blocks
==18063==    still reachable: 1,024 bytes in 1 blocks
==18063==         suppressed: 0 bytes in 0 blocks
==18063== 
```

可以看到**疑似泄漏**1152bytes

在日志中寻找`go`或者`cgo`的block

```bash
==18063== 1,024 bytes in 1 blocks are still reachable in loss record 5 of 5
==18063==    at 0x4C33045: malloc (vg_replace_malloc.c:381)
==18063==    by 0x50E113B: _IO_file_doallocate (filedoalloc.c:101)
==18063==    by 0x50F1328: _IO_doallocbuf (genops.c:365)
==18063==    by 0x50F0447: _IO_file_overflow@@GLIBC_2.2.5 (fileops.c:759)
==18063==    by 0x50EE98C: _IO_file_xsputn@@GLIBC_2.2.5 (fileops.c:1266)
==18063==    by 0x50E3A3E: puts (ioputs.c:40)
==18063==    by 0x46896F: runtime.asmcgocall.abi0 (asm_amd64.s:765)
==18063==    by 0x1FFF000257: ???
==18063==    by 0x441A57: runtime.newproc.func1 (proc.go:4261)
==18063==    by 0x466BAA: runtime.morestack.abi0 (asm_amd64.s:461)
==18063==    by 0x46B1C8: runtime.newproc.abi0 (<autogenerated>:1)
==18063==    by 0x67C90F: ??? (in /mnt/hgfs/code/go-leak-example/go-leak-example)
```

可以看到cgo的调用

```bash
==18063==    by 0x50E3A3E: puts (ioputs.c:40)
==18063==    by 0x46896F: runtime.asmcgocall.abi0 (asm_amd64.s:765)
==18063==    by 0x1FFF000257: ???
==18063==    by 0x441A57: runtime.newproc.func1 (proc.go:4261)
```

```bash
==18063==    by 0x67C050: _cgo_try_pthread_create (gcc_libinit.c:100)
==18063==    by 0x67C24E: _cgo_sys_thread_start (gcc_linux_amd64.c:73)
==18063==    by 0x46896F: runtime.asmcgocall.abi0 (asm_amd64.s:765)
```

于是我们找到代码中的cgo部分

```go
/*
#include <stdio.h>
#include <stdlib.h>
void hello(const char *s) {
	puts(s);
}
*/
import "C"
```

