---
title: 使用frp内网穿透
name: frp
date: 2021-05-23 21:07:09
id: 0
tags: [linux]
categories: [linux]
abstract: ""
---

使用`frp`进行内网穿透


<!--more-->

使用`frp`进行内网穿透

<!--more-->

# 什么是frp

frp is a fast reverse proxy to help you expose a local server behind a NAT or firewall to the Internet. As of now, it supports **TCP** and **UDP**, as well as **HTTP** and **HTTPS** protocols, where requests can be forwarded to internal services by domain name.

frp also has a P2P connect mode.

frp 是一个专注于内网穿透的高性能的反向代理应用，支持 TCP、UDP、HTTP、HTTPS 等多种协议。可以将内网服务以安全、便捷的方式通过具有公网 IP 节点的中转暴露到公网。

## 使用场景

在内网大环境下配置带有公网ip的服务器进行内网穿透

使得外部用户可以直接通过公网访问内网的服务

## 服务端与客户端

frp分为两部分`frpc`和`frps`

服务端frps运行于公网服务器上，客户端frpc运行于内网环境

## 配置文件

**客户端frpc.ini**

```ini
# frpc.ini
[common]
server_addr = x.x.x.x
server_port = 7000

[ssh]
type = tcp
local_ip = 127.0.0.1
local_port = 22
remote_port = 6000
```

 **参数解析**

`common` 通用配置键

`server_addr` 远程服务器的公网ip地址

`server_port` 远程服务器开放的用于和client通信的端口（此端口双端一致）

`ssh` 开放ssh服务

`type` 通信协议 一般ssh为`tcp`，web服务为`http`

`local_port` 服务的本机端口

`remote_port` 暴露的外部通信端口

> 解释：在客户端remote_port定义后
>
> 假如ssh服务本身运行在内网的22端口现在会被映射到`remote_port`上 
>
> 访问公网ip:remote_port即可访问内网的ssh服务

**服务端frps.ini**

```ini
# frps.ini
[common]
bind_port = 7000
```

### 配置域名

如果不想通过端口跳转的方式访问服务而是域名

只需要修改客户端配置，加上`custom_domains`

```bash
# frpc.ini
[common]
server_addr = x.x.x.x
server_port = 7000

[web]
type = http
local_port = 80
custom_domains = www.example.com
```



### 配置通信密钥

常规的访问只需要保证客户端和服务端的请求通信端口一致即可

如果需要保证一定的安全性可以使用密钥

```ini
[common]
server_addr = 45.134.82.237
server_port = 6996
privilage_token = 123456
```

在客户端和服务端配置中都加入`privilage_token`字段作为二者的通信密钥

###   安全tcp

通过配置stcp协议保证连接的安全性

使用 `stcp(secret tcp)` 类型的代理可以避免让任何人都能访问到要穿透的服务，但是访问者也需要运行另外一个 frpc 客户端。

在内网服务上

```bash
[common]
server_addr = x.x.x.x
server_port = 7000

[secret_ssh]
type = stcp
# 只有 sk 一致的用户才能访问到此服务
sk = abcdefg
local_ip = 127.0.0.1
local_port = 22
```

在要访问的机器上同样需要运行frpc

```ini
[common]
server_addr = x.x.x.x
server_port = 7000

[secret_ssh_visitor]
type = stcp
# stcp 的访问者
role = visitor
# 要访问的 stcp 代理的名字
server_name = secret_ssh
sk = abcdefg
# 绑定本地端口用于访问 SSH 服务
bind_addr = 127.0.0.1
bind_port = 6000
```



### 配置http服务

一般情况下暴露的端口是`bind_port`这个是普通的tcp协议暴露端口

通过`vhost_http_port`可以指定暴露http端口 然后只需要访问http://ip:vhost_http_port即可

```bash
# frps.ini
[common]
bind_port = 7000
vhost_http_port = 8080
```

此时客户端需要配置对应的http服务和协议

```bash
# frpc.ini
[common]
server_addr = x.x.x.x
server_port = 7000

[web]
type = http
local_port = 80
custom_domains = www.example.com
```



### 配置多服务多域名

```ini
[common]
server_addr = x.x.x.x
server_port = 7000

[web]
type = http
local_port = 80
custom_domains = www.yourdomain.com

[web2]
type = http
local_port = 8080
custom_domains = www.yourdomain2.com
```

