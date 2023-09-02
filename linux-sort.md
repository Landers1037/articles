---
title: sort命令使用
name: linux-sort
date: 2020-03-09
id: 0
tags: [linux]
categories: []
abstract: ""
---


Linux的`sort`指令使用，排序指令

<!--more-->

## sort

```bash
root@skypig:/home/landers# sort --version
sort (GNU coreutils) 8.28
Copyright (C) 2017 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Written by Mike Haertel and Paul Eggert.
```

使用`sort --help`查看所有参数

### 使用

#### **排序**

默认的排序是按照字符串ascall排序，默认从首字母开始

test

```txt
1 2 3
4 5 6
2 3 5
10 11 12
hello
Hello world
倒数第三行
倒数第二行
倒数第一行
```

**简单排序**

```bash
root@skypig:/home/landers# sort test
1 2 3
10 11 12
2 3 5
4 5 6
apple
banana
```

看到10比2大却排在2的前面，因为按照字符算10首位是1，1比2小所以10<2

**按照数值大小排序**

```bash
root@skypig:/home/landers# sort -n test
apple
banana
1 2 3
2 3 5
4 5 6
10 11 12
```

使用`-n`参数将数字看作数字而不是字符

**反序输出**

```bash
root@skypig:/home/landers# sort -n -r test
10 11 12
4 5 6
2 3 5
1 2 3
banana
apple
```

**输出到文件**

```bash
root@skypig:/home/landers# sort -n -r test -o newtest
root@skypig:/home/landers# cat newtest
10 11 12
4 5 6
2 3 5
1 2 3
banana
apple
```

使用`-o`参数指定新文件的名称，同样可以使用文件流`>`的方式，但是文件流不能保存回原文件

```bash
root@skypig:/home/landers# sort -n -r test > newtest
```

**忽略字符前的空格**

```bash
root@skypig:/home/landers# sort -n -b test
apple
banana
1 2 3
2 3 5
4 5 6
10 11 12
```

使用`-b`参数

### 进阶 多列匹配

在处理多列时，默认使用`" "`空格作为列的分隔符，可以使用`-t`参数指定分隔符

默认的排序都是从第一列进行的，也就是按照第一列的大小顺序排序

test

```bash
1 2 3
4 5 6
2 3 5
10 11 12
```

假如想要按照第二列来排序

```bash
root@skypig:/home/landers# sort -k 2n test
1 2 3
2 3 5
4 5 6
10 11 12
```

使用`-k`参数指定列，这里的`2`就是第二列，`n`表示按照数值排序，可以写为`-k 2 -n`

假如test如下

```txt
1 2 3
1 3 6
2 3 5
10 11 12
```

此时第二列有两个`3`，遇到相同的值后，sort又会默认转回第一列按照第一列的顺序排序

即

```bash
root@skypig:/home/landers# sort -n -k 2 test
1 2 3
1 3 6
2 3 5
10 11 12
```

#### 优先级排序

假如我们想设置排序优先级，当第二列值相同时，按照第三列的值大小排序

```bash
root@skypig:/home/landers# sort -n -k 2 -k 3 test
1 2 3
2 3 5
1 3 6
10 11 12
```

可以看到在第二列相同时，比较第三列`5>6`所以`235`在`136`前面

#### 设置分隔符

```txt
1 :2: 3
2 :3 :5
1 :3 :6
10 :11 :12
```

使用`-t :`设置`:`为分隔符

```bash
root@skypig:/home/landers# sort -n -k 2 -k 3 -t : test
1 :2: 3
2 :3 :5
1 :3 :6
10 :11 :12

root@skypig:/home/landers# sort -n -k 2  -t : test
1 :2: 3
1 :3 :6
2 :3 :5
10 :11 :12
```

#### 起止点

默认的起止是从第一个字符开始的，如`-k 1`就是从第一列的第一个字符开始排序

`-k 1.2`就是从第一列的第二个字符开始排序

test

```bash
root@skypig:/home/landers# cat test
12 :2: 3
21 :3 :5
11 :3 :6
10 :11 :12
```

按照第一列的第二个字符排序

```bash
root@skypig:/home/landers# sort -n -k 1.2  -t : test
10 :11 :12
11 :3 :6
21 :3 :5
12 :2: 3
```

