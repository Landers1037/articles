---
title: docker入门教程2
name: docker-start2
date: 2021-08-17 12:14:55
id: 0
tags: [docker]
categories: [docker]
abstract: ""
---

docker入门 - 常用docker命令操作

<!--more-->

## docker命令

```bash

Usage:  docker [OPTIONS] COMMAND

A self-sufficient runtime for containers

Options:
      --config string      Location of client config files (default "/root/.docker")
  -c, --context string     Name of the context to use to connect to the daemon (overrides DOCKER_HOST
                           env var and default context set with "docker context use")
  -D, --debug              Enable debug mode
  -H, --host list          Daemon socket(s) to connect to
  -l, --log-level string   Set the logging level ("debug"|"info"|"warn"|"error"|"fatal") (default "info")
      --tls                Use TLS; implied by --tlsverify
      --tlscacert string   Trust certs signed only by this CA (default "/root/.docker/ca.pem")
      --tlscert string     Path to TLS certificate file (default "/root/.docker/cert.pem")
      --tlskey string      Path to TLS key file (default "/root/.docker/key.pem")
      --tlsverify          Use TLS and verify the remote
  -v, --version            Print version information and quit

Management Commands:
  app*        Docker App (Docker Inc., v0.9.1-beta3)
  builder     Manage builds
  buildx*     Build with BuildKit (Docker Inc., v0.5.1-docker)
  config      Manage Docker configs
  container   Manage containers
  context     Manage contexts
  image       Manage images
  manifest    Manage Docker image manifests and manifest lists
  network     Manage networks
  node        Manage Swarm nodes
  plugin      Manage plugins
  scan*       Docker Scan (Docker Inc., v0.8.0)
  secret      Manage Docker secrets
  service     Manage services
  stack       Manage Docker stacks
  swarm       Manage Swarm
  system      Manage Docker
  trust       Manage trust on Docker images
  volume      Manage volumes

Commands:
  attach      Attach local standard input, output, and error streams to a running container
  build       Build an image from a Dockerfile
  commit      Create a new image from a container's changes
  cp          Copy files/folders between a container and the local filesystem
  create      Create a new container
  diff        Inspect changes to files or directories on a container's filesystem
  events      Get real time events from the server
  exec        Run a command in a running container
  export      Export a container's filesystem as a tar archive
  history     Show the history of an image
  images      List images
  import      Import the contents from a tarball to create a filesystem image
  info        Display system-wide information
  inspect     Return low-level information on Docker objects
  kill        Kill one or more running containers
  load        Load an image from a tar archive or STDIN
  login       Log in to a Docker registry
  logout      Log out from a Docker registry
  logs        Fetch the logs of a container
  pause       Pause all processes within one or more containers
  port        List port mappings or a specific mapping for the container
  ps          List containers
  pull        Pull an image or a repository from a registry
  push        Push an image or a repository to a registry
  rename      Rename a container
  restart     Restart one or more containers
  rm          Remove one or more containers
  rmi         Remove one or more images
  run         Run a command in a new container
  save        Save one or more images to a tar archive (streamed to STDOUT by default)
  search      Search the Docker Hub for images
  start       Start one or more stopped containers
  stats       Display a live stream of container(s) resource usage statistics
  stop        Stop one or more running containers
  tag         Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE
  top         Display the running processes of a container
  unpause     Unpause all processes within one or more containers
  update      Update configuration of one or more containers
  version     Show the Docker version information
  wait        Block until one or more containers stop, then print their exit codes

Run 'docker COMMAND --help' for more information on a command.

To get more help with docker, check out our guides at https://docs.docker.com/go/guides/
```



## 常用操作

### 查找镜像

```bash
docker search xxx
```

会在docker镜像源中搜索

### 拉取镜像

```bash
docker pull nginx:latest
```

### 查看镜像

```bash
docker images
```

### 删除镜像

```
docker rmi 镜像名:版本 or 镜像id
```

### 镜像重新打tag

用于修改镜像的tag标签名称

```bash
Usage:  docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]

Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE

REPOSITORY                                                                    TAG        IMAGE ID       CREATED         SIZE
landers1037/blog                                                              v6.0       4d5f9c77233e   4 weeks ago     18.3MB

# 重命名此镜像
docker tag landers1037/blog:v6.0 git.io/blog:v1
```

