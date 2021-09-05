---
title: git stash的使用
name: git-stash-tutorial
date: 2021-07-24 21:12:28
tags: [git]
categories: [git]
abstract: 
---
git命令`stash`(贮藏)的使用

<!--more-->

`stash`贮藏提供了对仓库未提交代码的暂存管理

对于想要保存当前已经修改的代码，却不想产生一个脏提交的时候，可以使用

## 初始化git仓库

```bash
root@ubuntu:/home/landers# mkdir stash-test
root@ubuntu:/home/landers# cd stash-test/
root@ubuntu:/home/landers/stash-test# git init
Initialized empty Git repository in /home/landers/stash-test/.git/

root@ubuntu:/home/landers/stash-test# tree
.

0 directories, 0 files
```

初始化一个空的仓库

### 创建一次初始提交

```bash
root@ubuntu:/home/landers/stash-test# echo "hello" > README.md
root@ubuntu:/home/landers/stash-test# git add .
root@ubuntu:/home/landers/stash-test# git commit -m "init"
[master a654e8d] init
 1 file changed, 1 deletion(-)
```



## stash用法

### stash当前的更改

对于未提交的代码直接使用git stash可以将代码推送到一个新的储藏，此时的工作目录就会变为全新

```bash
root@ubuntu:/home/landers/stash-test# touch test1.txt
root@ubuntu:/home/landers/stash-test# git status
On branch master

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        test1.txt

nothing added to commit but untracked files present (use "git add" to track)

root@ubuntu:/home/landers/stash-test# git add .
root@ubuntu:/home/landers/stash-test# git status
On branch master
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        new file:   test1.txt
```

我们创建了一个文件，但是并未提交它

此时如果此文件内容未编写完毕，无需被commit

```bash
root@ubuntu:/home/landers/stash-test# git stash
Saved working directory and index state WIP on master: a654e8d init
root@ubuntu:/home/landers/stash-test# git status
On branch master
nothing to commit, working tree clean
```

对于未提交的test1文件，使用`git stash`后当前的工作目录变为全新

### 添加新的文件

```bash
root@ubuntu:/home/landers/stash-test# touch test2.txt
root@ubuntu:/home/landers/stash-test# git add .
root@ubuntu:/home/landers/stash-test# git status
On branch master
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        new file:   test2.txt

root@ubuntu:/home/landers/stash-test# git commit -m "add test2"
[master 469e802] add test2
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 test2.txt
root@ubuntu:/home/landers/stash-test# git status
On branch master
nothing to commit, working tree clean
```

此时可以创建test2文件，然后提交到本地仓

那么我们之前创建的test1文件此时想要修改，并重新提交怎么操作

### 重新应用储藏的stash

```bash
root@ubuntu:/home/landers/stash-test# git stash pop
On branch master
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        new file:   test1.txt

Dropped refs/stash@{0} (fba1b4c7d11ee8a2570eb1687c645a437f465ba9)
root@ubuntu:/home/landers/stash-test# git status
On branch master
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        new file:   test1.txt
```

使用`pop`命令可以将最近一次stash恢复到工作区

pop指令将缓存堆栈中的第一个stash删除，并将对应修改应用到当前的工作目录下。

### 查看现有的全部stash

在此之前先创建几个stash

```bash
root@ubuntu:/home/landers/stash-test# git status
On branch master
nothing to commit, working tree clean
root@ubuntu:/home/landers/stash-test# touch test3.txt
root@ubuntu:/home/landers/stash-test# git add .
root@ubuntu:/home/landers/stash-test# git stash
Saved working directory and index state WIP on master: 150cf5e add test1
root@ubuntu:/home/landers/stash-test# touch test4.txt
root@ubuntu:/home/landers/stash-test# git add .
root@ubuntu:/home/landers/stash-test# git stash
Saved working directory and index state WIP on master: 150cf5e add test1
```



```bash
root@ubuntu:/home/landers/stash-test# git stash list
stash@{0}: WIP on master: 150cf5e add test1
stash@{1}: WIP on master: 150cf5e add test1
```

### 应用stash

使用`git stash apply`默认将储藏中的stash多次应用到工作目录 但不会删除stash拷贝

默认应用最近的一次

### 应用某次的stash

```bash
root@ubuntu:/home/landers/stash-test# git stash list
stash@{0}: WIP on master: 150cf5e add test1
stash@{1}: WIP on master: 150cf5e add test1

root@ubuntu:/home/landers/stash-test# git stash apply stash@{0}
On branch master
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        new file:   test4.txt
```

### 移除stash

使用git stash drop可以移除储藏，默认移除最近的一次

```bash
root@ubuntu:/home/landers/stash-test# git stash drop stash@{1}
Dropped stash@{1} (bbcf70daaceebba4efe21552dca88b8687b176e2)

root@ubuntu:/home/landers/stash-test# git status
On branch master
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        new file:   test4.txt
```

### 删除stash

使用git stash clear删除全部缓存

```bash
root@ubuntu:/home/landers/stash-test# git stash clear
root@ubuntu:/home/landers/stash-test# git stash list
```