保证ini文件中的键不重复即可

配置完成后保证域名提供商已经绑定了服务器ip

通过http://www.yourdomain.com:80即可访问服务1

通过http://www.yourdomain2.com:8080即可访问服务2

### 配置子域名

适合一台frps多个frpc的场景

配置子域名可以更加方便的分发

```ini
# frps.ini
subdomain_host = frps.com
```

```ini
# frpc.ini
[web]
type = http
local_port = 80
subdomain = test
```

通过访问`test.frps.com`即可访问web服务

### 代理静态文件

```bash
[common]
server_addr = x.x.x.x
server_port = 7000

[test_static_file]
type = tcp
remote_port = 6000
plugin = static_file
# 要对外暴露的文件目录
plugin_local_path = /tmp/file
# 用户访问 URL 中会被去除的前缀，保留的内容即为要访问的文件路径
plugin_strip_prefix = static
plugin_http_user = abc
plugin_http_passwd = abc
```

通过浏览器访问 `http://x.x.x.x:6000/static/` 来查看位于 `/tmp/file` 目录下的文件，会要求输入已设置好的用户名和密码。

### 提供https服务

为http服务添加https

```ini
[common]
server_addr = x.x.x.x
server_port = 7000

[test_htts2http]
type = https
custom_domains = test.yourdomain.com

plugin = https2http
plugin_local_addr = 127.0.0.1:80

# HTTPS 证书相关的配置
plugin_crt_path = ./server.crt
plugin_key_path = ./server.key
plugin_host_header_rewrite = 127.0.0.1
plugin_header_X-From-Where = frp
```

### 开启http简单认证

```ini
# frpc.ini
[web]
type = http
local_port = 80
custom_domains = test.example.com
http_user = abc
http_pwd = abc
```



## 使用

```bash
# client side
./frpc -c frpc.ini

# server side
./frps -c frps.ini
```

查看效果

```bash
2021/05/23 02:01:10 [I] [service.go:449] [a552e647080fc1e9] client login info: ip [58.19.0.134:25128] version [0.36.2] hostname [] os [linux] arch [amd64]
2021/05/23 02:01:10 [I] [http.go:92] [a552e647080fc1e9] [nas] http proxy listen for host [git.renj.io] location [] group []
2021/05/23 02:01:10 [I] [control.go:446] [a552e647080fc1e9] new proxy [nas] success
```

### 搭配nginx

当配置多个域名时不想通过域名+端口号访问可以使用nginx进行转发

```nginx
server {
	# 监听的80端口
	listen 80;
	server_name a.frp.net b.frp.net;
	location / {
		proxy_pass http: //127.0.0.1:8080;
		proxy_set_header Host $host;
	}
}
```

frpc配置

```ini
[web1]
type = http
local_port = 1110
custom_domains = a.frp.net

[web2]
type = http
local_port = 1111
custom_domains = b.frp.net
```

然后分别使用域名即可访问不同的web服务

**注意**

nginx需要根据host头进行判断转发所以转发header一定要设置

## 参数列表

### frpc

