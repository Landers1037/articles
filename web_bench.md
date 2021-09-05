---
title: 接口压力测试工具的使用
name: web_bench
date: 2021-04-12 23:46:45
id: 0
tags: [linux]
categories: [linux]
abstract: ""
---

介绍两款压力测试工具`wrk`和`webbench`的基本使用


<!--more-->

介绍两款压力测试工具`wrk`和`webbench`的基本使用

<!--more-->

压力测试 一般在后端服务编写完毕后需要进行

通过压力测试我们可以得知服务能够承载的并发压力了解数据库io上限

以下分工具介绍

# wrk

wrk是一款基于c的压力测试工具，使用时需要通过源码编译为二进制文件

源码地址https://github.com/wg/wrk

```bash
cd wrk && make
```

编译完成后可以查看程序的使用帮助

```bash
wrk

Usage: wrk <options> <url>                            
  Options:                                            
    -c, --connections <N>  Connections to keep open   
    -d, --duration    <T>  Duration of test           
    -t, --threads     <N>  Number of threads to use   
                                                      
    -s, --script      <S>  Load Lua script file       
    -H, --header      <H>  Add header to request      
        --latency          Print latency statistics   
        --timeout     <T>  Socket/request timeout     
    -v, --version          Print version details      
                                                      
  Numeric arguments may include a SI unit (1k, 1M, 1G)
  Time arguments may include a time unit (2s, 2m, 2h)
```

### 参数解释

`-c` 指定跟服务器建立的连接数

`-d` 指定压力测试的时间

`-t `指定使用的线程数

`-s` 加载lua脚本

`-H `自定义http请求的请求头

`--latency` 结束时打印延迟统计信息

`--timeout` 指定请求超时时间

## 使用

假设现在有一个测试服务 提供接口`/hello` get方式

```bash
curl http://127.0.0.1:5000/hello

{"message":"hello this is my blog"}
```

### 并发测试

指定线程数10 测试时间10s 模拟500个请求

```bash
wrk -t 10 -d 10 -c 500 http://127.0.0.1:5000/hello

Running 10s test @ http://127.0.0.1:5000/hello
  10 threads and 500 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    87.93ms  110.04ms 878.24ms   85.42%
    Req/Sec   829.30      1.29k   11.21k    93.65%
  81851 requests in 10.08s, 40.36MB read
Requests/sec:   8121.05
Transfer/sec:      4.00MB
```

解释：

 `Avg`平均值  `Stdev`标准差 `Max`最大值  `+/- Stdev`正负一个标准差所占比例

 `Latency` 延迟时间

`Req/Sec` 每秒的请求数

`81851 requests in 10.08s, 40.36MB read`即在10.08s内发送了81851个请求 响应数据大小为40.36mb

 `Requests/sec:   8121.05` 即qps 每秒的平均处理的请求数

`Transfer/sec:      4.00MB` 即每秒平均传递的数据量

### 发送post请求

wrk发送带请求体的数据时 必须使用lua配合测试

编写一段lua脚本

```lua
wrk.method = "POST"
wrk.body = '{"data": "test"}'
wrk.headers["Content-Type"] = "application/json"
```

使用lua测试

```bash
wrk -t 10 -c 500 -s ./post.lua http://127.0.0.1:5000/hello
```

## 关于wrk的运行阶段

使用lua脚本时可以实现复杂的wrk请求

wrk发送请求大致分为三个阶段：init(初始化) request(请求) response(响应)

### wrk.format

```lua
function wrk.format(method, path, headers, body)

    wrk.format returns a HTTP request string containing the passed parameters
    merged with values from the wrk table.
```

format函数返回一个格式化的http请求字符串

### wrk.lookup

```lua
function wrk.lookup(host, service)

    wrk.lookup returns a table containing all known addresses for the host
    and service pair. This corresponds to the POSIX getaddrinfo() function.
```

lookup函数返回一个表，其中包含主机和服务对的所有已知地址

### wrk.connect

```lua
function wrk.connect(addr)

    wrk.connect returns true if the address can be connected to, otherwise
    it returns false. The address must be one returned from wrk.lookup().
```

connect函数返回接口的请求是否成功，该接口地址必须是wrk.lookup方法返回的地址

### setup

此过程接口地址已经传递给了wrk解析完成，线程已经初始化完成但是还未启动

### running

运行会调用init()函数 然后循环调用request()和response()函数

`delay()`返回延迟发送下一个请求的毫秒数。

