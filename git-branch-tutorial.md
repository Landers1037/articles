---
title: git分支管理
name: git-branch-tutorial
date: 2021-07-25 23:30:41
tags: [git]
categories: [git]
abstract: 
---
使用`git branch`进行分支管理

<!--more-->

## git branch命令

```bash
$ git branch -h
usage: git branch [<options>] [-r | -a] [--merged | --no-merged]
   or: git branch [<options>] [-l] [-f] <branch-name> [<start-point>]
   or: git branch [<options>] [-r] (-d | -D) <branch-name>...
   or: git branch [<options>] (-m | -M) [<old-branch>] <new-branch>
   or: git branch [<options>] (-c | -C) [<old-branch>] <new-branch>
   or: git branch [<options>] [-r | -a] [--points-at]
   or: git branch [<options>] [-r | -a] [--format]

Generic options
    -v, --verbose         show hash and subject, give twice for upstream branch
    -q, --quiet           suppress informational messages
    -t, --track           set up tracking mode (see git-pull(1))
    -u, --set-upstream-to <upstream>
                          change the upstream info
    --unset-upstream      unset the upstream info
    --color[=<when>]      use colored output
    -r, --remotes         act on remote-tracking branches
    --contains <commit>   print only branches that contain the commit
    --no-contains <commit>
                          print only branches that don't contain the commit
    --abbrev[=<n>]        use <n> digits to display SHA-1s

Specific git-branch actions:
    -a, --all             list both remote-tracking and local branches
    -d, --delete          delete fully merged branch
    -D                    delete branch (even if not merged)
    -m, --move            move/rename a branch and its reflog
    -M                    move/rename a branch, even if target exists
    -c, --copy            copy a branch and its reflog
    -C                    copy a branch, even if target exists
    -l, --list            list branch names
    --show-current        show current branch name
    --create-reflog       create the branch's reflog
    --edit-description    edit the description for the branch
    -f, --force           force creation, move/rename, deletion
    --merged <commit>     print only branches that are merged
    --no-merged <commit>  print only branches that are not merged
    --column[=<style>]    list branches in columns
    --sort <key>          field name to sort on
    --points-at <object>  print only branches of the object
    -i, --ignore-case     sorting and filtering are case insensitive
    --format <format>     format to use for the output
```

### 显示分支

```bash
➜ git branch
* master
```

默认新建的分支为`master`

使用以下命令可以变更默认的分支名而不影响提交历史

```bash
$ git branch -m master main
```

但这只个变更只是本地的，需要同步到远端。

```bash
$ git push -u origin main
```

上述命令将新建的 `master` 分支同步到远端并设置 upstream 到了该分支。

### 删除旧分支

删除原来的 master 分支 `git push origin --delete master`或者使用简写`-d`

删除远程分支

``` bash
git push origin --delete branch
```

删除远程分支的关联

```bash
git branch --delete --remotes <remote>/<branch>
```

此操作只会删除追踪的分支，但是并不会删除远程分支

需要删除远程分支使用

``` bash
git push origin --delete branch
```



### 创建分支

```bash
➜ git branch dev

➜ git branch
  dev
* master
```

创建一个分支dev

### 切换本地分支

```bash
➜ git checkout dev
Switched to branch 'dev'

➜ git branch
* dev
  master
```

### 创建并切换分支

```bash
➜ git checkout -b dev
```



## git的upstream

分支upstream其实就是与远程分支的关联，告诉git默认此分支为每次推动拉取的默认分支

```bash
git branch --set-upstream-to=origin/master
# or
git branch -u origin/master
```

设置当前分支为远程origin的master分支

### 推送时设置upstream

```bash
git push -u origin master
```

命令的含义是，推送master分支到远程origin仓库master分支，并且建立本地分支master的upstream为origin/master

### 设置分支

```bash
git br -u origin/br01-remote br01
# or
git push -u origin br01:br01
```

设置本地分支`br01`的upstream为`origin/br01-remote`

### 取消upstream

取消当前分支的upstream

``` bash
git branch --unset-upstream
```

取消其他分支的upstream

```bash
git branch --unset-upstream [分支名]
```

### 查看upstream

```bash
remote.origin.url=git@github.com:Landers1037/config.git
remote.origin.fetch=+refs/heads/*:refs/remotes/origin/*
branch.master.remote=origin
branch.master.merge=refs/heads/master
```

`branch.master.remote`就是对应远程仓的upstream名

`branch.master.merge`表示远程仓跟踪的分支名

### 使用git remote show查看

``` bash
➜ git remote show origin
* remote origin
  Fetch URL: git@github.com:Landers1037/medserver.git
  Push  URL: git@github.com:Landers1037/medserver.git
  HEAD branch: master
  Remote branch:
    master tracked
  Local branch configured for 'git pull':
    master merges with remote master
  Local ref configured for 'git push':
    master pushes to master (up to date)
```

`Remote branch` 表示远程仓库的分支

`git pull`表示upstream跟踪的分支

## 配置remote仓

通过`git remote -v`可以查看当前的远程仓

```bash
➜ git remote -v
origin	git@github.com:Landers1037/blog.git (fetch)
origin	git@github.com:Landers1037/blog.git (push)
```

`remote.origin.url`就是远程仓的地址

### 修改远程仓地址

``` bash
git remote add origin git@github.com:<username>/<repo>.git  
```

### 删除远程仓地址

```bash
git remote remove origin
```

## 分支合并

在一个分支开发完毕后将此分支合并到主线分支

```bash
git merge <branch>
```

### 取消合并，重建合并前的状态

```bash
git merge --abort
```

### 合并dev分支到当前分支且以dev分支为准

```bash
git merge dev -Xtheirs
```

## rebase衍合

在一个分支的基础上重新应用，将一个分支合并到当前分支

假如有远程分支origin，现基于master分支创建分支dev

```bash
git checkout -b dev origin
```

现在在`dev`分支做修改，产生提交历史

同时远程分支上，合作者在`origin`分支也做了修改提交

那么此时的`origin`分支和`dev`分支就产生了分叉

此时使用pull拉取origin，然后基于`dev`的修改合并就会产生一次新的merger提交历史

如果需要让`dev`分支的历史看上去没发生过任何合并，使用`rebase`

```bash
git checkout dev
git rebase origin
```

命令会将dev中的每次提交历史取消掉，临时保存为补丁到`.git/rebase`，然后将`dev`分支更新为`origin`的最新分支，再应用补丁到`dev`分支上

### 区别

和merge的区别在于，rebase的历史跟origin保持一致

在双方都修改提交时不会产生分叉

在`rebase`的过程中，也许会出现冲突(conflict)。在这种情况，Git会停止`rebase`并会让你去解决冲突

在解决完冲突后，用`git add`命令去更新这些内容的索引(index), 然后，你无需执行 `git commit`,只要执行:

```shell
$ git rebase --continue
```

在任何时候，可以用`--abort`参数来终止`rebase`的操作，并且”`mywork`“ 分支会回到`rebase`开始前的状态。

```shell
$ git rebase --abort
```

## 参考资料

[易百教程](https://www.yiibai.com/git/git_rebase.html)

[php中文网](https://www.php.cn/tool/git/469391.html)

