---
title: grep指令
name: linux-grep
date: 2020-03-09
id: 0
tags: [linux]
categories: []
abstract: ""
---


Linux的`grep`指令的使用

<!--more-->

### grep

```bash
root@skypig:/home/landers# grep --version
grep (GNU grep) 3.1
Copyright (C) 2017 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Written by Mike Haertel and others, see <http://git.sv.gnu.org/cgit/grep.git/tree/AUTHORS>.
```

一个GNU开源程序，用于文本的匹配

### 使用

grep的模式匹配串支持正则和普通字符串

**test**

```txt
1 2 3
4 5 6
2 3 5
hello
Hello world
倒数第三行
倒数第二行
倒数第一行
```

**读取当前文件**

```bash
root@skypig:/home/landers# cat test
1 2 3
4 5 6
2 3 5
hello
Hello world
倒数第三行
倒数第二行
倒数第一行
```

**获取包含hello的行**

```bash
root@skypig:/home/landers# grep "hello" test
hello
```

注意到这里只提取到`hello`，没有提取到`Hello`，默认grep对大小写敏感

**获取包含hello的行，忽略大小写**

```bash
root@skypig:/home/landers# grep -i "hello" test
hello
Hello world
```

使用`-i`参数忽略大小写

**统计匹配hello的次数**

```bash
root@skypig:/home/landers# grep -i -c "hello" test
2
```

使用`-c`参数计算匹配总数

**只显示匹配的部分**

```bash
root@skypig:/home/landers# grep -i -o "hello" test
hello
Hello
```

使用`-o`参数

**只显示不匹配的部分**

```python
root@skypig:/home/landers# grep -i -v "hello" test
1 2 3
4 5 6
2 3 5
倒数第三行
倒数第二行
倒数第一行
```

使用`-v`参数

**显示匹配结果和其前N行**

```bash
root@skypig:/home/landers# grep -i -B 2 "hello" test
4 5 6
2 3 5
hello
Hello world
```

使用`-B`参数即before，显示匹配行和其前两行

**显示匹配结果和其后N行**

```bash
root@skypig:/home/landers# grep -i -A 2 "hello" test
hello
Hello world
倒数第三行
倒数第二行
```

使用`-A`参数即after，显示匹配行和其后两行

### 匹配串

**在后缀是t的文件里寻找**

```bash
root@skypig:/home/landers# grep -i -B 2 "hello" *t
4 5 6
2 3 5
hello
Hello world
```

**在首字母是t的文件里寻找**

```bash
root@skypig:/home/landers# grep -i -B 2 "hello" t*
test-4 5 6
test-2 3 5
test:hello
test:Hello world
grep: tmp: Is a directory
```

**使用正则式匹配字符**

```bash
root@skypig:/home/landers# grep -i  "^he.*o" test
hello
Hello world

root@skypig:/home/landers# grep -i  "^he.*d$" test
Hello world

root@skypig:/home/landers# grep -i  "[0-9]" test
1 2 3
4 5 6
2 3 5
```

可以使用`-e`参数添加多个正则表达式

### 搭配cat，wc使用

**统计文件的行数**

```bash
root@skypig:/home/landers# cat test | wc -l
9
```

wc使用`-c`提取byte数，`-m`提取字符数，`-l`提取行数

**通过管道传递**

```bash
root@skypig:/home/landers# cat test | grep "hello"
hello
```

使用`grep`统计匹配的行数(注意不是出现次数，`-c`用于统计出现次数)

```bash
root@skypig:/home/landers# grep -i "hello" test | wc -l
2
```

### 搭配head和tail使用

**取出首行**

```bash
root@skypig:/home/landers# cat test | head -1
1 2 3
```

**取出前两行**

```bash
root@skypig:/home/landers# cat test | head -2
1 2 3
4 5 6
```

**取出末尾两行**

```bash
root@skypig:/home/landers# cat test | tail -2
倒数第二行
倒数第一行
```

可以一同使用取出某一行

**取出第二行**

```
root@skypig:/home/landers# cat test | head -2|tail -1
4 5 6
```

先使用`head`取出头两行，再使用`tail`取出末尾

**搭配grep**

```bash
root@skypig:/home/landers# grep -i "hello"  test | head -1
hello
```

