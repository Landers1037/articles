---
title: 实用Git教程
name: useful-git-tutorial
date: 2020-10-09
id: 0
tags: [学习笔记,git]
categories: []
abstract: ""
---



Git的实用命令及使用示例，简单高效地管理代码


<!--more-->

官方教程请[参考](http://git-scm.com/docs)

本示例将从创建，克隆，提交，回退几部分介绍git的使用

## 创建

使用git命令在本地创建一个全新的仓库，假设在本地空目录`gittest`下

```bash
landers@skypig:/mnt/d/tmp/gittest$ tree
.

0 directories, 0 files

$ git init
Initialized empty Git repository in D:/tmp/gittest/.git/
```

使用git初始化后，会在当前目录下多出一个`.git`目录

使用`git init repo_name`创建自定义名称的仓库

### 添加文件

当我们在本地编辑新建了文件后，将新建的文件加入到git仓库内

我们新建一个`README.md`文件

```bash
landers@skypig:/mnt/d/tmp/gittest$ touch README.md

landers@skypig:/mnt/d/tmp/gittest$ echo "this's test fot git tutorial" > README.md
```

使用`git add`将文件添加到仓内

```bash
$ git add README.md
```

一般情况下修改的文件很多，不可能一个一个add，所以add支持通配符

```bash
$ git add .
# 把当前目录下的全部文件加入
$ git add *.java
# 把所有的java文件加入
```

然后使用`git commit -m "message"`将更改内容提交，具体将在后面演示

### .gitignore

在提交时我们不希望一些测试文件，比如生成的二进制文件或者拥有敏感信息的文件比如.env文件被添加到仓内提交，此时可以使用`.gitignore`

为了看到变化我们多创建几个文件，我们新建一个密钥文件`secret.txt`它应该是不能被提交的

新建两个目录`include`和`exclude`，每个目录下有一个txt文件

```bash
landers@skypig:/mnt/d/tmp/gittest$ touch secret.txt
landers@skypig:/mnt/d/tmp/gittest$ mkdir include
landers@skypig:/mnt/d/tmp/gittest$ mkdir exclude
landers@skypig:/mnt/d/tmp/gittest$ touch include/test.txt
landers@skypig:/mnt/d/tmp/gittest$ touch exclude/test.txt
landers@skypig:/mnt/d/tmp/gittest$ tree
.
├── README.md
├── exclude
├── include
└── secret.txt

2 directories, 2 files
```

首先使用`git status`命令查看当前可以被add的文件

```bash
$ git status
On branch master

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)

        new file:   README.md

Untracked files:
  (use "git add <file>..." to include in what will be committed)

        exclude/
        include/
        secret.txt
```

可以看到刚刚新建的目录和文件都是可以被add进仓库的(即使用`git add .`命令)，现在在`.gitignore`文件内添加需要排除的文件和文件夹

```bash
$ nano .gitignore || touch .gitignore
// 加入内容
secret.txt
exclude/

$ git status
On branch master

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)

        new file:   README.md

Untracked files:
  (use "git add <file>..." to include in what will be committed)

        .gitignore
        include/
```

再次执行`git status`命令后，会发现secret.txt文件和exclude目录已经被排除在外，此时使用`git add .`添加文件时就会排除掉这些文件和文件夹

## 提交

在刚才我们向仓库内添加了`README.md`, `include`目录，使用`add`添加的文件只是保存在暂存区，需要使用提交命令将暂存区的文件添加进仓库中

```bash
$ git commit -m "first commit"
[master (root-commit) 12b7f0c] first commit
 3 files changed, 3 insertions(+)
 create mode 100644 .gitignore
 create mode 100644 README.md
 create mode 100644 include/test.txt
```

使用`-m`参数，添加提交时的附加信息

使用`-a`参数，可以不执行add直接提交文件到仓库

使用`-am`参数，在不执行add的同时添加备注信息

> 如果备注信息写错了怎么办？
>
> 在没有push到远程仓库前可以使用amend修改备注信息

### 修改备注信息

```bash
$ git commit --amend
[master 4339321] first commit by landers
 Date: Fri Oct 9 22:50:19 2020 +0800
 3 files changed, 3 insertions(+)
 create mode 100644 .gitignore
 create mode 100644 README.md
 create mode 100644 include/test.txt

$ git log
commit 4339321137b56e90d0d810582c793d8c4aeda1ba (HEAD -> master)
Author: Landers1037 <32225052+Landers1037@users.noreply.github.com>
Date:   Fri Oct 9 22:50:19 2020 +0800

    first commit by landers
```

默认会打开vi页面，此时可以修改备注信息，当前修改为`first commit by landers`

使用`git log`查看提交记录可以看到备注已经改过来了

`--amend`的原理

对最后一次提交使用`git commit --amend`，会将此次提交作为最后一次提交更新。在不产生新的commit记录的前提下修改提交信息，前提是最后一次提交没有被`merge`进远程仓库



### push

作为三部曲的最后一步，发布到远程仓库后就会合并到远程代码仓

我们是本地新建的git仓库，所以需要为其指定一个远程仓

使用方式`git push <远程主机名> <本地分支名>:<远程分支名>`

```bash
$ git remote add origin https://github.com/username/gittest.git
$ git push -u origin main
```

使用remote命令指定远程仓库

使用`push`命令推送，默认的推送分支就是master分支(自2020年10月9日master分支修改为main，所以这里使用main分支)

> 一般情况下，我们都是clone远程仓库，本地修改然后push



## 克隆与拉取

一个正常的代码开发流程往往从clone开始

```bash
$ git clone https://github.com/username/gittest.git
```

执行clone命令拉取你个人仓库下的代码仓，这里默认拉取的是master分支

如果需要特定分支可以使用`-b`参数或者`checkout`

```bash
$ git clone -b br_1 https://github.com/username/gittest.git
```

指定拉取分支`br_1`

### 获取最新代码

使用`fetch`命令获取远程仓库代码

假设远程仓库有合作者，此时他已经提交了一次更新到远程仓库，这个更新是你的本地仓库没有的

```bash
$ git fetch origin
...
$ git merge origin/master
```

在执行fetch获取更新后，需要使用merge命令更新内容到自己本地的仓库

### 拉取

使用`pull`命令拉取

`git pull`事实上是fetch和merge的合并形式

```bash
$ git pull origin master
```

解释：拉取远程仓的master分支代码更新内容，合并到本地仓库

使用`git remote -v`命令可以查看远程仓库的信息

```bash
$ git remote -v
origin  https://github.com/username/gittest.git (fetch)
origin  https://github.com/username/gittest.git (push)
```

### 分支

使用`branch`命令创建分支

```bash
$ git branch (branchname)
```

使用`checkout`命令切换到某个分支

```bash
$ git checkout (branchname)
```

```bash
git branch # 列出全部分支
git branch -d branchname # 删除分支
git merge brancename # 合并分支
```

## 回退

当提交的代码有问题，我们需要回退

回退分为两种情况，**本地commit后远程还没合并**与**远程已经合并**

```bash
git reset [--soft | --mixed | --hard] [HEAD]
```

参数分为三种soft，mixed，hard

`soft`用于回退到某一个版本

`mixed`为默认参数，可以不加表示重置暂存区文件内容与上一次提交保持一致，工作区内容保持不变

`hard`用于撤销工作区所有未提交的内容，将暂存区和工作区内容回退到上一个版本，并删除之前的提交信息(即commit的备注信息)

- HEAD~0 表示当前版本
- HEAD~1 上一个版本
- HEAD^2 上上一个版本
- HEAD^3 上上上一个版本

**soft示例**

```bash
$ git reset --soft HEAD
# 回退到某个版本
$ git reset --soft HEAD~1
# 回退到上个版本
$ git reset --soft HEAD~3
# 回退到上上上个版本
$ git reset HEAD^ a.txt
# 回退a.txt文件到指定的版本
```

**mixed示例**

```bash
$ git reset HEAD
# 回退到某个版本
$ git reset HEAD~1
# 回退到上个版本
$ git reset HEAD~3
# 回退到上上上个版本
```

**hard示例**

```bash
$ git reset --hard HEAD
# 回退到某个版本
$ git reset --hard HEAD~1
# 回退到上个版本
$ git reset --hard HEAD~3
# 回退到上上上个版本
$ git reset –hard bae128  # 回退到某个版本回退点之前的所有信息。 
$ git reset --hard origin/master    # 将本地的状态回退到和远程的一样 
```

谨慎使用 `–-hard` 参数，它会删除回退点之前的所有信息。

**具体示例**

在本地添加一个文件`a.txt`并提交这次更改

```bash
$ touch a.txt
$ git add a.txt
$ git commit -m "second commit"
[master a476b10] second commit
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 a.txt
 
$ git status
On branch master
nothing to commit, working tree clean

$ git log
commit a476b10d8cbbdd3d0fe3f817b45f1328ab05cdf1 (HEAD -> master)
Author: Landers1037 <32225052+Landers1037@users.noreply.github.com>
Date:   Fri Oct 9 23:34:55 2020 +0800

    second commit

commit 4339321137b56e90d0d810582c793d8c4aeda1ba
Author: Landers1037 <32225052+Landers1037@users.noreply.github.com>
Date:   Fri Oct 9 22:50:19 2020 +0800

    first commit by landers
```

在提交后使用`status`查看是否有需要提交的内容显示为空，查看`log`显示已经有两次提交记录

假设我们第二次提交的文件有问题，又不想进行新的提交，那就撤销上一次提交的记录

**使用`soft`回退测试**

```bash
$ git reset --soft HEAD~1

$ git log
commit 4339321137b56e90d0d810582c793d8c4aeda1ba (HEAD -> master)
Author: Landers1037 <32225052+Landers1037@users.noreply.github.com>
Date:   Fri Oct 9 22:50:19 2020 +0800

    first commit by landers
```

可以看到本地的提交记录只有一次，第二次提交记录已经被撤销

**使用`hard`回退测试**

```bash
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        new file:   a.txt
$ git commit -m "second commit"
[master a351221] second commit
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 a.txt

$ git reset --hard HEAD~1
HEAD is now at 4339321 first commit by landers

$ git log
commit 4339321137b56e90d0d810582c793d8c4aeda1ba (HEAD -> master)
Author: Landers1037 <32225052+Landers1037@users.noreply.github.com>
Date:   Fri Oct 9 22:50:19 2020 +0800

    first commit by landers
```

首先使用`status`查看有无需要提交的文件，然后提交第二次记录

使用hard参数撤销本次提交，可以看到`log`里只存在了一次提交记录

> 那么hard和soft的区别在哪？
>
> 我们使用tree查看目录

```bash
landers@skypig:/mnt/d/tmp/gittest$ tree
.
├── README.md
├── exclude
│   └── test.txt
├── include
│   └── test.txt
└── secret.txt
```

之前新建的a.txt文件消失了，使用`--hard`参数后，工作区的内容也会回退到上一次提交的样子，所以a.txt文件被删除了。

**谨慎使用`--hard`参数**

# 扩充内容

## 状态查看status

使用`git status`命令查看状态信息

```bash
$ git status
On branch master
nothing to commit, working tree clean
```

用于查看自上次提交后，是否有新的内容更改需要提交

## 日志log

使用`git log`命令看到历史提交记录

```bash
$ git log
commit 4339321137b56e90d0d810582c793d8c4aeda1ba (HEAD -> master)
Author: Landers1037 <32225052+Landers1037@users.noreply.github.com>
Date:   Fri Oct 9 22:50:19 2020 +0800

    first commit by landers
```

`git blame file`以列表的形式展示某个文件的提交记录

```bash
$ git blame README.md
^4339321 (Landers1037 2020-10-09 22:50:19 +0800 1) this's test fot git tutorial
```

使用`--online`参数简化输出结果

## 贮藏stash

使用`git stash`命令可以将当前工作区的代码储藏起来，提供一个干净的工作区和暂存区

用在当你想要保存当前修改，又想要在上一次提交的代码上操作时

```bash
$ git stash
$ git stash pop # 取出上次存贮的代码
```

一般用法

```bash
$ git stash
$ git pull # 更新远程代码到本地仓库
$ git stash pop # 取出存贮的代码到更新后的仓库内
```



## 版本比较diff

使用`git diff`命令比较暂存区和工作区文件的不同

```bash
$ git diff file

$ git diff --cached|--staged file
# 暂存区和上一次提交内容对比

$ touch "hello" > README.md

$ git diff README.md
diff --git a/README.md b/README.md
index c55ae6f..ce01362 100644
--- a/README.md
+++ b/README.md
@@ -1 +1 @@
-this's test fot git tutorial
+hello
```

我们修改README.md的内容为hello，指定diff命令可以看到和上次比较新增一行删除一行

## 添加tag标签

```bash
git tag v1.0
```

### 查看所有tag

```bash
git tag

# landers in blog on master via 🐹 v1.16.5 took 21s
v3.2
v3.3
v4.2-upx
v5.1
v5.2
v5.3
v5.4
v5.5
v5.6
v6.0
v6.1
```

### 查看标签的状态

```bash
landers in blog on master via 🐹 v1.16.5 took 21s
➜ git checkout v6.0
Note: checking out 'v6.0'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by performing another checkout.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -b with the checkout command again. Example:

  git checkout -b <new-branch-name>

HEAD is now at 63c9c9d... blog v6.0
```



### 添加tag并推送

```bash
git tag -a v1.0 -m "提交信息"

git push origin v1.0
```

### 删除本地标签

```bash
git tag -d v1.0
```

### 同步删除到远程仓

```bash
# 两种方法
git push origin --delete v1.0

git push origin :refs/tags/v1.0
```



## 配置config

使用`git config`命令配置git

常用的配置

```bash
$ git config --list
$ git config --global user.name "username"
$ git config --global user.email mail@username.com
```

### 配置http代理
```bash
git config --global http.proxy http://127.0.0.1:1080
# 还原
git config --global --unset http.proxy
```

### 配置远程仓

```bash
# 查看远程仓
git remote -v

# landers in config on master took 5s
➜ git remote -v
origin	git@github.com:Landers1037/config.git (fetch)
origin	git@github.com:Landers1037/config.git (push)

# 删除远程仓
git remote rm origin
# 添加远程仓
git remote add origin 地址

# 重新拉取
git pull origin master
# 直接配置远程url
git remte origin set-url URL
```