`request()`需要为每个请求返回HTTP对象。在此函数中，我们可以修改method，headers，path，和 body。

`response()`响应返回时调用。需要status（http响应状态），headers（http响应头），body（http响应体）三个参数，解析响应头和响应体非常消耗计算机资源，因此，如果调用init（）方法之后全局变量response是nil，wrk将不会解析响应头和响应体。

```lua
Running

  function init(args)
  function delay()
  function request()
  function response(status, headers, body)

  The running phase begins with a single call to init(), followed by
  a call to request() and response() for each request cycle.

  The init() function receives any extra command line arguments for the
  script which must be separated from wrk arguments with "--".

  delay() returns the number of milliseconds to delay sending the next
  request.

  request() returns a string containing the HTTP request. Building a new
  request each time is expensive, when testing a high performance server
  one solution is to pre-generate all requests in init() and do a quick
  lookup in request().

  response() is called with the HTTP response status, headers, and body.
  Parsing the headers and body is expensive, so if the response global is
  nil after the call to init() wrk will ignore the headers and body.
```



### done

当所有请求都已经完成并且请求结果已经被统计时调用。

`done()`函数接收一个包含结果数据的表(表是lua内置数据类型)，以及两个统计对象：每个请求的延迟时间（以微秒为单位）、每个线程的请求速率（以每秒请求数为单位）。Done函数内，可以使用以下属性：

```lua
function done(summary, latency, requests)

  The done() function receives a table containing result data, and two
  statistics objects representing the per-request latency and per-thread
  request rate. Duration and latency are microsecond values and rate is
  measured in requests per second.



  latency.min              -- minimum value seen 延时最小值
  latency.max              -- maximum value seen 延时最大值
  latency.mean             -- average value seen 延时平均值
  latency.stdev            -- standard deviation 延时标准差
  latency:percentile(99.0) -- 99th percentile value 第99%的值
  latency[i]              -- raw value and count 请求i的原始延迟数据

  summary = {
    duration = N,  -- run duration in microseconds 运行持续时间（以微秒为单位）
    requests = N,  -- total completed requests 完成的请求总数
    bytes    = N,  -- total bytes received 接受的总字节数
    errors   = {
      connect = N, -- total socket connection errors    socket连接错误的总数目
      read    = N, -- total socket read errors    socket读取错误的总数目
      write   = N, -- total socket write errors    socket写入错误的总数目
      status  = N, -- total HTTP status codes > 399   HTTp状态码大于399的总数目
      timeout = N  -- total request timeouts    所有请求的超时时间总和
    }
  }
```

# webbench

webbench是一个c编写的简易接口测试工具

```bash
webbench [option]... URL
  -f|--force               Don't wait for reply from server.
  -r|--reload              Send reload request - Pragma: no-cache.
  -t|--time <sec>          Run benchmark for <sec> seconds. Default 30.
  -p|--proxy <server:port> Use proxy server for request.
  -c|--clients <n>         Run <n> HTTP clients at once. Default one.
  -9|--http09              Use HTTP/0.9 style requests.
  -1|--http10              Use HTTP/1.0 protocol.
  -2|--http11              Use HTTP/1.1 protocol.
  --get                    Use GET request method.
  --head                   Use HEAD request method.
  --options                Use OPTIONS request method.
  --trace                  Use TRACE request method.
  -?|-h|--help             This information.
  -V|--version             Display program version.
```

参数解释

`-f` 不需要等待服务器响应

`-r` 发送重新加载请求

`-t` 指定测试时长默认30s

`-p` 使用代理服务器

`-c` 指定并发的用户数

需要注意的是webbench仅支持发送get请求 但是可以定制header头部

webbench最高支持并发3w的接口请求测试

### 并发测试

指定并发用户数100 测试10s

```bash
webbench -c 100 -t 10 http://127.0.0.1:5000/hello

Webbench - Simple Web Benchmark 1.5
Copyright (c) Radim Kolar 1997-2004, GPL Open Source Software.

Request:
GET /hello HTTP/1.0
User-Agent: WebBench 1.5
Host: 127.0.0.1


Runing info: 100 clients, running 10 sec.

Speed=180942 pages/min, 1559116 bytes/sec.
Requests: 30157 susceed, 0 failed.
```

Speed 为每分钟多个个请求
Requests 成功多少个请求，失败多少个请求

## 参考资料

[github wrk](https://github.com/wg/wrk)

[wrk笔记](http://www.manongjc.com/detail/22-hstcrmxjwpdqova.html)