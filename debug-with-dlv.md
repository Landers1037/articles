---
title: 使用delve对go程序debug
name: debug-with-dlv
date: 2021-09-13 02:34:17
id: 0
tags: [go]
categories: [go]
abstract: ""
---

使用`delve`进行远程调试

在Goland上配置`dlv`进行远程调试

<!--more-->

## 安装dlv

参考官方的安装方式 [github](https://github.com/go-delve/delve/blob/master/Documentation/installation/README.md)

对于linux系统使用`go get`可以直接安装

无法使用go代理时，官方提供了git方式下载并编译

```bash
$ git clone https://github.com/go-delve/delve
$ cd delve
$ go install github.com/go-delve/delve/cmd/dlv
```

使用go直接安装

```bash
$ go install github.com/go-delve/delve/cmd/dlv@latest
```

默认安装的路径为`$GOPATH/bin`，如果想要全局使用需要将路径添加到`PATH`

```bash
$ export PATH=$PATH:$GOPATH/bin
```

```bash
$ dlv
Delve is a source level debugger for Go programs.

Delve enables you to interact with your program by controlling the execution of the process,
evaluating variables, and providing information of thread / goroutine state, CPU register state and more.

The goal of this tool is to provide a simple yet powerful interface for debugging Go programs.

Pass flags to the program you are debugging using `--`, for example:

`dlv exec ./hello -- server --config conf/config.toml`

Usage:
  dlv [command]

Available Commands:
  attach      Attach to running process and begin debugging.
  connect     Connect to a headless debug server.
  core        Examine a core dump.
  dap         [EXPERIMENTAL] Starts a headless TCP server communicating via Debug Adaptor Protocol (DAP).
  debug       Compile and begin debugging main package in current directory, or the package specified.
  exec        Execute a precompiled binary, and begin a debug session.
  help        Help about any command
  run         Deprecated command. Use 'debug' instead.
  test        Compile test binary and begin debugging program.
  trace       Compile and begin tracing program.
  version     Prints version.

Flags:
      --accept-multiclient               Allows a headless server to accept multiple client connections.
      --allow-non-terminal-interactive   Allows interactive sessions of Delve that don't have a terminal as stdin, stdout and stderr
      --api-version int                  Selects API version when headless. New clients should use v2. Can be reset via RPCServer.SetApiVersion. See Documentation/api/json-rpc/README.md. (default 1)
      --backend string                   Backend selection (see 'dlv help backend'). (default "default")
      --build-flags string               Build flags, to be passed to the compiler. For example: --build-flags="-tags=integration -mod=vendor -cover -v"
      --check-go-version                 Checks that the version of Go in use is compatible with Delve. (default true)
      --disable-aslr                     Disables address space randomization
      --headless                         Run debug server only, in headless mode.
  -h, --help                             help for dlv
      --init string                      Init file, executed by the terminal client.
  -l, --listen string                    Debugging server listen address. (default "127.0.0.1:0")
      --log                              Enable debugging server logging.
      --log-dest string                  Writes logs to the specified file or file descriptor (see 'dlv help log').
      --log-output string                Comma separated list of components that should produce debug output (see 'dlv help log')
      --only-same-user                   Only connections from the same user that started this instance of Delve are allowed to connect. (default true)
  -r, --redirect stringArray             Specifies redirect rules for target process (see 'dlv help redirect')
      --wd string                        Working directory for running the program.

Additional help topics:
  dlv backend  Help about the --backend flag.
  dlv log      Help about logging flags.
  dlv redirect Help about file redirection.

Use "dlv [command] --help" for more information about a command.
```

## 使用示例

`main.go`

```go
/*
Name: playgroundx
File: main.go
Author: Landers
*/

package main

import (
	"encoding/json"
	"fmt"
	"os"
)

func main() {
	f, e := os.ReadFile("test.json")
	if e != nil {
		fmt.Println(e)
		return
	}
	var data map[string]interface{}
	e = json.Unmarshal(f, &data)
	if e != nil {
		fmt.Println(e)
		return
	}

	fmt.Println(data)
}
```

一个简单的读取**test.json**并输出的程序

### 启动debug

```bash
$ dlv debug main.go
Type 'help' for list of commands.
(dlv) break main.main
Breakpoint 1 set at 0x4f5792 for main.main() ./main.go:15
(dlv) continue
> main.main() ./main.go:15 (hits goroutine(1):1 total:1) (PC: 0x4f5792)
    10:         "encoding/json"
    11:         "fmt"
    12:         "os"
    13: )
    14:
=>  15: func main() {
    16:         f, e := os.ReadFile("test.json")
    17:         if e != nil {
    18:                 fmt.Println(e)
    19:                 return
    20:         }
(dlv) c
open test.json: no such file or directory
Process 404507 has exited with status 0
```

使用`break`命令在main函数位置设置断点

使用`continue`或者`c`命令运行至断点处

可以看到因为当前目录不存在json文件所以在`os.ReadFile("test.json")`处报错退出程序

```bash
# 添加json文件
echo '{"name": "landers"}' > test.json

Type 'help' for list of commands.
(dlv) break main.main
Breakpoint 1 set at 0x4f5792 for main.main() ./main.go:15
(dlv) c
> main.main() ./main.go:15 (hits goroutine(1):1 total:1) (PC: 0x4f5792)
    10:         "encoding/json"
    11:         "fmt"
    12:         "os"
    13: )
    14:
=>  15: func main() {
    16:         f, e := os.ReadFile("test.json")
    17:         if e != nil {
    18:                 fmt.Println(e)
    19:                 return
    20:         }
(dlv) c
map[name:landers]
Process 407615 has exited with status 0
```

程序执行完毕

### 单步执行

使用`next(n)`单步运行程序

```bash
> main.main() ./main.go:15 (hits goroutine(1):1 total:1) (PC: 0x4f5792)
    10:         "encoding/json"
    11:         "fmt"
    12:         "os"
    13: )
    14:
=>  15: func main() {
    16:         f, e := os.ReadFile("test.json")
    17:         if e != nil {
    18:                 fmt.Println(e)
    19:                 return
    20:         }
(dlv) n
> main.main() ./main.go:16 (PC: 0x4f57a9)
    11:         "fmt"
    12:         "os"
    13: )
    14:
    15: func main() {
=>  16:         f, e := os.ReadFile("test.json")
    17:         if e != nil {
    18:                 fmt.Println(e)
    19:                 return
    20:         }
    21:         var data map[string]interface{}
(dlv) n
> main.main() ./main.go:17 (PC: 0x4f584a)
    12:         "os"
    13: )
    14:
    15: func main() {
    16:         f, e := os.ReadFile("test.json")
=>  17:         if e != nil {
    18:                 fmt.Println(e)
    19:                 return
    20:         }
    21:         var data map[string]interface{}
    22:         e = json.Unmarshal(f, &data)
```

使用`locals`变量可以打印当前的变量

```bash
(dlv) locals
f = []uint8 len: 20, cap: 512, [...]
e = error nil
```

## dlv常用命令

`attach pid` 直接对正在运行的进程调试

`debug` 对某个文件编译后debug，在退出调试模式后二进制会被删除

`exec` 直接从二进制程序启动debug模式

`core executable_file core_file` 以core模式启动调试，为了找出二进制程序core的原因

`trace file func` 追踪函数的调用轨迹



`break(b)` 打断点，例如`b FuncName`或者`b main.go:10`

`continue(c)` 继续执行到断点处

`bp` 查看全部断点

`restart(r)` 重启debug进程

`on` 在执行到某处断点时执行命令

`condition(cord)` 有条件的断点，只有某个表达式成立才进入中断

`next(n)` 逐步执行代码，不进入函数内部

`step(s)` 逐步执行代码，遇到函数会进入函数内部

`stepout` 跳出函数

`args` 查看被调用函数传入的参数值

`locals` 查看局部变量，支持指定某个变量名

`clear` 清除单个断点

`clearall` 清除全部断点 

`list` 打印当前断点位置的源码

`bt` 打印当前栈信息

`print(p)` 打印变量或者表达式

**协程**

`goroutines` 显示所有协程

`goroutine num` 切换到指定协程

## 使用goland进行远程debug

在`Run`中添加`Go Remote`

![](/images/debug-with-dlv-1.png)

配置远程机器的ip和端口号

![](/images/debug-with-dlv-2.png)

根据提示在远程机器上使用dlv编译程序或使用`gcflags`编译

```bash
Before running this configuration, start your application and Delve as described bellow.  Allow Delve to compile your application: 
dlv debug --headless --listen=:6666 --api-version=2 --accept-multiclient
 Or compile the application using Go 1.10 or newer: 
go build -gcflags \"all=-N -l\" github.com/app/demo
 and then run it with Delve using the following command: 
dlv --listen=:6666 --headless=true --api-version=2 --accept-multiclient exec ./demo
```

### 在远程服务器启动dlv

```bash
$ dlv debug --headless --listen=:2345 --api-version=2 main.go 
API server listening at: [::]:2345
```



### 在本机debug调试

```go
/*
Name: playgroundx
File: main.go
Author: Landers
*/

package main

import (
	"fmt"
	"github.com/gin-gonic/gin"
)

func main() {
	g := gin.Default()
	g.GET("/", func(c *gin.Context) {
		fmt.Println(c.Request)
    // add break point
		c.Set("name", "test")
	})
	g.Run()
}
```

```bash
$ curl http://10.211.55.4:8080
```

![](/images/debug-with-dlv-3.png)

请求接口，触发进入断点，打印出所有的`c.request`

