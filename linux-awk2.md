---
title: awk指令2
name: linux-awk2
date: 2020-03-10
id: 0
tags: [linux]
categories: []
abstract: ""
---


Linux的awk指令 变量续


<!--more-->


Linux的awk指令 变量续

<!--more-->

## 变量续

### RS

输入行换行符，默认使用`\n`换行符来判断何时换行

可以使用`-v RS=xxx`的方式来指定换行符

```bash
root@skypig:/home/landers# awk '{print}' newtest
1: 2: 3

root@skypig:/home/landers# awk -v RS=" " '{print}' newtest
1:
2:
3
```

我们指定了空格为默认的换行符，awk只要遇到了空格就会把它当作新的一行输出

**配合wc来计算行数**

```bash
root@skypig:/home/landers# awk -v RS=" " '{print}' newtest | wc -l
4
```

### ORS

输出行换行符，默认同样是回车换行符

```bash
root@skypig:/home/landers# awk -v RS=" " -v ORS="?"  '{print}' newtest
1:?2:?3
```

我们指定使用`?`来代替换行符

### FILENAME

文件名称

```bash
1 :2 :3
root@skypig:/home/landers# awk -v RS=":"  '{print FILENAME,$0}' newtest
newtest 1
newtest  2
newtest  3
```

我们使用`:`作为输入换行符，每次输出都打印一次文件名

### ARGV / ARGC

ARGV是变量组成的数组，ARGC是变量数量

```bash
root@skypig:/home/landers# awk -v RS=":"  '{print$1,ARGV[0],ARGV[1]}' newtest
1 awk newtest
2 awk newtest
3 awk newtest
```

此时打印两个变量`ARGV[0]`和`ARGV[1]`，Argv[0]就是指令自身，Argv[1]就是输入的文件名

同其他编程语言argv接收来自终端的信息，第一个参数就是指令本身

使用`ARGC`统计变量数

```bash
root@skypig:/home/landers# awk -v RS=":"  '{print$1,ARGV[0],ARGV[1],ARGC}' newtest
1 awk newtest 2
2 awk newtest 2
3 awk newtest 2
```

可以看到`ARGV[]`数组的长度为2

## 自定义变量

定义方法，使用-v参数赋予，或者在program部分直接添加

```bash
root@skypig:/home/landers# awk -v var="nihao" 'BEGIN{print var}'
nihao
```

这里没有文件输入所以使用BEGIN先打印，自定义变量`var`为hello

```bash
root@skypig:/home/landers# awk  'BEGIN{var="hello" ;print var}'
hello
```

也可以直接在program内部定义变量，这里类似c定义后使用`;`隔开按步骤执行

```bash
root@skypig:/home/landers# awk  'BEGIN{var="hello" ;var2="world";print var ,var2}'
hello world
```

可以通过分号定义多个变量

### 格式化printf

使用printf可以帮助快速进行格式化输出文本

和print的区别，printf默认会将内容全部输出为一行，使用%s格式化符可以快速格式化输出内容

```bash
 1 2 3
 4 5 6
 7 8 9
 
root@skypig:/home/landers# awk '{printf  "%s\n",$1}' 1.txt
1
4
7
```

注意使用的格式化符数量，需要和后面的输入变量数相同

```bash
root@skypig:/home/landers# awk '{printf  "%s%s%s\n",$1,$2,$3}' 1.txt
123
456
789
```

我们还可以在处理输入前格式化

```bash
root@skypig:/home/landers# awk 'BEGIN{printf "%s %s %s\n" ,"first","second","third"} {printf  "%s %s %s\n",$1,$2,$3}' 1.txt
first second third
1 2 3
4 5 6
7 8 9
```

在输出前 我们为文本添加首行。可以看到这样的数据并没有对齐，可以使用格式化符处理

```bash
root@skypig:/home/landers# awk 'BEGIN{printf "%5s %5s %5s\n" ,"first","second","third"} {printf  "%5s %5s %5s\n",$1,$2,$3}' 1.txt
first second third
    1     2     3
    4     5     6
    7     8     9
```

使用`%5s`指定输出每列占5个字符位置，不足的在前面补空格

还可以使用`%-5s`表示左对齐，在右边补齐空格