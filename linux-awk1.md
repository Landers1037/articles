---
title: awk指令1
name: linux-awk1
date: 2020-03-09
id: 0
tags: [暂时没有标签]
categories: []
abstract: ""
---


Linux的awk命令，初级使用


<!--more-->


Linux的awk命令，初级使用

<!--more-->

## awk

```bash
gawk is a pattern scanning and processing language.
By default it reads standard input and writes standard output.

Examples:
        gawk '{ sum += $1 }; END { print sum }' file
        gawk -F: '{ print $1 }' /etc/passwd
root@skypig:/home/landers# awk --version
GNU Awk 4.1.4, API: 1.1 (GNU MPFR 4.0.1, GNU MP 6.1.2)
Copyright (C) 1989, 1991-2016 Free Software Foundation.

This program is free software; you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation; either version 3 of the License, or
(at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License
along with this program. If not, see http://www.gnu.org/licenses/.
```

`awk`和`grep`的区别，grep更加擅长查找匹配处理文本，而awk擅长格式化复杂的文本输出格式化的文本

### 使用

awk [option] 'pattern{action}' file

支持option参数输入，引号内填入pattern匹配模式串或者action即动作，常用的输出动作`print`和`printf`

**使用print打印一个文件**

```bash
root@skypig:/home/landers# awk '{print}' test
12 :2: 3
21 :3 :5
11 :3 :6
10 :11 :12
```

awk内置两个变量`$0`,`$NF`,$0表示一个整行，$NF表示末行

同sort类似，awk读取文件时是按照行来读，默认使用**空格**作为分割符(多个连续空格看作一个)，分割完成后的每一列使用`$N`来提取，比如`$1`就是第一列

```bash
root@skypig:/home/landers# cat newtest
1 2 3

root@skypig:/home/landers# awk '{print$1}' newtest
1
```

**同时还可以输出多列**

```bash
root@skypig:/home/landers# awk '{print$1,$3}' newtest
1 3
```

**添加自定义字符串进行拼接**

```bash
root@skypig:/home/landers# awk '{print "first:" $1,"third:"$3}' newtest
first:1 third:3
```

每一列使用`,`隔开，`$N`属于变量，前后不能添加字符串

### 模式

特殊模式`BEGIN`，`END`分别表示在执行操作前和执行操作后做什么

```bash
root@skypig:/home/landers# awk 'BEGIN{print "hello"}''{print$1}' newtest
hello
1
```

在打印第一列前先打印`hello`

```bash
root@skypig:/home/landers# awk 'END{print "over"}''{print$1}' newtest
1
over
```

在打印第一列后打印`over`

结合使用

```bash
root@skypig:/home/landers# awk 'BEGIN{print "start"}{print$1} END{print "end"}' newtest
start
1
end
```

### 分隔符

分隔符即field separator，分为**输入分隔符**和**输出分隔符**

#### 输入分隔符

就是处理输入的文本时的分隔符，默认为空格

> 有两个方法设置分隔符
>
> -F 分隔符 如-F#
>
> FS=分隔符 如FS=#使用时需要使用-v FS=#来指定这是变量

```bash
root@skypig:/home/landers# awk -F: '{print}' newtest
1: 2: 3
root@skypig:/home/landers# awk -F: '{print$2}' newtest
 2
```

#### 输出分隔符

默认的输出分隔符是空格，即输出多列时，列与列之间使用空格分开

> 设置输出分隔符OFS的方法
>
> -v OFS=#

```bash
root@skypig:/home/landers# awk -F: -v OFS=# '{print$1,$2}' newtest
1# 2
```

可以注意到分列输出每一列变量间使用`,`隔开，这代表着每一个变量是单独的一列

如果想要把列合并为一列输出可以直接去掉`,`

```bash
root@skypig:/home/landers# awk -F: -v OFS=# '{print$1 $2}' newtest
1 2
```

现在输出就是1列和2列合并后的列

### 内置变量

已知的有`NF`,`FS`,`OFS`分别表示一行分隔后的总列数，输入分隔符，输出分隔符

| RS       | 指定输入时的换行符 |      |
| -------- | ------------------ | ---- |
| ORS      | 替换输出时的换行符 |      |
| NF       | 当前行的列数       |      |
| NR       | 当前处理的行号     |      |
| FNR      | 各文件的行号       |      |
| FILENAME | 当前文件名         |      |

#### 输出行号 对应的行包含的列数

```bash
root@skypig:/home/landers# cat test
12 :2: 3
21 :3 :5
11 :3 :6
10 :11 :12

root@skypig:/home/landers# awk -F: '{print NR,NF}' test
1 3
2 3
3 3
4 3
```

#### 输出匹配列和所在的行号

```bash
root@skypig:/home/landers# awk -F: '{print NR,$1}' test
1 12
2 21
3 11
4 10
```

#### 在处理多文件时

**不添加FNR**

```bash
root@skypig:/home/landers# cat test  newtest
12 :2: 3
21 :3 :5
11 :3 :6
10 :11 :12

1: 2: 3

root@skypig:/home/landers# awk '{print NR,$1}' test newtest
1 12
2 21
3 11
4 10
5 1:
```

看到按照文件的读取顺序，打印出看每一列和对应的行号。但是如果我们想要知道列在自己文件中所处的位置

**使用FNR**

```bash
root@skypig:/home/landers# awk '{print FNR,$1}' test newtest
1 12
2 21
3 11
4 10
1 1:
```