| 参数                | 类型   | 说明                            | 默认值     | 可选值                          | 备注                                                         |
| ------------------- | ------ | ------------------------------- | ---------- | ------------------------------- | ------------------------------------------------------------ |
| server_addr         | string | 连接服务端的地址                | 0.0.0.0    |                                 |                                                              |
| server_port         | int    | 连接服务端的端口                | 7000       |                                 |                                                              |
| http_proxy          | string | 连接服务端使用的代理地址        |            |                                 | 格式为 {protocol}://user:passwd@192.168.1.128:8080 protocol 目前支持 http、socks5、ntlm |
| log_file            | string | 日志文件地址                    | ./frpc.log |                                 | 如果设置为 console，会将日志打印在标准输出中                 |
| log_level           | string | 日志等级                        | info       | trace, debug, info, warn, error |                                                              |
| log_max_days        | int    | 日志文件保留天数                | 3          |                                 |                                                              |
| disable_log_color   | bool   | 禁用标准输出中的日志颜色        | false      |                                 |                                                              |
| pool_count          | int    | 连接池大小                      | 0          |                                 |                                                              |
| user                | string | 用户名                          |            |                                 | 设置此参数后，代理名称会被修改为 {user}.{proxyName}，避免代理名称和其他用户冲突 |
| dns_server          | string | 使用 DNS 服务器地址             |            |                                 | 默认使用系统配置的 DNS 服务器，指定此参数可以强制替换为自定义的 DNS 服务器地址 |
| login_fail_exit     | bool   | 第一次登陆失败后是否退出        | true       |                                 |                                                              |
| protocol            | string | 连接服务端的通信协议            | tcp        | tcp, kcp, websocket             |                                                              |
| tls_enable          | bool   | 启用 TLS 协议加密连接           | false      |                                 |                                                              |
| tls_cert_file       | string | TLS 客户端证书文件路径          |            |                                 |                                                              |
| tls_key_file        | string | TLS 客户端密钥文件路径          |            |                                 |                                                              |
| tls_trusted_ca_file | string | TLS CA 证书路径                 |            |                                 |                                                              |
| tls_server_name     | string | TLS Server 名称                 |            |                                 | 为空则使用 server_addr                                       |
| heartbeat_interval  | int    | 向服务端发送心跳包的间隔时间    | 30         |                                 |                                                              |
| heartbeat_timeout   | int    | 和服务端心跳的超时时间          | 90         |                                 |                                                              |
| udp_packet_size     | int    | 代理 UDP 服务时支持的最大包长度 | 1500       |                                 | 服务端和客户端的值需要一致                                   |
| start               | string | 指定启用部分代理                |            |                                 | 当配置了较多代理，但是只希望启用其中部分时可以通过此参数指定，默认为全部启用 |

### 权限验证

| 参数                        | 类型   | 说明                    | 默认值 | 可选值      | 备注                                 |
| --------------------------- | ------ | ----------------------- | ------ | ----------- | ------------------------------------ |
| authentication_method       | string | 鉴权方式                | token  | token, oidc | 需要和服务端一致                     |
| authenticate_heartbeats     | bool   | 开启心跳消息鉴权        | false  |             | 需要和服务端一致                     |
| authenticate_new_work_conns | bool   | 开启建立工作连接的鉴权  | false  |             | 需要和服务端一致                     |
| token                       | string | 鉴权使用的 token 值     |        |             | 需要和服务端设置一样的值才能鉴权通过 |
| oidc_client_id              | string | oidc_client_id          |        |             |                                      |
| oidc_client_secret          | string | oidc_client_secret      |        |             |                                      |
| oidc_audience               | string | oidc_audience           |        |             |                                      |
| oidc_token_endpoint_url     | string | oidc_token_endpoint_url |        |             |                                      |

### frps

| 参数                      | 类型   | 说明                                   | 默认值       | 可选值                          | 备注                                         |
| ------------------------- | ------ | -------------------------------------- | ------------ | ------------------------------- | -------------------------------------------- |
| bind_addr                 | string | 服务端监听地址                         | 0.0.0.0      |                                 |                                              |
| bind_port                 | int    | 服务端监听端口                         | 7000         |                                 | 接收 frpc 的连接                             |
| bind_udp_port             | int    | 服务端监听 UDP 端口                    | 0            |                                 | 用于辅助创建 P2P 连接                        |
| kcp_bind_port             | int    | 服务端监听 KCP 协议端口                | 0            |                                 | 用于接收采用 KCP 连接的 frpc                 |
| proxy_bind_addr           | string | 代理监听地址                           | 同 bind_addr |                                 | 可以使代理监听在不同的网卡地址               |
| log_file                  | string | 日志文件地址                           | ./frps.log   |                                 | 如果设置为 console，会将日志打印在标准输出中 |
| log_level                 | string | 日志等级                               | info         | trace, debug, info, warn, error |                                              |
| log_max_days              | int    | 日志文件保留天数                       | 3            |                                 |                                              |
| disable_log_color         | bool   | 禁用标准输出中的日志颜色               | false        |                                 |                                              |
| detailed_errors_to_client | bool   | 服务端返回详细错误信息给客户端         | true         |                                 |                                              |
| heart_beat_timeout        | int    | 服务端和客户端心跳连接的超时时间       | 90           |                                 | 单位：秒                                     |
| user_conn_timeout         | int    | 用户建立连接后等待客户端响应的超时时间 | 10           |                                 | 单位：秒                                     |
| udp_packet_size           | int    | 代理 UDP 服务时支持的最大包长度        | 1500         |                                 | 服务端和客户端的值需要一致                   |
| tls_cert_file             | string | TLS 服务端证书文件路径                 |              |                                 |                                              |
| tls_key_file              | string | TLS 服务端密钥文件路径                 |              |                                 |                                              |
| tls_trusted_ca_file       | string | TLS CA 证书路径                        |              |                                 |                                              |

