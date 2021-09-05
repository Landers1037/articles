---
title: Hexo 搭建静态页面的心得
name: hexo-study
date: 2019-02-01
id: 0
tags: [暂时没有标签]
categories: []
abstract: ""
---


# Hexo+Github

## 安装环境

node.js

github账户

<!--more-->


# Hexo+Github

## 安装环境

node.js

github账户
<!--more-->

第一步：使用npm安装hexo

```bash
$ npm install -g hexo-cli  
$ npm install hexo-server --save
```

第二步：在硬盘新建文件夹blog ，进入，安装hexo

```bash
$ npm install hexo-cli -g
$ hexo init blog
$ cd blog
$ npm install
$ hexo server
```

第三步：测试

在浏览器中输入localhost:4000查看生成的网页

如果出现访问错误，检查是否使用hexo s的命令，server一定要是处于开启的状态

如果还有错误，检查电脑的4000端口是否被占用，根据网上反馈，安装有foxit阅读器的用户，需要关闭它的服务，或者使用下面的命令改变端口

```
hexo s -p 5000
```

运行正常即可在本地查看网页

第四步：远程发布至github

新建一个仓库，命名规则：用户名.github.io

每一个用户只能建立一个这样的仓库，然后进入仓库复制自己的ssh地址

在本地的博客目录下找到_config.yml文件，翻到最下面，如下填写

```yaml
deploy:
  type: git
  repo: https://github.com/你的用户名/你的用户名.github.io.git
  branch: master
```

配置完成后使用如下命令发布到github

```bash
hexo clean
hexo g
hexo d 或者使用hexo d -g
```

上传文章出现warning：CRLF问题

```bash
git config –-global core.autocrlf false
```

