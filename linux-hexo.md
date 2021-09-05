---
title: Linux下的hexo安装
name: linux-hexo
date: 2019-05-10
id: 0
tags: [linux,hexo]
categories: []
abstract: ""
---


## ubuntu18.04

### 安装nvm

```bash
wget -qO- https://raw.github.com/creationix/nvm/master/install.sh | sh
```


<!--more-->


## ubuntu18.04

### 安装nvm

```bash
wget -qO- https://raw.github.com/creationix/nvm/master/install.sh | sh
```

<!--more-->

### 安装node js

```bash
nvm install 7
#node.js更新的很快，hexo3.8需求node>6.9
nvm ls-remote #可以查看所有的库
```

### 配置环境变量

```bash
#这句话写入到bashrc
 export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"
#然后source一下
```

### 安装hexo

```bash
$npm install  hexo-cli -g
$npm install hexo-server -g
$npm install hexo-deployer-git -g
$npm install hexo-util -g
#也可以直接 install hexo -g
```

### 新建一个hexo文件夹初始化

```bash
#进入文件夹下
hexo init
npm install #安装依赖的库
```

### 卸载旧版node.js

```bash
nvm uninstall 4 #卸载4.*版本的node.js
```