### 查看stash的diff变更

```bash
root@ubuntu:/home/landers/stash-test# tree
.
├── README.md
├── test1.txt
├── test2.txt
└── test4.txt

0 directories, 4 files

root@ubuntu:/home/landers/stash-test# echo word >> README.md 
root@ubuntu:/home/landers/stash-test# git status
On branch master
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        new file:   test4.txt

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   README.md

root@ubuntu:/home/landers/stash-test# git stash
Saved working directory and index state WIP on master: 150cf5e add test1
```

我们向readme中追加内容，然后stash本次修改

```bash
root@ubuntu:/home/landers/stash-test# git stash show
 README.md | 1 +
 test4.txt | 0
 2 files changed, 1 insertion(+)
```

使用-p参数可以查看全部的diff

```bash
root@ubuntu:/home/landers/stash-test# git stash show -p
diff --git a/README.md b/README.md
index e69de29..4f5b278 100644
--- a/README.md
+++ b/README.md
@@ -0,0 +1 @@
+word
diff --git a/test4.txt b/test4.txt
new file mode 100644
index 0000000..e69de29
```

## 从stash中创建分支

一般修改本地文件后，从远程分支拉取可能遇到合并冲突

此时可以创建一个新的分支来在新分支上继续未提交的工作

```bash
root@ubuntu:/home/landers/stash-test# echo "another line" >> README.md 
root@ubuntu:/home/landers/stash-test# git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   README.md

no changes added to commit (use "git add" and/or "git commit -a")

root@ubuntu:/home/landers/stash-test# git stash
Saved working directory and index state WIP on master: b83cf8d modify readme
root@ubuntu:/home/landers/stash-test# git stash branch test1
Switched to a new branch 'test1'
On branch test1
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   README.md

no changes added to commit (use "git add" and/or "git commit -a")
Dropped refs/stash@{0} (2f848182ea852a2c49e30da7dd58b3a1e500f9f3)
```

我们修改README.md文件，stash本次未提交内容

然后使用`stash branch name`的方式为此次stash创建新的分支

### 在新的分支中继续

```bash
root@ubuntu:/home/landers/stash-test# git status
On branch test1
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   README.md

no changes added to commit (use "git add" and/or "git commit -a")
root@ubuntu:/home/landers/stash-test# git branch
  master
* test1
```

在stash创建新分支后，我们已经进入了新的test1分支中，之前的未提交内容可以继续

### 合并分支

在新分支修改完毕后，将新分支合并到master分支中

```bash
root@ubuntu:/home/landers/stash-test# git add .
root@ubuntu:/home/landers/stash-test# git commit -m "modify readme2"
[test1 03d403e] modify readme2
 1 file changed, 1 insertion(+)
root@ubuntu:/home/landers/stash-test# git status
On branch test1
nothing to commit, working tree clean

# 合并到master
root@ubuntu:/home/landers/stash-test# git checkout master
Switched to branch 'master'
root@ubuntu:/home/landers/stash-test# git merge test1
Updating b83cf8d..03d403e
Fast-forward
 README.md | 1 +
 1 file changed, 1 insertion(+)
```

```bash
root@ubuntu:/home/landers/stash-test# git log
commit 03d403e74bb4b0fe277996df1b56a1215fe1ad18 (HEAD -> master, test1)
Author: landers <liaorenj@gmail.com>
Date:   Sat Jul 24 22:16:13 2021 +0800

    modify readme2

commit b83cf8ddd74b43477ef4d8a4f628cd9da2f38a82
Author: landers <liaorenj@gmail.com>
Date:   Sat Jul 24 22:09:16 2021 +0800

    modify readme

commit 150cf5e848873f9481aebe88bf389ed110c0054e
Author: landers <liaorenj@gmail.com>
Date:   Sat Jul 24 21:55:26 2021 +0800

    add test1

commit 469e8024d33c54a4a38f1d2efc59a322e0c7e96b
Author: landers <liaorenj@gmail.com>
Date:   Sat Jul 24 21:46:15 2021 +0800

    add test2

commit a654e8d2c02383ed3fa625703ba95e03d1c81d2c
Author: landers <liaorenj@gmail.com>
Date:   Sat Jul 24 21:35:57 2021 +0800

    init

commit a17522a1fb19e4d7125057c88dcb5b44c0fd31cc
Author: landers <liaorenj@gmail.com>
Date:   Sat Jul 24 21:34:18 2021 +0800

    init
```



## 未追踪的文件

默认情况下，`git stash`会缓存下列文件：

- 添加到暂存区的修改（staged changes）
- Git跟踪的但并未添加到暂存区的修改（unstaged changes）

但不会缓存以下文件：

- 在工作目录中新的文件（untracked files）
- 被忽略的文件（ignored files）

`git stash`命令提供了参数用于缓存上面两种类型的文件。使用`-u`或者`--include-untracked`可以stash untracked文件。使用`-a`或者`--all`命令可以stash当前目录下的所有修改。