```bash
REPOSITORY                                                                    TAG        IMAGE ID       CREATED         SIZE
landers1037/blog                                                              v6.0       4d5f9c77233e   4 weeks ago     18.3MB

git.io/blog                                                                   v1         4d5f9c77233e   4 weeks ago     18.3MB
```

可以看到新的镜像以`git.io/blog`命名

### 运行

```bash
docker run 镜像名:版本

Usage: docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

**常用的运行参数**

- -a --attach登录后台运行的容器
- -c 设置cpu权重默认0
- -d 后台运行
- -e 设置容器内环境变量
- --entrypoint 覆盖镜像的默认入口如(`/bin/bash`)
- -p 指定暴露的容器端口
- -m 指定容器的内存使用上限
- --name 指定容器的别名
- -net 配置容器网络(`bridge`桥接模式 `host`容器使用本机网络 `container`使用其他容器网络 `none`容器使用自己的网络)
- --restart 指定容器停止后的重启策略（例如always总是重启）
- --rm 指定容器停止后即删除容器 不适用-d方式启动的容器
- -- privileged 指定容器是否拥有特权
- -t 分配tty设备支持终端登录一般与-i配合
- -i 打开stdin用于控制台交互
- -u  --user 指定容器内的用户
- -v 指定挂载存储卷
- -w --workdir 指定容器的工作目录

```bash
docker run -it --name=test -v /home:/app -p 8080:80 git.io/blog:v1 /bin/bash
```

例如使用刚才的blog镜像启动一个容器名称`test` 挂载本地的`/home`到容器内的`/app` 暴露容器端口`80`到外部端口`8080` 同时启动一个伪终端启动入口为`/bin/bash`

### 停止容器

```bash
docker stop 容器id or 容器name
```

### 查看容器

```bash
# 查看正在运行的容器
docker ps
# 查看全部
docker ps -a
```

### 进入正在运行的容器

```bash
docker attach 容器id
```

一般使用`start`启动容器后可以使用`attach`进入

或者

```bash
docker exec -it 容器id /bin/bash
```

### 启动容器

```bash
docker start 容器id
# 重启
docker restart 容器id
```

### 删除容器

```bash
docker rm 容器id
```

docker会自动识别id，如果id不重复只需要填写id的前几位

**删除全部容器**

```bash
docker rm -f $( docker  ps -aq) 
```

### 强行停止容器

```bash
docker kill 容器id
```

### 重命名容器

```bash
docker rename name1 name2
```



### 查看历史

```bash
docker history 容器id
```

### 查看容器进程信息

```bash
docker top 容器id
```

### 查看镜像的元数据

```bash
docker inspect 容器id
```

### 导出容器

一般用于离线构建

```bash
docker export 容器id > xxx.tar
```

### 导出镜像

```bash
docker save 镜像 > xxx.tar
```

### 加载镜像

```bash
docker load <xxx.tar
# 或者
cat xxx.tar | docker import - xxx:import
```

注意输入输出符`> <`

docker的`export`和`import`导出和导入的是容器的快照而不是镜像本身所以不存在layer层

快照文件将丢弃所有历史记录和元数据信息

docker的`load`和`save`用于保存和导入镜像

### 向容器中拷贝文件

```bash
docker cp 容器id:/home/path/xxx /home
```

将`/home/path/xxx`拷贝到容器的`/home`目录下

### 查看容器日志

```bash
docker logs 容器id

Usage:  docker logs [OPTIONS] CONTAINER

Fetch the logs of a container

Options:
      --details        Show extra details provided to logs
  -f, --follow         Follow log output
      --since string   Show logs since timestamp (e.g. 2013-01-02T13:23:37Z) or relative (e.g. 42m for 42 minutes)
  -n, --tail string    Number of lines to show from the end of the logs (default "all")
  -t, --timestamps     Show timestamps
      --until string   Show logs before a timestamp (e.g. 2013-01-02T13:23:37Z) or relative (e.g. 42m for 42
                       minutes)
```

查看某一时间后的日志

```bash
docker logs -t --since=2020-01-01T18:00:00 CONTAINER_ID
```

查看最后100行

```bash
docker logs -f -t --tail=100 CONTAINER_ID
```

