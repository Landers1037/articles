---
title: awk指令3
name: linux-awk3
date: 2020-03-10
id: 0
tags: [linux]
categories: []
abstract: ""
---


Linux的awk指令模式


<!--more-->


Linux的awk指令模式

<!--more-->

## 模式

即Pattern，匹配条件

模式和动作都包含在`''`组成的program里

例子

```bash
root@skypig:/home/landers# cat 1.txt
 1 2 3
 4 5 6
 7 8 9
 
root@skypig:/home/landers# awk 'NF==2 {print $1}' 1.txt
root@skypig:/home/landers# awk 'NF==3 {print $1}' 1.txt
1
4
7 
```

`NF`表示每行的列数，一开始我们指定条件为列数为2，这个条件不成立所以没有输出

同样支持各种关系运算符`<,>,<=,>=,!=`，以及正则使用的`~,!~`,表示对应的正则表达式为真或假

```bash
root@skypig:/home/landers# awk '$1==4 {print $1}' 1.txt
4
```

打印首列为4的

### 正则表达式

grep里使用正则的方式 `grep "^abc"`

在awk里正则表达式前后使用`/`包裹，`awk '/^abc/ {print $1}'`

**从文本里获取网址**

```bash
root@skypig:/home/landers# cat 2.txt
this is a test http://127.0.0.1 the end

#使用grep方式
root@skypig:/home/landers# grep -o  "http:.*." 2.txt
http://127.0.0.1 the end
#使用awk
root@skypig:/home/landers# awk '/http:.*/{print $5}' 2.txt
http://127.0.0.1
```

对于特殊字符有时需要使用转义`\`

```bash
root@skypig:/home/landers# awk '/\/\/.*/{print $5}' 2.txt
http://127.0.0.1
```

#### 扩展正则

当需要使用扩展正则时，需要加上参数`--posix`或者`--re-interval`

```bash
root@skypig:/home/landers# cat 2.txt
aaaab
aaab
aab
ab

root@skypig:/home/landers# awk --posix '/a{3,4}/{print $0}' 2.txt
aaaab
aaab
```

只匹配a最少出现3次，最多出现4次的字符串

#### 变量匹配正则

加入我们需要提取第二列满足是`http`开头的字符串

```bash
root@skypig:/home/landers# cat 2.txt
first http://127
second ht://127
third ws://

root@skypig:/home/landers# awk '$2 ~/^http.*/{print $0}' 2.txt
first http://127
```

使用`~`表示匹配正则，只输出匹配这个模式的行