### 权限验证

| 参数                        | 类型   | 说明                   | 默认值 | 可选值      | 备注                               |
| --------------------------- | ------ | ---------------------- | ------ | ----------- | ---------------------------------- |
| authentication_method       | string | 鉴权方式               | token  | token, oidc |                                    |
| authenticate_heartbeats     | bool   | 开启心跳消息鉴权       | false  |             |                                    |
| authenticate_new_work_conns | bool   | 开启建立工作连接的鉴权 | false  |             |                                    |
| token                       | string | 鉴权使用的 token 值    |        |             | 客户端需要设置一样的值才能鉴权通过 |
| oidc_issuer                 | string | oidc_issuer            |        |             |                                    |
| oidc_audience               | string | oidc_audience          |        |             |                                    |
| oidc_skip_expiry_check      | bool   | oidc_skip_expiry_check |        |             |                                    |
| oidc_skip_issuer_check      | bool   | oidc_skip_issuer_check |        |             |                                    |

### 管理配置

| 参数                 | 类型   | 说明                               | 默认值 | 可选值 | 备注                            |
| -------------------- | ------ | ---------------------------------- | ------ | ------ | ------------------------------- |
| allow_ports          | string | 允许代理绑定的服务端端口           |        |        | 格式为 1000-2000,2001,3000-4000 |
| max_pool_count       | int    | 最大连接池大小                     | 5      |        |                                 |
| max_ports_per_client | int    | 限制单个客户端最大同时存在的代理数 | 0      |        | 0 表示没有限制                  |
| tls_only             | bool   | 只接受启用了 TLS 的客户端连接      | false  |        |                                 |

### Dashboard, 监控

| 参数              | 类型   | 说明                          | 默认值  | 可选值 | 备注                                                         |
| ----------------- | ------ | ----------------------------- | ------- | ------ | ------------------------------------------------------------ |
| dashboard_addr    | string | 启用 Dashboard 监听的本地地址 | 0.0.0.0 |        |                                                              |
| dashboard_port    | int    | 启用 Dashboard 监听的本地端口 | 0       |        |                                                              |
| dashboard_user    | string | HTTP BasicAuth 用户名         |         |        |                                                              |
| dashboard_pwd     | string | HTTP BasicAuth 密码           |         |        |                                                              |
| enable_prometheus | bool   | 是否提供 Prometheus 监控接口  | false   |        | 需要同时启用了 Dashboard 才会生效                            |
| asserts_dir       | string | 静态资源目录                  |         |        | Dashboard 使用的资源默认打包在二进制文件中，通过指定此参数使用自定义的静态资源 |

### HTTP & HTTPS

| 参数               | 类型   | 说明                                            | 默认值 | 可选值 | 备注                                      |
| ------------------ | ------ | ----------------------------------------------- | ------ | ------ | ----------------------------------------- |
| vhost_http_port    | int    | 为 HTTP 类型代理监听的端口                      | 0      |        | 启用后才支持 HTTP 类型的代理，默认不启用  |
| vhost_https_port   | int    | 为 HTTPS 类型代理监听的端口                     | 0      |        | 启用后才支持 HTTPS 类型的代理，默认不启用 |
| vhost_http_timeout | int    | HTTP 类型代理在服务端的 ResponseHeader 超时时间 | 60     |        |                                           |
| subdomain_host     | string | 二级域名后缀                                    |        |        |                                           |
| custom_404_page    | string | 自定义 404 错误页面地址                         |        |        |                                           |

### TCPMUX

| 参数                    | 类型 | 说明                         | 默认值 | 可选值 | 备注                                       |
| ----------------------- | ---- | ---------------------------- | ------ | ------ | ------------------------------------------ |
| tcpmux_httpconnect_port | int  | 为 TCPMUX 类型代理监听的端口 | 0      |        | 启用后才支持 TCPMUX 类型的代理，默认不启用 |

## 参考资料

- [frp on github](https://github.com/fatedier/frp)
- [frp中文文档](https://github.com/fatedier/frp/blob/dev/README_zh.md)

